FROM php:7.4-fpm-alpine3.15

LABEL name="Walofz"

# Set locales = UTF-8
ENV MUSL_LOCALE_DEPS cmake make musl-dev gcc gettext-dev libintl 
ENV MUSL_LOCPATH /usr/share/i18n/locales/musl
RUN apk add --no-cache \
    $MUSL_LOCALE_DEPS \
    && wget https://gitlab.com/rilian-la-te/musl-locales/-/archive/master/musl-locales-master.zip \
    && unzip musl-locales-master.zip && cd musl-locales-master \
    && cmake -DLOCALE_PROFILE=OFF -D CMAKE_INSTALL_PREFIX:PATH=/usr . && make && make install \
    && cd .. && rm -r musl-locales-master

# Install Dependencies
RUN apk add --no-cache unixodbc unixodbc-dev iputils kde-dev-scripts readline-dev libltdl freetype-dev \
    wget unzip libzip-dev zip libjpeg-turbo-dev libpng-dev libjpeg-turbo-dev autoconf g++ gnupg \
    && docker-php-ext-install pdo mysqli pdo_mysql

# Add Microsoft repo for Microsoft ODBC Driver 17 for Linux
RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.9.1.1-1_amd64.apk && curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.9.1.1-1_amd64.apk

RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.9.1.1-1_amd64.sig && curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.9.1.1-1_amd64.sig

# Install ODBC 17
RUN curl https://packages.microsoft.com/keys/microsoft.asc  | gpg --import - \
    && gpg --verify msodbcsql17_17.9.1.1-1_amd64.sig msodbcsql17_17.9.1.1-1_amd64.apk \
    && gpg --verify mssql-tools_17.9.1.1-1_amd64.sig mssql-tools_17.9.1.1-1_amd64.apk \
    && apk add --allow-untrusted msodbcsql17_17.9.1.1-1_amd64.apk \
    && apk add --allow-untrusted mssql-tools_17.9.1.1-1_amd64.apk

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd zip opcache

# Enable the php extensions.
RUN pecl install pdo_sqlsrv-5.10.0 sqlsrv-5.10.0 \
    && docker-php-ext-enable pdo_sqlsrv sqlsrv 

# Install && Enable ldap
RUN apk add --no-cache ldb-dev libldap openldap-dev 
# RUN ln -s /usr/lib/x86_64-linux-gnu/libldap.so /usr/lib/libldap.so \
#     && ln -s /usr/lib/x86_64-linux-gnu/liblber.so /usr/lib/liblber.so
ARG DOCKER_PHP_ENABLE_LDAP
RUN docker-php-ext-configure ldap && docker-php-ext-install ldap 

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

CMD ["php-fpm", "-F"]

EXPOSE 9000    