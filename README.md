# RPI4-HA-installation
installation process and resources for RPi4 with HA

# Required Hardware:
- RaspberryPi 4 model B
- MHS 3.5inch display & touchscreen

# Installation process
1. Assemble all the parts like explained [here](http://www.lcdwiki.com/MHS-3.5inch_RPi_Display).
2. Download latest raspberryPi O.S. (Raspbian) from [here](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit).
3. flash the image with your preferred tool balena etcher is the one RPi foundation recommends.
4. place a ssh.txt emtpy file on the root directory of the accesible partition of the newly flashed O.S. on the SD card to enable ssh.
5. put the SD card on the PI and switch it on for the first time.
6. connect through ssh using user pi and password raspberry.
7. run the following commands through ssh:
  ```shell
  sudo rm -rf LCD-show
  git clone https://github.com/goodtft/LCD-show.git
  chmod -R 755 LCD-show
  cd LCD-show/
  sudo ./MHS35-show
  ```
8. wait for the RPi to restart, you should be able to use the screen once rebooted.
9. run `sudo raspi-config` to change the location, timezone and default display resolution (640x480).
10. start installing Home Assistant
    1. follow the instructions as stated below, if other hardware is being in use, follow [these](https://community.home-assistant.io/t/installing-home-assistant-supervised-on-debian-10/200253) instead.
    2. execute the following commands through ssh
    ```shell
    # elevate account to root privileges
    sudo -i
    
    # Update, upgrade & clean debian install, after install dependencies
    apt update && sudo apt upgrade -y && sudo apt autoremove -y
    apt-get install -y software-properties-common apparmor-utils apt-transport-https ca-certificates curl dbus jq network-manager
    
    # Disable & stop ModemManager
    systemctl disable ModemManager
    systemctl stop ModemManager
    
    # Install Docker with official script
    curl -fsSL get.docker.com | sh
    
    # Install home assistant with supervisor using explicit device flag
    curl -sL "https://raw.githubusercontent.com/Kanga-Who/home-assistant/master/supervised-installer.sh" | bash -s -- -m raspberrypi4
    ```
    3. After install, access https://YourRPiIPAddress:8123 to know when the process has finished and to configure an account
    4. Access this account from chromium at the 3.5 inch screen with your user and password so they stay registered for that device
11. Setup RPi4 in kiosk mode by editing this file
```shell
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```
12. Add the following lines to the end of the file
```shell
xset -dpms
xset s off
xset s noblank
@chromium-browser --kiosk --incognito --noerrdialogs --disable-translate --no-first-run --fast --fast-start --disable-infobars --app=http://localhost:8123
```
13. add the following line at the end of /etc/ssh/sshd_config to prevent ssh freeze
```
sudo nano /etc/ssh/sshd_config
```
```
IPQoS cs0 cs0
```
14. after this changes, you're done! to test this either run `sudo systemctl restart lightdm.service` or reboot the whole RPi, it should now start and log in to HomeAutomation in kiosk mode.
