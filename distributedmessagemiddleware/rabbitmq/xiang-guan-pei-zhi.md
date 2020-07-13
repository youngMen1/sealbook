# 1.SpringBoot配置RabbitMq

```
  rabbitmq:
    username: admin
    password: admin@123
    addresses: 192.168.1.28:5672
    listener:
      type: simple
      simple:
        concurrency: 5
        max-concurrency: 20
        # 设置手动确认
        acknowledge-mode: manual 
        retry:
          enabled: true 
          # 最大重试次数
          max-attempts: 3 
          # 重试时间间隔
          initial-interval: 1000ms 
          # 重试失败后是否回队
        default-requeue-rejected: false 
    connection-timeout: 5000ms
    cache:
      channel:
        size: 5
    # 发布者消息确认
    publisher-confirms: true 
    # 发布者消息回调
    publisher-returns: true  
    template:
      retry:
        # 开启消费者(程序出现异常)重试机制，默认开启并一直重试
        enabled: true
        # 最大重试次数
        max-attempts: 3
        # 重试间隔时间(毫秒)
        initial-interval: 1000ms
    virtual-host: /
```



