# Sdr Pi

Deploy standalone PI with SOAPYSDR a wireless access point and the ability to host training material and GNURadio Companion Graphs for easy usage. 

## Installing Base PI

Tested on:
-  Raspberry Pi 3b

**Note:** the build process currently assumes you have a wired interface with dhcp for initial setup. Therefore building on a WiFi connection only may be unreliable.

### Prepare Raspberry Pi OS Image

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Grab a micro-sd card (32Gb+ has been tested) and a suitable card reader
3. In Pi Imager select:
    - Storage -> Your micro-sd
    - Chose OS:
        - Raspberry Pi OS (Other)
            - Raspberry Pi OS Lite (32-bit) (Debian Bullseye)
    - Settings:
        - Set Hostname: sdr-pi.local
        - Enable SSH:
            - Use Password Authentication
        - Set Username and Password
            - Username: pi
            - Password: ${remember_this}
        - Configure Wifi: deselect
        - Set Locale Settings
            - Timezone: As appropriate
            - Keyboard Layout: As appropriate
            - Skip first-run wizard: do not skip
    - Write
4. Put SD Card into Pi
5. Connect Pi
    - Network
    - Power
    - HDMI (Optional)
    - Keyboard (Optional)
6. Boot Pi allow first-run wizard to complete
7. Test ssh access:
    - ```ssh pi@sdr-pi.local```

## Deploy Playbook

The sdr-pi playbook can be executed from a host with both ansible and sshpass installed.

To run the playbook use: 

```Shell
eval `ssh-agent`
ssh-add 
ssh-add -l
ansible-playbook -i inventory.yml sdr-pi.yml --ask-pass
```