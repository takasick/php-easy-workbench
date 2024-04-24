FROM docker.io/library/php:8.3-apache-bookworm

RUN apt update \
    && apt install -y supervisor rsyslog cron \
    \
    && apt install -y git \
    \
    && apt install -y libzip-dev unzip \
    && docker-php-ext-install zip \
    \
    && apt install -y libbz2-dev \
    && docker-php-ext-install bz2 \
    \
    && docker-php-ext-install pdo pdo_mysql mysqli \
    \
    && apt install -y libzip-dev libpng-dev libwebp-dev libjpeg62-turbo-dev libxpm-dev libfreetype-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp --with-xpm \
    && docker-php-ext-install gd \
    \
    && docker-php-ext-install bcmath \
    \
    && docker-php-ext-install exif \
    \
    && apt install -y libicu-dev \
    && docker-php-ext-install intl

# supervisor
RUN { \
        echo '[supervisord]'; \
        echo 'nodaemon=true'; \
        echo 'user=root'; \
        echo ''; \
        echo '[program:syslog]'; \
        echo 'command=/usr/sbin/rsyslogd -n'; \
        echo 'stdout_logfile=/dev/stdout'; \
        echo 'stdout_logfile_maxbytes=0'; \
        echo ''; \
        echo '[program:cron]'; \
        echo 'command=/usr/sbin/cron -f -L 15'; \
        echo ''; \
        echo '[program:apache2]'; \
        echo 'command=/usr/local/bin/apache2-foreground'; \
        echo 'stdout_logfile=/dev/stdout'; \
        echo 'stdout_logfile_maxbytes=0'; \
        echo 'stderr_logfile=/dev/stderr'; \
        echo 'stderr_logfile_maxbytes=0'; \
    } > /etc/supervisor/conf.d/supervisord.conf

# rsyslog
RUN sed -i -r \
      -e 's!^(module\(load="imklog"\) )!#\1 !' \
      -e 's!^(\*\.\*;auth,authpriv\.none)\t!\1;cron.none\t!' \
      -e 's!-/var/log/syslog!/dev/stdout!' \
      -e 's!/var/log/auth\.log!/dev/null!' \
      -e 's!-/var/log/cron\.log!/dev/stdout!' \
      /etc/rsyslog.conf

# PHP
RUN mv ${PHP_INI_DIR}/php.ini-development ${PHP_INI_DIR}/php.ini

## composer
COPY --from=docker.io/composer/composer:latest-bin /composer /usr/bin/composer

## phpmyadmin
RUN export COMPOSER_ALLOW_SUPERUSER=1; composer create-project phpmyadmin/phpmyadmin --no-dev /usr/share/phpmyadmin \
    && mkdir /etc/phpmyadmin \
    && { \
        echo '<?php'; \
        echo ''; \
        echo 'declare(strict_types=1);'; \
        echo ''; \
        /bin/echo -e '$cfg[\x27blowfish_secret\x27] = \x27\x27;'; \
        echo ''; \
        echo '$i = 0;'; \
        echo ''; \
        echo '$i++;'; \
        /bin/echo -e '$cfg[\x27Servers\x27][$i][\x27auth_type\x27] = \x27cookie\x27;'; \
        /bin/echo -e '$cfg[\x27Servers\x27][$i][\x27host\x27] = \x27mysql\x27;'; \
        /bin/echo -e '$cfg[\x27Servers\x27][$i][\x27compress\x27] = false;'; \
        /bin/echo -e '$cfg[\x27Servers\x27][$i][\x27AllowNoPassword\x27] = true;'; \
        echo ''; \
        /bin/echo -e '$cfg[\x27UploadDir\x27] = \x27\x27;'; \
        /bin/echo -e '$cfg[\x27SaveDir\x27] = \x27\x27;'; \
        /bin/echo -e '$cfg[\x27TempDir\x27] = \x27/var/lib/phpmyadmin/tmp\x27;'; \
       } > /etc/phpmyadmin/config.inc.php \
    && ln -sf /etc/phpmyadmin/config.inc.php /usr/share/phpmyadmin/config.inc.php \
    && mkdir -p /var/lib/phpmyadmin/tmp \
    && chown -R www-data:www-data /var/lib/phpmyadmin/tmp

# httpd
RUN a2enmod rewrite \
    && sed -i -r \
        -e 's!^(</VirtualHost>)!Alias /phpMyAdmin /usr/share/phpmyadmin\n<Directory /usr/share/phpmyadmin>\nRequire all granted\n</Directory>\n\1!' \
        /etc/apache2/sites-enabled/000-default.conf

# Run
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
