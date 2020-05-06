---
layout: post
title: "Nginx Security"
date: 2020-01-21 21:06:06 +0100
categories: ['Nginx']
---

## Security

### HTTPS

```
# ...

http {
  # ...
  # Redirect all traffic to HTTPS
  server {
    listen 80;
    server_name 15.236.5.206;
    return 301 https://$host$request_uri;
  }

  server {
    # ...
		
    listen 443 ssl http2;
    server_name 15.236.5.206;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    # Enables the specified protocols
    # Disable SSL, as it's outdated
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # Optimise cipher suits
    # Specifies that server ciphers should be preferred over client ciphers
    ssl_prefer_server_ciphers on;

    # Specifies the enabled ciphers
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enable DH Params
    # Specifies a file with DH parameters for DHE ciphers
    # By default, DHE ciphers will not be used
    # `openssl dhparam 2048 -out /etc/nginx/ssl/dhparam.pem`
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    # Enable HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL sessions
    # Sets the types and sizes of caches that store session parameters
    ssl_session_cache shared:SSL:40m;

    # Specifies a time during which a client may reuse the session parameters
    ssl_session_timeout 4h;

    # Enables or disables session resumption through TLS session tickets
    ssl_session_tickets on;

    # ...
  }
}

```

### Rate limiting

Rate limiting is used to control the rate of traffic sent or received by a network interface controller and is used to prevent DoS attacks.

- **Security** - Brute force protection
- **Reliability** - Prevent traffic spikes
- **Shaping** - Service priority

`limit_req_zone key zone=name:size rate=rate [sync];`

* `key` Rate limiting limit the request processing rate per a defined key, the key can contain text, variables, and their combination.
* `zone=name:size` defines a zone to store the states.
* `rate` defines the limit of frequency for each key.
  * For NGINX, `300r/m` and `5r/s` are treated the same way: allow one request every 0.2 seconds for this zone. Every 0.2 second, in this case, NGINX will set a flag to remember it can accept a request. When a request comes in that fits in this zone, NGINX sets the flag to false and processes it. If another request comes in before the timer ticks, it will be rejected immediately with a 503 status code. If the timer ticks and the flag was already set to accept a request, nothing changes.

`limit_req zone=name [burst=number] [nodelay | delay=number];`

* `zone` lets you define a bucket, a shared ‘space’ in which to count the incoming requests. All requests coming into the same bucket will be counted in the same rate limit. This is what allows you to limit per URL, per IP, or anything fancy.
* `burst` Excessive requests are delayed until their number exceeds the maximum burst size in which case the request is terminated with an [error](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_status).
* `delay` specifies a limit at which excessive requests become delayed. Default value is zero, i.e. all excessive requests are delayed.
* `nodelay` tells NGINX that the requests it accepts in the burst window should be processed immediately, like regular requests.

```
# ...
http {
  # ...

  # Define limit zone
  limit_req_zone $request_uri zone=MYZONE:10m rate=1r/s;

  # ...

  server {
    # ...

    location / {
      limit_req zone=MYZONE burst=5 nodelay;
      try_files $uri $uri/ =404;
    }

    # ...
  }
}
```

Test:

```shell
$ sudo apt-get install siege

# -v verbose mode, print result of each connection
# -r request per connection
# -c concurrent number of users
$ siege -v -r 2 -c 5 https://35.180.48.247/thumb.png
```

For more info: [NGINX rate-limiting in a nutshell](https://www.freecodecamp.org/news/nginx-rate-limiting-in-a-nutshell-128fe9e0126c/), [Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx/)

### Basic Auth

Generate password with:

```shell
htpasswd -c /etc/nginx/.htpasswd username
```

Set the auth:

```
location / {
  auth_basic "Secure Area";
  auth_basic_user_file /etc/nginx/.htpasswd;
  # ...
}
```

### Hide Nginx version

Hackers may attack your server if you use a old Nginx version. 

```
http {
  # Remove the Nginx version number from the response header
  server_tokens off;
  # ...
}
```

### Clickjacking and XSS

```
server {
  # ...
  # Indicate whether or not a browser should be allowed 
  # to render a page in a <frame>, <iframe>, <embed> or <object>
  add_header X-Frame-Options "SAMEORIGIN";

  # stops pages from loading when they detect 
  # reflected cross-site scripting (XSS) attacks
  add_header X-XSS-Protection "1; mode=block";
  # ...
}
```

### Remove useless modules

Some of the built-in modules in Nginx are useless and dangerous, we can install Nginx without these modules, like `http_autoindex_module`.
