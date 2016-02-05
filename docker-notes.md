Install Docker

    sudo apt-get install docker.io
    sudo ln -sf usr/bin/docker.io /usr/local/bin/docker

    ls -l /usr/bin/docker*
    -rwxr-xr-x 1 root root 15240483 Aug 21  2014 /usr/bin/docker
    lrwxrwxrwx 1 root root        6 Aug 21  2014 /usr/bin/docker.io -> docker

    ls -l /usr/local/bin/docker*
    lrwxrwxrwx 1 root root 18 Oct 17 16:04 /usr/local/bin/docker -> /usr/bin/docker.io

    So the docker executable (about 15 MB) is in /usr/bin/docker. And we have a couple of links to it.

Try it

    sudo docker run hello-world
    sudo docker run -p 8080 google/nodejs-hello
    sudo docker run --rm -ti ubuntu:latest /bin/bash

[https://docs.docker.com/reference/commandline/run/](https://docs.docker.com/reference/commandline/run/)
[https://docs.docker.com/reference/run/](https://docs.docker.com/reference/run/)

    docker run: Run a command in a new container.
    --rm   Automatically remove the container when it exits.
    -t   Allocate a pseudo-TTY.
    -i   Keep STDIN open even if not attached.
    The image is ubuntu:latest.
    The command is /bin/bash.

Deploy the Node.js (MongoDB) app. SSH into one of the managed VMS. List the Docker containers:

    sudo docker ps

    CONTAINER ID        IMAGE
    1b0ddb299ed7        gcr.io/google_appengine/mvm-agent:latest
    592e8e1481ee        gcr.io/google_appengine/nginx-proxy:latest
    da83e0377870        appengine.gcr.io/388964074776919141/steve-41.default.20151201t210516:latest
    4bb7712373d9        gcr.io/google_appengine/memcache-proxy:latest
    6bd096583910        gcr.io/google_appengine/fluentd-logger:latest

List the Docker images:

    sudo docker images

    REPOSITORY                                                             TAG                 IMAGE ID
    appengine.gcr.io/388964074776919141/steve-41.default.20151201t210516   latest              bbeb5fb67051
    gcr.io/google_appengine/monitoring                                     2015-11-09b         bbd4e3d96b8a
    gcr.io/google_appengine/monitoring                                     latest              bbd4e3d96b8a
    ....
    gcr.io/google_containers/pause                                         0.8.0               2c40b0526b63

See Docker image layers

    sudo docker history appengine.gcr.io/390467708463985433/steve-49.default.20160204t090356

    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    3cd377324f84        4 hours ago         /bin/sh -c #(nop) CMD ["/bin/sh" "-c" "npm st   0 B
    19c86b46d556        4 hours ago         /bin/sh -c /usr/local/bin/install_node '>=0.1   0 B
    31ae39eb5cbf        4 hours ago         /bin/sh -c npm install --unsafe-perm ||   ((i   59.12 MB
    46ebc20df40b        4 hours ago         /bin/sh -c #(nop) COPY dir:b89ae56b062ef1b768   33.66 kB
    d0d258f887ad        8 weeks ago         /bin/sh -c #(nop) CMD ["npm" "start"]           0 B
    f9361aab81dd        8 weeks ago         /bin/sh -c #(nop) WORKDIR /app                  0 B
    57a6938d5c4a        8 weeks ago         /bin/sh -c #(nop) ENV NODE_ENV=production       0 B
    e5a210e985ec        8 weeks ago         /bin/sh -c #(nop) ADD file:35d266db05f8155d7d   2.907 kB
    4c180d2197f3        8 weeks ago         /bin/sh -c npm install https://storage.google   239.7 kB
    84d8a851e8dd        8 weeks ago         /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
    e7bb6c4563d4        8 weeks ago         /bin/sh -c mkdir /nodejs && curl https://node   35.48 MB
    f6a7fb8d0061        8 weeks ago         /bin/sh -c apt-get update -y && apt-get insta   225.3 MB
    e8aed8091139        10 weeks ago        /bin/sh -c apt-get -q update &&     apt-get i   56.91 MB
    36e2f6c710be        10 weeks ago        /bin/sh -c #(nop) ENV PORT=8080                 0 B
    54f405e77b26        10 weeks ago        /bin/sh -c #(nop) ENV DEBIAN_FRONTEND=noninte   0 B
    16d49c9e1091        10 weeks ago        /bin/sh -c #(nop) CMD []                        0 B
    8f8068a6a6b4        6 months ago        /bin/sh -c #(nop) ADD file:6958cfcdc7475df054   163.5 MB
    559718b5f880        6 months ago        /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
    643a001c5ee0        6 months ago        /bin/sh -c #(nop) ENV DEBIAN_FRONTEND=noninte   0 B

I think we're looking at the layers that make up the appengine.gcr... image
(most recet layer at the top of the list). For each layer, we see a portion of the line
(in some Dockerfile ?) that created the layer. To see the whole line, we can use 
docker inspect.

Start at the bottom of the list:

    sudo docker inspect 643a001c5ee0
    "Cmd": ["/bin/sh", "-c", "#(nop) ENV DEBIAN_FRONTEND=noninteractive"]

Work up from the bottom of the list:

    "Cmd": ["/bin/sh","-c", "#(nop) ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"]

    "Cmd": ["/bin/sh", "-c", "#(nop) ADD file:6958cfcdc7475df054052ac9872193828b68036a34279b7916f6b00509797cbe in /"]

    "Cmd": ["/bin/sh", "-c", "#(nop) CMD []"]

    "Cmd": ["/bin/sh", "-c", "#(nop) ENV DEBIAN_FRONTEND=noninteractive"]

    ...

Does each of these layers correspond to an instruction in a Dockerfile?
Is it as if we had a Dockerfile like this?

    ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    ADD file:6958cfcdc7475df054052ac9872193828b68036a34279b7916f6b00509797cbe in /
    CMD []
    ENV DEBIAN_FRONTEND=noninteractive
    ENV PORT=8080
    apt-get -q update \u0026\u0026   apt-get install --no-install-recommends -y -q ca-certificates \u0026\u0026     apt-get -y -q upgrade \u0026\u0026     rm /var/lib/apt/lists/*_*
    apt-get update -y \u0026\u0026   apt-get install --no-install-recommends -y -q curl python build-essential git ca-certificates libkrb5-dev \u0026\u0026     apt-get clean \u0026\u0026 rm /var/lib/apt/lists/*_*
    mkdir /nodejs \u0026\u0026 curl https://nodejs.org/dist/v4.2.3/node-v4.2.3-linux-x64.tar.gz | tar xvzf - -C /nodejs --strip-components=1
    ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/nodejs/bin
    npm install https://storage.googleapis.com/gae_node_packages/semver.tar.gz
    ADD file:35d266db05f8155d7dfc9345d6a0b25ea3dbdaad0e1480fd5c954fdb889a1580 in /usr/local/bin/install_node
    ENV NODE_ENV=production
    WORKDIR /app
    CMD ["npm" "start"]
    ....

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

[Nice Dockerfile exercise](https://docs.docker.com/engine/examples/nodejs_web_app/)

Let's look at the Dockerfile for PHP on Managed VMs.

[GoogleCloudPlatform/php-docker](https://github.com/GoogleCloudPlatform/php-docker/blob/master/php-nginx/Dockerfile)

    FROM gcr.io/google_appengine/base

    RUN apt-get update && apt-get install -y --no-install-recommends \
    cron \
    curl \
    gettext \
    git \
    libbz2-1.0 \
    libicu52 \
    libmcrypt4 \
    libmemcached11 \
    libmemcachedutil2 \
    libpcre3 \
    libpng12-0 \
    libpq5 \
    libreadline6 \
    librecode0 \
    libsqlite3-0 \
    libxml2 \
    libxslt1.1 \
    logrotate \
    mercurial \
    subversion \
    supervisor \
    zlib1g

    ENV NGINX_DIR=/usr/local/nginx \
    PHP_DIR=/usr/local/php \
    PHP56_DIR=/usr/local/php56 \
    PHP7_DIR=/usr/local/php7 \
    LOG_DIR=/var/log/app_engine \
    APP_DIR=/app \
    NGINX_USER_CONF_DIR=/etc/nginx/conf.d \
    UPLOAD_DIR=/upload \
    SESSION_SAVE_PATH=/tmp/sessions \
    OPENSSL_VERSION=1.0.1p \
    PATH=/usr/local/php/bin:$PATH

    COPY openssl-version-script.patch /tmp/openssl-version-script.patch
    RUN mkdir /build-scripts

    COPY build-scripts/apt_build_deps.sh /build-scripts/apt_build_deps.sh
    RUN /bin/bash /build-scripts/apt_build_deps.sh install

    COPY build-scripts/import_pgp_keys.sh /build-scripts/import_pgp_keys.sh
    RUN /bin/bash /build-scripts/import_pgp_keys.sh

    ENV NGINX_VERSION=1.8.1
    COPY build-scripts/build_nginx.sh /build-scripts/build_nginx.sh
    RUN /bin/bash /build-scripts/build_nginx.sh

    ENV PHP56_VERSION=5.6.18
    COPY build-scripts/build_php56.sh /build-scripts/build_php56.sh
    RUN /bin/bash /build-scripts/build_php56.sh

    ENV PHP70_VERSION=7.0.3
    COPY build-scripts/build_php70.sh /build-scripts/build_php70.sh
    RUN /bin/bash /build-scripts/build_php70.sh

    RUN /bin/bash /build-scripts/apt_build_deps.sh uninstall

    EXPOSE 8080

    RUN mkdir -p $APP_DIR $LOG_DIR $UPLOAD_DIR $SESSION_SAVE_PATH \
        $NGINX_USER_CONF_DIR \
    && chown -R www-data.www-data \
        $APP_DIR $UPLOAD_DIR $SESSION_SAVE_PATH $LOG_DIR \
        $NGINX_USER_CONF_DIR \
    && chmod 755 $UPLOAD_DIR $SESSION_SAVE_PATH

    COPY nginx.conf fastcgi_params gzip_params $NGINX_DIR/conf/
    COPY php.ini $PHP56_DIR/lib/php.ini
    COPY php.ini $PHP7_DIR/lib/php.ini
    COPY php-fpm.conf $PHP56_DIR/etc/php-fpm.conf
    COPY php-fpm.conf $PHP7_DIR/etc/php-fpm.conf
    COPY supervisord.conf /etc/supervisor/supervisord.conf
    COPY logrotate.app_engine /etc/logrotate.d/app_engine

    COPY entrypoint.sh /entrypoint.sh
    RUN chmod +x /entrypoint.sh

    COPY composer.sh /composer.sh
    RUN chmod +x /composer.sh

    COPY detect_php_version.php /tmp/detect_php_version.php
    RUN cd /tmp && ${PHP_DIR}/bin/php \
        -d suhosin.executor.include.whitelist=phar \
        -d suhosin.executor.func.blacklist=none \
        /usr/local/bin/composer \
        require composer/semver

    ONBUILD COPY . $APP_DIR
    ONBUILD RUN chmod -R 550 $APP_DIR
    ONBUILD RUN chown -R www-data.www-data $APP_DIR

    WORKDIR $APP_DIR

    ONBUILD RUN /composer.sh
    ONBUILD RUN touch $LOG_DIR/suhosin.log
    ONBUILD RUN chown www-data.www-data $LOG_DIR/suhosin.log

    ENTRYPOINT ["/entrypoint.sh"]
    CMD ["/usr/bin/supervisord"]

Let's look at the layers for php-nginx.

    sudo docker pull gcr.io/php-mvm-a/php-nginx:latest

    sudo docker history gcr.io/php-mvm-a/php-nginx
    IMAGE               CREATED             CREATED BY                                      SIZE
    058534bf7f14        15 hours ago        /bin/sh -c #(nop) CMD ["/usr/bin/supervisord"   0 B
    dbaac6fd6b10        15 hours ago        /bin/sh -c #(nop) ENTRYPOINT ["/entrypoint.sh   0 B
    6a950ec26609        15 hours ago        /bin/sh -c #(nop) ONBUILD RUN chown www-data.   0 B
    c59213c2eebb        15 hours ago        /bin/sh -c #(nop) ONBUILD RUN touch $LOG_DIR/   0 B
    56f0ddf1454b        15 hours ago        /bin/sh -c #(nop) ONBUILD RUN /composer.sh      0 B
    2a23c1c1b3eb        15 hours ago        /bin/sh -c #(nop) WORKDIR /app                  0 B
    d4b223a9764b        15 hours ago        /bin/sh -c #(nop) ONBUILD RUN chown -R www-da   0 B
    f942c9822726        15 hours ago        /bin/sh -c #(nop) ONBUILD RUN chmod -R 550 $A   0 B
    dff80ab16e4f        15 hours ago        /bin/sh -c #(nop) ONBUILD COPY . $APP_DIR       0 B
    02a1f8bc0c58        15 hours ago        /bin/sh -c cd /tmp && ${PHP_DIR}/bin/php        9.307 MB
    a47be3b30687        15 hours ago        /bin/sh -c #(nop) COPY file:f51485401828348ec   1.447 kB
    19645d4b77ad        15 hours ago        /bin/sh -c chmod +x /composer.sh                3.192 kB
    fa17a450a315        15 hours ago        /bin/sh -c #(nop) COPY file:0c71c8b9cf41aed7d   3.192 kB
    b85ed6d71e3f        15 hours ago        /bin/sh -c chmod +x /entrypoint.sh              2.087 kB
    f40e48da5a9d        15 hours ago        /bin/sh -c #(nop) COPY file:e2efd7c41704151b1   2.087 kB
    c8d78cc49a4d        15 hours ago        /bin/sh -c #(nop) COPY file:700e9200883429de4   799 B
    e949340e6e4b        15 hours ago        /bin/sh -c #(nop) COPY file:74e1f41f42af0aeba   1.416 kB
    244752f239c8        15 hours ago        /bin/sh -c #(nop) COPY file:f023e11cccbc4c705   23.26 kB
    f3a3b532f0a8        15 hours ago        /bin/sh -c #(nop) COPY file:f023e11cccbc4c705   23.26 kB
    df610194d2b3        15 hours ago        /bin/sh -c #(nop) COPY file:62a5b8d47c12e39ab   73.83 kB
    e3481acdad61        15 hours ago        /bin/sh -c #(nop) COPY file:62a5b8d47c12e39ab   73.83 kB
    0f36a3d74430        15 hours ago        /bin/sh -c #(nop) COPY multi:f7f4dd89355444bf   4.701 kB
    408f9e270e04        15 hours ago        /bin/sh -c mkdir -p $APP_DIR $LOG_DIR $UPLOAD   0 B
    7a8a1fc206b7        15 hours ago        /bin/sh -c #(nop) EXPOSE 8080/tcp               0 B
    c034391cd78e        15 hours ago        /bin/sh -c /bin/bash /build-scripts/apt_build   650.7 kB
    21c59dc78a03        15 hours ago        /bin/sh -c /bin/bash /build-scripts/build_php   57.89 MB
    04e3458a8805        15 hours ago        /bin/sh -c #(nop) COPY file:ee18ef78042073900   3.872 kB
    10cee5bfacc7        15 hours ago        /bin/sh -c #(nop) ENV PHP70_VERSION=7.0.3       0 B
    42590eb1e1f3        15 hours ago        /bin/sh -c /bin/bash /build-scripts/build_php   58.54 MB
    669edccdf998        15 hours ago        /bin/sh -c #(nop) COPY file:b91633c41bae4d1fa   3.887 kB
    53f50e65a6ab        15 hours ago        /bin/sh -c #(nop) ENV PHP56_VERSION=5.6.18      0 B
    7af5a8ba1778        42 hours ago        /bin/sh -c /bin/bash /build-scripts/build_ngi   3.348 MB
    3ff71ec97f9f        42 hours ago        /bin/sh -c #(nop) COPY file:9e86458080c3fa814   1.287 kB
    9b60ddc956bb        42 hours ago        /bin/sh -c #(nop) ENV NGINX_VERSION=1.8.1       0 B
    0eda4c392ff3        42 hours ago        /bin/sh -c /bin/bash /build-scripts/import_pg   79.43 kB
    09ba8abc48da        42 hours ago        /bin/sh -c #(nop) COPY file:fab8050e7e5284400   1.032 kB
    db2af62cc814        42 hours ago        /bin/sh -c /bin/bash /build-scripts/apt_build   241.5 MB
    ec77c1af9ef3        42 hours ago        /bin/sh -c #(nop) COPY file:73841794fd2ed759c   1.739 kB
    41ad28903a1b        42 hours ago        /bin/sh -c mkdir /build-scripts                 0 B
    288f37760c98        42 hours ago        /bin/sh -c #(nop) COPY file:3a420aa9c53ae0cd1   110.7 kB
    331d1eb28211        42 hours ago        /bin/sh -c #(nop) ENV NGINX_DIR=/usr/local/ng   0 B
    ae64cc35933c        42 hours ago        /bin/sh -c apt-get update && apt-get install    192.4 MB
    ------------------------------------------------------------------------------------------------
    183e1ca00401        10 days ago         /bin/sh -c apt-get -q update &&     apt-get i   75.62 MB
    31f8b8307171        10 days ago         /bin/sh -c #(nop) ENV PORT=8080                 0 B
    ecf7eb0a5fb6        10 days ago         /bin/sh -c #(nop) ENV DEBIAN_FRONTEND=noninte   0 B
    f55a937a54ad        10 days ago         /bin/sh -c #(nop) CMD []                        0 B
    8f8068a6a6b4        6 months ago        /bin/sh -c #(nop) ADD file:6958cfcdc7475df054   163.5 MB
    559718b5f880        6 months ago        /bin/sh -c #(nop) ENV PATH=/usr/local/sbin:/u   0 B
    643a001c5ee0        6 months ago        /bin/sh -c #(nop) ENV DEBIAN_FRONTEND=noninte   0 B

We can see that each instruction in the Dockerfile has a corresponding layer in the image.
The layers below the dotted line are from the base image, gcr.io/google_appengine/base.

Let's look at how PHP gets installed in the image.

    ENV PHP56_VERSION=5.6.18
    COPY build-scripts/build_php56.sh /build-scripts/build_php56.sh
    RUN /bin/bash /build-scripts/build_php56.sh

We copy a script from ./build-scripts to /build-scripts in the image.
Then we run the script.

Here's the script.

    # A shell script for installing PHP 5.6.
    set -xe

    PHP_SRC=/usr/src/php

    curl -SL "http://php.net/get/php-$PHP56_VERSION.tar.gz/from/this/mirror" -o php.tar.gz
    curl -SL "http://us2.php.net/get/php-$PHP56_VERSION.tar.gz.asc/from/this/mirror" -o php.tar.gz.asc
    gpg --verify php.tar.gz.asc
    mkdir -p ${PHP_SRC}
    tar -zxf php.tar.gz -C ${PHP_SRC} --strip-components=1
    rm php.tar.gz
    rm php.tar.gz.asc
    ....
    pushd /usr/src/php
    rm -f configure
    ./buildconf --force
    ./configure \
    --prefix=$PHP56_DIR \
    --with-config-file-scan-dir=$APP_DIR:${PHP56_DIR}/lib/conf.d \
    --disable-cgi \
    --disable-memcached-sasl \
    --enable-apcu \
    --enable-bcmath=shared \
    --enable-calendar=shared \
    --enable-exif=shared \
    --enable-fpm \
    ...
    # Install shared extensions
    ${PHP56_DIR}/bin/pecl install memcache
    ${PHP56_DIR}/bin/pecl install mongodb
    ${PHP56_DIR}/bin/pecl install redis

    rm -rf /tmp/pear

    # Install composer
    curl -sS https://getcomposer.org/installer | \
    ${PHP56_DIR}/bin/php -- \
    --install-dir=/usr/local/bin \
    --filename=composer

We see that PHP is installed by getting the source code, configuring, and building.
Extensions like mongodb are installed by pecl.
Composer is installed by a curl command.

Another look at the layers in gcr.io/google_appengine/base (058534bf7f14).

    sudo docker run gcr.io/php-mvm-a/php-nginx:latest

In another terminal,

    cd /var/lib/docker/aufs/layers

    cat 058534bf7f14 TAB


Another look at the layers of a running container

http://blog.thoward37.me/articles/where-are-docker-images-stored/

    

    
    






















Get general Docker information:

    sudo docker info

    Containers: 6
    Images: 174
    Storage Driver: aufs
     Root Dir: /var/lib/docker/aufs
     Backing Filesystem: extfs
     Dirs: 186
     Dirperm1 Supported: true
    Execution Driver: native-0.2
    Kernel Version: 3.16.0-0.bpo.4-amd64
    Operating System: Debian GNU/Linux 7 (wheezy)
    CPUs: 1
    Total Memory: 1.664 GiB
    Name: gae-default-20151201t210516-bl1r
    ID: 7RCR:Y3WM:U5XZ:L3IL:U6B5:TUT5:KOWR:2W5F:ODWI:LCGR:3JOK:43O5
    WARNING: No swap limit support

Start a shell in a Docker container:

    sudo docker exec -i -t da83e0377870 bash

Look around. Notice that the container has its own environment variables and
its own view of the directory tree:

    root@da83e0377870:/app# ls
    app.js  app.yaml  books  config.js  node_modules  out.txt  package.json  tmp5lvsU6  views

    root@da83e0377870:/app# cd /
    root@da83e0377870:/# ls
    app  bin  boot  dev  etc  home  lib  lib64  media  mnt  nodejs  opt  proc  root  run  sbin  srv  sys  tmp  usr  var


    root@da83e0377870:/# printenv
    APPENGINE_LOADBALANCER=
    MEMCACHE_PORT_11211_TCP_PROTO=tcp
    HOSTNAME=da83e0377870
    GAE_APPENGINE_HOSTNAME=steve-41.appspot.com
    MEMCACHE_NAME=/gaeapp/memcache
    GAE_MODULE_NAME=default
    MEMCACHE_PORT_11211_TCP_ADDR=172.17.0.3
    GAE_AFFINITY=true
    GAE_LONG_APP_ID=steve-41
    GAE_MODULE_VERSION=20151201t210516
    GAE_MODULE_INSTANCE=1
    MEMCACHE_PORT_11211_TCP_PORT=11211
    APPENGINE_LOADBALANCER_IP=

    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/nodejs/bin
    ...
    GAE_VM=true
    ...
    PORT=8080
    ...

    root@da83e0377870:/# echo $JAVA_HOME

    root@da83e0377870:/# 

Here are the nodejs and npm executables:

    cd /nodejs
    root@da83e0377870:/nodejs# ls
    CHANGELOG.md  LICENSE  README.md  bin  etc  include  lib  share
    root@da83e0377870:/nodejs# cd bin
    root@da83e0377870:/nodejs/bin# ls
    node  npm


    root@da83e0377870:/nodejs/lib/node_modules/npm# node -v
    v4.2.1
    root@da83e0377870:/nodejs/lib/node_modules/npm# npm -v
    2.14.7

Look at the process tree, in this container:

    ps auxf
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root        26  0.0  0.1  20220  3200 ?        Ss   23:32   0:00 bash
    root       133  0.0  0.1  17484  2168 ?        R+   23:38   0:00  \_ ps auxf
    root         1  0.0  0.0   4324   640 ?        Ss   22:02   0:00 /bin/sh -c npm start
    root         9  0.0  2.2 1075908 39904 ?       Sl   22:02   0:00 npm
    root        20  0.0  0.0   4328   756 ?        S    22:02   0:00  \_ sh -c node app.js
    root        21  0.0  2.1 932644 37448 ?        Sl   22:02   0:02      \_ node app.js

Install GDB:

    apt-get update
    apt-get install gdb

Here's another way to find the process ID (PID), in this container, of the Node.js executable:

    ps -C node
    PID TTY          TIME CMD
     20 ?        00:00:01 node

Attach GDB to the Node.js process:

    gdb -p 20

Look at the modules that are mapped into the process address space:

    info proc mappings
 
    process 20
    Mapped address spaces:
          Start Addr           End Addr       Size     Offset objfile
            0x400000          0x16cd000  0x12cd000        0x0 /nodejs/bin/node
           0x18cc000          0x18e5000    0x19000  0x12cc000 /nodejs/bin/node
           0x18e5000          0x18f7000    0x12000        0x0 
           0x1df8000          0x201e000   0x226000        0x0 [heap]
           ...
      0x7f76e7d63000     0x7f76e7d64000     0x1000     0x3000 /lib/x86_64-linux-gnu/libdl-2.19.so
      0x7f76e7d64000     0x7f76e7d84000    0x20000        0x0 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f78000     0x7f76e7f7e000     0x6000        0x0 
      0x7f76e7f82000     0x7f76e7f84000     0x2000        0x0 
      0x7f76e7f84000     0x7f76e7f85000     0x1000    0x20000 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f85000     0x7f76e7f86000     0x1000    0x21000 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f86000     0x7f76e7f87000     0x1000        0x0 
      0x7fffbb9ed000     0x7fffbba0f000    0x22000        0x0 [stack]
      0x7fffbba7d000     0x7fffbba7e000     0x1000        0x0 [vdso]
      0x7fffbba7e000     0x7fffbba80000     0x2000        0x0 [vvar]
      0xffffffffff600000 0xffffffffff601000     0x1000        0x0 [vsyscall]

Look at the threads in the process:
TODO: Understand why there are 5 threads.

    (gdb) info threads
      Id   Target Id         Frame 
      5    Thread 0x7f0550996700 (LWP 22) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      4    Thread 0x7f0550195700 (LWP 23) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      3    Thread 0x7f054f994700 (LWP 24) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      2    Thread 0x7f054f193700 (LWP 25) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
    * 1    Thread 0x7f0551da0740 (LWP 21) "node" syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38

Look at the call stack for the current thread:

    (gdb) back
    #0  syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38
    #1  0x0000000000fbc88a in uv__epoll_wait (epfd=<optimized out>, ... ./deps/uv/src/unix/linux-syscalls.c:321
    #2  0x0000000000fba748 in uv__io_poll (loop=loop@entry=0x18f3b00 ... ../deps/uv/src/unix/linux-core.c:243
    #3  0x0000000000fabce6 in uv_run (loop=0x18f3b00 ... ../deps/uv/src/unix/core.c:341
    #4  0x0000000000dfaa50 in node::Start(int, char**) ()
    #5  0x00007f05509b8b45 in __libc_start_main ... libc-start.c:287
    #6  0x00000000007222fd in _start ()


New Experiment

Deploy the PHP Hello World app. SSH into a Managed VM instance.

    sudo docker ps
    CONTAINER ID        IMAGE
    ...
    60786d1e1c82        appengine.gcr.io/389014331831046975/steve-43.default.20151204t010509:latest
    ...
    stevepe@gae-default-20151204t010509-ikhg:/$ sudo docker exec -i -t 60786d1e1c82 bash

    root@60786d1e1c82:/app# ps auxf
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root        68  0.0  0.1  20220  3052 ?        Ss   19:09   0:00 bash
    root        74  0.0  0.1  17484  2076 ?        R+   19:10   0:00  \_ ps auxf
    root         1  0.0  0.8  55596 15540 ?        Ss   07:08   0:07 /usr/bin/python /usr/bin/supervisord
    root        17  0.0  1.0 166456 18576 ?        S    07:08   0:01 php-fpm: master process (/usr/local/php56/etc/php-fpm.conf)
    www-data    21  0.2  0.7 166576 12604 ?        S    07:08   1:49  \_ php-fpm: pool app
    www-data    22  0.2  0.7 166576 12544 ?        S    07:08   1:49  \_ php-fpm: pool app
    www-data    29  0.2  0.7 166576 12540 ?        S    08:49   1:28  \_ php-fpm: pool app
    root        18  0.0  0.1  28636  3132 ?        S    07:08   0:00 nginx: master process /usr/local/nginx/sbin/nginx
    www-data    20  0.0  0.2  30312  3896 ?        S    07:08   0:05  \_ nginx: worker process
    root        19  0.0  0.1  25888  2272 ?        S    07:08   0:00 /usr/sbin/cron -f

In the container, php-fpm has PID = 17, and nginx has PID = 18.

Back out of the container:

    exit

    ps auxf
    ...
    root      3406  0.0  0.8  55596 15540 ?        Ss   07:08   0:07  \_ /usr/bin/python /usr/bin/supervisord
    root      3568  0.0  1.0 166456 18576 ?        S    07:08   0:01  |   \_ php-fpm: master process (/usr/local/php56/etc/php-fpm.conf)
    www-data  3600  0.2  0.7 166576 12604 ?        S    07:08   1:50  |   |   \_ php-fpm: pool app
    www-data  3601  0.2  0.7 166576 12544 ?        S    07:08   1:49  |   |   \_ php-fpm: pool app
    www-data  4339  0.2  0.7 166576 12540 ?        S    08:49   1:29  |   |   \_ php-fpm: pool app
    root      3570  0.0  0.1  28636  3132 ?        S    07:08   0:00  |   \_ nginx: master process /usr/local/nginx/sbin/nginx
    www-data  3590  0.0  0.2  30312  3896 ?        S    07:08   0:06  |   |   \_ nginx: worker process
    ...

Here in the root (namespace?), php-fpm has PID=3568, and nginx has PID=3570.

Look at the namespace identifiers for the two processes:

    root@gae-default-20151204t010509-ikhg:/proc/3568/ns# ls -l
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 ipc -> ipc:[4026532276]
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 mnt -> mnt:[4026532274]
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 net -> net:[4026532279]
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 pid -> pid:[4026532277]
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 user -> user:[4026531837]
    lrwxrwxrwx 1 root root 0 Dec  4 19:01 uts -> uts:[4026532275]

    root@gae-default-20151204t010509-ikhg:/proc/3570/ns# ls -l
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 ipc -> ipc:[4026532276]
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 mnt -> mnt:[4026532274]
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 net -> net:[4026532279]
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 pid -> pid:[4026532277]
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 user -> user:[4026531837]
    lrwxrwxrwx 1 root root 0 Dec  4 19:02 uts -> uts:[4026532275]

We see that all of the namespaces for the two processes are the same. So these
two processes have the same view of the directory tree, the network, the list of processes,
the list of users, the ipc stuff (investigate), and the uts stuff (investigate).

References

* [Namespaces](http://unix.stackexchange.com/questions/113530/how-to-find-out-namespace-of-a-particular-process)






