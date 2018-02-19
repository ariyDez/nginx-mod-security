#################################################################
# Dockerfile to build Nginx Installed Containers With ModSecurity
# Based on Ubuntu
#################################################################

FROM ubuntu

MAINTAINER ariy.dezmond@gmail.com

ENV DEBIAN_FRONTEND noninteractive

# Install Nginx

# Add application repository URL to the default sources
# RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list

RUN mkdir /keys
COPY ./nginx_signing.key /keys/
RUN apt-key add /keys/nginx_signing.key
RUN echo "deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx" >> /etc/apt/sources.list
RUN echo "deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx" >> /etc/apt/sources.list

# Update the repository
RUN apt-get update

# Install necessary tools
RUN apt-get install -y nano wget dialog net-tools apt-utils


# Download and Install Nginx
RUN apt-get install -y nginx

RUN echo "nginx -v"

COPY ./sites-enabled/default /etc/nginx/
COPY ./default.conf /etc/nginx/conf.d/
RUN apt-get install -y autoconf \
                       automake \
                       build-essential \
                       git \
                       libcurl4-openssl-dev \
                       libgeoip-dev \
                       liblmdb-dev \
                       libpcre++-dev \
                       libtool \
                       libxml2-dev \
                       libyajl-dev \
                       pkgconf \
                       zlib1g-dev
RUN git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
RUN cd /ModSecurity && \
    git submodule init && \
    git submodule update && \
    ./build.sh && \
    ./configure && \
    make && \
    make install
RUN git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
RUN wget http://nginx.org/download/nginx-1.13.8.tar.gz
RUN tar zxvf nginx-1.13.8.tar.gz
RUN cd /nginx-1.13.8 && \
    ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx && \
    make modules && \
    cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
COPY nginx.conf /etc/nginx/
RUN mkdir /etc/nginx/modsec 
RUN wget -P /etc/nginx/modsec https://raw.githubusercontent.com/SpiderLabs/ModSecurity/master/modsecurity.conf-recommended
RUN mv /etc/nginx/modsec/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
RUN sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf
COPY main.conf /etc/nginx/modsec/
COPY modsecurity.conf /etc/nginx/modsec/

# Append "daemon off;" to the beginning of the configuration
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

EXPOSE 80

CMD service nginx start