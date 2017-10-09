---
title: 端口映射+nginx对外请求控制
date: 2017-10-09 13:59:27
categories:
- 开发随记
tags: [开发随记,Nginx]
toc: true
reward: false
---



​	**前几天接到个任务,要和另一家公司对接,具体就是我这开一个接口给对面调用.因为一开始在内网测试,那问题来了,怎么才能让对面访问到呢?**

​        当然是找运维大兄弟.....操作也很简单,就是用路由做一个端口映射,用公网ip做一个端口映射到我本机地址.这样外网就能访问到我的tomcat了.

比如说公网ip是`192.168.1.0`,我的ip是`192.168.1.1`,tomcat端口是8080.

那就在路由上配一个端口9876直接映射到本地tomcat 192.168.1.1:8080.那现在外网就可以通过`http://192.168.1.0:9876/ `访问到本地`http://192.168.1.1:8080/`了.

就这样,做好了接口,问题又来了,因为这个接口是在核心系统里,那这样就会把所有的接口都暴露了,肯定不行..

<!-- more -->

怎么做呢,怎么才能拦截这些请求呢? 当然又去问了运维大兄弟,再查了些资料,得知用Nginx可以只允许访问指定的url,其他的都直接对外禁止访问.

那怎么做呢?

重新修改下映射规则,不直接映射到tomcat,先经过nginx,通过nginx再把请求发送到tomcat.

这里就重新映射一个端口8000,在nginx中监听这个端口,然后再配置访问规则,再代理到tomcat

在nginx.config中添加一段server

```nginx
server {
  		#监听8080端口,这个8080是路由映射到本机的端口
        listen       8000;
        server_name  0.0.0.0;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
    		#阻止所有请求,这里将永远输出403错误
            deny all;
        }

  		#允许访问 /test/processe接口
        location ~ /test/processe {
    		# 代理本地项目url
        	proxy_pass http://192.168.4.48:8080;
        }
  		#如果还有其他接口,就再添加一个location 
  		 #location ~ /test/processe1 {
        	#proxy_pass http://192.168.4.48:8080;
  		#}
 		 #location ~ /test/processe2 {
        	#proxy_pass http://192.168.4.48:8080;
       	 #}
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

这里的流程就是,外网通过访问公网ip+给定的端口,在路由根据映射规则,再访问到我这台电脑,这个时候请求不是直接去访问本机的接口,而是进入了Nginx,在这里,会去检查请求的url是否与配置允许的地址相同,不同的话就会403 Forbidden错误啦,当uri是`/test/processe`时,就能访问实际代理的`http://192.168.4.48:8080/test/processe`接口了

那这样就可以让对面只能访问指定的接口.