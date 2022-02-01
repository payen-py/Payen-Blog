---
title: nginx1.14部署hexo博客
tags:
  - Nginx
  - Hexo
  - 博客
categories:
  - - Linux
    - Nginx
  - - Linux
    - Hexo
  - - 博客部署
date: 2022-01-29 20:47:11
updated: 2022-02-01 00:00:00
---


总结一下我在Ubuntu18.04服务器上使用nginx部署hexo博客的过程

## hexo操作

hexo的安装和基本操作见[Hexo官方文档](https://hexo.io/docs/)

### 修改_config.yml文件

修改文件中的url，后面nginx中的location要与它的相对路径与保持一致

```yaml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
# 我使用相对路径/blog作为我博客的url，与nginx设置的location保持一致
# 也可以直接使用根url作为博客的url
# hexo生成的静态文件内使用的是相对路径，所以这里的域名可以随便写，不影响网站部署
url: http://pengyu.website/blog
```

### 清除hexo缓存文件和已生成的静态文件

进入hexo项目文件夹

```bash
$ hexo clean
```

### 生成静态文件

```bash
$ hexo g -d
```

得到一个名为public的文件夹

## Nginx配置（直接使用apt安装）

### 修改配置文件

使用vim打开nginx配置文件

```bash
$ sudo vim /etc/nginx/nginx.conf
```

在http块中写入新的server块，具体内容如下

```yaml
# server config
server {
    listen 80;
    server_name 服务器ip或域名;
    charset utf-8;
    location /blog { # 这里与_config.yml中设置的相对路径/blog一致
        # 由于博客没有映射到根路由，需要设置alias到静态文件夹的目录
        alias /home/pengyu/project/web/blog/public;
        # 若直接将博客映射到根路由，设置root为静态文件夹目录即可
        # root /home/pengyu/project/web/blog/public;
        index index.html;
    }
}
```

在http块中找到以下两行并将其注释，不然会使用默认的配置文件，使新写入的server块无效，当时我在这卡了几小时/(ㄒoㄒ)/~~

```yaml
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*; # 主要是这行
```

### 测试配置文件是否正确

输出以下信息则成功

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 运行nginx

```bash
$ sudo nginx -s reload
```

若输出

```output
nginx: [error] open() "/run/nginx.pid" failed (2: No such file or directory)
```

说明nginx未启动，使用以下命令启动nginx

```bash
$ sudo nginx -c /etc/nginx/nginx.conf
```

## 未完待续

之后考虑使用ssl加强连接的安全性
