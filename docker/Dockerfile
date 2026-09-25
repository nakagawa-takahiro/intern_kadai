FROM php:7.3-apache

# php.iniをコピー
COPY ./docker/php.ini /usr/local/etc/php/

# Debian 11 (bullseye) は2026/8/31にLTSが終了し、security.debian.org のパッケージが
# アーカイブへ移行中のため取得できない。security リポジトリを外し、期限切れチェックを無効化する。
RUN sed -i '/security\.debian\.org/d' /etc/apt/sources.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until

# 必要なパッケージと拡張機能をインストール
RUN apt-get update && \
    apt-get install -y \
    git \
    unzip \
    zlib1g-dev \
    libgmp-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libgd-dev \
    cron \
    vim \
    libonig-dev \
    libzip-dev \
    locales && \
    ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h && \
    docker-php-ext-configure gd --with-freetype-dir --with-jpeg-dir && \
    docker-php-ext-install mbstring pdo_mysql mysqli zip gd gmp bcmath

# 日本語ロケールの生成
RUN locale-gen ja_JP.UTF-8 && \
    echo "LANG=ja_JP.UTF-8" > /etc/default/locale && \
    echo "LANGUAGE=ja_JP:ja" >> /etc/default/locale && \
    echo "LC_ALL=ja_JP.UTF-8" >> /etc/default/locale

# Apacheのユーザーとグループの権限を更新
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

# Apacheモジュールを有効化
RUN a2enmod rewrite headers

# DocumentRootを変更し、環境変数を設定
RUN DEFAULT_SITE_FILE=/etc/apache2/sites-enabled/000-default.conf && \
    TMP=$(mktemp) && \
    sed 's!/var/www/html!/var/www/html/my_fuel_project/public!' $DEFAULT_SITE_FILE > $TMP && \
    echo SetEnv FUEL_ENV development >> $TMP && \
    mv $TMP $DEFAULT_SITE_FILE

# FuelPHPのログディレクトリとキャッシュディレクトリを作成し、権限を設定
RUN mkdir /var/log/fuel /var/cache/fuel && \
    chmod 777 /var/log/fuel /var/cache/fuel

# Composerをインストール
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# Oilをインストール
RUN mkdir /var/www/html/my_fuel_project && \
    cd /var/www/html/my_fuel_project && \
    curl https://get.fuelphp.com/oil | sh

EXPOSE 80

# Apacheとcronをフォアグラウンドで起動
CMD ["apache2-foreground"]
