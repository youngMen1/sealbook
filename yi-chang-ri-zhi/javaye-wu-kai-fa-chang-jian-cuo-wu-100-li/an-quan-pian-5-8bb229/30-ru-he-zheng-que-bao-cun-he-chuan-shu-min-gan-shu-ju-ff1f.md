# 30 | 如何正确保存和传输敏感数据？

今天，我们从安全角度来聊聊用户名、密码、身份证等敏感信息，应该怎么保存和传输。同时，你还可以进一步复习加密算法中的散列、对称加密和非对称加密算法，以及 HTTPS 等相关知识。

## 应该怎样保存用户密码？

最敏感的数据恐怕就是用户的密码了。黑客一旦窃取了用户密码，或许就可以登录进用户的账号，消耗其资产、发布不良信息等；更可怕的是，有些用户至始至终都是使用一套密码，密码一旦泄露，就可以被黑客用来登录全网。

为了防止密码泄露，最重要的原则是不要保存用户密码。你可能会觉得很好笑，不保存用户密码，之后用户登录的时候怎么验证？其实，我指的是**不保存原始密码，这样即使拖库也不会泄露用户密码。**

我经常会听到大家说，不要明文保存用户密码，应该把密码通过 MD5 加密后保存。这的确是一个正确的方向，但这个说法并不准确。

首先，MD5 其实不是真正的加密算法。所谓加密算法，是可以使用密钥把明文加密为密文，随后还可以使用密钥解密出明文，是双向的。

而 MD5 是散列、哈希算法或者摘要算法。不管多长的数据，使用 MD5 运算后得到的都是固定长度的摘要信息或指纹信息，无法再解密为原始数据。所以，MD5 是单向的。**最重要的是，仅仅使用 MD5 对密码进行摘要，并不安全。**

比如，使用如下代码在保持用户信息时，对密码进行了 MD5 计算：


```

UserData userData = new UserData();
userData.setId(1L);
userData.setName(name);
//密码字段使用MD5哈希后保存
userData.setPassword(DigestUtils.md5Hex(password));
return userRepository.save(userData);
```

通过输出，可以看到密码是 32 位的 MD5：

```
"password": "325a2cc052914ceeb8c19016c091d2ac"
```

到某 MD5 破解网站上输入这个 MD5，不到 1 秒就得到了原始密码：

e1b3638dea64636494c3dcb0bb9b8ade.png

其实你可以想一下，虽然 MD5 不可解密，但是我们可以构建一个超大的数据库，把所有 20 位以内的数字和字母组合的密码全部计算一遍 MD5 存进去，需要解密的时候搜索一下 MD5 就可以得到原始值了。这就是字典表。

目前，有些 MD5 解密网站使用的是彩虹表，是一种使用时间空间平衡的技术，即可以使用更大的空间来降低破解时间，也可以使用更长的破解时间来换取更小的空间。

**此外，你可能会觉得多次 MD5 比较安全，其实并不是这样。**比如，如下代码使用两次 MD5 进行摘要：


userData.setPassword(DigestUtils.md5Hex(DigestUtils.md5Hex( password)));

得到下面的 MD5：

```
"password": "ebbca84993fe002bac3a54e90d677d09"
```

也可以破解出密码，并且破解网站还告知我们这是两次 MD5 算法：

ce87f65a3289e50d4e29754073b7eab1.png

所以直接保存 MD5 后的密码是不安全的。一些同学可能会说，还需要加盐。是的，但是加盐如果不当，还是非常不安全，比较重要的有两点。

第一，**不能在代码中写死盐，且盐需要有一定的长度**，比如这样：

```
userData.setPassword(DigestUtils.md5Hex("salt" + password));
```
得到了如下 MD5：


```

"password": "58b1d63ed8492f609993895d6ba6b93a"
```
对于这样一串 MD5，虽然破解网站上找不到原始密码，但是黑客可以自己注册一个账号，使用一个简单的密码，比如 1：


```
"password": "55f312f84e7785aa1efa552acbf251db"
```

321dfe5822da9fe186b17f283bda1fca.png


其实，知道盐是什么没什么关系，关键的是我们是在代码里写死了盐，并且盐很短、所有用户都是这个盐。这么做有三个问题：

* 因为盐太短、太简单了，如果用户原始密码也很简单，那么整个拼起来的密码也很短，这样一般的 MD5 破解网站都可以直接解密这个 MD5，除去盐就知道原始密码了。
* 相同的盐，意味着使用相同密码的用户 MD5 值是一样的，知道了一个用户的密码就可能知道了多个。
* 我们也可以使用这个盐来构建一张彩虹表，虽然会花不少代价，但是一旦构建完成，所有人的密码都可以被破解。

**所以，最好是每一个密码都有独立的盐，并且盐要长一点，比如超过 20 位。**

第二，**虽然说每个人的盐最好不同，但我也不建议将一部分用户数据作为盐。**比如，使用用户名作为盐：

```
userData.setPassword(DigestUtils.md5Hex(name + password));
```

如果世界上所有的系统都是按照这个方案来保存密码，那么 root、admin 这样的用户使用再复杂的密码也总有一天会被破解，因为黑客们完全可以针对这些常用用户名来做彩虹表。**所以，盐最好是随机的值，并且是全球唯一的，意味着全球不可能有现成的彩虹表给你用。**

正确的做法是，使用全球唯一的、和用户无关的、足够长的随机值作为盐。比如，可以使用 UUID 作为盐，把盐一起保存到数据库中：


```

userData.setSalt(UUID.randomUUID().toString());
userData.setPassword(DigestUtils.md5Hex(userData.getSalt() + password));
```

并且每次用户修改密码的时候都重新计算盐，重新保存新的密码。你可能会问，盐保存在数据库中，那被拖库了不是就可以看到了吗？难道不应该加密保存吗？

在我看来，盐没有必要加密保存。盐的作用是，防止通过彩虹表快速实现密码“解密”，如果用户的盐都是唯一的，那么生成一次彩虹表只可能拿到一个用户的密码，这样黑客的动力会小很多。

更好的做法是，不要使用像 MD5 这样快速的摘要算法，而是使用慢一点的算法。比如 Spring Security 已经废弃了 MessageDigestPasswordEncoder，推荐使用 BCryptPasswordEncoder，也就是BCrypt来进行密码哈希。BCrypt 是为保存密码设计的算法，相比 MD5 要慢很多。
`https://en.wikipedia.org/wiki/Bcrypt`
写段代码来测试一下 MD5，以及使用不同代价因子的 BCrypt，看看哈希一次密码的耗时。


```

private static BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

@GetMapping("performance")
public void performance() {
    StopWatch stopWatch = new StopWatch();
    String password = "Abcd1234";
    stopWatch.start("MD5");
    //MD5
    DigestUtils.md5Hex(password);
    stopWatch.stop();
    stopWatch.start("BCrypt(10)");
    //代价因子为10的BCrypt
    String hash1 = BCrypt.gensalt(10);
    BCrypt.hashpw(password, hash1);
    System.out.println(hash1);
    stopWatch.stop();
    stopWatch.start("BCrypt(12)");
    //代价因子为12的BCrypt
    String hash2 = BCrypt.gensalt(12);
    BCrypt.hashpw(password, hash2);
    System.out.println(hash2);
    stopWatch.stop();
    stopWatch.start("BCrypt(14)");
    //代价因子为14的BCrypt
    String hash3 = BCrypt.gensalt(14);
    BCrypt.hashpw(password, hash3);
    System.out.println(hash3);
    stopWatch.stop();
    log.info("{}", stopWatch.prettyPrint());
}
```
可以看到，MD5 只需要 0.8 毫秒，而三次 BCrypt 哈希（代价因子分别设置为 10、12 和 14）耗时分别是 82 毫秒、312 毫秒和 1.2 秒：

13241938861dd3ca9ba984776cc90846.png

也就是说，如果制作 8 位密码长度的 MD5 彩虹表需要 5 个月，那么对于 BCrypt 来说，可能就需要几十年，大部分黑客应该都没有这个耐心。

我们写一段代码观察下，BCryptPasswordEncoder 生成的密码哈希的规律：


```

@GetMapping("better")
public UserData better(@RequestParam(value = "name", defaultValue = "zhuye") String name, @RequestParam(value = "password", defaultValue = "Abcd1234") String password) {
    UserData userData = new UserData();
    userData.setId(1L);
    userData.setName(name);
    //保存哈希后的密码
    userData.setPassword(passwordEncoder.encode(password));
    userRepository.save(userData);
    //判断密码是否匹配
    log.info("match ? {}", passwordEncoder.matches(password, userData.getPassword()));
    return userData;
}
```

我们可以发现三点规律。

第一，我们调用 encode、matches 方法进行哈希、做密码比对的时候，不需要传入盐。**BCrypt 把盐作为了算法的一部分，强制我们遵循安全保存密码的最佳实践。**

第二，生成的盐和哈希后的密码拼在了一起：$是字段分隔符，其中第一个$后的 2a 代表算法版本，第二个$后的 10 是代价因子（默认是 10，代表 2 的 10 次方次哈希），第三个$后的 22 个字符是盐，再后面是摘要。所以说，我们不需要使用单独的数据库字段来保存盐。
```
"password": "$2a$10$wPWdQwfQO2lMxqSIb6iCROXv7lKnQq5XdMO96iCYCj7boK9pk6QPC"
//格式为：$<ver>$<cost>$<salt><digest>
```
第三，代价因子的值越大，BCrypt 哈希的耗时越久。因此，对于代价因子的值，更建议的实践是，根据用户的忍耐程度和硬件，设置一个尽可能大的值。

最后，我们需要注意的是，虽然黑客已经很难通过彩虹表来破解密码了，但是仍然有可能暴力破解密码，也就是对于同一个用户名使用常见的密码逐一尝试登录。因此，除了做好密码哈希保存的工作外，我们还要建设一套完善的安全防御机制，在感知到暴力破解危害的时候，开启短信验证、图形验证码、账号暂时锁定等防御机制来抵御暴力破解。

## 应该怎么保存姓名和身份证？
我们把姓名和身份证，叫做二要素。

现在互联网非常发达，很多服务都可以在网上办理，很多网站仅仅依靠二要素来确认你是谁。所以，二要素是比较敏感的数据，如果在数据库中明文保存，那么数据库被攻破后，黑客就可能拿到大量的二要素信息。如果这些二要素被用来申请贷款等，后果不堪设想。

之前我们提到的单向散列算法，显然不适合用来加密保存二要素，因为数据无法解密。这个时候，我们需要选择真正的加密算法。可供选择的算法，包括对称加密和非对称加密算法两类。

对称加密算法，是使用相同的密钥进行加密和解密。使用对称加密算法来加密双方的通信的话，双方需要先约定一个密钥，加密方才能加密，接收方才能解密。如果密钥在发送的时候被窃取，那么加密就是白忙一场。因此，这种加密方式的特点是，加密速度比较快，但是密钥传输分发有泄露风险。

非对称加密算法，或者叫公钥密码算法。公钥密码是由一对密钥对构成的，使用公钥或者说加密密钥来加密，使用私钥或者说解密密钥来解密，公钥可以任意公开，私钥不能公开。使用非对称加密的话，通信双方可以仅分享公钥用于加密，加密后的数据没有私钥无法解密。因此，这种加密方式的特点是，加密速度比较慢，但是解决了密钥的配送分发安全问题。

但是，对于保存敏感信息的场景来说，加密和解密都是我们的服务端程序，不太需要考虑密钥的分发安全性，也就是说使用非对称加密算法没有太大的意义。在这里，我们使用对称加密算法来加密数据。

接下来，我就重点与你说说对称加密算法。对称加密常用的加密算法，有 DES、3DES 和 AES。

虽然，现在仍有许多老项目使用了 DES 算法，但我不推荐使用。在 1999 年的 DES 挑战赛 3 中，DES 密码破解耗时不到一天，而现在 DES 密码破解更快，使用 DES 来加密数据非常不安全。因此，**在业务代码中要避免使用 DES 加密。**

而 3DES 算法，是使用不同的密钥进行三次 DES 串联调用，虽然解决了 DES 不够安全的问题，但是比 AES 慢，也不太推荐。

AES 是当前公认的比较安全，兼顾性能的对称加密算法。不过严格来说，AES 并不是实际的算法名称，而是算法标准。2000 年，NIST 选拔出 Rijndael 算法作为 AES 的标准。

AES 有一个重要的特点就是分组加密体制，一次只能处理 128 位的明文，然后生成 128 位的密文。如果要加密很长的明文，那么就需要迭代处理，而迭代方式就叫做模式。网上很多使用 AES 来加密的代码，使用的是最简单的 ECB 模式（也叫电子密码本模式），其基本结构如下：

27c2534caeefcac4a5dd1a2814957d8b.png


可以看到，这种结构有两个风险：明文和密文是一一对应的，如果明文中有重复的分组，那么密文中可以观察到重复，掌握密文的规律；因为每一个分组是独立加密和解密的 ，如果密文分组的顺序，也可以反过来操纵明文，那么就可以实现不解密密文的情况下，来修改明文。

我们写一段代码来测试下。在下面的代码中，我们使用 ECB 模式测试：
* 加密一段包含 16 个字符的字符串，得到密文 A；然后把这段字符串复制一份成为一个 32 个字符的字符串，再进行加密得到密文 B。我们验证下密文 B 是不是重复了一遍的密文 A。

* 模拟银行转账的场景，假设整个数据由发送方账号、接收方账号、金额三个字段构成。我们尝试改变密文中数据的顺序来操纵明文。



```

private static final String KEY = "secretkey1234567"; //密钥
//测试ECB模式
@GetMapping("ecb")
public void ecb() throws Exception {
  Cipher cipher = Cipher.getInstance("AES/ECB/NoPadding");
  test(cipher, null);
}
//获取加密秘钥帮助方法
private static SecretKeySpec setKey(String secret) {
    return new SecretKeySpec(secret.getBytes(), "AES");
}
//测试逻辑
private static void test(Cipher cipher, AlgorithmParameterSpec parameterSpec) throws Exception {
    //初始化Cipher
    cipher.init(Cipher.ENCRYPT_MODE, setKey(KEY), parameterSpec);
    //加密测试文本
    System.out.println("一次：" + Hex.encodeHexString(cipher.doFinal("abcdefghijklmnop".getBytes())));
    //加密重复一次的测试文本
    System.out.println("两次：" + Hex.encodeHexString(cipher.doFinal("abcdefghijklmnopabcdefghijklmnop".getBytes())));
    //下面测试是否可以通过操纵密文来操纵明文    
    //发送方账号
    byte[] sender = "1000000000012345".getBytes();
    //接收方账号
    byte[] receiver = "1000000000034567".getBytes();
    //转账金额
    byte[] money = "0000000010000000".getBytes();
    //加密发送方账号
    System.out.println("发送方账号：" + Hex.encodeHexString(cipher.doFinal(sender)));
    //加密接收方账号
    System.out.println("接收方账号：" + Hex.encodeHexString(cipher.doFinal(receiver)));
    //加密金额
    System.out.println("金额：" + Hex.encodeHexString(cipher.doFinal(money)));
    //加密完整的转账信息
    byte[] result = cipher.doFinal(ByteUtils.concatAll(sender, receiver, money));
    System.out.println("完整数据：" + Hex.encodeHexString(result));
    //用于操纵密文的临时字节数组
    byte[] hack = new byte[result.length];
    //把密文前两段交换
    System.arraycopy(result, 16, hack, 0, 16);
    System.arraycopy(result, 0, hack, 16, 16);
    System.arraycopy(result, 32, hack, 32, 16);
    cipher.init(Cipher.DECRYPT_MODE, setKey(KEY), parameterSpec);
    //尝试解密
    System.out.println("原始明文：" + new String(ByteUtils.concatAll(sender, receiver, money)));
    System.out.println("操纵密文：" + new String(cipher.doFinal(hack)));
}
```
输出如下：
![](/static/image/cd506b4cf8a020d4b6077fdfa3b34959.png)
可以看到：
* 两个相同明文分组产生的密文，就是两个相同的密文分组叠在一起。
* 在不知道密钥的情况下，我们操纵密文实现了对明文数据的修改，对调了发送方账号和接收方账号。

所以说，**ECB 模式虽然简单，但是不安全，不推荐使用。**我们再看一下另一种常用的加密模式，CBC 模式。

CBC 模式，在解密或解密之前引入了 XOR 运算，第一个分组使用外部提供的初始化向量 IV，从第二个分组开始使用前一个分组的数据，这样即使明文是一样的，加密后的密文也是不同的，并且分组的顺序不能任意调换。这就解决了 ECB 模式的缺陷：

7955a199e2400adc7ac7577b3712bae8.png



```

 private static final String initVector = "abcdefghijklmnop"; //初始化向量

@GetMapping("cbc")
public void cbc() throws Exception {
    Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
    IvParameterSpec iv = new IvParameterSpec(initVector.getBytes("UTF-8"));
    test(cipher, iv);
}
```
可以看到，相同的明文字符串复制一遍得到的密文并不是重复两个密文分组，并且调换密文分组的顺序无法操纵明文：
8b79074d6533a84c32e48eab3daef808.png
其实，除了 ECB 模式和 CBC 模式外，AES 算法还有 CFB、OFB、CTR 模式，你可以参考这里了解它们的区别。《实用密码学》一书比较推荐的是 CBC 和 CTR 模式。还需要注意的是，ECB 和 CBC 模式还需要设置合适的填充模式，才能处理超过一个分组的数据。

```
https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
```

对于敏感数据保存，除了选择 AES+ 合适模式进行加密外，我还推荐以下几个实践：

* 不要在代码中写死一个固定的密钥和初始化向量，最好和之前提到的盐一样，是唯一、独立并且每次都变化的。

* 推荐使用独立的加密服务来管控密钥、做加密操作，千万不要把密钥和密文存在一个数据库，加密服务需要设置非常高的管控标准。

* 数据库中不能保存明文的敏感信息，但可以保存脱敏的信息。普通查询的时候，直接查脱敏信息即可。

接下来，我们按照这个策略完成相关代码实现。

第一步，对于用户姓名和身份证，我们分别保存三个信息，脱敏后的明文、密文和加密 ID。加密服务加密后返回密文和加密 ID，随后使用加密 ID 来请求加密服务进行解密：