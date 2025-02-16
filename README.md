# Лабораторная работа №1. Виртуальный сервер

## Студент
**Gachayev Dmitrii I2302**  
**Выполнено 16.02.2025**  

## Описание задачи
В этой лабораторной работе рассматривается процесс виртуализации операционных систем и настройка виртуального HTTP сервера. В рамках выполнения работы будет развернута виртуальная машина с Debian с использованием гипервизора QEMU, установлен LAMP, а также настроены PhpMyAdmin и Drupal.
## Выполнение работы
### Установка доп. по: 
1. Скачиваю дистрибутив Debian для серверов для архитектуры x64 (без графического интерфейса) и систему виртуализации (гипервизор) QEMU.

https://www.debian.org/distrib/

https://www.qemu.org/download/

## Установка ОС Debian на виртуальную машину
1. В консоли выполняю следующую команду, тем самым создаю образ диска для виртуальной машины размером 8 ГБ, формата qcow2:
   ```sh
   qemu-img create -f qcow2 debian.qcow2 8G
   ```
   
   ![image](screenshots/Screenshot_1.png)
   
3. Запускаю установку OC Debian командой:
   ```sh
   qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G
   ```
   
   ![image](screenshots/Screenshot_3.png)
   
4. Установку Debian произвожу со следующими параметрами: 
   - Имя компьютера: **debian**
   - Хостовое имя: **debian.localhost**
   - Имя пользователя: **user**
   - Пароль пользователя: **password**
     
     ![image](screenshots/Screenshot_2.png)
     
5. Запуск виртуальной машины

Данная команда запускает виртуальную машину с QEMU, используя образ диска debian.qcow2, выделяя 2 ГБ оперативной памяти (-m 2G) и 2 процессорных ядра (-smp 2). Сетевое устройство e1000 подключается через виртуальный сетевой адаптер (-netdev user,id=net0).
   ```
   qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 \ -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22
   ```
## Установка LAMP
1. Выполняю следующие команды:
 ```
su
apt update -y
apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip

```

**apache2** — это веб-сервер, который обрабатывает HTTP-запросы и раздаёт веб-страницы пользователям, позволяя запускать сайты и веб-приложения.

**php** — интерпретатор языка PHP, необходимый для выполнения серверных скриптов и генерации динамических веб-страниц.

**libapache2-mod-php** — модуль для Apache, который позволяет серверу обрабатывать файлы PHP, интегрируя интерпретатор PHP напрямую с веб-сервером.

**php-mysql** — расширение PHP, обеспечивающее взаимодействие PHP-скриптов с базой данных MySQL или MariaDB.

**mariadb-server** — серверная часть системы управления базами данных MariaDB, которая является форком MySQL и обеспечивает хранение, управление и обработку данных.

**mariadb-client** — клиент для взаимодействия с MariaDB, позволяющий подключаться к серверу базы данных и выполнять запросы через командную строку.
### На этом этапе я столкнулся с проблемами, связанными с отсутствием источников в файле sources.list, после их изменения все пакеты были скачаны

![image](screenshots/Screenshot_4.png)

Также в будущем оказалось что устанавливаемая версия php слишком стара для Drupal, поэтому я также установил ее отдельно по гайду из интернета


### Установка PhpMyAdmin и CMS Drupal
1. Скачал с помощью следующих команд PhpMyAdmin и CMS Drupal:
   ```sh
   wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip
   wget https://ftp.drupal.org/files/projects/drupal-11.1.1.zip
   ```

   ![image](screenshots/Screenshot_7.png)

   ![image](screenshots/Screenshot_8.png)
   
3. Проверка наличия файлов:
   ```sh
   ls -l
   ```
   ![image](screenshots/Screenshot_9.png)

## Распаковка и перемещение файлов:
   ```sh
   unzip phpMyAdmin-5.2.2-all-languages.zip
   unzip drupal-11.1.1.zip
   ```
![image](screenshots/Screenshot_10.png)

![image](screenshots/Screenshot_11.png)
   
   ```sh
   mkdir /var/www
   mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin
   mv drupal-11.1.1 /var/www/drupal
   ```

 ### Настройка базы данных

   Создаю через командную строку базу данных drupal_db и пользователя базы данных с названием ***gachayev***
   ```sh
   mysql -u root
   CREATE DATABASE drupal_db;
   CREATE USER 'gachayev'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON drupal_db.* TO 'gachayev'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```
![image](screenshots/Screenshot_16.png)

### Настройка виртуальных хостов Apache
1. Создаю файлы:
   - `nano /etc/apache2/sites-available/01-phpmyadmin.conf`

     Вписываю:
      ```
      <VirtualHost *:80>
         ServerAdmin webmaster@localhost
         DocumentRoot "/var/www/phpmyadmin"
         ServerName phpmyadmin.localhost
         ServerAlias www.phpmyadmin.localhost
         ErrorLog "/var/log/apache2/phpmyadmin.localhost-error.log"
         CustomLog "/var/log/apache2/phpmyadmin.localhost-access.log" common
      </VirtualHost>
      ```
      ![image](screenshots/Screenshot_18.png)
     
     `nano /etc/apache2/sites-available/02-drupal.conf`
     
      Вписываю:
      ```
      <VirtualHost *:80>
         ServerAdmin webmaster@localhost
         DocumentRoot "/var/www/drupal"
         ServerName drupal.localhost
         ServerAlias www.drupal.localhost
         ErrorLog "/var/log/apache2/drupal.localhost-error.log"
         CustomLog "/var/log/apache2/drupal.localhost-access.log" common
      </VirtualHost>
      
      ```
      ![image](screenshots/Screenshot_19.png)

   Далее регистрирую конфигурации и добавляю их в *hosts*:
   ```sh
   /usr/sbin/a2ensite 01-phpmyadmin
   /usr/sbin/a2ensite 02-drupal
   ```
   ![image](screenshots/Screenshot_20.png)

   ## Запуск и тестирование
   Проверяю версию системы:
   ```sh
   uname -a
   ```
   Команда uname -a выводит информацию о системе, такую как: имя ядра, имя хоста, версия ядра Линукс, архитектура процессора и тд
   
   ![image](screenshots/Screenshot_29.png)
   
   Перезапускаю Apache:
   ```sh
   systemctl restart apache2
   ```
   Проверка доступности сайтов в браузере:
   - `http://drupal.localhost:1080`
   - `http://phpmyadmin.localhost:1080`
  
   Оба сайта доступны, продолжаю настройку Drupal
   На этом этапе я столкнулся с проблемной версией php, что я упоминал выше, а так-же с отсутствием директории ***sites/default/files***, которую исправил следующим образом:
   
   ![image](screenshots/Screenshot_24.png)

   ![image](screenshots/Screenshot_22.png)

   ![image](screenshots/Screenshot_23.png)

   ![image](screenshots/Screenshot_25.png)

   ![image](screenshots/Screenshot_26.png)

   ![image](screenshots/Screenshot_27.png)

   ![image](screenshots/Screenshot_28.png)

   ## Ответы на контрольные вопросы
**Каким образом можно скачать файл в консоли при помощи утилиты wget?**
   Для скачивания файла в консоли с помощью wget используется команда:
   wget "url"

Зачем необходимо создавать для каждого сайта свою базу и своего пользователя?
   Это делается для обеспечения безопасности и изоляции данных. Если сайты используют одну базу, то взлом одного из них может привести к компрометации всех данных. 
   
Как поменять доступ к системе управления БД на порт 1234?
   Для смены порта MariaDB/MySQL на 1234 нужно:
   Открыть конфигурационный файл:
   ```
   sudo nano /etc/mysql/mariadb.conf.d
   ```
Найти строку port и заменить её

Какие преимущества, с вашей точки зрения, даёт виртуализация?

Изоляция окружений: можно запускать несколько ОС на одной машине.
   Гибкость: лёгкое создание, клонирование и восстановление систем.
   Эффективное использование ресурсов: запуск нескольких сервисов на одном сервере.
   Тестирование: безопасная среда для экспериментов без риска для основной системы.

Для чего необходимо устанавливать время / временную зону на сервере?
Корректное время необходимо для:
   Ведения логов (анализ событий и отладка).
   Корректной работы задач cron и автоматизированных процессов.
   Поддержания точности сертификатов и аутентификации (например, TLS).

Сколько места занимает установленная вами ОС (виртуальный диск) на хостовой машине?
Это можно проверить различными командами, как и в виртуальной машине, так и на хосте
В моем случае установленная ОС занимает 2,7 GB

Какие есть рекомендации по разбиению диска для серверов? Почему рекомендуется так разбивать диск?
Обычные рекомендации для серверов:
/ (root) – основной раздел (20–50 ГБ).
/home – для пользовательских данных (по необходимости).
/var – для логов, баз данных, веб-серверов (50 ГБ и больше).
/tmp – для временных файлов (1–5 ГБ).
## Вывод
В ходе лабораторной работы была установлена виртуальная машина с Debian Bookworm, используя гипервизор QEMU. Для обеспечения работы веб-приложений был развернут стек LAMP (Apache, MariaDB, PHP), а также установлены и настроены PhpMyAdmin и CMS Drupal. Благодаря данной работе я изучил основы создания виртуального сервера и принципов виртуализации.

## Библиография
1. Официальный сайт Debian. [https://www.debian.org](https://www.debian.org)
2. Официальная документация QEMU. [https://www.qemu.org/documentation](https://www.qemu.org/documentation)
3. Курс Контейнеризация и виртуализация [https://moodle.usm.md/course/view.php?id=6806](https://moodle.usm.md/course/view.php?id=6806)
