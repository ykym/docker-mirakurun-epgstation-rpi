# docker-mirakurun-epgstation-rpi
====

[Mirakurun](https://github.com/Chinachu/Mirakurun) + [EPGStation](https://github.com/l3tnun/EPGStation) の Raspberry Pi 用 Docker コンテナ

## 構築例

Raspberry Pi 構築

### buster lite イメージを SDカードに dd
https://www.raspberrypi.org/downloads/raspbian/
（デスクトップ環境が欲しかったら with desktop）
boot パーティションのディレクトリに 「ssh」(拡張子なし) のファイルを作っておくと最初から SSH が有効化される

### 起動

### login
user pi
pass raspberry

### IPを固定する場合
```
sudo vi /etc/dhcpcd.conf

以下の部分のコメントアウトを外して適時設定

#interface eth0
#static ip_address=192.168.0.10/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
#static routers=192.168.0.1
#static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1

面倒ならDHCPサーバ側で固定振り
```

### 初期設定
```
sudo timedatectl set-timezone 'Asia/Tokyo'

sudo apt update && sudo apt upgrade -y

sudo apt install locales-all
sudo localectl set-locale 'LANG=ja_JP.utf8'
```
### チューニング
```
sudo sed -i 's/$/ coherent_pool=4M dwc_otg.host_rx_fifo_size=2048/' /boot/cmdline.txt
sudo sed -i 's/^CONF_SWAPSIZE=100/CONF_SWAPSIZE=1024/' /etc/dphys-swapfile
echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0666"' | sudo tee /etc/udev/rules.d/10-vchiq-permissions.rules
echo "options px4_drv xfer_packets=51 urb_max_packets=816 max_urbs=6" | sudo tee /etc/modprobe.d/px4_drv.conf
```

### ドライバ用パッケージインストール (例:[px4_drv](https://github.com/nns779/px4_drv))
```
sudo apt -y install raspberrypi-kernel-headers dkms git

git clone https://github.com/nns779/px4_drv.git
cd px4_drv
cd fwtool
make
wget http://plex-net.co.jp/plex/pxw3u4/pxw3u4_BDA_ver1x64.zip -O pxw3u4_BDA_ver1x64.zip
unzip -oj pxw3u4_BDA_ver1x64.zip pxw3u4_BDA_ver1x64/PXW3U4.sys
./fwtool PXW3U4.sys it930x-firmware.bin
sudo mkdir -p /lib/firmware
sudo cp it930x-firmware.bin /lib/firmware/
cd ../
sudo cp -a ./ /usr/src/px4_drv-0.2.1
sudo dkms add px4_drv/0.2.1
sudo dkms install px4_drv/0.2.1
cd
```

### install docker
```
curl -sSL https://get.docker.com | sh
sudo apt -y install docker-compose
```

### 一旦再起動
```
sudo reboot
```

### install Mirakurun+EPGStation
```
cd
git clone http://github.com/ykym/docker-mirakurun-epgstation-rpi.git
cd docker-mirakurun-epgstation-rpi

カスタムしてる libaribb25 を使ってる場合は以下を差し替え
mirakurun/libairbb25

sudo docker-compose build

### 環境に応じて設定ファイル書き換え
```
vi epgstation/config/config.json
vi mirakurun/conf/tuners.yml
vi mirakurun/conf/channels.yml
vi docker-compose.yml
```

### 録画領域がローカル以外ならマウントさせる
```
sudo mkdir /mnt/recorded
sudo vi /etc/fstab

sudo mount -a
```

### 起動
```
sudo docker-compose up -d
```
