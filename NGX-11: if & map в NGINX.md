NGX-11: if & map в NGINX 

Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Для выполнения задания используйте динамический модуль geoip первой версии (как было в 8 задании).
    Используя if & map, настройте виртуальный хост access.${base_domain} следующим образом:
        Создайте map - $blocked, который зависит от страны пользователя (двухбуквенный код страны) следующим образом:
        Все запросы из стран ru / ua / us должны быть разрешены (возвращать 200)
        Все запросы из остальных стран должны быть запрещены (возвращать 403)
        При проверке условия в if запрещается использовать сравнение (знак =)
        Ваш сервер должен обслуживать все запросы на 80 порту, как сервер по умолчанию
        Не забудьте выключить default сервер и удалить его конфигурационный файл
    Проверьте что все условия выполнены и отправляйте задание на проверку.


Server IP: 165.22.27.247, root password: eekouygnrczh, base_domain: 4e513a2cf8b3ced7cfb19f54929f11bc.kis.im

  garry@home-pc:~/devops_learn/nginx-rebrain/ngx-ansible$ ansible-playbook -i "157.230.114.8," -u root -k  -e 'ansible_python_interpreter=/usr/bin/python3' nginx.yml







map $geoip_country_code $blocked {
  default 1;
  ua 0;
  us 0;
  ru 0;
}

server {
  listen 80 default_server;
  server_name access.4e513a2cf8b3ced7cfb19f54929f11bc.kis.im; 
  
  if ($blocked) {
     return 403;
  }

  location / {
    add_header X-Country $geoip_country_code;
    return 200;
  }
}


https://techexpert.tips/nginx/nginx-blocking-access-from-country/