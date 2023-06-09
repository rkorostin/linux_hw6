# Операционные системы и виртуализация (Linux)
## Урок 6. Запуск стека для веб-приложения
### 1. Установить Nginx и настроить его на работу с PHP-FPM.
 - sudo apt update
 - sudo apt install nginx
 - sudo apt install php8.1-fpm

В файле default настраиваем nginx на работу с PHP-FPM
 - sudo vim /etc/nginx/sites-available/default

Добавляем в location регулярное выражение, которое "ловит" php
```
location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        root /var/www/html;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        }
```
 - sudo touch /var/www/html/info.php
 - sudo vim /var/www/html/info.php
 ```
<?php
phpinfo();
?>
 ```
 - sudo systemctl reload nginx

Проверяем. Сначала заходим на web (у меня мост на VM)

![localhost](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_1_2.png)

Затем заходим на localhost/info.php

Видим, что обработка PHP происходит в PHP-FPM

![localhost/info.php](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_1.png)


### 2. Установить Apache. Настроить обработку PHP. Добиться одновременной работы с Nginx.
 - sudo apt install apache2

Для одновременной работы с Nginx нужно настроить порты:
 - sudo vim /etc/apache2/ports.conf
```
Listen 8080

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```
 - sudo vim /etc/apache2/sites-enabled/000-default.conf
```
<VirtualHost *:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
 - sudo systemctl start apache2

Для обработки PHP в Apache:
 - sudo apt install libapache2-mod-php8.1 php8.1
- sudo systemctl reload apache2

Проверяем.

*http://localhost*

Работает Nginx

![localhost](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_2-nginx.png)

*http://localhost:8080*

Работает Apache

![localhost:8080](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_2-apache.png)

*http://localhost:8080/info.php*

PHP обрабатывается на строне Apache

![localhost:8080/info.php](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_2-php_apache.png)


### 3. Настроить схему обратного прокси для Nginx (динамика - на Apache).
Добавляем директивы в location:
 - sudo vim /etc/nginx/sites-available/default
```
#Динамические запросы
location / {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    }
```
 - sudo systemctl reload nginx

Проверяем. Запускаем http://localhost и получаем проброс на apache. Но отвечает нам nginx

![localhost](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_3.png)
### 4. Установить MySQL. Создать новую базу данных и таблицу в ней.
 - sudo apt install mysql-server-8.0
 - sudo mysql
 - mysql> CREATE DATABASE mydatabase;
```
Query OK, 1 row affected (0,01 sec)
```
 - mysql> USE mydatabase;
```
Database changed
```
 - mysql> CREATE TABLE test(i INT);
```
Query OK, 0 rows affected (0,02 sec)
```
 - mysql> INSERT INTO test (i) VALUES (1),(2),(3),(4);
```
Query OK, 4 rows affected (0,01 sec)
Records: 4  Duplicates: 0  Warnings: 0
```
 - mysql>  SELECT * FROM test;
```
+------+
| i    |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
+------+
4 rows in set (0,00 sec)
```
### 5. Установить пакет phpmyadmin и запустить его веб-интерфейс для управления MySQL.
 - sudo apt-get install phpmyadmin

Во время установки вибараем конфигурацию на apache2 и заводим пароль для учетки phpmyadmin. После установки заходим на *localhost/phpmyadmin*

![localhost/phpmyadmin](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_5.png)

![localhost/phpmyadmin](https://github.com/rkorostin/Images/blob/main/linux_hw6_task_5-1.png)
### 6. Настроить схему балансировки трафика между несколькими серверами Apache на стороне Nginx с помощью модуля ngx_http_upstream_module.
 - sudo touch /etc/nginx/conf.d/upstream.conf
Далее в созданном файле upstream.conf настраиваем балансировщик для нескольких серверов Apache:
```
upstream backend {
    server apache1.ru:80;
    server apache2.ru:80;
    server apache3.ru:80;
}
```
 - sudo vim /etc/nginx/sites-available/default

Меняем директиву proxy_pass:
```
location / {
        #       proxy_pass http://localhost:8080;
                proxy_pass http://backend;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
        }
```
 - sudo systemctl reload nginx



