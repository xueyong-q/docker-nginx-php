ARG PHP_VERSION
FROM php:${PHP_VERSION}

ARG USERNAME
ARG USER_UID
ARG USER_GID=${USER_UID}
ARG LANG
ENV LANG=${LANG}

# 扩展配置
COPY conf.d/ $PHP_INI_DIR/conf.d/
COPY ohmyzsh.sh /usr/bin/
COPY startService.sh /usr/bin/
COPY installExt.sh /

# 构建PHP
RUN sed -ri 's/https?:\/\/.*\.debian\.org/https:\/\/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list \
    && apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    && apt-get -y install git procps iproute2 lsb-release curl wget \
        vim zsh zip libzip-dev mlocate lsof net-tools libfreetype6-dev \
        libjpeg62-turbo-dev libpng-dev make autoconf gcc g++ libpcre3-dev \
        libpcre3 openssl libssl-dev sudo jq gnupg2 ca-certificates \
    #
    # add permissions
    && chmod +x /usr/bin/ohmyzsh.sh /installExt.sh /usr/bin/startService.sh \
    #
    # 安装扩展
    && /installExt.sh \
    && rm -f /installExt.sh \
    #
    # 处理用户权限
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/zsh --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME} \
    && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME} \
    #
    # Composer
    && sudo -u ${USERNAME} mkdir -p /home/${USERNAME}/bin \
    && sudo -u ${USERNAME} curl -o /home/${USERNAME}/bin/composer https://getcomposer.org/composer-1.phar \
    && chmod +x /home/${USERNAME}/bin/composer \
    && sudo -u ${USERNAME} /home/${USERNAME}/bin/composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
    #
    # 环境变量配置
    && echo "export PATH=\$PATH:\$HOME/.composer/vendor/bin" >> /home/${USERNAME}/.envrc \
    && echo "export PATH=/home/${USERNAME}/bin:\$PATH " >> /home/${USERNAME}/.envrc \
    && sudo -u ${USERNAME} touch /home/${USERNAME}/.zshrc \
    && echo ". ~/.envrc" > /home/${USERNAME}/.zshrc \
    && echo "startService.sh && ohmyzsh.sh" >> /home/${USERNAME}/.zshrc \
    #
    # 修改 PHP 运行用户
    && sed -ri "s/user.*=.*www-data/user = ${USERNAME}/g" /usr/local/etc/php-fpm.d/www.conf \
    && sed -ri "s/group.*=.*www-data/group = ${USERNAME}/g" /usr/local/etc/php-fpm.d/www.conf
    
# Install Nginx、Nodejs And yarn
RUN echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" | tee /etc/apt/sources.list.d/nginx.list \
    && curl -fsSL https://nginx.org/keys/nginx_signing.key | apt-key add - \
    && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y nginx nodejs yarn \
    #
    # 修改 Nginx 配置
    && sed -ri "s/user.*nginx/user ${USERNAME}/g" /etc/nginx/nginx.conf \
    #
    # 清理包缓存信息
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*