# Nginx

## Nginx简介
Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器

### Nginx作用
1. 高并发
2. web服务器
3. 反向代理服务器
4. 负载均衡

### 与其他服务器的比较
1. Apache的优点是稳定，bug少，模块多，rewrite强大
2. Nginx的优点是配置简单，运行效率高，高并发，占用资源少

## Nginx使用

基本配置`/etc/nginx/nginx.cnf`

```
user nginx;
worker_processes 8;
pid    /var/run/nginx.pid;
worker_rlimit_nofile 51200;
events
{
    use epoll;
    worker_connections 51200;
}
http{
    include       mime.types;
    default_type application/octet-stream;
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;
    sendfile on;
    tcp_nopush     on;
    keepalive_timeout 60;
    tcp_nodelay on;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    upstream pinpoint-web
    {
        ip_hash;
        server 172.20.1.19:28086;
        server 172.20.0.179:28086;
        server 172.20.0.123:28086;
    }

    server {
        listen 8888;
        server_name pinpoint.web.hand;
        location / {
            root /var/www/html ;
            index index.php index.htm index.html;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://pinpoint-web;
        }

        location /nginx {
            access_log off;
            auth_basic "NginxStatus";
            #auth_basic_user_file /usr/local/nginx/htpasswd;
        }
    }

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

}
```