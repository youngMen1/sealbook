# 1.Rabbitmq Channel参数详解
## 1.1.Channel 
1.1 channel.exchangeDeclare()：
type：有direct、fanout、topic三种
durable：true、false true：服务器重启会保留下来Exchange。警告：仅设置此选项，不代表消息持久化。即不保证重启后消息还在。原文：true if we are declaring a durable exchange (the exchange will survive a server restart)
autoDelete:true、false.true:当已经没有消费者时，服务器是否可以删除该Exchange。原文1：true if the server should delete the exchange when it is no longer in use。