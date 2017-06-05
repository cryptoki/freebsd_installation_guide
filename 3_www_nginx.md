## Webserver nginx Installation

### nginx
```
root@www-nginx:/ # cd /usr/ports/www/nginx
root@www-nginx:/usr/ports/www/nginx # make install clean
```
### php 7.1
```
root@www-nginx:/usr/ports/www/nginx # whereis php71
php71: /usr/ports/lang/php71
root@www-nginx:/usr/ports/www/nginx # cd /usr/ports/lang/php71
root@www-nginx:/usr/ports/lang/php71 # make install clean
```

### configuration
/etc/rc.conf for service start
```
# nginx/php
nginx_enable="YES"
php_fpm_enable="YES"
```

we have to set the correct timezone in the jail using tzsetup command.
```
root@www-nginx:/ # tzsetup
```

change the firewall rules in pf.conf
```
open_tcp = "{ 22, 80 }"
open_udp = "{ 22, 80 }"

rdr on $if proto tcp from any to $if port { 80, 443 } -> $www
rdr on $if proto udp from any to $if port { 80 } -> $www
```

reload rules
```
pfctl -f /etc/pf.conf
service pf reload
```

#### nginx
/usr/local/etc/nginx/nginx.conf
```
### Change the number of workers to the same number of cores your server has
worker_processes  4;

pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    ### MIME TYPES ### 
    include       mime.types;
    default_type  application/octet-stream;

    ### LOGGING ### 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    ### GENERAL ###
    index   index.php index.htm index.html;
    server_tokens  off;
    client_header_timeout  5;
    client_body_timeout  10;
    client_max_body_size  16m;
    ignore_invalid_headers  on;
    send_timeout  10;


    ### PERFORMANCE ###
    sendfile        on;
    server_names_hash_bucket_size  128;
    tcp_nodelay    on;
    tcp_nopush     on;
    keepalive_timeout   15;

    ### GZIP ### 
    gzip               on;
    gzip_vary          on;
    gzip_proxied       any;
    gzip_comp_level    6;
    gzip_buffers       16 8k;
    gzip_http_version  1.1;
    gzip_min_length    256;
    gzip_types         text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/vnd.ms-fontobject application/x-font-ttf font/opentype image/svg+xml image/x-icon;
    gzip_disable       "MSIE [1-6]\.(?!.*SV1)";

    ### SSL ###


    ### VHOST INCLUDES ###


    ### DEFAULT SERVER ###
    server {
        listen       80;
        server_name  _;

        access_log  /var/log/nginx/access.log  main;
        error_log   /var/log/nginx/error.log  warn;

        location / {
            root   /usr/local/www/nginx-dist;
            index  index.html index.htm index.php;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           /usr/local/www/nginx-dist;
            fastcgi_pass   10.1.1.2:9000;
            fastcgi_index  index.php;
            #fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/local/www/nginx-dist;
        }
    }
}
```
#### php / php-fpm
```
cp php.ini-production php.ini

vi php.ini

# change the timezone entry in the php.ini
date.timezone = Europe/Berlin
```

we using the default config files from php-fpm

##### to be discuss
```
;cgi.fix_pathinfo=1
```


#### testing
```
root@www-nginx:/ # cd /usr/local/www/nginx-dist
root@www-nginx:/usr/local/www/nginx-dist # echo "<?php phpinfo(); ?>" >> xyz_your_own_name.php
```

use u're browser and 
```
http://www.yourdomain.de => standard nginx page
http://www.yourdomain.de/xyz_your_own_name.php => php info page
```

#### ssl
https://search.thawte.com/support/ssl-digital-certificates/index?page=content&actp=CROSSLINK&id=SO17666
```
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
```

follow the instruction on the hetzner page for generating a ssl certificate.

#### curl
for better testing I install the curl tool
```
pkg search curl
whereis curl
cd /usr/ports/ftp/curl
make install clean
pkg autoremove

curl -k -v http://10.1.1.2:80
curl -H "Accept-Encoding: gzip" -I http://localhost:80
```