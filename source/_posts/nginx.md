---
title: Linux下Nginx与php-fpm的优化配置
date: 2017-11-28 16:26:52
tags:
    - go-micro
    - kafka
    - Go微服务
---
### linux下nginx与php-fpm的优化配置
前段时间魅族服务在访问高峰期出现概率性502服务器错误，10%的概率出现502，出现在新的服务器上，
旧服务器没问题。问题出现在新服务器Linux文件句柄限制上，默认是1024，改为65535即可。以下为
服务器常见配置：

#### linux文件句柄限制：
```
vim /etc/sysctl.conf
fs.file-max = 65535 #重启生效
```
#### nginx 参数
```
worker_processes  8; #，8核CPU，设置8个进程，最多不应超过8个进程，即便CPU核心数超过8个
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;   #8核CPU，开启8个进程对应的CPU，
worker_rlimit_nofile 50000; #单个nginx进程最大打开文件数，默认只有1024
events
{
     use epoll; #开启高性能方式
     worker_connections 50000; #单个worker最大并发数，8个worker的总并发数为8*50000
}

http
{
     server_names_hash_bucket_size 128; #设置缓存区
     sendfile on;
     tcp_nopush     on; #为sendfile时启用
     tcp_nodelay on; #为长连接时启用
     # 处理时间
     keepalive_timeout 120; #默认是65
     # 用户请求头的超时时间
     client_header_timeout 3m;
     # 用户请求体的超时时间
     client_body_timeout 3m;
     # 用户请求体最大字节数
     client_max_body_size 100m;

     # proxy 用
     #send_timeout 3m;
      connection_pool_size 256;
      client_header_buffer_size 64k;
      large_client_header_buffers 4 64k;
      request_pool_size 64k;
      output_buffers 4 64k;
      postpone_output 1460;
      client_body_buffer_size 256k;
      fastcgi_connect_timeout 1000;
      fastcgi_send_timeout 1000;
      fastcgi_read_timeout 1000;
      fastcgi_buffer_size 256k;
      fastcgi_buffers 8 256k;
      fastcgi_busy_buffers_size 256k;
      fastcgi_temp_file_write_size 256k;
      #     fastcgi_temp_path /dev/shm;
      #     fastcgi_intercept_errors on;

      #     open_file_cache max=50000 inactive=20s;
      #     # 多长时间检查一次缓存的有效信息
      #     open_file_cache_min_uses 1;
      #     open_file_cache_valid 30s;
      #打开压缩，默认关闭
      gzip on;
      gzip_min_length  4k;
      gzip_buffers     4 16k;
      gzip_http_version 1.1;
      gzip_comp_level 2;
      gzip_types       text/plain application/x-javascript text/css application/xml application/font-woff application/vnd.ms-fontobject application/font-sfnt image/svg+xml application/octet-stream;
      gzip_vary on;

      server {
          listen 80 default;
          return 403;
       }
}
```
#### php-fpm 参数
```
pm = dynamic //动态进程分配
pm.max_children = 192
pm.start_servers = 40
pm.min_spare_servers = 25
pm.max_spare_servers = 192
pm.max_requests = 4096
```