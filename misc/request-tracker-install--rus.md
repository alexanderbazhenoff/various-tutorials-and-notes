# Установка Request Tracker

[Request Tracker](https://bestpractical.com/)  -- система учёта и отслеживания заявок уровня предприятия (enterprise) с открытыми исходниками от Best Practical Solutions. RT написана на Perl и работает на Apache, Nginx и lighttpd веб-серверах с использованием mod_perl или FastCGI и возможностью хранения данных в MySQL, PostgreSQL, Oracle или SQLite. Существует возможность расширения интерфейса с использованием [плагинов](https://medium.com/r/?url=https%3A%2F%2Fmetacpan.org%2Fsearch%3Fp%3D3%26q%3DRT%253A%253AExtension%26search_type%3Dmodules) на Perl.

Ниже будет рассмотрена установка RT из исходников с использованием Apache и MySQL. Установку будем производить на Debian Wheezy (7.11). Данной инструкцией можно так же воспользоваться для установки RT в Ubuntu и других версиях Debian, однако при компиляции исходников RT в Debian Wheezy могут возникнуть [некоторые сложности](https://medium.com/r/?url=https%3A%2F%2Fforum.bestpractical.com%2Ft%2Frt-build-from-sources-errors-at-debian-jessie%2F32120). Мануал по установке RT для Centos 7 можно найти в официальном [RT wiki](https://medium.com/r/?url=https%3A%2F%2Frt-wiki.bestpractical.com%2Fwiki%2FCentOS7Install).

Предполагается, что уже произведена установка netinstall дистрибутива в любой удобной для вас конфигурации. LAMP на этапе установки системы выбирать не нужно, достаточно выбрать стандартные утилиты и SSH-сервер. После установки системы отредактируем hostname и resolv.conf. Например:
```
# hostnamectl set-hostname rt.example.com
# nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       rt.example.com   rt
```
Желательно так же установить ntp-клиент:
```
# apt-get install ntp
```
Все его настройки можно оставить дефолтными, но если потребуется синхронизация с другим сервером времени (например, контроллером домена, где уже установлен сервер NTP):
```
# nano /etc/ntp.conf
server 192.168.0.202 iburst

# service ntp restart
```
Проверить его настройки можно командой:
```
# ntpq -pn
```
Устанавливаем MySQL:
```
# apt-get install mysql-server mysql-client libmysqlclient-dev
```
Задаем пароль root и осуществим небольшой тюнинг MySQL путем задания innodb_buffer_pool_size в пределах от 128 до 256 Мб (здесь все зависит от нагрузки на БД, можно выставить примерно равным объему физической памяти):
```
# nano /etc/mysql/my.cnf
# * Fine Tuning
#
key_buffer              = 16M
max_allowed_packet      = 16M
thread_stack            = 192K
thread_cache_size       = 8
innodb_buffer_pool_size = 256M
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
```
Запускаем скрипт, отвечаем на вопросы (где задаем пароль root, разрешаем по необходимости логин для root, удаляем anonimous user и тестовую базу) и перезапускаем службу:
```
# mysql_secure_installation
# service mysql restart
```
Скачиваем и устанавливаем Apache, а так же другие зависимости, необходимые для RT (для Ubuntu `libgd2-xpm-dev` следует заменить на `libgd-dev` и `libgd-perl` - на `libgd-gd2-perl`)::
```
# apt-get install make apache2 libapache2-mod-fcgid libssl-dev libyaml-perl libgd2-xpm-dev libgd-gd2-perl libgraphviz-perl
```
Создаем системного пользователя и группу rt, добавляем пользователя *www-data* в группу *rt*, скачиваем и распаковываем исходники RT:
```
# adduser --system --group rt
# usermod -aG rt www-data
# mkdir -p /root/src
# wget http://download.bestpractical.com/pub/rt/release/rt.tar.gz
# tar xf rt.tar.gz -C /root/src
# cd /root/src/rt-*
```
Запускаем конфигурационный скрипт, указывающий некоторые параметры (например, если не указывать дополнительно, то путь для установки по умолчанию будет: `/opt/rt4`):
```
# ./configure --with-web-user=www-data --with-web-group=www-data --enable-graphviz --enable-gd
```
И прежде, чем запустить проверку зависимостей, заходим в CPAN:
```
# cpan
CPAN.pm requires configuration, but most of it can be done automatically.
If you answer 'no' below, you will enter an interactive dialog for each
configuration option instead.
Would you like to configure as much as possible automatically? [yes] <enter>
...
Would you like me to automatically choose some CPAN mirror
sites for you? (This means connecting to the Internet) [yes] <enter>
...
cpan[1]> o conf prerequisites_policy follow
cpan[2]> o conf build_requires_install_policy yes
cpan[3]> o conf commit
cpan[4]> q
```
Теперь можно запустить проверку зависимостей, до тех пор пока не получим сообщение *"All dependencies have been found"*, а так же их исправление командой make `fixdeps`. Может потребоваться несколько циклов до тех пор пока все пакеты и зависимости не будут установлены:
```
# make testdeps
# make fixdeps
```
Проверяем:
```
# make testdeps /usr/bin/perl ./sbin/rt-test-dependencies --with-mysql --with-fastcgi

perl:
        >=5.10.1(5.14.2) ...found
users:
        rt group (rt) ...found
        bin owner (root) ...found
        libs owner (root) ...found
        libs group (bin) ...found
        web owner (www-data) ...found
        web group (www-data) ...found
CLI dependencies:
        Text::ParseWords ...found
        Term::ReadKey ...found
        Getopt::Long >= 2.24 ...found
        HTTP::Request::Common ...found
        Term::ReadLine ...found
        LWP ...found
CORE dependencies:
        Storable >= 2.08 ...found
        Net::IP ...found
        URI::QueryParam ...found
        Business::Hours ...found
        Encode >= 2.64 ...found
        Crypt::Eksblowfish ...found
        Module::Versions::Report >= 1.05 ...found
        List::MoreUtils ...found
        Errno ...found
        DBI >= 1.37 ...found
        Devel::StackTrace >= 1.19 ...found
        HTTP::Message >= 6.0 ...found
        Text::Password::Pronounceable ...found
        Devel::GlobalDestruction ...found
        Time::ParseDate ...found
        IPC::Run3 ...found
        Tree::Simple >= 1.04 ...found
        HTML::Scrubber >= 0.08 ...found
        HTML::Quoted ...found
        Data::Page::Pageset ...found
        Sys::Syslog >= 0.16 ...found
        Mail::Mailer >= 1.57 ...found
        Data::GUID ...found
        HTML::Mason >= 1.43 ...found
        HTML::Entities ...found
        LWP::Simple ...found
        Symbol::Global::Name >= 0.04 ...found
        URI >= 1.59 ...found
        DateTime::Format::Natural >= 0.67 ...found
        Plack >= 1.0002 ...found
        File::Glob ...found
        Text::Wrapper ...found
        Regexp::Common::net::CIDR ...found
        Log::Dispatch >= 2.30 ...found
        HTML::FormatText::WithLinks::AndTables >= 0.06 ...found
        DateTime >= 0.44 ...found
        CGI::Emulate::PSGI ...found
        Text::Quoted >= 2.07 ...found
        Regexp::IPv6 ...found
        CGI >= 3.38 ...found
        Class::Accessor::Fast ...found
        CSS::Squish >= 0.06 ...found
        DateTime::Locale >= 0.40 ...found
        CGI::PSGI >= 0.12 ...found
        Apache::Session >= 1.53 ...found
        Date::Extract >= 0.02 ...found
        Digest::SHA ...found
        HTML::Mason::PSGIHandler >= 0.52 ...found
        MIME::Entity >= 5.504 ...found
        Locale::Maketext::Lexicon >= 0.32 ...found
        Role::Basic >= 0.12 ...found
        Module::Refresh >= 0.03 ...found
        Digest::base ...found
        File::Temp >= 0.19 ...found
        Date::Manip ...found
        Locale::Maketext >= 1.06 ...found
        HTML::RewriteAttributes >= 0.05 ...found
        Text::Template >= 1.44 ...found
        Scalar::Util ...found
        CGI::Cookie >= 1.20 ...found
        XML::RSS >= 1.05 ...found
        Text::WikiFormat >= 0.76 ...found
        File::Spec >= 0.8 ...found
        DBIx::SearchBuilder >= 1.65 ...found
        File::ShareDir ...found
        Regexp::Common ...found
        Digest::MD5 >= 2.27 ...found
        CSS::Minifier::XS ...found
        Data::ICal ...found
        Pod::Select ...found
        HTML::FormatText::WithLinks >= 0.14 ...found
        Scope::Upper ...found
        Mail::Header >= 2.12 ...found
        Locale::Maketext::Fuzzy >= 0.11 ...found
        Time::HiRes ...found
        MIME::Types ...found
        Email::Address::List >= 0.02 ...found
        Convert::Color ...found
        JavaScript::Minifier::XS ...found
        Net::CIDR ...found
        JSON ...found
        UNIVERSAL::require ...found
        Email::Address >= 1.897 ...found
        Plack::Handler::Starlet ...found
FASTCGI dependencies:
        FCGI >= 0.74 ...found
GD dependencies:
        GD::Text ...found
        GD ...found
        GD::Graph >= 1.47 ...found
GPG dependencies:
        File::Which ...found
        PerlIO::eol ...found
        GnuPG::Interface ...found
GRAPHVIZ dependencies:
        IPC::Run >= 0.90 ...found
        GraphViz ...found
MAILGATE dependencies:
        Pod::Usage ...found
        LWP::UserAgent >= 6.0 ...found
        Crypt::SSLeay ...found
        Getopt::Long ...found
        Net::SSL ...found
        LWP::Protocol::https ...found
        Mozilla::CA ...found
MYSQL dependencies:
        DBD::mysql >= 2.1018 ...found
SMIME dependencies:
        String::ShellQuote ...found
        File::Which ...found
        Crypt::X509 ...found
All dependencies have been found.
```
Устанавливаем:
```
# make install

...
Congratulations. RT is now installed.


You must now configure RT by editing /opt/rt4/etc/RT_SiteConfig.pm.
(You will definitely need to set RT's database password in 
/opt/rt4/etc/RT_SiteConfig.pm before continuing. Not doing so could be 
very dangerous. Note that you do not have to manually add a 
database user or set up a database for RT. These actions will be 
taken care of in the next step.)
After that, you need to initialize RT's database by running
 'make initialize-database'
```
Создаем БД для RT с помощью скрипта (вводим пароль для root):
```
# make initialize-database

/usr/bin/perl -I/opt/rt4/local/lib -I/opt/rt4/lib sbin/rt-setup-database --action init --prompt-for-dba-password
In order to create or update your RT database, this script needs to connect to your  mysql instance on localhost (port '') as root
Please specify that user's database password below. If the user has no database
password, just press return.
Password:
Working with:
Type:   mysql
Host:   localhost
Port:
Name:   rt4
User:   rt_user
DBA:    root
Now creating a mysql database rt4 for RT.
Done.
Now populating database schema.
Done.
Now inserting database ACLs.
Done.
Now inserting RT core system objects.
Done.
[14364] [Mon Jul 17 11:04:58 2017] [warning]: Subroutine quant redefined at /root/src/rt-4.4.1/sbin/../lib/RT/I18N/ru.pm line 58. (/root/src/rt-4.4.1/sbin/../lib/RT/I18N/ru.pm:58)
[14364] [Mon Jul 17 11:04:58 2017] [warning]: Subroutine numerate redefined at /root/src/rt-4.4.1/sbin/../lib/RT/I18N/ru.pm line 67. (/root/src/rt-4.4.1/sbin/../lib/RT/I18N/ru.pm:67)
Now inserting data.
Done inserting data.
Done.
```
Далее нужно перенаправить с HTTP на HTTPS все входящие HTTP запросы. Дефолтная конфигурация для Debian 7.x находится в файле `default` (для Ubuntu и Debian 8.x - `000-default`, или `000-default.conf`). Нужно исправить конфиг следующим образом:
```
# nano /etc/apache2/sites-available/default

<VirtualHost *:80>

        ServerName rt.example.com:80
        Redirect / https://rt.example.com/
        
        #ServerAdmin webmaster@localhost
        #DocumentRoot /var/www
```
Создаем конфигурационный файл для сайта rt (для Debian 8.x, или Ubuntu - `default-ssl.conf`):
```
# cp /etc/apache2/sites-available/default-ssl /etc/apache2/sites-available/rt
```
Редактируем конфигурацию (для Debian 8.x, или Ubunut, опять же, файл назвается `rt.conf`):
```
# nano /etc/apache2/sites-available/rt

<IfModule mod_ssl.c>
        <VirtualHost _default_:443>    
            # Request Tracker
            ServerName rt.example.com:443
            AddDefaultCharset UTF-8
            DocumentRoot /opt/rt4/share/html
            Alias /NoAuth/images/ /opt/rt4/share/html/NoAuth/images/
            ScriptAlias / /opt/rt4/sbin/rt-server.fcgi/
            <Location />
                    ## раскомментируйте в зависимости от версии:
                    ## для Apache версии < 2.4 (Debian 7.x)
                    #Order allow,deny
                    #Allow from all
                    ## Для Apache 2.4
                    #Require all granted
            </Location>
            <Directory "/opt/rt4/sbin">
                    SSLOptions +StdEnvVars
            </Directory>
```
Активируем модули ssl и fcgid (fcgid может быть уже активирован):
```
# a2enmod ssl fcgid

Enabling module ssl.
See /usr/share/doc/apache2.2-common/README.Debian.gz on how to configure SSL and create self-signed certificates.
Module fcgid already enabled
To activate the new configuration, you need to run:
  service apache2 restart
```
Активируем сайт rt:
```
# a2ensite rt

Enabling site rt.
To activate the new configuration, you need to run:
  service apache2 reload
```
Проверяем конфигурацию Apache:
```
# apachectl configtest

Syntax OK
```
RT очень гибок по своим [настройкам](https://medium.com/r/?url=https%3A%2F%2Fdocs.bestpractical.com%2Frt%2F4.4.2%2FRT_Config.html). Но на начальном этапе достаточно добавить следующие строки:
```
# nano /opt/rt4/etc/RT_SiteConfig.pm

Set( $rtname, 'rt.example.com');
Set( $Organization, 'Example Company');
Set( $Timezone, 'Europe/Moscow');
Set( $WebDomain, 'rt.example.com');
Set( $WebPort, 443);
Set( $WebPath, '');
```
Перезапускаем Apache:
```
# service apache2 restart
```
и логинимся в веб-интерфейсе - открываем в барузере ссылку: `http://rt.corp.example.com` (или `https://rt.example.com`) и логинимся:
```
root
password
```
Финальным штрихом в настройке будет задание пароля в MySQL, для пользователя *rt_user*:
```
# mysql -u root -p

Enter password: <пароль_MySQL_root>

mysql> SET PASSWORD FOR 'rt_user'@'localhost' = PASSWORD('пароль');
mysql> \q

# service mysql restart
```
и укажем его в конфиге RT:
```
# nano /opt/rt4/etc/RT_SiteConfig.pm

...
Set( $WebDomain, 'rt.example.com');
Set( $WebPort, 443);
Set( $WebPath, '');
Set( $DatabasePassword, 'пароль');
```
