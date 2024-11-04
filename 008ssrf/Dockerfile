FROM php:7.3.8-fpm-alpine3.9

LABEL maintainer="Ric Harvey <ric@ngd.io>"

# Basic environment setup
ENV php_conf /usr/local/etc/php-fpm.conf
ENV fpm_conf /usr/local/etc/php-fpm.d/www.conf
ENV php_vars /usr/local/etc/php/conf.d/docker-vars.ini

ENV NGINX_VERSION 1.16.0
ENV LUA_MODULE_VERSION 0.10.14
ENV DEVEL_KIT_MODULE_VERSION 0.3.0
ENV LUAJIT_LIB=/usr/lib
ENV LUAJIT_INC=/usr/include/luajit-2.1

# Add iconv support
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Install build tools and wget for libiconv
RUN apk update && apk add --no-cache build-base wget

# Build and install gnu-libiconv
RUN wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.16.tar.gz && \
    tar -xzf libiconv-1.16.tar.gz && \
    cd libiconv-1.16 && \
    ./configure --prefix=/usr/local && \
    make && make install && \
    cd .. && rm -rf libiconv-1.16 libiconv-1.16.tar.gz

# Configure NGINX
RUN GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
  && CONFIG="\
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/usr/lib/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-http_xslt_module=dynamic \
    --with-http_image_filter_module=dynamic \
    --with-http_geoip_module=dynamic \
    --with-http_perl_module=dynamic \
    --with-threads \
    --with-stream \
    --with-stream_ssl_module \
    --with-stream_ssl_preread_module \
    --with-stream_realip_module \
    --with-stream_geoip_module=dynamic \
    --with-http_slice_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-compat \
    --with-file-aio \
    --with-http_v2_module \
    --add-module=/usr/src/ngx_devel_kit-$DEVEL_KIT_MODULE_VERSION \
    --add-module=/usr/src/lua-nginx-module-$LUA_MODULE_VERSION \
  " \
  && addgroup -S nginx \
  && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
  && apk add --no-cache --virtual .build-deps \
    autoconf \
    gcc \
    libc-dev \
    make \
    libressl-dev \
    pcre-dev \
    zlib-dev \
    linux-headers \
    curl \
    gnupg \
    libxslt-dev \
    gd-dev \
    geoip-dev \
    perl-dev \
    luajit-dev \
  && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
  && curl -fSL http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
  && curl -fSL https://github.com/simpl/ngx_devel_kit/archive/v$DEVEL_KIT_MODULE_VERSION.tar.gz -o ndk.tar.gz \
  && curl -fSL https://github.com/openresty/lua-nginx-module/archive/v$LUA_MODULE_VERSION.tar.gz -o lua.tar.gz \
  && export GNUPGHOME="$(mktemp -d)" \
  && gpg --keyserver keyserver.ubuntu.com --recv-keys "$GPG_KEYS" \
  && gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
  && mkdir -p /usr/src \
  && tar -zxC /usr/src -f nginx.tar.gz \
  && tar -zxC /usr/src -f ndk.tar.gz \
  && tar -zxC /usr/src -f lua.tar.gz \
  && rm nginx.tar.gz ndk.tar.gz lua.tar.gz \
  && cd /usr/src/nginx-$NGINX_VERSION \
  && ./configure $CONFIG \
  && make -j$(getconf _NPROCESSORS_ONLN) && make install \
  && apk del .build-deps \
  && rm -rf /usr/src/nginx-$NGINX_VERSION

# Install other necessary packages and PHP extensions
RUN apk add --no-cache \
    bash \
    openssh-client \
    supervisor \
    curl \
    libzip-dev \
    bzip2-dev \
    imap-dev \
    openssl-dev \
    git \
    python3 \
    py3-pip \
    libressl-dev \
    ca-certificates \
    autoconf \
    make \
    libpng-dev \
    icu-dev \
    libpq \
    libxslt-dev \
    freetype-dev \
    sqlite-dev \
    libjpeg-turbo-dev \
    postgresql-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install iconv pdo_mysql pdo_sqlite pgsql pdo_pgsql mysqli gd exif intl xsl json soap dom zip opcache \
    && pecl install xdebug-2.7.2 redis && docker-php-ext-enable xdebug redis \
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php --quiet --install-dir=/usr/bin --filename=composer \
    && rm composer-setup.php \
    && pip3 install -U pip certbot \
    && apk del autoconf make \
    && rm -rf /var/cache/apk/*

# Add necessary configuration and start scripts
COPY conf/supervisord.conf /etc/supervisord.conf
COPY conf/nginx.conf /etc/nginx/nginx.conf
COPY conf/nginx-site.conf /etc/nginx/sites-available/default.conf
COPY conf/nginx-site-ssl.conf /etc/nginx/sites-available/default-ssl.conf
RUN ln -s /etc/nginx/sites-available/default.conf /etc/nginx/sites-enabled/default.conf

# Set up PHP-FPM settings
RUN echo "cgi.fix_pathinfo=0" > ${php_vars} && \
    echo "upload_max_filesize = 100M" >> ${php_vars} && \
    echo "post_max_size = 100M" >> ${php_vars} && \
    echo "memory_limit = 128M" >> ${php_vars} && \
    sed -i \
        -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" \
        -e "s/user = www-data/user = nginx/g" \
        -e "s/group = www-data/group = nginx/g" \
        -e "s/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm.sock/g" \
        ${fpm_conf}

# Copy app files
COPY src/ /var/www/html/
COPY errors/ /var/www/errors
ADD scripts/start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 80 443
WORKDIR "/var/www/html"
CMD ["/start.sh"]
