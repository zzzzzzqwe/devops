# Лабораторная работа №1. Виртуальный сервер

## Студент
**Gachayev Dmitrii I2302**  
**Выполнено 16.02.2025_**  

## Описание задачи
В этой лабораторной работе рассматривается процесс виртуализации операционных систем и настройка виртуального HTTP сервера. В рамках выполнения работы будет развернута виртуальная машина с Debian с использованием гипервизора QEMU, установлен LAMP, а также настроены PhpMyAdmin и Drupal.
## Выполнение работы
## Установка доп. по: 
1. Скачиваю дистрибутив Debian для серверов для архитектуры x64 (без графического интерфейса) и систему виртуализации (гипервизор) QEMU.
## Установка ОС Debian на виртуальную машину
1. В консоли выполняю следующую команду, тем самым создаю образ диска для виртуальной машины размером 8 ГБ, формата qcow2:
   ```sh
   qemu-img create -f qcow2 debian.qcow2 8G
   ```
   ![image](screenshots/Screenshot_1.png)
2. Запускаю установку OC Debian командой:
   ```sh
   qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G
   ```
   ![image](screenshots/Screenshot_3.png)
3. Установку Debian произвожу со следующими параметрами: 
   - Имя компьютера: **debian**
   - Хостовое имя: **debian.localhost**
   - Имя пользователя: **user**
   - Пароль пользователя: **password**
     ![image](screenshots/Screenshot_2.png)
4. Запуск виртуальной машины
   После установки с помощью следующей команды система загружается с виртуального жесткого диска debian.qcow2, используя 2 ГБ оперативной памяти и 2 ядра процессора. Также создаётся NAT-сеть и  настраивается проброс портов, что позволяет подключаться к веб-серверу, работающему на виртуальной машине.
   ```
   qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 \ -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22
      ```
