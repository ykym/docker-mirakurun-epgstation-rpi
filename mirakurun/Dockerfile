FROM node:12-buster-slim
EXPOSE 40772
ENV DOCKER="YES"
ENV DEV="libpcsclite-dev build-essential curl libssl-dev yasm libtool autoconf pkg-config gcc git g++ make automake"

ADD ./libaribb25 /usr/local/libaribb25
ADD ./recfriio.diff /tmp/recfriio.diff

RUN apt-get update && \
    apt-get -y install pcscd libpcsclite1 libccid pcsc-tools libboost-all-dev libboost-thread-dev libboost-filesystem-dev libglib2.0-dev libusb-dev psmisc dvb-tools && \
    apt-get -y install $DEV && \
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
    echo '#!/bin/sh\n\
sh -c cat | b25 -v 0 /dev/stdin /dev/stdout | cat' > /usr/local/bin/arib-b25-stream.sh && \
    chmod +x /usr/local/bin/arib-b25-stream.sh && \
\
# recdvb
\
    git clone https://github.com/epgdatacapbon/recdvb /tmp/recdvb && \
    cd /tmp/recdvb && \
    ./autogen.sh && \
    ./configure --enable-b25 && \
    make && \
    make install && \
\
# recpt1
\
    git clone https://github.com/stz2012/recpt1 /tmp/recpt1 && \
    cd /tmp/recpt1/recpt1 && \
    sed -i -e 's/arib25/aribb25/g' configure.ac && \
    sed -i -e 's/arib25/aribb25/g' -e 's/HAVE_LIBARIB25/HAVE_LIBARIBB25/g' -e '/#include <aribb25\/arib_std_b25.h>/a#include <aribb25\/arib_std_b25_error_code.h>' decoder.h && \
    sed -i -e 's/HAVE_LIBARIB25/HAVE_LIBARIBB25/g' decoder.c && \
    sed -i -e 's/HAVE_LIBARIB25/HAVE_LIBARIBB25/g' recpt1.c && \
    ./autogen.sh && \
    ./configure --enable-b25 && \
    make && \
    make install && \
\
# recfsusb2n
\
    git clone https://github.com/epgdatacapbon/recfsusb2n /tmp/recfsusb2n && \
    cd /tmp/recfsusb2n/src && \
    make B25=1 && \
    make install && \
\
# recfsusb2n K0905
\
    git clone https://github.com/ykym/recfsusb2n /tmp/recfsusb2n-k0905 && \
    cd /tmp/recfsusb2n-k0905 && \
    git checkout aribb25 && \
    cd src && \
    make && \
    cp -rp recfsusb2n /usr/local/bin/recfsusb2n-k0905 && \
\
# recfriio
\
    git clone https://github.com/stz2012/recfriio /tmp/recfriio && \
    cd /tmp/recfriio && \
    patch -p1 < /tmp/recfriio.diff && \
    cd /tmp/recfriio/recfriior5 && \
    make && \
    cp -rp recfriio /usr/local/bin/ && \
\
# pm2
\
    npm install pm2 -g && \
\
# arib-b25-stream-test
\
    npm install arib-b25-stream-test -g --unsafe && \
\
# mirakurun
    npm install mirakurun -g --unsafe --production && \
\
# 不要なパッケージを削除
\
    apt-get -y remove $DEV && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/libaribb25 && \
    rm -rf /tmp/recpt1 && \
    rm -rf /tmp/recfsusb2n && \
    rm -rf /tmp/recfsusb2n-k0905 && \
    rm -rf /tmp/recfriio

WORKDIR /usr/local/lib/node_modules/mirakurun

ENTRYPOINT /etc/init.d/pcscd start && \
    npm start

