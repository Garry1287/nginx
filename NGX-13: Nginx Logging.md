NGX-13: Nginx Logging 

Задание:

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Создайте виртуалхост log.${base_domain} , который слушает 80 порт со следующими настройками:
        Файл содержащий error_log должен отсутствовать (чтобы логи нельзя было прочитать)
        Создайте log_format - themain, который будет использовать формат: <remote ip address> <response code> <request_time> <upstream response time> (формат должен быть определен одной строкой - без переносов)
        access_log должен писаться в /var/log/nginx/log.access.log в формате themain
        location /yandex должен проксировать запросы на https://yandex.ru/
    Установите из deb пакета и настройте nginxlog exporter следующим образом:
        Метрики должны быть доступны извне на порту 8080
        Конфигурационный файл должен располагаться в /etc/prometheus-nginxlog-exporter.hcl
        nginxlog-exporter должен парсить файл access_log, который был создан для виртуального хост (строка парсинга должна быть записана в одну строку)
        nginxlog-exporter должен быть запущен как сервис и отображаться в статусе running systemd
    Проверьте что все условия выполнены и отправляйте задание на проверку

Server IP: 165.22.92.193, root password: bgmwldxwthut, base_domain: 2afcbed83359957f09df1b709f074de0.kis.im

garry@home-pc:~/devops_learn/nginx-rebrain/ngx-ansible$ ansible-playbook -i "165.22.92.193," -u root -k  -e 'ansible_python_interpreter=/usr/bin/python3' nginx.yml
 Format:<$remote_addr> <$status> <$request_time> <$upstream_response_time>

http {

  log_format themain '<$remote_addr> <$status> <$request_time> <$upstream_response_time>';
  error_log /dev/null crit;
}




server {
  listen 80;
  server_name log.2afcbed83359957f09df1b709f074de0.kis.im;

  access_log /var/log/nginx/log.access.log themain;

  location / {
    return 200 "Hello\n";
  }

   location /yandex {
   proxy_pass https://yandex.ru/;
  }

}





$ wget https://github.com/martin-helmich/prometheus-nginxlog-exporter/releases/download/v1.8.0/prometheus-nginxlog-exporter_1.8.0_linux_amd64.deb
$ apt install ./prometheus-nginxlog-exporter_1.8.0_linux_amd64.deb


cat /etc/prometheus-nginxlog-exporter.hcl
listen {
  port = 8080
}

namespace "nginx" {
source_files = ["/var/log/nginx/log.access.log"]


  source = {
    files = [
      "/var/log/nginx/log.access.log"
    ]
  }

  format = "$remote_addr $status $request_time $upstream_response_time"
#  format = "$remote_addr - $remote_user [$time_local] \"$request\" $status $body_bytes_sent \"$http_referer\" \"$http_user_agent\" \"$http_x_forwarded_for\""

  labels {
    app = "default"
  }
}



 curl -D - -s log.2afcbed83359957f09df1b709f074de0.kis.im/yandex
 curl -D - -s log.2afcbed83359957f09df1b709f074de0.kis.im/

root@2afcbed83359957f09df1b709f074de0:~# tail -f /var/log/nginx/log.access.log 
167.71.58.123 "200" 0.000 -
167.71.58.123 "200" 0.514 0.516
167.71.58.123 "200" 0.589 0.592
167.71.58.123 "200" 0.000 -
167.71.58.123 "200" 0.000 -
167.71.58.123 "200" 0.592 0.592
192.241.215.78 "200" 0.000 -


