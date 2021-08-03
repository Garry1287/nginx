NGX-12: Cross-Origin Resource Sharing
Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Настройте виртуальный хост api.${base_domain}, который будет реализовывать следующую логику:
        Добавлять header Access-Control-Allow-Origin только если запрос пришел от домена https://example.org/ и метод запроса был GET.
        Значение header Access-Control-Allow-Origin должно быть https://example.org/
        Остальные хедеры, относящиеся к CORS, добавлять не надо.
        Данную задачку нужно решить БЕЗ использования директивы map.
        На все запросы virtual host должен возвращать 200
    Проверьте что все условия выполнены и отправляйте задание на проверку

Server IP: 157.230.99.147, root password: iiswonmakadd, base_domain: 92ff032b2a542d5a056cf841998107de.kis.im


garry@home-pc:~/devops_learn/nginx-rebrain/ngx-ansible$ ansible-playbook -i "157.230.99.147," -u root -k  -e 'ansible_python_interpreter=/usr/bin/python3' nginx.yml



https://blog.amet13.name/2016/03/if-nginx.html
https://stackoverflow.com/questions/54313216/nginx-config-to-enable-cors-with-origin-matching
https://gist.github.com/Stanback/7145487


map $http_origin $cors_origin_header {
    default "";
    "https://example.org" "$http_origin";
}

server {
    listen 80;

	server_name  api.domain.com;

    add_header Access-Control-Allow-Origin $cors_origin_header always;
    add_header Access-Control-Allow-Methods "GET, POST, OPTIONS, HEAD";
    add_header Access-Control-Allow-Headers "Origin, Content-Type, Accept";

    location / {
      if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
      }
    }
}





Server IP: 167.99.129.149, root password: vuohwmimemza, base_domain: 92ff032b2a542d5a056cf841998107de.kis.im

server {
  listen 80 default_server;
  server_name  api.92ff032b2a542d5a056cf841998107de.kis.im;

    location / {
          set $cors '';
          if ($request_method = GET ) {
              set $cors 'true';
          }

          if ($http_origin = 'https://example.org/') {
              set $cors '${cors}true';
          }

          if ($cors = 'truetrue') {
                add_header 'Access-Control-Allow-Origin' "https://example.org/" always;
      #    return 200;
         }  

         return 200; 
    }
}





root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s -H "Origin: https://example.org" -X POST  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:10:54 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s -H "Origin: https://example.org" -X POST  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:10:57 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s -H "Origin: https://example.org" -X POST  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:10:58 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s -H "Origin: https://example.org" -X POST  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:10:59 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s -H "Origin: https://example.org" -X GET  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:11:06 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: https://example.org

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s  -X GET  api.92ff032b2a542d5a056cf841998107de.kis.im
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:11:12 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s  -X GET  api.92ff032b2a542d5a056cf841998107de.kis.im/dfds
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:11:17 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive

root@92ff032b2a542d5a056cf841998107de:~# curl -D - -s   api.92ff032b2a542d5a056cf841998107de.kis.im/dfds
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Mon, 02 Aug 2021 14:11:23 GMT
Content-Type: application/octet-stream
Content-Length: 0
Connection: keep-alive


