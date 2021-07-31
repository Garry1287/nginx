NGX-10: Upstream check module

Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Скомпилируйте nginx с динамическим (!) модулем geoip2 и статическим (!) модулем upstream_check_module (p.s. - модуль geoip2 будет лежать в /opt/nginx-1.14.0/debian/build-core/objs/ngx_http_geoip2_module.so - его надо будет скопировать в modules nginx: cp /opt/nginx-1.14.0/debian/build-core/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules/).
    Подключите модуль geoip2 в конфигурации nginx, по аналогии с другими подключаемыми модулями.
    Установите geoipupdate и укажите свой лицензионный ключ в /etc/GeoIP.conf (необходимо получить аккаунт в maxmind.com - при получении лицензионного ключа вам потребуется указать что geoipupdate идет версией ниже 3.1.1 - https://www.maxmind.com/en/my_license_key), после чего скачайте базы - они появятся в /var/lib/GeoIP/. ВНИМАНИЕ: если для установки дополнительных библиотек будет добавлен репозиторий, то базы будут загружены не в указанное место. Устанавливаете из стандартных репозиториев ubuntu.
    Настройте переменную $geoip2_data_country_code в nginx, которая будет получать двухбуквенный код страны из которых пришел запрос (WARNING: В примере конфига с сайта разработчика указана неверная директива в source).
    Настройте два виртуальных хоста на порту 81 и 82, которые будут отдавать при запросе / код 201 и 202 соответственно, а при запросе /status - код 200. (Они должны быть доступны извне).
    Настройте основной виртуальный хост backend.${base_domain} на порту 80, который все запросы (/) будет проксировать на backend сервера
    Основной виртуальный хост nginx должен проверять состояние backend серверов следующим образом:
        Интервал проверки - 1 секунда
        После 2 (двух) неверных ответов хост исключается из балансировки
        После 1 (одного) удачного ответа хост возвращается в балансировку
        Проверка должна запрашивать страницу GET /status
    При проксировании основной хост должен передавать backend хостам хедер X-Country, который содержит двухбуквенный код страны пользователя.
    При ответе пользователю основной virtual host (на 80 порту) должен отдавать хедер X-Internal-Country, который содержит двухбуквенный код страны пользователя (хедер должен отдаваться всегда - вне зависимости от кода ответа backend сервера).
    Проверьте что все условия выполнены и отправляйте задание на проверку.


Сборка прошла по мануалу, только ошибался в синтаксисе
Плюс при установке была проблема из-за того, что удалил nginx модули, которые уже были в системе. Пришлось снова их ставить
apt --fix-broken install
dpkg -i /opt/nginx-core_1.14.0-0ubuntu1.9_amd64.deb


http {
    ...
    geoip2 /var/lib/GeoIP/GeoLite2-Country.mmdb {
        auto_reload 5m;
        $geoip2_metadata_country_build metadata build_epoch;
        # $geoip2_data_country_code default=US source=$variable_with_ip country iso_code;
        $geoip2_data_country_code default=US country iso_code;
        $geoip2_data_country_name country names en;
        }





upstream backend {
  server 127.0.0.1:81;
  server 127.0.0.1:82;

  check interval=1000 rise=2 fall=1 timeout=1000 type=http;
  check_http_send "GET /status HTTP/1.0\r\n\r\n";
  check_http_expect_alive http_2xx http_3xx;

}


server {
  listen 80 default_server;
  server_name backend.52cff97b039e67f68a8d35e3e32f2259.kis.im;

  location / {
    proxy_pass http://backend;
    proxy_set_header X-Country $geoip2_data_country_code;
    add_header X-Internal-Country $geoip2_data_country_code always;
  }

}


server {
  listen 81;

  location / {
    return 201 "Backend 1\n";
  }

  location /status {
    return 200;
  }

}

server {
  listen 82;

  location / {
    return 202 "Backend 2\n";
  }

  location /status {
    return 200;
  }

}








Нашёл пример (https://searchengines.guru/ru/forum/1007844)

http://www.bsdportal.ru/viewtopic.php?f=58&t=29015
https://serverfault.com/questions/585204/nginx-geoipblocking-allowing-lan-ips

Я нашел эту команду, но что-то ошибка какая-то вылетела.

В общем пересобрал, все работает. Но так как инфы на русском в сети мало, выложу сюда конфиг, чтобы русскоязычным удобно было.

В конфиг Nginx в самом начале нужно прописать:
user www-data;
worker_processes auto;
pid /run/nginx.pid;
load_module "modules/ngx_http_geoip2_module.so";
......

Далее в секции http {
http {
.........

geoip2 /usr/share/GeoIP/GeoLite2-Country.mmdb {
$geoip2_data_city_name city names en;
$geoip2_data_postal_code postal code;
$geoip2_data_latitude location latitude;
$geoip2_data_longitude location longitude;
$geoip2_data_state_name subdivisions 0 names en;
$geoip2_data_state_code subdivisions 0 iso_code;
}

geoip2 /usr/share/GeoIP/GeoLite2-City.mmdb {
$geoip2_data_country_code default=US country iso_code;
$geoip2_data_city_name city names en;
$geoip2_data_country_name country names en;
$geoip2_data_state_code subdivisions 0 iso_code;
}

..............

В конфиг нужного сайта в секцию
location ~ \.php$ {
...........
fastcgi_param COUNTRY_CODE $geoip2_data_country_code;
fastcgi_param COUNTRY_NAME $geoip2_data_country_name;
fastcgi_param CITY_NAME $geoip2_data_city_name;
fastcgi_param STATE_CODE $geoip2_data_state_code;

Для тестирования создайте пхп файл

<?php
$geoip2_data_city_name = getenv(CITY_NAME);
echo $geoip2_data_city_name;
?>
<br>
<?php
$geoip2_data_country_code = getenv(COUNTRY_CODE);
echo $geoip2_data_country_code;
?>
<br>
<?php
$geoip2_data_country_name = getenv(COUNTRY_NAME);
echo $geoip2_data_country_name;
?>
<br>
<?php
$geoip2_data_state_code = getenv(STATE_CODE);
echo $geoip2_data_state_code;
?>

Для Москвы я получил:
Moscow
RU
Russia
MOW


Отличная статья
https://www.getpagespeed.com/server-setup/nginx/upgrade-to-geoip2-with-nginx-on-centos-rhel
https://mclouds.ru/2020/12/nginx-geo2ip/


https://dev.maxmind.com/geoip/updating-databases
https://www.maxmind.com/en/accounts/588635/license-key/create
https://github.com/leev/ngx_http_geoip2_module
https://github.com/yaoweibin/nginx_upstream_check_module


add_header X-Country $geoip2_data_country_name always;

Надо было добавить слово always
https://stackoverflow.com/questions/57041647/reverse-proxy-when-proxied-service-returns-http-202
















Сообщение куратора
Оценка 4
autoTeacher
31.07.2021 09:22

Checking nginx upstream module installed: OK

Checking nginx geoip2 module installed: OK

Checking nginx geoip2 module loaded: OK

Checking geoipupdate package installed: OK

Checking geoip2 database present: OK

Checking geoip2_data_country_code variable present: FAILED

Checking check module interval: OK

Checking check module rise: FAILED

Checking check module fall: FAILED

Checking check module check status page: OK

Checking if X-Country header set: OK

Checking backend1 configuration by http: OK

Checking backend2 configuration by http: OK

Checking backend1 status page: OK

Checking backend2 status page: OK

Checking backend configuration: OK







Проверка
garry@g-notebook:~/Downloads$ curl -D - -s backend.52cff97b039e67f68a8d35e3e32f2259.kis.im
HTTP/1.1 201 Created
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 31 Jul 2021 06:28:46 GMT
Content-Type: application/octet-stream
Content-Length: 10
Connection: keep-alive
X-Internal-Country: RU

Backend 1
garry@g-notebook:~/Downloads$ curl -D - -s backend.52cff97b039e67f68a8d35e3e32f2259.kis.im
HTTP/1.1 202 Accepted
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 31 Jul 2021 06:28:47 GMT
Content-Type: application/octet-stream
Content-Length: 10
Connection: keep-alive
X-Internal-Country: RU

Backend 2
garry@g-notebook:~/Downloads$ curl -D - -s backend.52cff97b039e67f68a8d35e3e32f2259.kis.im
HTTP/1.1 201 Created
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 31 Jul 2021 06:28:49 GMT
Content-Type: application/octet-stream
Content-Length: 10
Connection: keep-alive
X-Internal-Country: RU

Backend 1
garry@g-notebook:~/Downloads$ 







root@52cff97b039e67f68a8d35e3e32f2259:/etc/nginx# curl -D - -s backend.52cff97b039e67f68a8d35e3e32f2259.kis.im
HTTP/1.1 202 Accepted
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 31 Jul 2021 06:29:24 GMT
Content-Type: application/octet-stream
Content-Length: 10
Connection: keep-alive
X-Internal-Country: DE

Backend 2
root@52cff97b039e67f68a8d35e3e32f2259:/etc/nginx# curl -D - -s backend.52cff97b039e67f68a8d35e3e32f2259.kis.im
HTTP/1.1 201 Created
Server: nginx/1.14.0 (Ubuntu)
Date: Sat, 31 Jul 2021 06:29:25 GMT
Content-Type: application/octet-stream
Content-Length: 10
Connection: keep-alive
X-Internal-Country: DE

Backend 1
root@52cff97b039e67f68a8d35e3e32f2259:/etc/nginx# 






root@52cff97b039e67f68a8d35e3e32f2259:/etc/nginx# ls -al modules-enabled/
total 16
drwxr-xr-x 2 root root 4096 Jul 31 05:37 .
drwxr-xr-x 8 root root 4096 Jul 31 05:45 ..
lrwxrwxrwx 1 root root   54 Jul 31 05:31 50-mod-http-geoip.conf -> /usr/share/nginx/modules-available/mod-http-geoip.conf
lrwxrwxrwx 1 root root   61 Jul 31 05:31 50-mod-http-image-filter.conf -> /usr/share/nginx/modules-available/mod-http-image-filter.conf
lrwxrwxrwx 1 root root   60 Jul 31 05:31 50-mod-http-xslt-filter.conf -> /usr/share/nginx/modules-available/mod-http-xslt-filter.conf
lrwxrwxrwx 1 root root   48 Jul 31 05:31 50-mod-mail.conf -> /usr/share/nginx/modules-available/mod-mail.conf
lrwxrwxrwx 1 root root   59 Jul 31 05:37 50-mod-ngx_http_geoip2.conf -> /usr/share/nginx/modules-available/mod-ngx_http_geoip2.conf
lrwxrwxrwx 1 root root   50 Jul 31 05:31 50-mod-stream.conf -> /usr/share/nginx/modules-available/mod-stream.conf






  ##

    geoip2 /var/lib/GeoIP/GeoLite2-Country.mmdb {
        auto_reload 5m;
        $geoip2_metadata_country_build metadata build_epoch;
        # $geoip2_data_country_code default=US source=$variable_with_ip country iso_code;
        $geoip2_data_country_code default=US country iso_code;
        $geoip2_data_country_name country names en;
        }
