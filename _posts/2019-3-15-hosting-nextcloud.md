---
layout: article
title: Hosting Nextcloud
key: hosting-nextcloud
tags: Assignments
---

Steps to host your own Nextcloud service.  

<!-- more -->

# The Task

## The Goals

- [x] Manual deploy, i.e. no Docker or Snap.
- [x] Support for online previewing for pictures, videos and documents.
- [x] Nginx for reverse proxy


## The Steps

__1. Set-Up Nginx__  

1. `sudo apt install nginx`
2. `cat /etc/passwd | grep www-data` to make sure we have an HTTP user.
    - Here it's `www-data` under Ubuntu.
3. `sudo service nginx start`
4. Open a browser tab and visit `127.0.0.1` to check if Nginx is running properly.
    - If there exists a previous installation of other web servers, good chance is that an `index.html` already exists
        in `/var/www/html/`. However, for the default Nginx configuration, the HelloWorld page is loaded in the following
        order ...  

        ```text
        index index.html index.htm index.nginx-debian.html;
        ```

        So don't panic when you are greeted by Apache :smile_cat:!  

    - Also, do remember to change the default port configuration! (see __3. Configure Nginx__)

__2. Install Nextcloud__  

1. Install dependencies.

    ```bash
    sudo apt install apache2 mariadb-server libapache2-mod-php7.2
    sudo apt install php7.2-gd php7.2-json php7.2-mysql php7.2-curl php7.2-mbstring
    sudo apt install php7.2-intl php-imagick php7.2-xml php7.2-zip
    ```

    > Note  
    > - `libapace*` libraries are installed though we want to run Nginx, but it packs PHP libraries, less bother.
    > - `miradb` can be replaced with `mysql`.

2. Download `nextcloud-15.0.5.zip` from official website and `unzip` it.
3. Copy site files to webserver's data root: `sudo cp -r nextcloud /var/www`.
4. Change owner / executor to user `www-data`.
5. Create database `nextcloud` and database user `nextclouduser@localhost`.

__3. Configure Nginx__  

1. Create configuration file `/etc/nginx/conf.d/nextcloud.conf`.
    - Copied from [here](https://www.linuxbabe.com/ubuntu/install-nextcloud-ubuntu-18-04-nginx-lemp) (or [Nginx Configuration](https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html) offered in reference).
2. `sudo apt install php7.2-fpm`
    - Required for nginx to run PHP files.
3. `sudo systemctl reload nginx.service`

__Final Touches__  

1. Open browser tab and navigate to <localhost/nextcloud>.
2. Walk Nextcloud initialization in the fancy GUI.
    - Which __can__ be done in command line.
3. Enjoy.

__Optional__  

- Remove service defined in Apache.


# References

- [Nextcloud 15 Administration Manual 15 documentation](https://docs.nextcloud.com/server/15/admin_manual/#)
    - [Nginx Configuration](https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html)
- [How to Install LEMP Stack (Nginx, MariaDB, PHP7.2) on Ubuntu 18.04 LTS](https://www.linuxbabe.com/ubuntu/install-lemp-stack-nginx-mariadb-php7-2-ubuntu-18-04-lts)
- [How to Install NextCloud on Ubuntu 18.04 with Nginx (LEMP Stack)](https://www.linuxbabe.com/ubuntu/install-nextcloud-ubuntu-18-04-nginx-lemp)
- [Reset Forgotten MySQL Root Password](https://www.howtoforge.com/reset-forgotten-mysql-root-password) :smile_cat:

