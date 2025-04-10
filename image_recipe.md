# Recipe for creating the Picroft IMG

These are the steps followed to create the base image for Picroft on PI OS.  This was performed on a Raspberry Pi 4

NOTE: At startup Picroft will automatically update itself to the latest version of released software, scripts and Skills.


### Start with the official Raspberry Pi OS Image
* Download and burn [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit).
  <br>_Last used 2022-08-24 version_
* Use the Raspberry Pi Imager
  - Select the image you downloaded from above
  - Select the SD card
  - click cog at the bottom right corner
    - Click Enable SSH
    - Set the password
    - Click Configure wireless LAN (if you are on wifi it should all be filled in for you)
      - change country for the wifi
    - Click Set local settings (make sure it is set for your area)
* Install into Raspberry Pi and boot
  - login: pi
  - password: what you set the password to from above step
### Configure USB Audio
  - sudo apt update
  - sudo apt upgrade





speaker-test -c2 -twav -l7









### General configuration (If the above steps where fallowed these are not needed)
  - ```sudo raspi-config```
  - 1 Change User Password
      - Enter and verify ```mycroft```
  - 2 Network Options
      - N1 Hostname
        - Enter ```picroft```
      - N3 Network interface names
        - pick *Yes*
  - 3 Boot Options
      - B1 Desktop / CLI
        - B2 Console Autologin
      - B2 Wait for network
        - pick *No*
  - 4 Localization Options
      - I3 Change Keyboard Layout
          - Pick *Generic 104-key PC*
          - Pick *Other*
          - Pick *English (US)*
          - Pick *English (US)*
          - Pick *The default for the keyboard layout*
          - Pick *No compose key*
      - I4 Change Wi-fi Country
          - Pick *United States*
  - 5 Interfacing Options
      - P2 SSH
          - Pick *Yes*
  - Finish and reboot

### Set the device to not use locale settings provided by ssh
* ```sudo nano /etc/ssh/sshd_config``` and comment out the line (prefix with '#')
  ```
  AcceptEnv LANG LC_*
  ```

### Connect to the network
* Either plug in Ethernet or

  __or__ (If the above steps where fallowed these are not needed)
* Guided wifi setup
  * ```sudo raspi-config```
    - 2 Network Options
      - N2 Wi-fi

  __or__
* Manually setup wifi
  * ```sudo nano /etc/wpa_supplicant/wpa_supplicant.conf```
  * Enter network creds:
    ```
    network={
            ssid="NETWORK"
            psk="WIFI_PASSWORD"  # for network with password
            key_mgmt=NONE        # for open network
    }
    ```

## Install Picroft scripts
* cd ~
* wget -N https://rawgit.com/MycroftAI/enclosure-picroft/buster/home/pi/update.sh (Pi OS uses pulsaudio)
* bash update.sh

**The update.sh script will perform all of the following steps in this section...**
When asked by dev_setup, answer as follows:
- Y) run on the stable 'master' branch
- Y) automatically check for updates
- Y) build Mimic locally
- Y) add Mycroft helper commands to path
- Y) check code style

##### Enable Autologin as the 'pi' user

* ```sudo nano /etc/systemd/system/getty@tty1.service.d/autologin.conf``` and enter:
   ```
   [Service]
   ExecStart=
   ExecStart=-/sbin/agetty --autologin pi --noclear %I     38400 linux
   ```

* ```sudo systemctl enable getty@tty1.service```

##### Create RAM disk and point to it
  - ```sudo nano /etc/fstab``` and add the line:
    ```
    tmpfs /ramdisk tmpfs rw,nodev,nosuid,size=20M 0 0
    ```

##### Environment setup (part of update.sh)

* ```sudo mkdir /etc/mycroft```
* ```sudo nano /etc/mycroft/mycroft.conf```
* mkdir ~/bin

##### Customize .bashrc for startup
* ```nano ~/.bashrc```
   uncomment *#alias ll='ls -l'* near the bottom of the file
   at the bottom add:
   ```
   #####################################
   # This initializes Mycroft
   #####################################
   source ~/auto_run.sh
   ```

##### Install git and mycroft-core
* ```sudo apt-get install git```
* ```git clone https://github.com/MycroftAI/mycroft-core.git```
* ```cd mycroft-core```
* ```git checkout master```
* ```bash dev_setup.sh```

(Wait an hour on a RPi 3B+/4)

## Final steps
* Run ```. ~/bin/mycroft-wipe --keep-skills```
* Remove the SD card
* Create an IMG file named `"Picroft_v${CORE_VERSION}_${YYYYMMDD}.img"` (optionally include a release candidate suffix `"_rc-${num}.img"`)
* Compress the IMG using pishrink.sh
* Upload to downloads.mycroft.ai and adjust redirect link from https://mycroft.ai/to/picroft-image or https://mycroft.ai/to/picroft-unstable
