# Настройка Дентал-Софт для работы с MySQL в Linux
(Feb 13, 2016)

В этой заметке рассмотрено, как настроить СУБД MySQL в Linux-системах для клиентского ПО Дентал-Софт под Microsoft Windows. Процесс установки [Дентал-Софт](http://dental-soft.ru/soft/dental-soft/) и [MySQL Connector/ODBC для Windows](https://dev.mysql.com/downloads/connector/odbc/) тривиален и расписывать его я не буду. Рассмотрю лишь саму настройку MySQL под Linux. Все нижеизложенное справедливо для:

- [Дентал-Софт 1.7.14](http://дентал-софт.xn--p1ai/installers/dental-soft_v1.7.14.zip)
- MySQL 5.5.47
- [Debian Wheezy 7.9](https://www.debian.org/News/2015/2015090502)

но, с большей вероятностью, процесс настройки на более поздних версиях мало чем отличается.

## Устанавовка и настройка MySQL.
```
# apt-get install mysql-server mysql-client
```
Далее вас спросят задать пароль root. По завершению установки открываем и редактируем конфигурационный файл:
```
# nano /etc/mysql/my.cnf
```
и закоментируем строчку, как в примере ниже, для того, чтобы к БД можно было подключаться с любого IP адреса:
```
#bind-address = 0.0.0.0
```
Дентал-Софт работает в кодировках UTF8 и CP1251. Установим кодировку UTF8 для MySQL принудительно. Находим в том же my.cnf раздел [mysqld]:
```
[mysqld]
# UTF8 setings
character-set-server = utf8
collation-server = utf8_unicode_ci
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
skip-character-set-client-handshake
Так же нужно переключить север MySQL на использование нижнего регистра в именах таблиц:
lower_case_table_names = 1
```
Сохраняем файл конфигурации (в nano нажимаем cntrl+O) и для обновления параметров перезапускаем MySQL:
```
# /etc/init.d/mysql restart
```
После чего логинимся в консоль mysql:
```
# mysql -u root -p
```
Вводим пароль root и проверяем заданные нами настройки кодировок - вводим в консоли MySQL:
```
show variables like "%character%";show variables like "%collation%";
```
Результат должен быть:
```
mysql> show variables like "%character%";show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_unicode_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)
```

## Создаем базу данных и пользователей.
Далее, не выходя из консоли MySQL создаем нового пользователя:
```
mysql> CREATE USER 'DentalSoft'@'localhost' IDENTIFIED BY 'some_password';
```
*где:* some_password - пароль пользователя DentalSoft.
Разрешаем пользователю DentalSoft доступ с любого IP адреса и назначем права DBA:
```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'DentalSoft'@'%' IDENTIFIED BY 'some_password';
```
Проверяем права пользователя DentalSoft:
```
mysql> SHOW GRANTS FOR 'DentalSoft'@'%';
```
Результатом выполнения должно быть:
```
Grants for DentalSoft@%
+------------------------------------------------------------------+
GRANT ALL PRIVILEGES ON *.* TO 'DentalSoft'@'%' IDENTIFIED BY PASSWORD '*0DC3BCF65D8832D42E3856FEF31D25647E8FA3DD'
Не выходя из консоли MySQL, cоздаем пустую БД и смотрим её параметры:
mysql> CREATE DATABASE dental_soft;
mysql> SHOW CREATE DATABASE dental_soft;
```
Результатом должно быть:
```
Database    | Create Database
--------------------------------------------------------------------
dental_soft | CREATE DATABASE `dental_soft` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci */
```

## Настраиваем Дентал-Софт.

Открываем на клиентской windows-машине в проводнике:
```
C:\Program Files (x86)\Дентал-Софт\Дентал-Софт\_SYS\SQL
```
и в зависимости от версии установленного драйвера MySQL Connector/ODBC находим файл:
```
init_sql_1.sql - для версии ODBC 3.5;
init_sql_4.sql - для версии ODBC 5.1;
init_sql_9.sql - для версии ODBC 5.2;
init_sql_10.sql - для версии ODBC 5.3.
```
открываем его в блокноте и вставляем содержимое с заменой:
```
SET CHARACTER SET utf8;
set character_set_client='utf8';
set character_set_results='utf8';
set collation_connection='utf8_unicode_ci';
```
Запускаем Дентал-Софт и создаем подключение:

- IP-адрес вашего Debian сервера с MySQL
- Тип базы данных: MySQL ODBC 5.1 (3.5, 5.2 или 5.3 - в зависимости от версии установленного драйвера ODBC);
- Имя базы данных: dental_soft;
- Имя пользователя, пароль и наименование подключаемой базы.

При запуске вас предупредит о том, что базы не существует и нужно её создать (схему мы создали, но данных там еще нет). Соглашаемся, ждем завершения. Настройка завершена.

**ВНИМАНИЕ!** Дентал-Софт до версии 1.7.20 некорректно работает с кодировками 'utf8_unicode_ci' и 'utf8_general_ci'. Если на одном из клиентов произошла конверсия базы и при каждом посоледующем запуске предлагает сконвертировать, то нужно выполнить:

Для смены кодировки базы на сервере:
```
mysql> ALTER DATABASE dental_soft CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```
и в конфигурации соединения для ODBC-коннектора указать:
```
SET CHARACTER SET utf8;
set character_set_client='utf8';
set character_set_results='utf8';
set collation_connection='utf8_general_ci';
```
Работать будет, но каждый раз будет выводит сообщение, что:

> Переменные collation_server и collation_connection не равны друг-другу.

Обратите так же внимание, что язык интерфейса Windows и параметр "использовать язык для программ не поддерживающих Unicode" для Дентал-Софт должен быть обязательно *русский*, а для активации по программным ключам нужно войти *под локальным* (не доменным) администратором.
