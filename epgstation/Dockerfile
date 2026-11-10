#FROM node:14-buster-slim
FROM l3tnun/epgstation:v1.7.6-debian
EXPOSE 8888
ARG CPUCORE="1"
ENV DEV="make gcc git g++ automake curl wget autoconf build-essential python python3 python3-yaml unzip" 
ENV FFMPEG_VERSION=7.0

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    sed -i '/stretch-updates/d' /etc/apt/sources.list

RUN apt-get update && \
    apt-get -y install $DEV && \
    apt-get -y install openssh-client rsync nkf && \
    apt-get -y install sqlite3 && \
\
# ffmpeg static build download
\
    wget -O /tmp/ffmpeg-static.tar.xz https://johnvansickle.com/ffmpeg/releases/ffmpeg-${FFMPEG_VERSION}-arm64-static.tar.xz && \
    mkdir /tmp/ffmpeg-static/ && \
    tar Jxf /tmp/ffmpeg-static.tar.xz -C /tmp/ffmpeg-static/ --strip-components 1 && \
    cp /tmp/ffmpeg-static/ffmpeg /tmp/ffmpeg-static/ffprobe /usr/local/bin/ && \
\
# SQLite regex
\
    wget -O /tmp/sqlite-amalgamation-3270200.zip https://www.sqlite.org/2019/sqlite-amalgamation-3270200.zip && \
    wget -O /tmp/sqlite-src-3270200.zip https://www.sqlite.org/2019/sqlite-src-3270200.zip && \
    unzip /tmp/sqlite-amalgamation-3270200.zip -d /tmp/ && \
    unzip /tmp/sqlite-src-3270200.zip -d /tmp/ && \
    cp /tmp/sqlite-src-3270200/ext/misc/regexp.c /tmp/sqlite-amalgamation-3270200 && \
    cd /tmp/sqlite-amalgamation-3270200 && \
    gcc -g -fPIC -shared regexp.c -o regexp.so && \
    cp regexp.so /app/ && \
    cd /app && \
\
# remove packages
\
    apt-get -y remove $DEV && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /tmp/ffmpeg-static* && \
    rm -rf /tmp/sqlite-* && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

ENTRYPOINT npm start

