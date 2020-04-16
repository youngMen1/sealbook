# SpringBoot配置

```
  rabbitmq:
    username: admin
    password: admin
    addresses: 172.18.21.XXX:5672,172.18.21.XXX:5672,172.18.21.XXX:5672
    listener:
      type: simple
      simple:
        concurrency: 5
        max-concurrency: 20
        acknowledge-mode: manual #设置手动确认
        retry:
          enabled: true
          max-attempts: 3
          initial-interval: 1000ms #尝试时间间隔
        default-requeue-rejected: false #重试失败后是否回队
    connection-timeout: 5000ms
    cache:
      channel:
        size: 5
    publisher-confirms: true #发布者消息确认
    publisher-returns: true  #发布者消息回调
    template:
      retry:
        enabled: true
        max-attempts: 3
        initial-interval: 1000ms #尝试时间间隔
    virtual-host: /
```



