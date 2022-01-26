FROM ubuntu:focal-20220105
LABEL  Narges Ahmadi <narges.se.ahmadi@gmail.com>
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive  apt-get install -y            \
    nginx curl wget nano unzip    \
    php php-cli php-fpm php-json  \
    php-mysql php-zip php-gd      \
    php-mbstring php-curl php-xml \  
    php-pear php-bcmath &&        \
    apt-get clean 
WORKDIR /var/www/html
COPY ./defaultsite /etc/nginx/sites-enabled/default
COPY ./php.ini /etc/php/7.0/fpm/php.ini
COPY ./index.php ./
CMD service php7.4-fpm start && nginx -g "daemon off;"
EXPOSE 80
