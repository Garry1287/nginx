NGX-09: Reverse Proxy

Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Создайте два виртуальных хоста:
        Один слушает на порту 81, другой на 82 (должны быть доступны извне)
        При запросе location / они должны отдавать код 200 и текст "Backend 1" или "Backend 2"
    Создайте третий virtual host reverse.${base_domain} на порту 80:
        При запросе /backend, nginx должен проксировать запрос на бекенды на порту 81 и 82
        При запросе /yandex, nginx должен проксировать запрос на https://yandex.ru/
    Проверьте что все условия выполнены и отправляйте задание на проверку


Server IP: 161.35.209.115, root password: rkwayddolksl, base_domain: f357ef338d8f375aa5cb705746504751.kis.im

    garry@home-pc:~/devops_learn/nginx-rebrain/ngx-ansible$ ansible-playbook -i "161.35.209.115," -u root -k  -e 'ansible_python_interpreter=/usr/bin/python3' nginx.yml


upstream backend {
  server 127.0.0.1:81;
  server 127.0.0.1.82;

}


server {
  listen 80 default_server;
  server_name reverse.f357ef338d8f375aa5cb705746504751.kis.im; 

  location /backend {
    proxy_pass http://backend;
  }

  location /yandex {
    proxy_pass https://yandex.ru;
  }

}


server {
  listen 81;

  location / {
    return 200 "Backend 1\n";
  }

}

server {
  listen 82;

  location / {
    return 200 "Backend 2\n";
  }

}