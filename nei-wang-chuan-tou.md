# 1.内网穿透

## 1.natapp

因为要直接用内网本机开发调试，微信网页授权在回调时要访问本机，所以直接做个内网穿透，可以直接在外网访问到本机，做法如下：

1.登录 `https://natapp.cn/` （我用的natapp.cn，你可以用其他类似的，个人感觉这个不错）

2.购买隧道：  
![](/static/image/20180320183743444.png)  
购买前需要认证一下，不要用免费的，因为免费的是随机分配域名的，每次都会变，之前VIP\_1型是5元，现在涨到9元了，自行选择（官方9折优惠码709ABD4F），我买的是VIP\_2型，购买后如下图：  
![](/static/image/20180320184001958.png)

**购买后使用方式：**

`https://natapp.cn/article/natapp_newbie`

使用后会得到natapp分配的网址，如 xxx.natapp.cn，这个地址就可以访问到开发本机。

## 2.Ngrok 实现内网穿透教程

官网：`https://dashboard.ngrok.com/login`

## 3.花生壳



