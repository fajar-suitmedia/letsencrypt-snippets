# letsencrypt-snippets
# cara mudah dapetin ssl letsencrypt
Buat Dockerfile isinya:
```bash
FROM nginx:alpine
RUN apk add python3 python3-dev py3-pip build-base libressl-dev musl-dev libffi-dev rust cargo
RUN pip3 install pip --upgrade
RUN pip3 install certbot-nginx
RUN mkdir /etc/letsencrypt
COPY default.conf /etc/nginx/conf.d/default.conf``
```

## file default.conf
```bash
server {
    listen       80;
    server_name  yourdomain.com;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## generate dh-param.pem
```bash
openssl dhparam -out /etc/nginx/ssl/dhparam-2048.pem 2048
```

## file options-ssl-nginx.conf
```bash
# This file contains important security parameters. If you modify this file
# manually, Certbot will be unable to automatically provide future security
# updates. Instead, Certbot will print and log an error message with a path to
# the up-to-date file that you will need to refer to when manually updating
# this file.

ssl_session_cache shared:le_nginx_SSL:10m;
ssl_session_timeout 1440m;
ssl_session_tickets off;

ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;

ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
```

## jalankan perintah berikut:
```bash
$ docker build -t nginx-certbot .
$ docker run -v $(pwd)/letsencrypt:/etc/letsencrypt --name nginx -ti -p 80:80 nginx-certbot sh
$ certbot certonly --standalone --preferred-challenges http -d example.com
$ exit container
$ docker cp nginx-certbot:/etc/letsencrypt .
```

## file yg dibutuhkan untuk ssl pada config di nginx
- copy file dari /etc/letsencrypt/live/example.com/fullchain.pem #ssl_certificate
- copy file dari /etc/letsencrypt/live/example.com/privkey.pem #ssl_certificate_key
- file dh-param.pem #ssl_dhparam
- options-ssl-nginx.conf #include
