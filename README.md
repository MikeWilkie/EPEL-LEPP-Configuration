EPEL-LEPP-Configuration
=======================

##Optimized for Magento

Configuration files for:

1. Percona Server
2. nginx
3. php-fpm
4. vsftp
5. memcached
6. redis


##Timezone
```
tzselect
touch ~/.profile
echo -e "TZ='America/New_York'; export TZ" >> ~/.profile
mv /etc/localtime /etc/localtime.bak
ln -s /usr/share/zoneinfo/$TZ /etc/localtime
```

##Enable Repositories
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
rpm -Uhv http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm
```
```
sed -i "s/enable=0/enable=1/" /etc/yum.repos.d/epel.repo
sed -i "s/enable=0/enable=1/" /etc/yum.repos.d/remi.repo
sed -i "s/gpgkey=file:\/\/\/etc\/pki\/rpm-gpg\/RPM-GPG-KEY-remi/gpgkey=file:\/\/\/etc\/pki\/rpm-gpg\/RPM-GPG-KEY-remi\nexclude=mysql-*/" /etc/yum.repos.d/remi.repo
```

##Core Packages

```
yum update -y
```
```
yum groupinstall \
'Yum Utilities' \
'Development tools' \
'Development Libraries' \
'Additional Development' \
-y
```
```
yum remove \
postfix \
mysql \
mysql-devel \
mysql-libs \
-y
```
```
rm -rf /var/log/exim /var/log/maillog* /var/log/spooler*
```
```
yum install \
aspell-devel \
bzip2-devel \
cronie \
cronie-anacron \
crontabs \
curl-devel \
enchant \
enchant-devel \
freetype-devel \
gd \
gd-devel \
GeoIP \
GeoIP-devel \
gmp-devel \
gperf \
gperftools-devel \
gperftools-libs \
libatomic_ops-devel \
libevent-devel \
libicu-devel \
libjpeg-devel \
libjpeg-turbo-devel \
libmcrypt \
libmcrypt-devel \
libmhash \
libpng-devel \
libtidy-devel \
libvpx-devel \
libxml2-devel \
libXpm-devel \
libxslt-devel \
ncurses-devel \
net-snmp-devel \
ntp \
pcre-devel \
perl-ExtUtils-* \
systemtap-sdt-devel \
t1lib \
t1lib-devel \
t1lib-static \
vim-enhanced \
-y
```

##User Tweaks
```
cd ~
mkdir -p ~/.vim/autoload ~/.vim/bundle; \
curl -Sso ~/.vim/autoload/pathogen.vim \
    https://raw.github.com/tpope/vim-pathogen/master/autoload/pathogen.vim
```
```
echo -e "execute pathogen#infect()\n filetype plugin indent on\n" >> ~/.vimrc
cd ~/.vim/bundle
git clone git://github.com/tpope/vim-sensible.git
git clone git://github.com/altercation/vim-colors-solarized.git
cd ~
```
```
echo -e "syntax enable\nlet g:solarized_termtrans=1\nif (&t_Co >= 256 || "'$TERM'" == 'xterm-256color')\n\tlet g:solarized_termcolors=256\nelse\n\tlet g:solarized_termcolors=16\nendif\nset background=dark\ncolorscheme solarized" >> ~/.vimrc
cp -r ~/.vim* /etc/skel/
rm -rf /etc/skel/.gnome2
mkdir ~/git
cd ~/git
```
```
git clone https://github.com/MikeWilkie/EPEL-LEPP-Configuration
```

##nginx

```
cd ~/git
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
echo -e "\nexport PATH="'$PATH'":/root/git/depot_tools" >> ~/.bashrc
```
```
mkdir ~/git/nginx
cd nginx
wget http://nginx.org/download/nginx-1.4.3.tar.gz
tar -xzvf nginx-1.4.3.tar.gz
rm -rf nginx-1.4.3.tar.gz
```
```
git clone git://github.com/pagespeed/ngx_pagespeed.git
cd ngx_pagespeed
wget https://dl.google.com/dl/page-speed/psol/1.6.29.7.tar.gz
tar -xzvf 1.6.29.7.tar.gz
rm -rf 1.6.29.7.tar.gz
```
```
cd ~/git/nginx
git clone git://github.com/agentzh/headers-more-nginx-module.git
wget http://people.FreeBSD.org/~osa/ngx_http_redis-0.3.6.tar.gz
tar -xzvf ngx_http_redis-0.3.6.tar.gz
rm -rf ngx_http_redis-0.3.6.tar.gz
```
```
cd ~/git/nginx/nginx-1.4.3
```
```
./configure \
--prefix=/usr \
--sbin-path=/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/lock/subsys/nginx \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--user=nginx \
--group=www-data \
--with-google_perftools_module \
--with-http_secure_link_module \
--with-http_geoip_module \
--with-http_realip_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_mp4_module \
--with-http_degradation_module \
--with-http_stub_status_module \
--with-http_addition_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_image_filter_module \
--with-http_xslt_module \
--with-http_spdy_module \
--with-http_perl_module \
--with-poll_module \
--with-select_module \
--with-rtsig_module \
--with-libatomic \
--with-file-aio \
--with-ipv6 \
--with-file-aio \
--without-mail_pop3_module \
--without-mail_imap_module \
--without-mail_smtp_module \
--add-module=$HOME/git/nginx/headers-more-nginx-module \
--add-module=$HOME/git/nginx/ngx_pagespeed \
--add-module=$HOME/git/nginx/ngx_http_redis-0.3.6
```
```
make && make install
```
```
mkdir /var/www
mkdir -p /etc/skel/{backup,bin,error,html,log,tmp}
mkdir -p /var/www/$DOMAIN/tmp/ngx-pagespeed
mkdir -p /etc/nginx/conf.d
mkdir -p /etc/nginx/speed.d
mkdir -p /etc/nginx/ssl.d
mkdir -p /etc/nginx/framework.d
```
```
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
mv /etc/nginx/mime.types /etc/nginx/mime.types.bak
mv /etc/nginx/fastcgi_params /etc/nginx/fastcgi_params.bak
```
```
chmod 755 /var
chmod 755 /var/www
chmod -R go+rx /var
chmod -R go+rx /var/www
chmod -R go+rx /var/www/*
```
```
useradd $USER --home=/var/www/$DOMAIN --shell=/bin/bash --user-group --create-home
```
```
cp -r ~/.ssh /var/www/$DOMAIN/
chown -R $USER:$USER /var/www/$DOMAIN/
chmod -R o+wrx /var/www/$DOMAIN/tmp
chmod 700 /var/www/$DOMAIN/.ssh
chmod 600 /var/www/$DOMAIN/.ssh/authorized_keys
echo -e "$USER ALL=(ALL) ALL" >> /etc/sudoers
```
```
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/nginx.conf /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/mime.types /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/fastcgi_params /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/conf.d/default.conf /etc/nginx/conf.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/conf.d/domain.conf /etc/nginx/conf.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/gzip.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/headers-cache.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/headers-nocache.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/pagespeed.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/ssl.d/ssl.conf /etc/nginx/ssl.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/framework.d/*.conf /etc/nginx/framework.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/nginx /etc/init.d/
```
```
mv /etc/nginx/conf.d/domain.conf /etc/nginx/conf.d/$DOMAIN.conf
sed -i 's/$VAR_DOMAIN/$DOMAIN/' /etc/nginx/conf.d/$DOMAIN.conf
```
```
chmod -R 755 /etc/init.d/nginx
service nginx start
chkconfig nginx on
```

##nginx-ssl

```
cd /etc/nginx/ssl.d
openssl req -new -newkey rsa:2048 -nodes -keyout $DOMAIN.key -out $DOMAIN.csr
vim /etc/nginx/ssl.d/$DOMAIN.crt
vim /etc/nginx/ssl.d/chain.ca.crt
```
```
cat /etc/nginx/ssl.d/chain.ca.crt >> /etc/nginx/ssl.d/$DOMAIN.crt
sed -i "s/80 default;/80 default;\n\tlisten\t\t\t\t443 ssl;\n\tssl_certificate\t\t\t\/etc\/nginx\/ssl.d\/$DOMAIN.crt;\n\t\ssl_certificate_key\t\t\/etc\/nginx\/ssl.d\/$DOMAIN.key;\n\tinclude\t\t\t\t\/etc\/nginx\/ssl.d\/ssl.conf;/" /etc/nginx/conf.d/$DOMAIN.conf
```

##php-fpm

```
yum install \
php-cli \
php-common \
php-devel \
php-fpm \
php-pear \
-y
```
```
yum install \
php-gd \
php-ioncube-loader \
php-magickwand \
php-markdown \
php-mbstring \
php-mcrypt \
php-mysqlnd \
php-redis \
php-pdo \
php-soap \
php-tidy \
php-xml \
-y
```
```
yum install \
php-pear-XML-Beautifier \
php-pear-XML-Parser \
php-pear-XML-SVG \
php-pear-XML-Serializer \
php-pecl-geoip \
php-pecl-igbinary \
php-pecl-imagick \
php-pecl-lzf \
php-pecl-memcache \
php-pecl-pdflib \
php-pecl-zendopcache \
-y
```
```
mv /etc/php.ini /etc/php.ini.bak
mv /etc/php-fpm/php-fpm.conf /etc/php-fpm/php-fpm.conf.bak
mkdir -p /tmp/pear/cache
chmod -R 777 /tmp/pear/
```
```
pecl config-set php_ini /etc/php.ini
pecl config-set ext_dir /lib64/php/modules
```
```
mkdir -p /var/log/php-fpm
mkdir -p /var/run/php-fpm
chown -R nobody:nobody /var/run/php-fpm
chown -R nobody:nobody /var/log/php-fpm
chmod -R 755 /var/log/php-fpm
chmod -R 755 /var/run/php-fpm
```
```
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php.ini /etc/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php-fpm.d/domain.conf /etc/php-fpm.d/www.conf
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php-fpm.conf /etc/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/php-fpm /etc/init.d/
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/php.d/opcache.ini /etc/php.d/opcache.ini
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/php.d/memcache.ini /etc/php.d/memcache.ini
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/php-fpm /etc/init.d/
```
```
mv /etc/php-fpm.d/domain.conf /etc/php-fpm.d/$DOMAIN.conf
sed -i 's/$VAR_USER/$USER/' /etc/php-fpm.conf
sed -i 's/$VAR_DOMAIN/$DOMAIN/' /etc/php-fpm.d/$DOMAIN.conf
```
```
chmod -R 755 /etc/init.d/php-fpm
service php-fpm start
chkconfig php-fpm on
```

##Percona Server (MySQL)

```
yum install \
Percona-Server-client-55 \
Percona-Server-server-55 \
Percona-Server-shared-55 \
Percona-Server-devel-55 \
Percona-Server-shared-compat \
percona-toolkit \
percona-xtrabackup \
-y
```
```
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/my.cnf /etc/
sed -i 's/datadir=\/var\/lib\/mysql/datadir=\/var\/lib\/mysql\/data/' /etc/init.d/mysql
```
```
rm -rf /var/log/mysql*
mkdir /var/log/mysql
mkdir -p /var/lib/mysql/data
mkdir -p /var/run/mysql
chown -R mysql:mysql /var/lib/mysql /var/log/mysql /var/run/mysql
chmod -R 755 /var/lib/mysql /var/log/mysql /var/run/mysql
```
```
mysql_install_db
```
```
chmod -R 755 /etc/init.d/mysql
service mysql start
ln -s /var/run/mysql/mysql.sock /var/lib/mysql/mysql.sock
mysql_secure_installation
chkconfig mysql on
```

##vsFTP
```
yum install vsftpd -y
```
```
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/vsftpd/vsftpd.conf /etc/vsftpd/
chmod -R 755 /etc/init.d/vsftpd
```
```
service vsftpd start
chkconfig vsftpd on
```

##memcached

```
yum install \
memcached \
memcached-devel \
-y
```
```
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/sysconfig/memcache.ini /etc/sysconfig/memcache.ini
```
```
mkdir -p /var/run/memcached
chown -R nobody:nobody /var/run/memcached
chmod -R 755 /var/run/memcached
```
```
sed -i 's/PORT=.*/PORT="11211"/' /etc/sysconfig/memcached
sed -i 's/USER=.*/"$USER"/' /etc/sysconfig/memcached
sed -i 's/MAXCONN=.*/MAXCONN="2048"/' /etc/sysconfig/memcached
sed -i 's/CACHESIZE=.*/CACHESIZE="512"/' /etc/sysconfig/memcached
sed -i 's/OPTIONS=.*/OPTIONS="-s /var/run/memcached/memcached.sock -a 0777"/' /etc/sysconfig/memcached
sed -i 's/PORT=.*/PORT="11211"/' /etc/init.d/memcached
sed -i 's/USER=.*/"$USER"/' /etc/init.d/memcached
sed -i 's/MAXCONN=.*/MAXCONN="2048"/' /etc/init.d/memcached
sed -i 's/CACHESIZE=.*/CACHESIZE="512"/' /etc/init.d/memcached
sed -i 's/OPTIONS=.*/OPTIONS="-s /var/run/memcached/memcached.sock -a 0777"/' /etc/init.d/memcached
```
```
chmod -R 755 /etc/init.d/memcached
service memcached start
chkconfig memcached on
```

##redis

```
yum install \
redis \
-y
```
```
mv /etc/redis.conf /etc/redis.conf.bak
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/redis /etc/init.d/redis
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/redis.conf /etc/
```
```
chmod -R 755 /etc/init.d/redis
service redis start
chkconfig redis on
```

##ntp

```
yum install \
ntp \
-y
```
```
ntpdate pool.ntp.org
service ntpd start
chkconfig ntpd on
```
