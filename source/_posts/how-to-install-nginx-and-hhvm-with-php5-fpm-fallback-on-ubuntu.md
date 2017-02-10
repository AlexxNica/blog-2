---
title: How to install NGINX and HHVM with PHP5-FPM Fallback on Ubuntu 16/14
date: 2017-01-19 14:00:00
tags:
- nginx
- hhvm
- php
- ubuntu
---
#How to install NGINX and HHVM with PHP5-FPM Fallback on Ubuntu 16/14

##In this tutorial I will be guiding you with steps to install NGINX stable, HHVM 3.9 and PHP 5 FPM.


In this tutorial I will be guiding you with steps to install NGINX stable, HHVM 3.9 and PHP 5 FPM.

* [Nginx](https://nginx.org) is a great webserver or reverse proxy that comes with a default configuration much faster than Apache 2.4 with its default MPM (Prefork).

* [HHVM](http://hhvm.com/) is a virtual machine that uses JIT (Just in time) compilation aproach.

* [PHP-FPM](https://php-fpm.org/) (FastCGI Process Manager) is an alternative PHP FastCGI implementation with some additional features useful for sites of any size, especially busier sites.

A little setup background:

* Clean KVM Machine with Ubuntu 16/14 LTS with 1GB RAM and 1 CPU Core from [Vultr](http://www.vultr.com/?ref=6823526) (Less than 1GB RAM will cause you trouble if you bundle MySQL)

* Root access (Else you must do sudo [command])

First, Nginx:

    add-apt-repository ppa:nginx/stable
    apt-get update && apt-get upgrade
    apt-get install nginx

Second, HHVM:

    apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449
    add-apt-repository "deb http://dl.hhvm.com/ubuntu $(lsb_release -sc) main"
    apt-get update
    apt-get install hhvm
    update-rc.d hhvm defaults
    /usr/bin/update-alternatives --install /usr/bin/php php /usr/bin/hhvm 60
    /usr/share/hhvm/install_fastcgi.sh

We want to run HHVM on a Unix Socket, so let’s edit it’s configuration:

    nano /etc/hhvm/server.ini

Comment “hhvm.server.port = 9000” with ; and add “hhvm.server.file_socket = /var/run/hhvm/hhvm.sock”

It should look like this:

    ; php options
    pid = /var/run/hhvm/pid

    ; hhvm specific
    ;hhvm.server.port = 9000
    hhvm.server.file_socket = /var/run/hhvm/hhvm.sock
    hhvm.server.type = fastcgi
    hhvm.server.default_document = index.php
    hhvm.log.use_log_file = true
    hhvm.log.file = /var/log/hhvm/error.log
    hhvm.repo.central.path = /var/run/hhvm/hhvm.hhbc

Save with Ctrl + X and then ‘Y’

Now PHP-FPM:

    apt-get install php5-fpm

This should install FPM and listen on /var/run/php5-fpm.sock by default

We are done installing! Let’s configure things now. Let’s start by creating a upstream conf file for nginx:

    nano /etc/nginx/conf.d/upstream.conf

Add the following and save:

    upstream php {
            server unix:/var/run/hhvm/hhvm.sock;
            server unix:/var/run/php5-fpm.sock backup;
    }

As you can see, PHP-FPM will be a backup in case HHVM fails.

Now let’s edit Nginx’s HHVM conf file:

    nano /etc/nginx/hhvm.confg

Replace “fastcgi_pass 127.0.0.1:9000;” with “fastcgi_pass php;” . It should look like this:

    location ~ \.(hh|php)$ {
        fastcgi_keep_conn on;
        fastcgi_pass   php;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

Restart the services with:

    service hhvm restart && service php5-fpm restart && service nginx restart

Test with:

    curl -I localhost/test.php

Should be a 404 (As the actual test.php doesn’t exist) with these headers:

    Server: nginx/1.8.0
    X-Powered-By: HHVM/3.9.0

And that’s all! We have latest stable Nginx with latest HHVM in fastcgi with php-fpm fallback!
