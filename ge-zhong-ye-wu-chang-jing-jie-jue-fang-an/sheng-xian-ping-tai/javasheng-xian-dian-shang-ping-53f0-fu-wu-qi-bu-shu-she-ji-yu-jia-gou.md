# Java生鲜电商平台-服务器部署设计与架构

补充说明：Java开源生鲜电商平台-服务器部署设计与架构，指的是通过服务器正式上线整个项目，进行正式的运营。

回顾整个章节，我们涉及到以下几个方面：

* 1.买家端

* 2.卖家端

* 3.销售端

* 4.配送端

* 5.系统运营端

* 6.公司网址

目前根据业务的情况，采购了阿里云服务器，由于是创业，我身上没多少钱，只采购了一台阿里云.(具体配置如下与域名规划如下)

公司网址: `http://www.netcai.com`

买家端:  `http://buyer.netcai.com`

卖家端:  `http://seller.netcai.com`

配送端：`http://delivery.netcai.com`

销售端：`http://sales.netcai.com`

后台端：`http://admin.netcai.com`

具体费用如下：
![](/static/image/641237-20180529085209332-1810363843.png)
 
说明：域名采用二级域名进行转发与配置。

服务器采用nginx进行根据域名转发。相关的配置我就贴在下面

如果需要进行业务的处理，比如说，我们发现买家的人数在增加，负载不够，我们可以把买家的域名绑定在一台新的服务器上面进行

最终也可以实现负载均衡的。

实现的基础业务逻辑如下：

域名---》nginx-->tomcat7

**nginx的核心配置如下：**


```
#admin port 8080
server
 {
        server_name admin.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-admin/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8080;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#buyer port 8081
server 
 {
        server_name buyer.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-buyer/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8081;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#seller port 8082
server
 {
        server_name seller.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-seller/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8082;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#delivery port 8083
server
 {
        server_name delivery.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-delivery/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8083;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#sales port 8085
server
 {
        server_name sales.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-sales/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8085;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#purchase port 8088
server
 {
        server_name purchase.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-purchase/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:8088;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#tongmei port 7070
server
 {
        server_name tongmei.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-tongmei/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:7070;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}


#users port 7080
server
 {
        server_name users.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/tomcat-users/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:7080;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}

#monitor port 19999
server
 {
        server_name monitor.netcai.com;
        index index.html index.htm;
    access_log  /webser/nginx/monitor/access/log/access.log  access;
    location / {
                 proxy_pass        http://localhost:19999;
                 proxy_set_header   Host         $host;
                 proxy_set_header   X-Real-IP        $remote_addr;
                 proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}
```


