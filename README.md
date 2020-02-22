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
    
## Update and restart
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot now
ssh pi@<ip-address>
```

## Set custom hostname
```bash
sudo raspi-config
```
Then navigate to Network Options -> Hostname. Set the hostname. Finish -> Reboot.

## Configure Pi as a Time Capsule for your Mac

### Partition Disk
Add an hfs+ partition by following along with these guides:
* https://www.calebwoods.com/2015/04/06/diy-time-capsule-raspberry-pi/
* https://blog.hqcodeshop.fi/archives/273-GNU-Parted-Solving-the-dreaded-The-resulting-partition-is-not-properly-aligned-for-best-performance.html

### Set up Time Capsule
Following along with this guide: https://gregology.net/2018/09/raspberry-pi-time-machine/  
Learn more about fstab here: https://linoxide.com/file-system/understanding-each-entry-of-linux-fstab-etcfstab-file/
