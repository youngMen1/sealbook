# jdk8排序


```java
  public static void main(String[] args) {

        List<UserInfo> userInfoList = new ArrayList<>();

        UserInfo userInfo = new UserInfo();
        userInfo.setName("2");
        userInfo.setAge(20);
        userInfo.setHeight(22);
        userInfoList.add(userInfo);

        UserInfo userInfo2 = new UserInfo();
        userInfo2.setName("2");
        userInfo2.setAge(20);
        userInfo2.setHeight(20);
        userInfoList.add(userInfo2);

        UserInfo userInfo3 = new UserInfo();
        userInfo3.setName("3");
        userInfo3.setAge(30);
        userInfo3.setHeight(10);
        userInfoList.add(userInfo3);

        UserInfo userInfo4 = new UserInfo();
        userInfo4.setName("4");
        userInfo4.setAge(40);
        userInfo4.setHeight(30);
        userInfoList.add(userInfo4);

        UserInfo userInfo5 = new UserInfo();
        userInfo5.setName("5");
        userInfo5.setAge(50);
        userInfo5.setHeight(40);
        userInfoList.add(userInfo5);

        // 先按年龄降序,遇见年龄相同的,再按身高降序
        List<UserInfo> list1 = userInfoList.stream().sorted(Comparator.comparing(UserInfo::getAge, Comparator.reverseOrder())
                        .thenComparing(UserInfo::getHeight, Comparator.reverseOrder()))
                .collect(Collectors.toList());
        System.out.println("先按年龄降序,遇见年龄相同的,再按身高降序:" + list1);

        // 先按年龄降序,遇见年龄相同的,再按身高升序
        List<UserInfo> list2 = userInfoList.stream().sorted(Comparator.comparing(UserInfo::getAge, Comparator.reverseOrder())
                        .thenComparing(UserInfo::getHeight))
                .collect(Collectors.toList());
        System.out.println("先按年龄降序,遇见年龄相同的,再按身高升序:" + list2);

        // 先按年龄升序,遇见年龄相同的,再按身高升序
        List<UserInfo> list3 = userInfoList.stream().sorted(Comparator.comparing(UserInfo::getAge)
                        .thenComparing(UserInfo::getHeight))
                .collect(Collectors.toList());
        System.out.println("先按年龄升序,遇见年龄相同的,再按身高升序:" + list3);

        // 先按姓名降序,遇见姓名相同的,再按身高降序
        List<UserInfo> list4 = userInfoList.stream().sorted(Comparator.comparing(UserInfo::getName, Comparator.reverseOrder())
                        .thenComparing(UserInfo::getHeight, Comparator.reverseOrder()))
                .collect(Collectors.toList());
        System.out.println("先按姓名降序,遇见姓名相同的,再按身高降序:" + list4);

        // 先按姓名升序,遇见姓名相同的,再按身高降序
        List<UserInfo> list5 = userInfoList.stream().sorted(Comparator.comparing(UserInfo::getName)
                        .thenComparing(UserInfo::getHeight, Comparator.reverseOrder()))
                .collect(Collectors.toList());
        System.out.println("先按姓名降序,遇见姓名相同的,再按身高降序:" + list5);

    }
    
先按年龄降序,遇见年龄相同的,再按身高降序:[UserInfo(name=5, age=50, height=40), UserInfo(name=4, age=40, height=30), UserInfo(name=3, age=30, height=10), UserInfo(name=2, age=20, height=22), UserInfo(name=2, age=20, height=20)]
先按年龄降序,遇见年龄相同的,再按身高升序:[UserInfo(name=5, age=50, height=40), UserInfo(name=4, age=40, height=30), UserInfo(name=3, age=30, height=10), UserInfo(name=2, age=20, height=20), UserInfo(name=2, age=20, height=22)]
先按年龄升序,遇见年龄相同的,再按身高升序:[UserInfo(name=2, age=20, height=20), UserInfo(name=2, age=20, height=22), UserInfo(name=3, age=30, height=10), UserInfo(name=4, age=40, height=30), UserInfo(name=5, age=50, height=40)]
先按姓名降序,遇见姓名相同的,再按身高降序:[UserInfo(name=5, age=50, height=40), UserInfo(name=4, age=40, height=30), UserInfo(name=3, age=30, height=10), UserInfo(name=2, age=20, height=22), UserInfo(name=2, age=20, height=20)]
先按姓名降序,遇见姓名相同的,再按身高降序:[UserInfo(name=2, age=20, height=22), UserInfo(name=2, age=20, height=20), UserInfo(name=3, age=30, height=10), UserInfo(name=4, age=40, height=30), UserInfo(name=5, age=50, height=40)]

```

