 NGX-07: TLS Authentication

    Внимание: В звязи с большим колличеством запросов на получение TLS сертификатов, вы можете столкнуться с ошибкой о превышении лимита сертификатов на домен kis.im. Поэтому в рамках данного задания допустипо получить сертификат не с основного, а со staging сервера letsencrypt.

    Установите nginx из системных (стандартных) репозиториев ubuntu.
    Сгенерируйте корневой CA сертификат и ключ, а также клиентский сертификат и разместите их в папке /opt/ssl со следующими именами (ключи должны быть формата RSA):
        ca.crt - корневой CA сертификат;
        ca.key - ключ корневого сертификата;
        client.crt - клиентский сертификат;
        client.key - клиентский ключ.
    Создайте virtual host для host tls.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
        tls.{base_domain} должен быть доступен по HTTPS с валидным сертификатом, выданным letsencrypt.
        На все запросы должна происходить проверка клиентского сертификата, и в случае успеха nginx должен возвращать стандартый index.html, входящий в поставку.
    Проверьте, что все условия выполнены и отправляйте задание на проверку.




    В /etc/nginx/ssl

Генерируем ключ
openssl genrsa -out ca.key 2048

Создаём корневой сертификат подписанный ключом
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt


Генерируем ключ для клиента
openssl genrsa -out client.key 2048

Генерируем запрос на клиентский сертификат
openssl req -new -key client.key -out client.csr


Подписываем клиентский запрос на сертификат с помощью корневого сертификата
openssl x509 -req -days 3650 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt