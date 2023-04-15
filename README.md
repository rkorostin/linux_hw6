# Операционные системы и виртуализация (Linux)
## Урок 6. Запуск стека для веб-приложения
### 1. Установить Nginx и настроить его на работу с PHP-FPM.
 - sudo apt update
 - sudo apt install nginx
 - sudo apt install php8.1-fpm

В файле default настраиваем nginx на работу с PHP-FPM
 - sudo vim /etc/nginx/sites-available/default
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
 - sudo touch /var/www/html/index.php
 - sudo vim /var/www/html/index.php
 ```
 <?php
phpinfo();
?>
 ```
- sudo systemctl reload apache2
### 3. Настроить схему обратного прокси для Nginx (динамика - на Apache).
 - sudo vim /etc/nginx/sites-available/default

*Динамические запросы*
```
location / {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    }
```
 - sudo systemctl reload nginx

