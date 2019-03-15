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

__1. Install Nextcloud__  

1. Install dependencies.

    ```bash
    sudo apt install apache2 mariadb-server libapache2-mod-php7.2
    sudo apt install php7.2-gd php7.2-json php7.2-mysql php7.2-curl php7.2-mbstring
    sudo apt install php7.2-intl php-imagick php7.2-xml php7.2-zip
    ```

    > __Note__  
    > - `miradb` can be replaced with `mysql`.

2. Download `nextcloud-15.0.5.zip` from official website and `unzip` it.
3. Copy site files to webserver's data root: `sudo cp -r nextcloud /var/www`.
4. Change owner / executor to user `www-data`.
5. Create database `nextcloud` and database user `nextclouduser@localhost`, and grant access.
6. Set-up default Apache server, in order to test reverse proxy (config file copied from [here](https://docs.nextcloud.com/server/15/admin_manual/installation/source_installation.html#apache-web-server-configuration)).
    - Here I bound service to `localhost:80/nextcloud`.

__2. Install Nginx__  

1. `sudo apt install nginx`
2. `cat /etc/passwd | grep www-data` to make sure we have an HTTP user.
    - Here it's `www-data` under Ubuntu.
3. `sudo service nginx start`
4. Open a browser tab and visit `127.0.0.1` to check if Nginx is running properly.
    - If there exists a previous installation of other web servers, say the Apache server we've just installed via
        `sudo apt install`, good chance is that an `index.html` already exists in `/var/www/html/`. However, for the
        default Nginx configuration, the HelloWorld page is loaded in the following order ...

        ```nginx
            index index.html index.htm index.nginx-debian.html;
        ```

        So don't panic when you are greeted by Apache :smile_cat: !  

    - Also, do remember to change the default port configuration, so it doesn't collide with the existing Apache server!

__3. [Optional] Configure Nginx for Nextcloud__  

_Nginx will be used for reverse proxy at the end of the day, therefore a dedicate Nginx-powered Nextcloud service is unnecessary!
I just did this for fun :smile_cat: !_  

1. Create configuration file `/etc/nginx/conf.d/nextcloud.conf`.
    - Copied from [here](https://www.linuxbabe.com/ubuntu/install-nextcloud-ubuntu-18-04-nginx-lemp) (or [Nginx Configuration](https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html) offered in reference).
    - Here I bound service to `localhost:9001`.
2. `sudo apt install php7.2-fpm`
    - Required for Nginx to run PHP files, or `502 Bad Gateway`.
3. `sudo systemctl start php7.2-fpm.service`
4. `sudo systemctl reload nginx.service`

At this point, __both__ `localhost:80/nextcloud` (via Apache) and `localhost:9001` (via Nginx) can reach our Nextcloud service.  

__4. Configure Nginx for Reverse Proxy__  

1. Replace `/etc/nginx/conf.d/nextcloud.conf` with following lines

    ```nginx
    upstream localserver1 {
        server 127.0.0.1:80;    # our local nextcloud service
        # you may have more than just one `server` statement in
        # this block, which enables Nginx to dynamically switch
        # between upstreams based on load.
    }

    server {
        listen 9001;
        server_name _;

        # redirect every subdir to our Apache-powered Nextcloud
        location / {
            proxy_pass http://localserver1/nextcloud/;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    ```

    > __Note__  
    > - The `proxy_set_header` lines are required, or Nextcloud will identify the Nginx proxy, disallowing access.

2. Visit `localhost:9001`.
    - After Nginx caching for sometime, you will see browser's URL stops at `localhost/nextcloud/index.php/xxxx`, which indicates our Nginx reverse proxy is working as expected.

__5. Final Touches__  

1. Open browser tab and visit Nextcloud service.
2. Walk through Nextcloud initialization step in its fancy WebApp.
    - Which __can__ be done in command line (see [Installing from command line](https://docs.nextcloud.com/server/15/admin_manual/installation/command_line_installation.html)).
3. Enjoy.

---

# References

- [Nextcloud 15 Administration Manual 15 documentation](https://docs.nextcloud.com/server/15/admin_manual/#)
    - [Nginx Configuration](https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html)
- [How to Install LEMP Stack (Nginx, MariaDB, PHP7.2) on Ubuntu 18.04 LTS](https://www.linuxbabe.com/ubuntu/install-lemp-stack-nginx-mariadb-php7-2-ubuntu-18-04-lts)
- [How to Install NextCloud on Ubuntu 18.04 with Nginx (LEMP Stack)](https://www.linuxbabe.com/ubuntu/install-nextcloud-ubuntu-18-04-nginx-lemp)
- [Reset Forgotten MySQL Root Password](https://www.howtoforge.com/reset-forgotten-mysql-root-password) :smile_cat:

