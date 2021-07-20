Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Создайте virtual host для host regex.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
        Все запросы на /admin/* должны возвращать 301 редирект на страницу /adminochka/* (путь запроса должен сохраняться в новом url).
        Все запросы на /user/* должны уходить на скрипт /index.php?user=* (все, что идет после /user/, должно стать аргументом для index.php?user=).
        Все запросы, которые идут на /index.php, должны возвращать код 200.
    Проверьте, что все location возвращают необходимые значения и отправляйте задание на проверку.

server {
  listen 80;
  server_name regex.cca92a30535d408335b1a6754cf66955.kis.im;

  location /admin/ {
    rewrite ^/admin/(.*)$ /adminochka/$1 permanent;
  }

  location /user/ {
    rewrite ^/user/(.*)$ /index.php?user=$1;
  }

  location = /index.php {
    return 200;
  }
}







root@cca92a30535d408335b1a6754cf66955:/etc/nginx# curl -D - -s regex.cca92a30535d408335b1a6754cf66955.kis.im/user/d
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 20 Jul 2021 06:45:41 GMT
Content-Type: text/html
Content-Length: 170
Location: http://regex.cca92a30535d408335b1a6754cf66955.kis.im/index.php?user=d
Connection: keep-alive

<html>
<head><title>302 Found</title></head>
<body bgcolor="white">
<center><h1>302 Found</h1></center>
<hr><center>nginx/1.14.0 (Ubuntu)</center>
</body>
</html>

При убирании redirect опции мы уже не увидим Location с преобразованием 

root@cca92a30535d408335b1a6754cf66955:/etc/nginx# vi conf.d/regex.cca92a30535d408335b1a6754cf66955.kis.im.conf 
root@cca92a30535d408335b1a6754cf66955:/etc/nginx# systemctl restart nginx
root@cca92a30535d408335b1a6754cf66955:/etc/nginx# curl -D - -s regex.cca92a30535d408335b1a6754cf66955.kis.im/user/d
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 20 Jul 2021 06:46:23 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive



