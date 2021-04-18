# home-pi

## Micro SD card
1) Download Raspbian Lite: https://downloads.raspberrypi.org/raspbian_lite_latest
2) Download Etcher: https://www.balena.io/etcher/
3) Flash the Raspbian image to the SD card using Etcher.

## Configure for remote access
1) Add a file entitled `ssh` to the SD card to enable SSH:
    ```bash
    touch ssh
    ```
2) Configure WiFi:
    ```bash
    sudo vi wpa_supplicant.conf
    ```
    Add this and set your network name/password:
    ```
    country=US # Your 2-digit country code
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    network={
        ssid="YOUR_NETWORK_NAME"
        psk="YOUR_PASSWORD"
        key_mgmt=WPA-PSK
    }
    ```

## Boot and connect via SSH
1) Power up the Raspberry Pi
2) After waiting a minute or so for the Pi to start up, find its IP via your router console
3) Connect via SSH:
    ```bash
    ssh pi@<ip-address>
    // Use the default password 'raspberry'
    ```
    
## Update libraries
```bash
sudo apt update
sudo apt upgrade -y
```

## Set custom hostname
```bash
sudo raspi-config
```
Then navigate to Network Options -> Hostname. Set the hostname.

## Rebot and login
```bash
sudo reboot now
ssh pi@<hostname>
```

## Install Docker
```bash
curl -sSL https://get.docker.com | sh
```

### Give `pi` user account access
```bash
sudo usermod -aG docker pi
sudo reboot now
```

## Install Portainer
```bash
docker pull portainer/portainer-ce:latest
```

### Create and run Portainer container
```bash
docker run --restart always -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
Open `http://<hostname>:9000` in a browser.

## Partition external drive
```bash
sudo parted
print
mktable gpt
print
mkpart logical hfs+  0%  50%
mkpart logical fat32 50% 100%
name 1 tm
name 2 share
print
```

## Make filesystem on partitions
```bash
sudo apt install hfsutils hfsprogs
sudo mkfs.hfsplus /dev/sda1 -n tm
sudo mkfs.vfat    /dev/sda2 -n share
```

## Create mount points
```bash
sudo mkdir /media/tm    && sudo chmod -R 777 /media/tm    && sudo chown pi:pi /media/tm
sudo mkdir /media/share && sudo chmod -R 777 /media/share && sudo chown pi:pi /media/share
```

### Get disk by UUID
Find the UUID for each partition
```bash
ls -lha /dev/disk/by-uuid
```
Add two entries to `/etc/fstab`:
```
UUID=<tm-uuid>    /media/tm    hfsplus force,rw,user,noauto       0 0
UUID=<share-uuid> /media/share auto    auto,rw,user,uid=pi,gid=pi 0 0
```

## Set up HomeAssistant
```bash
docker pull homeassistant/home-assistant:stable
```
Docker compose file:
```yaml
version: '2'
services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant:stable
    volumes:
      - /home/pi/homeassistant_config:/config
      - /media/share:/media
      - /etc/localtime:/etc/localtime
    environment:
      - TZ=America/Denver
    restart: always
    network_mode: host
```

## Set up ESPHome
```bash
sudo apt install python3-pip
pip3 install esphome tornado esptool
docker pull esphome/esphome
```
Docker compose file:
```yaml
version: '2'
services:
  esphome:
    container_name: esphome
    image: esphome/esphome:latest
    volumes:
      - /home/pi/esphome_config:/config
      - /etc/localtime:/etc/localtime
    restart: always
    network_mode: host
```

## Set up AirConnect
```bash
docker pull 1activegeek/airconnect:latest-arm64
```
Docker compose file:
```yaml
version: '2'
services:
  airconnect:
    container_name: airconnect
    image: 1activegeek/airconnect:latest
    volumes:
      - /etc/localtime:/etc/localtime
    restart: always
    network_mode: host
```

## Set up Plex
```bash
docker pull ghcr.io/linuxserver/plex
```
Docker compose file:
```yaml
version: "2.1"
services:
  plex:
    image: ghcr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    privileged: true
    environment:
      - PUID=1000
      - PGID=1000
      - VERSION=docker
      - UMASK_SET=022 #optional
      - PLEX_CLAIM= # https://www.plex.tv/claim/
    volumes:
      - /etc/localtime:/etc/localtime
      - /home/pi/plex_config:/config
      - /media/share/tv:/tv
      - /media/share/movies:/movies
    restart: always
```

## Set up Dogecoin node
```bash
docker pull tuckbick/dogecoin
```
Docker compose file:
```yaml
version: "2"
services:
  dogecoin:
    container_name: dogecoin
    image: tuckbick/dogecoin:latest
    ports:
      - 22556:22556
    volumes:
      - /etc/localtime:/etc/localtime
      - /my/dir/.dogecoin:/root/.dogecoin
    restart: always
    network_mode: host
```

----------

## Resources
* https://www.calebwoods.com/2015/04/06/diy-time-capsule-raspberry-pi/
* https://blog.hqcodeshop.fi/archives/273-GNU-Parted-Solving-the-dreaded-The-resulting-partition-is-not-properly-aligned-for-best-performance.html
* https://gregology.net/2018/09/raspberry-pi-time-machine/  
* https://linoxide.com/file-system/understanding-each-entry-of-linux-fstab-etcfstab-file/
* https://www.wundertech.net/portainer-raspberry-pi-install-how-to-install-docker-and-portainer/
* https://www.home-assistant.io/docs/installation/docker/
* https://esphome.io/guides/getting_started_command_line.html
* https://github.com/1activegeek/docker-airconnect
* https://hub.docker.com/r/linuxserver/plex
