FROM debian:12-slim

LABEL maintainer="dane@daneeveritt.com" \
      description="Local development environment for Pterodactyl Panel." \
      org.opencontainers.image.source=https://github.com/YunHerry/development

ENV DEBIAN_FRONTEND noninteractive
ENV LANG=C.UTF-8

USER root

# 步骤1: 安装基础工具
RUN apt -y update \
    && apt -y install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
        software-properties-common \
        apt-transport-https \
        sudo

# 步骤2: 添加 Node.js 18 源
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_18.x $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/nodesource.list

# 步骤3: 添加 Yarn 源
RUN curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor -o /etc/apt/keyrings/yarn.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# 步骤4: 添加 PHP 源（使用官方方式）
RUN curl -sSL https://packages.sury.org/php/README.txt \
    && curl -sSL https://packages.sury.org/php/apt.gpg | gpg --dearmor -o /etc/apt/keyrings/php.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/php.gpg] https://packages.sury.org/php/ $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/php.list

# 步骤5: 更新包列表并安装系统包
RUN apt -y update \
    && apt -y --no-install-recommends install \
        locales \
        iproute2 \
        iputils-ping \
        nginx \
        tar \
        unzip \
        git \
        supervisor \
        cron \
        nano

# 步骤6: 安装 Node.js 和 Yarn
RUN apt -y update \
    && apt -y install \
        nodejs \
        yarn

# 步骤7: 安装 PHP 和扩展
RUN apt -y update \
    && apt -y install \
        php8.2 \
        php8.2-cli \
        php8.2-common \
        php8.2-gd \
        php8.2-mysql \
        php8.2-mbstring \
        php8.2-bcmath \
        php8.2-xml \
        php8.2-fpm \
        php8.2-curl \
        php8.2-zip \
        php8.2-xdebug

# 步骤8: 安装 Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# 步骤9: 清理
RUN apt autoremove -y \
    && rm -rf /var/lib/apt/lists/*

# 用户和配置设置
COPY entrypoint /usr/local/bin/entrypoint
COPY setup-pterodactyl /usr/local/bin/setup-pterodactyl
COPY configs/supervisord.conf /etc/supervisord.conf
COPY configs/pterodactyl.conf /etc/nginx/sites-available/pterodactyl.conf

RUN useradd -m pterodactyl \
    && usermod -a -g www-data -G sudo pterodactyl \
    && echo "%sudo ALL=(ALL:ALL) NOPASSWD: ALL" | tee /etc/sudoers.d/sudoers \
    && chmod +x /usr/local/bin/setup-pterodactyl \
    && chmod +x /usr/local/bin/entrypoint \
    && rm -rf /etc/nginx/sites-enabled/* \
    && ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/ \
    && phpdismod -s cli xdebug \
    && mkdir -p /var/run/php \
    && mkdir -p /var/www/html \
    && (crontab -l 2>/dev/null; echo "* * * * * php /var/www/html/artisan schedule:run >> /dev/null 2>&1") | crontab -

WORKDIR /var/www/html

EXPOSE 80
EXPOSE 8080

ENTRYPOINT ["entrypoint"]
