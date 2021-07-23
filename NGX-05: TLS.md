NGX-05: TLS
    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Установите пакет letsencrypt.
    Создайте 2 virtual host для: ssl.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
        1-ый Virtualhost должен слушать на 80 порту для протокола http.
        Все запросы на /.well-known должны идти в папку /opt/www/acme.
        Все остальные запросы должны возвращать редирект на https-версию.
        Второй virtualhost должен слушать на 443 порту для протокола https.
        Для этого сервера должны быть прописаны полученные SSL сертификаты.
        Сервер должен поддерживать TLS версии 1.3 и только.
        Сервер https должен возвращать код 201 на все запросы.
    Проверьте, что все условия выполнены и отправляйте задание на проверку.



server {
  listen 80 default_server;
  server_name ssl.00882b1e465b13e557dc612ade138855.kis.im;

  location /.well-known {
    root /opt/www/acme;
  }

   location / { 
   return 301 https://ssl.00882b1e465b13e557dc612ade138855.kis.im$request_uri;
  }

}

server {
  listen 443 ssl;

  ssl_certificate /path/to/sslcert.pem;
  ssl_certificate_key /path/to/sslcert.key;

  location / { return 201; }
}






root@00882b1e465b13e557dc612ade138855:~# vi /etc/nginx/nginx.conf 
root@00882b1e465b13e557dc612ade138855:~# vi /etc/nginx/conf.d/ssl.00882b1e465b13e557dc612ade138855.kis.im.conf
root@00882b1e465b13e557dc612ade138855:~# systemctl restart nginx
root@00882b1e465b13e557dc612ade138855:~# mkdir -p /opt/www/acme
root@00882b1e465b13e557dc612ade138855:~# letsencrypt certonly --webroot -w /opt/www/acme -d ssl.00882b1e465b13e557dc612ade138855.kis.im
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): garry1287@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for ssl.00882b1e465b13e557dc612ade138855.kis.im
Using the webroot path /opt/www/acme for all unmatched domains.
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ssl.00882b1e465b13e557dc612ade138855.kis.im/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/ssl.00882b1e465b13e557dc612ade138855.kis.im/privkey.pem
   Your cert will expire on 2021-10-18. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

root@00882b1e465b13e557dc612ade138855:~# 


root@00882b1e465b13e557dc612ade138855:~# ls -al /etc/letsencrypt/live/ssl.00882b1e465b13e557dc612ade138855.kis.im/
total 28
drwxr-xr-x 2 root root 4096 Jul 20 20:17 .
drwx------ 3 root root 4096 Jul 20 20:17 ..
-rw-r--r-- 1 root root  682 Jul 20 20:17 README
lrwxrwxrwx 1 root root   67 Jul 20 20:17 cert.pem -> ../../archive/ssl.00882b1e465b13e557dc612ade138855.kis.im/cert1.pem
Мой сертификат без цепочки сертификационных центров


lrwxrwxrwx 1 root root   68 Jul 20 20:17 chain.pem -> ../../archive/ssl.00882b1e465b13e557dc612ade138855.kis.im/chain1.pem
Цепочка сертификационных центров

lrwxrwxrwx 1 root root   72 Jul 20 20:17 fullchain.pem -> ../../archive/ssl.00882b1e465b13e557dc612ade138855.kis.im/fullchain1.pem
Цепочка сертификационных центров + мой сертификат


lrwxrwxrwx 1 root root   70 Jul 20 20:17 privkey.pem -> ../../archive/ssl.00882b1e465b13e557dc612ade138855.kis.im/privkey1.pem
Приватный ключи



2021/07/20 20:28:36 [error] 3004#3004: *7 open() "/opt/www/acme/.well-known/non-existinng" failed (2: No such file or directory), client: 84.201.183.94, server: ssl.00882b1e465b13e557dc612ade138855.kis.im, request: "GET /.well-known/non-existinng HTTP/1.1", host: "ssl.00882b1e465b13e557dc612ade138855.kis.im"



server {
  listen 80 default_server;
  server_name ssl.00882b1e465b13e557dc612ade138855.kis.im;

  location /.well-known {
    root /opt/www/acme;
  }

   location / { 
   return 301 https://ssl.00882b1e465b13e557dc612ade138855.kis.im$request_uri;
  }

}

server {
  listen 443 ssl;

  ssl_certificate /etc/letsencrypt/live/ssl.00882b1e465b13e557dc612ade138855.kis.im/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/ssl.00882b1e465b13e557dc612ade138855.kis.im/privkey.pem;
  
  server_name ssl.00882b1e465b13e557dc612ade138855.kis.im;

  location /index.html { alias /tmp/index.html; }
}


location /index.html { alias /tmp/index.html; } Сделал чтобы посмотреть сертификат