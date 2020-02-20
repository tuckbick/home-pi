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
    sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
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
2) Find the IP address of the Raspberry Pi via your router
3) Connect via SSH:
    ```bash
    ssh pi@192.168.86.2
    // Use the default password 'raspberry'
    ```
