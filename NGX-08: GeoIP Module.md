NGX-08: GeoIP Module
Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Установите модуль geoip из системных репозиториев, а так же geoip-database.
    Создайте virtual host для host: country.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику: 1. При запросе location /country, в ответ должен добавляться header X-Country: и значение - трехбуквенный код страны, код ответа - 200 2. При запросе location /name, в ответ должен добавляться header X-Country-Name: и значение - название страны, код ответа - 200
    Проверьте что все условия выполнены и отправляйте задание на проверку




    Server IP: 165.227.137.184, root password: ifqnyxmaybql, base_domain: 66c6d472a13c2254c3fa3e76a898dffd.kis.im

    garry@home-pc:~/devops_learn/nginx-rebrain/ngx-ansible$ ansible-playbook -i "165.227.137.184," -u root -k  -e 'ansible_python_interpreter=/usr/bin/python3' nginx.yml






apt install geoip-database
Reading package lists... Done
Building dependency tree       
Reading state information... Done
geoip-database is already the newest version (20180315-1).
geoip-database set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 33 not upgraded.

root@66c6d472a13c2254c3fa3e76a898dffd:~# apt install libnginx-mod-http-geoip






    	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
#	include /etc/nginx/sites-enabled/*;
}




server {
  listen 80 default_server;
  server_name country.66c6d472a13c2254c3fa3e76a898dffd.kis.im; 

  location /country {
    add_header X-Country $geoip_country_code3;
    return 200;
  }

  location /name {
    add_header X-Country-Name $geoip_country_name;
    return 200;
  }


}




root@66c6d472a13c2254c3fa3e76a898dffd:~# curl -D - -s country.66c6d472a13c2254c3fa3e76a898dffd.kis.im/name
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 30 Jul 2021 03:13:22 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive
X-Country-Name: Germany

root@66c6d472a13c2254c3fa3e76a898dffd:~# curl -D - -s country.66c6d472a13c2254c3fa3e76a898dffd.kis.im/country
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 30 Jul 2021 03:13:34 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive
X-Country: DEU

root@66c6d472a13c2254c3fa3e76a898dffd:~# curl -D - -s country.66c6d472a13c2254c3fa3e76a898dffd.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 30 Jul 2021 03:13:38 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 17 Apr 2018 15:22:36 GMT
Connection: keep-alive
ETag: "5ad6113c-264"
Accept-Ranges: bytes

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>