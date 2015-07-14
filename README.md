EPEL-LEPP-Configuration
================
##Optimized for Magento

Configuration files for:

1. Percona Server
2. nginx
3. php-fpm
5. memcached
6. redis
7. ntp
8. iptables


##Timezone
```
tzselect
touch ~/.profile
echo -e "TZ='America/New_York'; export TZ" >> ~/.bash_profile
mv /etc/localtime /etc/localtime.bak
ln -s /usr/share/zoneinfo/$TZ /etc/localtime
```

##Create User Variables
```
echo -e "APP_USER='appusername'; export APP_USER" >> ~/.bash_profile
echo -e "APP_DOMAIN='appdomain.com'; export APP_DOMAIN" >> ~/.bash_profile
```

##Enable Repositories
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```
```
sed -i '0,/enabled=0/s//enabled=1/' /etc/yum.repos.d/epel.repo
sed -i '0,/enabled=0/s//enabled=1/' /etc/yum.repos.d/remi.repo
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
mysql \
mysql-devel \
mysql-libs \
-y
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
wget \
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

##ramfs

```
mkdir /mnt/{zoom_pagecache,ngx_pgspd_meta,ngx_pgspd_file}
chown -R nobody:nobody /mnt/*
chmod -R 777 /mnt/*
```
```
echo 'none            /mnt/zoom_pagecache     ramfs nodev,nosuid,size=64M,mode=1777 0 0' >> /etc/fstab
cat /etc/fstab
echo 'none            /mnt/ngx_pgspd_meta     ramfs nodev,nosuid,size=64M,mode=1777 0 0' >> /etc/fstab
cat /etc/fstab
echo 'none            /mnt/ngx_pgspd_file     ramfs nodev,nosuid,size=64M,mode=1777 0 0' >> /etc/fstab
cat /etc/fstab
```
```
mount -a
```


##openssl

```
rpm -ivh --nosignature http://rpm.axivo.com/axivo-release-6-1.noarch.rpm
sed -i '0,/enabled=0/s//enabled=1/' /etc/yum.repos.d/axivo.repo
sed -i "s/gpgkey=file:\/\/\/etc\/pki\/rpm-gpg\/RPM-GPG-KEY-AXIVO/gpgkey=file:\/\/\/etc\/pki\/rpm-gpg\/RPM-GPG-KEY-AXIVO\nincludepkgs=openssl*/" /etc/yum.repos.d/axivo.repo
yum install openssl openssl-libs openssl-devel openssl-static -y
yum update openssl openssl-libs openssl-devel openssl-static -y
```

##lua (required for certain nginx modules)
```
yum install lua \
lua-devel \
lua-static \
luarocks \
-y
luarocks install lua-cjson
```

##nginx

```
mkdir ~/git/nginx
cd ~/git/nginx
wget http://nginx.org/download/nginx-1.9.2.tar.gz
tar -xzvf nginx-1*.tar.gz
rm -rf nginx-1*.tar.gz
wget https://www.openssl.org/source/openssl-1.0.1p.tar.gz
tar xzvf openssl* && rm -rf openssl-1.0.1p.tar.gz
```
```
cd ~/git/nginx
git clone git://github.com/pagespeed/ngx_pagespeed.git
cd ~/git/nginx/ngx_pagespeed
NGX_PAGESPEED_VERSION=$(git describe --abbrev=0 --tags)
NGX_PAGESPEED_VERSION=${NGX_PAGESPEED_VERSION/v/}
NGX_PAGESPEED_VERSION=${NGX_PAGESPEED_VERSION/-beta/}
wget https://dl.google.com/dl/page-speed/psol/${NGX_PAGESPEED_VERSION}.tar.gz
tar -xzvf ${NGX_PAGESPEED_VERSION}.tar.gz
```
```
cd ~/git/nginx

git clone git://github.com/agentzh/headers-more-nginx-module.git
git clone git://github.com/agentzh/echo-nginx-module.git
git clone git://github.com/openresty/srcache-nginx-module.git
git clone git://github.com/openresty/lua-nginx-module.git
git clone git://github.com/bpaquet/ngx_http_enhanced_memcached_module.git
git clone git://github.com/HoopCHINA/ngx_http_redis.git
git clone git://github.com/anomalizer/ngx_aws_auth.git
git clone git://github.com/simpl/ngx_auto_lib.git

cd ~/git/nginx/nginx-1*
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
--group=nginx \
--group=www-data \
--with-openssl=$HOME/git/nginx/openssl-1.0.1p \
--with-google_perftools_module \
--with-http_ssl_module \
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
--add-module=$HOME/git/nginx/echo-nginx-module \
--add-module=$HOME/git/nginx/lua-nginx-module \
--add-module=$HOME/git/nginx/srcache-nginx-module \
--add-module=$HOME/git/nginx/ngx_http_enhanced_memcached_module \
--add-module=$HOME/git/nginx/ngx_http_redis \
--add-module=$HOME/git/nginx/ngx_pagespeed \
--add-module=$HOME/git/nginx/ngx_aws_auth \
--add-module=$HOME/git/nginx/ngx_auto_lib
```
```
make && make install
```
```
mkdir /var/www
mkdir -p /etc/skel/{backup,bin,error,html,log,tmp}
mkdir -p /etc/nginx/conf.d
mkdir -p /etc/nginx/speed.d
mkdir -p /etc/nginx/ssl.d
mkdir -p /etc/nginx/framework.d
mkdir -p /tmp/ngx_pagespeed
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
chown -R nobody:nobody /tmp/ngx_pagespeed
chmod -R 777 /tmp/ngx_pagespeed
```
```
useradd $APP_USER --home=/var/www/$APP_DOMAIN --shell=/bin/bash --user-group --create-home
```
```
cp -r ~/.ssh /var/www/$APP_DOMAIN/
chown -R $APP_USER:$APP_USER /var/www/$APP_DOMAIN/
chmod -R o+wrx /var/www/$APP_DOMAIN/tmp
chmod 700 /var/www/$APP_DOMAIN/.ssh
chmod 600 /var/www/$APP_DOMAIN/.ssh/authorized_keys
echo -e "$APP_USER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```
```
rm -rf /etc/nginx/conf.d/*
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/nginx.conf /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/mime.types /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/fastcgi_params /etc/nginx/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/conf.d/domain.conf /etc/nginx/conf.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/conf.d/domain.ssl.conf.bk /etc/nginx/conf.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/gzip.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/cors.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/cache.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/nocache.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/pagespeed.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/speed.d/pagespeed-server.conf /etc/nginx/speed.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/ssl.d/ssl.conf /etc/nginx/ssl.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/nginx/framework.d/*.conf /etc/nginx/framework.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/nginx /etc/init.d/
```
```
mv /etc/nginx/conf.d/domain.conf /etc/nginx/conf.d/$APP_DOMAIN.conf
mv /etc/nginx/conf.d/domain.ssl.conf.bk /etc/nginx/conf.d/$APP_DOMAIN.ssl.conf.bk
sed -i 's/$VAR_DOMAIN/'$APP_DOMAIN'/g' /etc/nginx/conf.d/$APP_DOMAIN.conf
sed -i 's/$VAR_USER/'$APP_USER'/g' /etc/nginx/nginx.conf
```
```
chmod -R 755 /etc/init.d/nginx
service nginx start
chkconfig nginx on
```

##nginx-ssl

```
cd /etc/nginx/ssl.d
openssl req -new -newkey rsa:2048 -nodes -keyout $APP_DOMAIN.key -out $APP_DOMAIN.csr
vim /etc/nginx/ssl.d/$APP_DOMAIN.crt
vim /etc/nginx/ssl.d/chain.ca.crt
```
```
cat /etc/nginx/ssl.d/chain.ca.crt /etc/nginx/ssl.d/$APP_DOMAIN.crt > trusted.crt;
mv /etc/nginx/conf.d/$APP_DOMAIN.ssl.conf.bk /etc/nginx/conf.d/$APP_DOMAIN.ssl.conf
sed -i "s/#listen;/ listen/" /etc/nginx/conf.d/$APP_DOMAIN.conf
```
```
openssl dhparam -out /etc/nginx/ssl.d/dhparam.pem 2048
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
rm -rf /etc/php-fpm.d/*
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php.ini /etc/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php-fpm.d/www.conf /etc/php-fpm.d/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/php-fpm.conf /etc/
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/init.d/php-fpm /etc/init.d/
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/php.d/opcache.ini /etc/php.d/opcache.ini
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/php.d/memcache.ini /etc/php.d/memcache.ini

```
```
mv /etc/php-fpm.d/www.conf /etc/php-fpm.d/$APP_DOMAIN.conf
sed -i 's/$VAR_USER/'$APP_USER'/' /etc/php-fpm.conf
sed -i 's/$VAR_DOMAIN/'$APP_DOMAIN'/' /etc/php-fpm.d/$APP_DOMAIN.conf
sed -i 's/$VAR_USER/'$APP_USER'/' /etc/php-fpm.d/$APP_DOMAIN.conf

```
```
mv /etc/php-fpm.d/www.conf /etc/php-fpm.d/$APP_DOMAIN.conf
sed -i 's/$VAR_USER/$APP_USER/' /etc/php-fpm.conf
sed -i 's/$VAR_DOMAIN/$APP_DOMAIN/' /etc/php-fpm.d/$APP_DOMAIN.conf
```
```
chmod -R 755 /etc/init.d/php-fpm
service php-fpm start
chkconfig php-fpm on
```

##Percona Server (MySQL)

```
rpm -ivh http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm
yum install \
Percona-Server-client-56 \
Percona-Server-server-56 \
Percona-Server-shared-56 \
Percona-Server-devel-56 \
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

##memcached

```
yum install \
memcached \
memcached-devel \
-y
```
```
cp -r ~/git/EPEL-LEPP-Configuration/conf/etc/sysconfig/memcached /etc/sysconfig/memcached
```
```
mkdir -p /var/run/memcached
chown -R nobody:nobody /var/run/memcached
chmod -R 755 /var/run/memcached
```
```
sed -i 's/PORT=.*/PORT="11211"/' /etc/sysconfig/memcached
sed -i 's/USER=.*/"nobody"/' /etc/sysconfig/memcached
sed -i 's/MAXCONN=.*/MAXCONN="2048"/' /etc/sysconfig/memcached
sed -i 's/CACHESIZE=.*/CACHESIZE="128"/' /etc/sysconfig/memcached
sed -i 's/OPTIONS=.*/OPTIONS="-l 127.0.0.1"/' /etc/sysconfig/memcached
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
cp -f ~/git/EPEL-LEPP-Configuration/conf/etc/redis.conf /etc/
```
```
chmod -R 755 /etc/init.d/redis-server
service redis-server start
chkconfig redis-server on
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

##iptables

```
iptables -F
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,53,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,53,80,443 -m state --state ESTABLISHED -j ACCEPT
iptables -I INPUT -p udp --dport 123 -j ACCEPT
iptables -I OUTPUT -p udp --sport 123 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -L -n
iptables-save | tee /etc/sysconfig/iptables
service iptables restart
```
