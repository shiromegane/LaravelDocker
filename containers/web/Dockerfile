FROM centos:latest
MAINTAINER Shiromegane

# Update yum
RUN yum -y update && yum clean all

# Install and setting eple,remi repository
RUN yum -y install epel-release && \
    yum -y install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm && \
    sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/epel.repo && \
	sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/remi.repo

# httpd, sshd, scp, openssl, sudo, which
RUN yum install -y httpd httpd-tools openssh-server openssh-clients openssl sudo which && yum clean all

# libmcrypt, supervisor
RUN yum install --enablerepo=epel -y libmcrypt supervisor && yum clean all

# git, vim
RUN yum -y install git && yum -y install vim-enhanced && \
    echo alias vi='vim' >> /etc/profile && source /etc/profile

# ImageMagick, file
RUN yum -y install ImageMagick file && yum clean all

# chrony, timezone
RUN yum install -y chrony && systemctl enable chronyd && \
    cp -p /usr/share/zoneinfo/Japan /etc/localtime && \
    echo -e 'ZONE="Asia/Tokyo"\nUTC=false' > /etc/sysconfig/clock

# libargon2, libraqm
RUN yum install --enablerepo=epel -y libargon2 libraqm && yum clean all

# php
RUN yum install --enablerepo=remi-php72 -y php php-cli php-fpm php-common php-devel php-opcache \
    php-zip php-gd php-mysqlnd php-xml php-mbstring php-mcrypt php-pecl-imagick \
    php-pecl-redis php-pecl-apcu php-pecl-zendopcache && yum clean all

# php.ini
RUN sed -i "s/;date.timezone *=.*$/date.timezone = 'Asia\/Tokyo'/" /etc/php.ini && \
    sed -i "s/;mbstring.language = Japanese/mbstring.language = Japanese/" /etc/php.ini && \
    sed -i "s/;mbstring.internal_encoding =/mbstring.internal_encoding = UTF-8/" /etc/php.ini && \
    sed -i "s/;mbstring.http_input =/mbstring.http_input = UTF-8/" /etc/php.ini && \
    sed -i "s/upload_max_filesize = 2M/upload_max_filesize = 10M/" /etc/php.ini

# composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# phpunit
RUN curl -L https://phar.phpunit.de/phpunit.phar > /usr/local/bin/phpunit && \
	chmod +x /usr/local/bin/phpunit

# initialize for ssh
RUN sed -i '/pam_loginuid\.so/s/required/optional/' /etc/pam.d/sshd && \
	ssh-keygen -A

# Laravel
RUN composer global require "laravel/installer"
ENV HOME /root
ENV PATH $PATH:$HOME/.composer/vendor/bin

# create login user
#RUN useradd -d /home/laravel -m -s /bin/bash laravel && \
#	echo laravel:****laravel | chpasswd && \
#	echo 'laravel ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

ENV DOCUMENT_ROOT /app/public
CMD mkdir $DOCUMENT_ROOT
WORKDIR $DOCUMENT_ROOT

ADD ./httpd.conf /etc/httpd/conf/httpd.conf
ADD ./vhosts.conf /etc/httpd/conf.d/vhosts.conf

EXPOSE 22 80 443

CMD /usr/sbin/httpd -DFOREGROUND
