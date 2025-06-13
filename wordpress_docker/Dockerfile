FROM debian:stretch

ENV DEBIAN_FRONTEND=noninteractive


# Fixer les sources pour l'archive Debian, supprimer stretch-updates et désactiver la vérif de validité
RUN sed -i '/stretch-updates/d' /etc/apt/sources.list && \
    sed -i 's|http://deb.debian.org/debian|http://archive.debian.org/debian|g' /etc/apt/sources.list && \
    sed -i 's|http://security.debian.org/debian-security|http://archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until

# Installer les dépendances nécessaires à la compilation
RUN apt-get update && apt-get install -y \
    build-essential wget curl unzip bzip2 \
    libxml2-dev libcurl4-openssl-dev \
    libssl-dev libsqlite3-dev libpng-dev libjpeg-dev \
    libmariadb-dev-compat libmariadb-dev libzip-dev \
    libonig-dev pkg-config apache2-utils \
    libpcre3-dev \
    nano \
    logrotate

RUN apt-get update && apt-get install -y perl

# Copier les sources téléchargées dans l'image
COPY httpd-2.4.29.tar.gz /tmp/
COPY apr-1.5.2.tar.gz /tmp/
COPY apr-util-1.5.4.tar.gz /tmp/
COPY php-7.2.15.tar.bz2 /tmp/
COPY wordpress-4.9.8.zip /tmp/
COPY phpMyAdmin-4.8.1-all-languages.zip /tmp/

RUN cd /tmp && \
    tar xzf httpd-2.4.29.tar.gz && \
    tar xzf apr-1.5.2.tar.gz && \
    tar xzf apr-util-1.5.4.tar.gz && \
    mv apr-1.5.2 httpd-2.4.29/srclib/apr && \
    mv apr-util-1.5.4 httpd-2.4.29/srclib/apr-util && \
    cd httpd-2.4.29 && \
    ./configure --enable-so --enable-rewrite --with-included-apr && \
    make -j"$(nproc)" && \
    make install && \
    cd / && rm -rf /tmp/httpd-2.4.29* /tmp/apr-1.5.2* /tmp/apr-util-1.5.4*

RUN ls -l /usr/local/apache2/bin/

# Compiler et installer PHP
RUN cd /tmp && \
    tar xjf php-7.2.15.tar.bz2 && \
    cd php-7.2.15 && \
    ./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysqli --with-pdo-mysql \
    --with-curl --with-xmlrpc --with-zlib --with-openssl --enable-mbstring \
    --with-gd --with-zip --with-gettext && \
    make -j"$(nproc)" && \
    make install && \
    cp php.ini-development /usr/local/lib/php.ini && \
    cd / && rm -rf /tmp/php-7.2.15*

# Installer WordPress
RUN unzip /tmp/wordpress-4.9.8.zip -d /tmp/ && \
    mv /tmp/wordpress /usr/local/apache2/htdocs && \
    rm /tmp/wordpress-4.9.8.zip

# Installer phpMyAdmin
RUN unzip /tmp/phpMyAdmin-4.8.1-all-languages.zip -d /tmp/ && \
    mv /tmp/phpMyAdmin-4.8.1-all-languages /usr/local/apache2/htdocs/phpmyadmin && \
    rm /tmp/phpMyAdmin-4.8.1-all-languages.zip

# Droits
RUN chown -R www-data:www-data /usr/local/apache2/htdocs

# Correction DirectoryIndex et handlers PHP (si besoin)
RUN sed -i 's/DirectoryIndex index.html/DirectoryIndex index.php index.html/' /usr/local/apache2/conf/httpd.conf && \
    echo 'AddType application/x-httpd-php .php' >> /usr/local/apache2/conf/httpd.conf && \
    echo 'AddHandler application/x-httpd-php .php' >> /usr/local/apache2/conf/httpd.conf

# Config phpMyAdmin pour host Docker MySQL
RUN echo "<?php \
\$cfg['Servers'][1]['host'] = 'mysql-db'; \
\$cfg['Servers'][1]['user'] = 'wordpress_user'; \
\$cfg['Servers'][1]['password'] = 'wordpress_pass'; \
\$cfg['Servers'][1]['auth_type'] = 'cookie'; \
" > /usr/local/apache2/htdocs/phpmyadmin/config.inc.php

#inshalla
RUN echo "<?php\n/**\n * The base configuration for WordPress\n *\n * The wp-config.php creation script uses this file during the\n * installation. You don't have to use the web site, you can\n * copy this file to \"wp-config.php\" and fill in the values.\n *\n * This file contains the following configurations:\n *\n * * MySQL settings\n * * Secret keys\n * * Database table prefix\n * * ABSPATH\n *\n * @link https://codex.wordpress.org/Editing_wp-config.php\n *\n * @package WordPress\n */\n\n// ** MySQL settings - You can get this info from your web host ** //\n/** The name of the database for WordPress */\ndefine('DB_NAME', 'wordpress_db');\n\n/** MySQL database username */\ndefine('DB_USER', 'wordpress_user');\n\n/** MySQL database password */\ndefine('DB_PASSWORD', 'wordpress_pass');\n\n/** MySQL hostname */\ndefine('DB_HOST', 'mysql-db');\n\n/** Database Charset to use in creating database tables. */\ndefine('DB_CHARSET', 'utf8mb4');\n\n/** The Database Collate type. Don't change this if in doubt. */\ndefine('DB_COLLATE', '');\n\n/**#@+\n * Authentication Unique Keys and Salts.\n *\n * Change these to different unique phrases!\n * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}\n * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.\n *\n * @since 2.6.0\n */\ndefine('AUTH_KEY',         'o$,Qyu<Q!oF<|BkZ36n8T,~sF5a-JXwd^J;2DKC/F>9!H?0Zr>)/iW[I5[O6Q(OD');\ndefine('SECURE_AUTH_KEY',  '7uy!{gF:44=K>v)|Mi82*{u+fl{F.M1z;X(rA8)wQVserNwTA~ZPXjcPO>r*~P3g');\ndefine('LOGGED_IN_KEY',    'eV{1M rG[Znw~oLD>Yk=8_H+Mc;eZhg0Ij^9A3LWo}_:GBaq1QBOl/Wk,c||.RSr');\ndefine('NONCE_KEY',        'l&[mp[? P`7XKU1DKi=Dcw,.?R~L}>w7w[ngYYyi%oh1x=}tK[?2YV(AKwT.yhM$');\ndefine('AUTH_SALT',        '4Y:e>XftCP}QBauz d&A5oh`8]j{Sp8(d,8?BxP9w(.Q /;QD6eH W~Ri)MQ4-ya');\ndefine('SECURE_AUTH_SALT', 'Sx`~P;gRQx>mnaJI:9&vzcUds?85)s({8Q$[**b0?:%rJ8M#&[b=Q~;uQz0cPCi ');\ndefine('LOGGED_IN_SALT',   '>h;y#ZIP9?^,:dTml;KIA.kcC(Ru-lu5B[5yM-7;J_#Vyy* L?/~Ky2sYBu}0 E>');\ndefine('NONCE_SALT',       'y>&}MX1qE+|7LZ+`q;}sdUIen+FX^q!/J>@-XdOqbTqu.f;0sd.z>c>9ZXuJQq5f');\n/**#@-*/\n\n/**\n * WordPress Database Table prefix.\n *\n * You can have multiple installations in one database if you give each\n * a unique prefix. Only numbers, letters, and underscores please!\n */\n\$table_prefix  = 'wp_';\n\n/**\n * For developers: WordPress debugging mode.\n *\n * Change this to true to enable the display of notices during development.\n * It is strongly recommended that plugin and theme developers use WP_DEBUG\n * in their development environments.\n *\n * For information on other constants that can be used for debugging,\n * visit the Codex.\n *\n * @link https://codex.wordpress.org/Debugging_in_WordPress\n */\ndefine('WP_DEBUG', false);\n\n/* That's all, stop editing! Happy blogging. */\n\n/** Absolute path to the WordPress directory. */\nif ( !defined('ABSPATH') )\n\tdefine('ABSPATH', dirname(__FILE__) . '/');\n\n/** Sets up WordPress vars and included files. */\nrequire_once(ABSPATH . 'wp-settings.php');\n" > /usr/local/apache2/htdocs/wordpress/wp-config.php

# Exposer le port 8080
EXPOSE 80

# Lancer Apache au démarrage
CMD ["/usr/local/apache2/bin/httpd", "-D", "FOREGROUND", "-k", "start", "-f", "/usr/local/apache2/conf/httpd.conf", "-C", "Listen 8080"]
