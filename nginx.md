# **nginx**

### 反向代理

客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器的IP地址。

### 负载均衡

**普通请求**  
普通的请求数量较少

![普通请求](https://ftp.bmp.ovh/imgs/2019/11/9966a7030ea33a00.png)

**大量请求**  
在面对大量的服务请求时，单一服务器可能会崩溃，解决办法有两种

1. **纵向升级**：提升服务器的配置，提升cpu加大内存等，但是硬件的升级跟不上日益增长的需求。
2. **横向升级**：集群的概念，增加服务器的数量，将请求分发到各个服务器上，负载将分发到不同的服务器上，也便是负载均衡。

![负载均衡](https://ftp.bmp.ovh/imgs/2019/11/2315f175b323c3e5.png)

### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度，降低原来单个服务器的压力。  
![动静分离](https://ftp.bmp.ovh/imgs/2019/11/2a09db9c0defced1.png)
  
***

## **nginx配置文件**

### **1. 全局块**

从配置文件开始到events之间的内容，主要为一些影响nginx服务器整体运行的配置指令

```conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
```

**work_processes**：该值越大，可以支持的并发数量也就越多

### **2. events块**

影响nginx服务器与用户的网络连接

```conf
events {
        worker_connections 768;
        # multi_accept on;
}
```

**worker_connections**：支持的最大连接数

### **3. http块**

***http全局块***  
http全局块配置配置的指令包括文件引入，MIME-TYPE定义，日志自定义，连接超时时间，单链接请求数上限等。  

***server块***
与虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
每个http块可以包括多个server块，而每个server块就相当于一个虚拟主机。server块也有全局server块，可以同时包含多个location块。  

* 全局server块
最常见的配置是本虚拟主机的监听配置和本虚拟主机的名称或IP配置。
* location块
主要作用是基于nginx服务器接受到的请求字符串，例如(server_name/uri-string)，对虚拟主机名称之外的字符串进行匹配，对特定的请求进行处理。地址定向，数据缓存和应答控制等，以及许多第三方模块的配置。




```conf
http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##
        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```