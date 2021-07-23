NGX-06: Basic Authentication 
    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Создайте virtual host для host: auth.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
        Все запросы на / должны быть закрыты аутентификацией с логином / паролем - установите логин user, пароль test
        Все запросы на /auth/off (и вложенные директории) должны проходить без аутентификации
        Все запросы после успешной аутентификации или при отключенной аутентификации должны возвращать код 200
    Проверьте что все условия выполнены и отправляйте задание на проверку










 server {
  listen 80 default_server;
  server_name auth.dfbba1d16806c400c7f9b30547c24e91.kis.im;

  location / {
    auth_basic "No_TEST";
    auth_basic_user_file /etc/nginx/.htpasswd;
    alias /tmp/;  # return 200;
  }

  location /auth/off {
    auth_basic off;
    return 200;
  }
}


root@dfbba1d16806c400c7f9b30547c24e91:~# htpasswd -n user
New password: 
Re-type new password: 
user:$apr1$ZojSsEXv$7yRLQUbbVUd0xvg7MfgAy1
