docker-mirakurun-epgstation-rpi
====

[Mirakurun](https://github.com/Chinachu/Mirakurun) v3.8.1 + [EPGStation](https://github.com/l3tnun/EPGStation) v1.7.6 の Raspberry Pi 用 Docker コンテナ

注意：開発が終了しているバージョンです。個人的に v1 が気に入ってるので構築手順を確立しておく。

## 構築例

Raspberry Pi 構築

### lite イメージを SDカードに

Raspberry Pi Imager 等で OS を入れる (SSH有効化しておく)

https://www.raspberrypi.org/downloads/raspbian/

（デスクトップ環境が欲しかったら with desktop）

### 起動

### login (インストール時に設定したもの)
```
例:
user: pi
pass: raspberry
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
sudo sed -i 's/$/ coherent_pool=4M dwc_otg.host_rx_fifo_size=2048/' /boot/firmware/cmdline.txt
echo "options px4_drv xfer_packets=51 urb_max_packets=816 max_urbs=6" | sudo tee /etc/modprobe.d/px4_drv.conf
echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0666"' | sudo tee /etc/udev/rules.d/10-vchiq-permissions.rules
echo 'SUBSYSTEM=="video*",GROUP="video",MODE="0666"' | sudo tee /etc/udev/rules.d/10-v4l2-permissions.rules
```

### 一旦再起動
```
sudo reboot
```

### ドライバインストール (例:[px4_drv](https://github.com/tsukumijima/px4_drv))
```
sudo apt -y install raspberrypi-kernel-headers dkms git

~~git clone https://github.com/tsukumijima/px4_drv~~
git clone https://github.com/tsukumijima/px4_drv
cd px4_drv

firmware インストール
sudo mkdir -p /lib/firmware && sudo cp ./etc/it930x-firmware.bin /lib/firmware/

ドライバインストール
cd driver
make
sudo make install
cd ../
```

### install docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
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

カスタムしてる libaribb25 を使っている場合は以下を差し替え
mirakurun/libaribb25

sudo docker compose build
```

### 環境に応じて設定ファイル書き換え
```
vi epgstation/config/config.json
vi mirakurun/conf/tuners.yml
vi mirakurun/conf/channels.yml
vi docker-compose.yml
```

### 録画領域がローカル以外ならマウントさせる 例
```
sudo mkdir /mnt/recorded
sudo vi /etc/fstab

sudo mount -a
```

### 起動
```
sudo docker compose up -d
```
