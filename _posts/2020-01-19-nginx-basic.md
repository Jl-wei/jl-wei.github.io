---
layout: post
title: "Nginx basic"
date: 2020-01-19 11:01:59 +0100
categories: ['Nginx']
---

## Installation

### Install from source

```shell
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
$ tar -zxvf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1

$ ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
$ make
$ sudo make install
```

#### Adding an Nginx Service

Save this [file](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/) as `/lib/systemd/system/nginx.service`, you may need to change some of the configuration based on your Nginx installation config.

```properties
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```shell
# Start nginx
systemctl start nginx

# Check status
systemctl status nginx

# Restart nginx, restart = stop + start
systemctl restart nginx

# Reload nginx, reload = remain running + re-read configuration files
systemctl reload nginx
```

## Configuration

```
user www-data;

events {
}

http {
  include mime.types;

  server {
    listen 80;
    server_name 15.236.5.206;

    root /sites/demo;
    index index.html;

    location = /thumb.png {
        image_filter rotate 180;
    }
  }
}
```

### Location

#### Location priority

It fires in this order:

1. `=`(exactly)

   ```
   location = /path
   ```

2. `^~`(forward match)

   ```
   location ^~ /path
   ```

3. `~`(regular expression case sensitive)

   ```
   location ~ /path/
   ```

4. `~*`(regular expression case insensitive)

   ```
   location ~* .(jpg|png|bmp)
   ```

5. `/`

   ```
   location /path
   ```

### Rewrites & Redirects

`rewrite` can replace the URI which match the regex by the second parameter, then continue executing.

```
server {
  listen 80;
  server_name 15.236.5.206;

  root /sites/demo;

  # With `last` flag, it stops processing the set of rewrite directives
  rewrite ^/user/(\w+) /greet/$1 last;
  rewrite ^/greet/john /thumb.png;

  location /greet {
    return 200 "Hello User";
  }

  location = /greet/john {
    return 200 "Hello John";
  }
}
```

### Logging

The location of the log file can be set with `access_log` and `error_log`.

These 2 directives can also be overwrite in child context.

They can also be closed by

```
access_log off;
```

### Worker processes

```
# The number of worker process, 
# with auto, same number as the CPU core number
worker_processes auto;

events {
  # Maximum connection of one worker,
  # should be less than `ulimit -n`
  worker_connections 1024;
}
```

### Buffers & Timeouts

These directives should be tweaked depends on your requests.

By default, the buffer are in the RAM. But if the buffer sizes are set too low, Nginx will have to write to a temporary file causing the disk to read and write constantly.

```
http {
  # Buffer size for POST submissions
  client_body_buffer_size 10K;
  client_max_body_size 8m;

  # Buffer size for Headers
  client_header_buffer_size 1k;

  # Max time to receive client headers/body
  client_body_timeout 12;
  client_header_timeout 12;

  # Max time to keep a connection open for
  keepalive_timeout 15;

  # Max time for the client accept/receive a response
  send_timeout 10;

  # Skip buffering for static files
  sendfile on;

  # Optimise sendfile packets
  tcp_nopush on;
  
  # ...
}
```

### Adding dynamic modules

Firstly, rebuild nginx from source with the module

```shell
$ cd nginx-1.16.1/
$ sudo apt-get install libgd-dev
$ ./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module --with-http_image_filter_module=dynamic --modules-path=/etc/nginx/modules
$ make
$ make install
$ systemctl reload nginx
```

Then include it in `nginx.conf` file then `systemctl reload nginx`

```yaml
load_module /etc/nginx/modules/ngx_http_image_filter_module.so;
```

### Backend application

#### FastCGI

FastCGI is one way to run the backend web application. It needs you to have the application which support FastCGI. In Ubuntu, you could run PHP as FastCIG with

```shell
$ sudo apt-get install php7-fpm
```

Then configure the PHP app and start it.

At the Nginx conf file:

```
location ~\.php$ {
  # Pass php requests to the php-fpm service (fastcgi)
  include fastcgi.conf;
  fastcgi_pass unix:/run/php/php7.2-fpm.sock;
}
```

#### Reverse proxy

Another way of using Nginx to serve a web application is reverse proxy, it allows Nginx to forward a request from client to your app, like Rails application:

```
http {
  upstream rails_app { 
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
  }

  server {
    listen 80;
    root /path/to/application/public;

    location / {
      error_page 404 /404.html; 
      error_page 500 502 503 504 /50x.html;

      try_files $uri $uri/index.html @rails;
    }
    
    location @rails {
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_pass http://rails_app;
    }
  }
}
```

In the example above, `upstream` create a load balancer named `rails_app` which balance the request to the three rails applications.

Reverse proxy is a more popular and scalable way for serving backend applications.