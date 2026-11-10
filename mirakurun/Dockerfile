FROM node:16-buster-slim
EXPOSE 40772
ENV DOCKER="YES"
ENV DEV="libpcsclite-dev build-essential curl libssl-dev yasm libtool autoconf pkg-config gcc git g++ make automake golang-go"

COPY ./libaribb25 /usr/local/libaribb25
COPY ./udev_start.sh /usr/bin/udev_start.sh

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    sed -i '/stretch-updates/d' /etc/apt/sources.list

RUN apt-get update && \
    apt-get -y install pcscd libpcsclite1 libccid pcsc-tools libboost-all-dev libboost-thread-dev libboost-filesystem-dev libglib2.0-dev libusb-dev psmisc dvb-tools udev procps && \
    apt-get -y install $DEV && \
\
    chmod 755 /usr/bin/udev_start.sh && \
\
# libaribb25
\
    test -f /usr/local/libaribb25/Makefile || git clone https://github.com/epgdatacapbon/libaribb25 /tmp/libaribb25 && cp -rp /usr/local/libaribb25 /tmp/ && \
    cd /tmp/libaribb25 && \
    make clean && \
    make && \
    make install && \
    ldconfig && \
\
# arib-b25-stream.sh
\
    echo '#!/bin/sh\n\
sh -c cat | b25 -v 0 /dev/stdin /dev/stdout | cat' > /usr/local/bin/arib-b25-stream.sh && \
    chmod +x /usr/local/bin/arib-b25-stream.sh && \
\
# go-dvb
\
    git clone https://github.com/ziutek/dvb /tmp/go-dvb && \
    cd /tmp/go-dvb && \
    go build -o dvb-tune examples/tune/main.go && \
    cp -rp dvb-tune /usr/local/bin/ && \
\
# recdvb
\
    git clone https://github.com/epgdatacapbon/recdvb /tmp/recdvb && \
    cd /tmp/recdvb && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install && \
\
# recpt1
\
    git clone https://github.com/stz2012/recpt1 /tmp/recpt1 && \
    cd /tmp/recpt1/recpt1 && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install && \
\
# recfsusb2n
\
    #git clone https://github.com/epgdatacapbon/recfsusb2n /tmp/recfsusb2n && \
    git clone https://github.com/toisme/recfsusb2n /tmp/recfsusb2n && \
    cd /tmp/recfsusb2n/src && \
    make && \
    make install && \
\
# pm2
\
    npm install npm@6 -g && \
    npm install pm2 -g && \
\
# arib-b25-stream-test
\
    npm install arib-b25-stream-test -g --unsafe && \
\
# mirakurun
    npm install mirakurun@3.8.1 -g --unsafe --production && \
\
# 不要なパッケージを削除
\
    apt-get -y remove $DEV && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/libaribb25 && \
    rm -rf /tmp/go-dvb && \
    rm -rf /tmp/recdvb && \
    rm -rf /tmp/recfsusb2n && \
    rm -rf /tmp/recpt1

WORKDIR /usr/local/lib/node_modules/mirakurun

ENTRYPOINT /etc/init.d/pcscd start && \
    /usr/bin/udev_start.sh && \
    npm start

