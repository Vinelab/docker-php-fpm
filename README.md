# Docker • PHP-FPM

A PHP-FPM image with the following packages (compactible with Laravel projects)

- gd
- iconv
- mcrypt
- mbstring

### Usage

#### With NGINX

- Create the host file `nginx.host.conf`

```conf
upstream php {
    server fpm:9000;
}

server {
    listen       80;
    server_name  localhost;
    root /code; # for Laravel projects use /code/public

    index index.php index.html index.htm;

    access_log off;
    error_log /var/log/nginx/error.log;

    client_max_body_size 20M;

    add_header Access-Control-Allow-Origin *;

    charset utf-8;

    location / {
        try_files $uri /index.php?$query_string;
    }

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt  { log_not_found off; access_log off; }

    location ~ \.(php)$ {
        fastcgi_split_path_info   ^(.+\.php)(/.*)$;
        include php.conf;
    }

    # Deny .htaccess file access
    location ~ /\.ht {
        deny all;
    }

    location /nginx_status {
        # Turn on nginx stats
        stub_status on;
        # I do not need logs for stats
        access_log   off;
        # Security: Only allow access from 172.17.0.0/16 IP #
        allow 172.17.0.0/16;
        # Send rest of the world to /dev/null #
        deny all;
   }
}
```

- Create the fastcgi params file `nginx.php.conf`

```conf
try_files                 $uri =404;
fastcgi_pass              php;
fastcgi_index             index.php;
fastcgi_param             SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_intercept_errors  on;
fastcgi_buffers         16 16k;
fastcgi_buffer_size     16k;
include                   fastcgi_params;
```

##### Run the Code Container
```
docker run -d \
    --name code \
    -v /path/to/code:/code \
    centos:7 \
    /bin/sh -c "while true; do echo hello world > /dev/null; sleep 1; done"
```

##### Run PHP-FPM
```
docker run -d \
    --name php \
    --expose 9000 \
    --volumes-from code \
    vinelab/php-fpm
```

##### Run NGINX
```
docker run -d \
    -p 80:80 \
    --volumes-from code \
    -v /path/to/nginx.host.conf:/etc/nginx/conf.d/default.conf \
    -v /path/to/nginx.php.conf:/etc/nginx/php.conf \
    --link php \
    nginx
```

For the full documentation please visit:
- PHP: https://hub.docker.com/_/php/
- NGINX: https://hub.docker.com/_/nginx/
