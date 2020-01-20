---
layout: post
title: "Nginx Performance"
date: 2020-01-20 20:06:06 +0100
categories: ['Nginx']
---

## Performance

### Headers & Expires

```
location ~* \.(css|js|jpg|png)$ {
    access_log off;
    # Enable cache
    add_header Cache-Control public;

    # Same as Cache-Control, old version
    add_header Pragma public;

    # Vary determines how to match future request headers to decide
    # whether a cached response can be used rather than 
    # requesting a fresh one from the origin server.
    add_header Vary Accept-Encoding;

    # The cache expires after 1 month
    expires 1M;
}
```

### Compressed Responses with gzip

If the client request has the header `Accept-Encoding: gzip`, we can then send the compressed responses with gzip.

```
http {
  # Turn on gzip
  gzip on;

  # The higher the level, the smaller the file
  # and the longer it takes for compressing
  gzip_comp_level 3;

  # Enables gzipping of responses for the specified MIME types
  gzip_types text/css;
  gzip_types text/javascript;
  
  # ...
}
```

This can be tested with

```shell
$ curl -I -H 'Accept-Encoding: gzip' ip_address/style.css
```

### FastCGI Cache

```
http {
  # ...
  # Configure microcache (fastcgi)
  # Sets the path and other parameters of a cache.
  fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=ZONE_1:100m inactive=60m;

  # Defines a key for caching
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
  add_header X-Cache $upstream_cache_status;

  server {
    # ...
    # Cache by default
    set $no_cache 0;

    # Check for cache bypass
    if ($arg_skipcache = 1) {
      set $no_cache 1;
    }

    location ~\.php$ {
      # Pass php requests to the php-fpm service (fastcgi)
      include fastcgi.conf;
      fastcgi_pass unix:/run/php/php7.1-fpm.sock;

      # Defines a shared memory zone used for caching.
      fastcgi_cache ZONE_1;

      # Sets caching time for different response codes.
      fastcgi_cache_valid 200 60m;

      # Defines conditions under which 
      # the response will not be taken from a cache.
      fastcgi_cache_bypass $no_cache;

      # Defines conditions under which 
      # the response will not be saved to a cache.
      fastcgi_no_cache $no_cache;
    }
  }
}
```

Test the performance:

```shell
$ sudo apt-get install apache2-utils

# -n number of requests in total
# -c number of concurrency
$ ab -n 100 -c 10 ip_address
```

### HTTP2

What does HTTP2 offer:

* Binary Protocal
* Compressed Headers
* Persistent Connections
* Multiplex Streaming
* Server Push

To use HTTP2, we have to install the HTTP2 module of nginx, and add ssl certification.

```
http {
  # ...
  server {
    listen 443 ssl http2;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # ...
  }
}
```

### Server push

This feature need HTTP2, it will improve the performance by reducing request number.

By default, the browser get index.html, it finds instructions that will require styles.css and script.js. At that point, the browser will issue requests to get these other two files.

With server push, the server can take the initiative by having rules that trigger content to be sent even before it is requested.

```
# whenever a client requests index.html, also push
# /style.css and /thumb.png
location = /index.html {
  http2_push /style.css;
  http2_push /thumb.png;
}
```

Test the performance:

```shell
$ sudo apt-get install nghttp2-client

$ nghttp -nys ip_address/index.html
# Before enabling server push, it should get only index.html
# After enabling server push,
# it should get the index.html, style.css and thumn.png
```

For more info: [Introducing HTTP/2 Server Push with NGINX 1.13.9](https://www.nginx.com/blog/nginx-1-13-9-http2-server-push)
