FROM node:12-buster-slim
EXPOSE 8888
ARG CPUCORE="1"
ENV DEV="make gcc git g++ automake curl wget autoconf build-essential python python3 unzip" 

RUN apt-get update && \
    apt-get -y install $DEV && \
    apt-get -y install openssh-client rsync nkf && \
    apt-get -y install sqlite3 && \
\
# install hardware acceleration ffmpeg
    echo "deb http://archive.raspberrypi.org/debian/ buster main" > /etc/apt/sources.list.d/raspi.list && \
    gpg --keyserver keys.gnupg.net --recv-key 82B129927FA3303E && \
    gpg -a --export 82B129927FA3303E| apt-key add - && \
    apt-get update && \
    apt-get -y install ffmpeg && \
\
# install EPGStation
    cd /usr/local/ && \
    git clone https://github.com/l3tnun/EPGStation.git && \
    cd /usr/local/EPGStation && \
    npm install && \
\
# ffmpeg thumbnail option patch
    sed -i 's/-i %INPUT% -ss ${ thumbnailPosition }/-ss ${ thumbnailPosition } -i %INPUT% -vf yadif=0:-1:1/g' /usr/local/EPGStation/src/server/Model/Operator/Thumbnail/ThumbnailManageModel.ts && \
\
    npm run build && \
\
# SQLite regex
wget -O /tmp/sqlite-amalgamation-3270200.zip https://www.sqlite.org/2019/sqlite-amalgamation-3270200.zip && \
wget -O /tmp/sqlite-src-3270200.zip https://www.sqlite.org/2019/sqlite-src-3270200.zip && \
unzip /tmp/sqlite-amalgamation-3270200.zip -d /tmp/ && \
unzip /tmp/sqlite-src-3270200.zip -d /tmp/ && \
cp /tmp/sqlite-src-3270200/ext/misc/regexp.c /tmp/sqlite-amalgamation-3270200 && \
cd /tmp/sqlite-amalgamation-3270200 && \
gcc -g -fPIC -shared regexp.c -o regexp.so && \
cp regexp.so /usr/local/EPGStation/ && \
cd /usr/local/EPGStation && \
\
# remove packages
\
    apt-get -y remove $DEV && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /tmp/sqlite-* && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/EPGStation

ENTRYPOINT npm start

