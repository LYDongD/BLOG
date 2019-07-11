## 搭建gitbook服务

### 1 拉取gitbook镜像

```
docker pull fellah/gitbook

```

该镜像的dockerfile

```

//gitbook需要node环境
FROM node:6-slim

MAINTAINER Roman Krivetsky <r.krivetsky@gmail.com>

ARG VERSION=3.2.0

LABEL version=$VERSION

//下载gitbook客户端
RUN npm install --global gitbook-cli &&\
	gitbook fetch ${VERSION} &&\
	npm cache clear &&\
	rm -rf /tmp/*

//工作目录
WORKDIR /srv/gitbook

//声明可挂载目录，/srv/gitbook用于挂载包含README.md和SUMARRY.md的目录
VOLUME /srv/gitbook /srv/html

//声明可暴露的端口
EXPOSE 4000 35729

//启动容器时启动gitbook服务
CMD /usr/local/bin/gitbook serve

```

###2 拉取blog静态文件

```

//先clone仓库或于远程主机建立关联
git pull origin master

```

###3 启动容器

```

docker run -v /home/blog:/srv/gitbook -p 4000:4000 -d --name blog fellah/gitbook

```

###4 配置nginx端口映射

```
server {
    listen       80;
    server_name  www.ponymew.com;
    #server_name 120.78.142.82;	
    #charset koi8-r;
    # 存到docker run挂载的目录
    access_log  /var/log/nginx/blog.access.log  main;
    error_log /var/log/nginx/blog.error.log;

    #以下是一些反向代理的配置，可选。
    client_max_body_size 10m; #允许客户端请求的最大单文件字节数
    client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
    proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
    proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
    proxy_buffer_size 64k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
    proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
    proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
    proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传



    location /static/ {
	expires 30d;
	alias /data/static/;
    }
    # 对 "/" 启用反向代理
    location / {
        proxy_pass              http://127.0.0.1:4000;
        proxy_redirect          off;
        proxy_set_header        Host            $host;
        proxy_set_header        X-Real-IP       $remote_addr;

        proxy_next_upstream http_502 http_504 error timeout invalid_header;

        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }
}

```

###5 启动nginx并访问

```
nginx -t
nginx -c /etc/nginx/nginx.conf
nginx -s reload

```
