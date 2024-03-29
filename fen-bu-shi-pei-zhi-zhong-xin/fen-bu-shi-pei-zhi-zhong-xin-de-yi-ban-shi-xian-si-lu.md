# 1.分布式配置中心

## 1.为什么

因为新公司没有采用独立的配置中心,每次修改配置参数只能通过手动修改配置文件的方式,然后再重启重启重启,而且机器又是多台,这种方式无疑是非常低下的,而且极容易出错,所以才有了下面的配置中心选型。

## 2.分布式配置中心需要重点考虑的几个点

其实自己开发一个简单的配置中心也是非常容易的,基于redis+DB就能简单实现。但是要设计一个合格的配置中心还需要考虑如下几点:

### 2.1.修改配置实时生效

就是修改配置不要依赖于定时调度,即使调度控制在很小的频率下还是会造成一定程度上的延迟.

### 2.2.只需加载被修改的配置

什么意思？就是只同步那些在配置中心改变的参数值。如果使用定时任务去不停的同步DB中的配置数据,无疑是大批量的,没有必要。

### 2.3.考虑系统容灾,保证高可用

即当配置中心挂的时候，或者DB不可用时不影响系统正常运行。

### 2.4.具备基本的权限管理。

限制用户登录,防止被恶意修改。

### 2.5.可以针对不同环境配置多套配置。

一般稍微大点的公司都有很多环境,像dev、test、pre、gray、pro.有可能不同的环境配置不同,需要做到灵活配置。

### 2.6.要提供多种配置方式,比如说properties配置,key-value配置

所以要自己开发一个独立的配置中心,还是要考虑得比较全面的。而且项目还是以业务为主,也没有足够人力来重新开发一套配置中心,所以就打算借助于开源的力量来满足目前的使用场景。

# 3.参考

链接：[https://www.jianshu.com/p/131c56f2a934](https://www.jianshu.com/p/131c56f2a934)

