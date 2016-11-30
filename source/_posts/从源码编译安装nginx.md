---
title: 从源码编译安装nginx
date: 2016-11-23 16:56:23
categories:
- Development
tags:
- nginx
---

1. 参考文档： [参考文档](http://nginx.org/en/docs/configure.html)

2. 下载nginx源码：[download](http://nginx.org/download/) 

3. 预先下载所需模块：[pcre](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/)     [nginx_ngx_cache_purge](http://labs.frickle.com/nginx_ngx_cache_purge/)    [zlib](http://zlib.net/)

4. 安装ssl模块：` yum -y install openssl openssl-devel`

5. 安装gcc编译器：` yum install -y gcc gcc-c++`

6. 目录结构如下：

   ```shell
   [devadmin_sam@hzserver nginx]$ ll
   总用量 24
   drwxr-xr-x  9 devadmin_sam dev  4096 11月 23 13:12 nginx-1.10.2
   drwxr-xr-x  3 devadmin_sam dev  4096 12月 24 2014 ngx_cache_purge-2.3
   drwxr-xr-x  9 devadmin_sam dev 12288 11月 23 13:14 pcre-8.39
   drwxr-xr-x 14 devadmin_sam dev  4096 11月 23 13:14 zlib-1.2.8
   ```

7. 编译参数：

   ```shell
   ./configure\
   --prefix=/etc/nginx\
   --sbin-path=/usr/sbin/nginx\
   --conf-path=/etc/nginx/nginx.conf\
   --error-log-path=/var/log/nginx/error.log\
   --http-log-path=/var/log/nginx/access.log\
   --pid-path=/var/run/nginx.pid\
   --lock-path=/var/run/nginx.lock\
   --http-client-body-temp-path=/var/cache/nginx/client_temp\
   --http-proxy-temp-path=/var/cache/nginx/proxy_temp\
   --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp\
   --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp\
   --http-scgi-temp-path=/var/cache/nginx/scgi_temp\
   --user=nginx\
   --group=nginx\
   --with-http_ssl_module\
   --with-http_realip_module\
   --with-http_addition_module\
   --with-http_sub_module\
   --with-http_dav_module\
   --with-http_flv_module\
   --with-http_mp4_module\
   --with-http_gunzip_module\
   --with-http_gzip_static_module\
   --with-http_random_index_module\
   --with-http_secure_link_module\
   --with-http_stub_status_module\
   --with-http_auth_request_module\
   --with-threads\
   --with-stream\
   --with-stream_ssl_module\
   --with-http_slice_module\
   --with-mail\
   --with-mail_ssl_module\
   --with-file-aio\
   --with-http_v2_module\
   --with-ipv6\
   --with-pcre=../pcre-8.39\
   --with-zlib=../zlib-1.2.8\
   --add-module=../ngx_cache_purge-2.3
   ```

8. `make && make install`

