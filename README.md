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

## Using it

### Wireless Access

A new Wireless Access point will be configured whenever the Pi boots up based on the current MAC address of eth0. 

*Note* You can move the installed SD card to a new Pi and it will create a new Wireless hotspot, however the Certificate Authority (see below) will still reference the original hardware used whilst running the playbook. If this is an issue; simply run the playbook on the hardware you want the Sdr-Pi solution to run on.

Accessing the Wireless Access point provides an easy way to use the remainder of the tools, but will be limited to WiFi speeds; be mindful of that if your SDR supports wide sampling as you may start dropping packets purely based on WiFi bandwidth. 


### Hardwired Access

By default you can connect the Ethernet port onto a DHCP enabled network and then pivot through your WiFi access port (traffic will be masqueraded). 

Alternatively, once you are happy that they Pi is installed, run: 
```enable_wired_access.sh``` 

This will disable the DHCP client, set a static IP and enable a DHCP server on the wired interface. You will need to reboot to finalise that configuration.

If you decide you no longer want to use hardwired access (e.g. to update the Pi), simply run:
```disable_wired_access.sh```
This will revert back to default mode of operation

### Readonly Pi
If you are not using the Pi to save experiments or for development, you may benefit from setting it into "read only" mode.

This will provide some measure of file system protection in the event of power failure. 

There are two mechanisms by which you can enable read only mode:
- Within the inventory file (inventory.yml) set the "make_raspberry_pi_readonly" variable to True
- Run the ```enable_readonly.sh``` script from a SSH session.

To turn a read only Pi back to read/write simply run ```disable_readonly.sh``` from a SSH session.

### Available tools:

#### Certificate Authority

A copy of easyrsa3 is installed (by default in /usr/sdr-pi), and the CA certificate is made available via the NginX reverse proxy, most of the tooling uses a wildcard ssl certificate suitable for the domain you configure in the inventory file (inventory.yml) prior to running the playbook for the first time. 

Connect to one of the DHCP enabled access methods (Wireless/Hardwired) and visit [http://cert](http://cert) to be presented with the CA file.

You will need to install the CA file to enable drive mappings in Windows/MacOS as they will not pass authentication credentials to untrusted WebDAV instances.

#### SoapySDR

Assuming Sdr-Pi can be found via the dns name "sdr.radio" (default in the inventory.yml file), then we can use a variety of hardware directly:

Specific configuration stanza's:

- UHD / Ettus research (osmocom Source)
    - Driver Arguments: "soapy=0,driver=remote,remote=sdr.radio:55132,remote:driver=uhd"
- BladeRF Micro A9
    - Driver Arguments: "soapy=0,driver=remote,remote=sdr.radio:55132,remote:driver=bladerf"
- HackRF Blue (Should be same as HackRF One)
    - Driver Arguments: "soapy=0,driver=remote,remote=sdr.radio:55132,remote:driver=hackrf"
- BladeRF Micro A9
    - Driver Arguments: "soapy=0,driver=remote,remote=sdr.radio:55132,remote:driver=rtlsdr"

#### WebDAV file store

A WebDAV folder is exposed on Apache as [https://sdr-pi.local/webdav](https://sdr-pi.local/webdav) using the credentials specified in the inventory file (inventory.yml).

This can be accessed in a browser, but to mount drives to is you will first need to add the CA file to your trusted certificate store. 

#### MediaWiki 

An instance of Media Wiki is exposed on [https://sdr-pi.local/wiki](https://sdr-pi.local/wiki), you can use the Git plugin to synchronise this with an existing MediaWiki instance. 

#### OpenWebRx

An instance of OpenWebRx is exposed on [https://sdr-pi.local/websdr](https://sdr-pi.local/websdr), this should autodetect the SDR in use (specifically it will brute force all known configurations).

If you do not have any SDR's detected, either reboot with your chosen hardware already installed or try the (very experimental) hot swap function.

*Note:* The hot-swap function makes use of udev to poll for a new addition to the USB bus and see if it looks like an SDR; from there it uses the "slow bounce" script to create a Mutex, identify if SoapySDR is playing nicely and where necessary restart services / OpenWebRx to get us in a known good state. 

This requires us to wait a while for certain hardware; so the hot swap function takes between 30-60 seconds to complete.
The easiest way to determine success is to just look at the websdr waterfall until you start to see some data stream (audio will also engage).

As you come across issues with the hot swap functions, please can you log them as issues on Github so that I can debug them with you.

#### Bettercap

Currently installed at [https://sdr-pi.local/bettercap](https://sdr-pi.local/bettercap), allows simple bluetooth enumeration / first past analysis to aid in resolving any EMC issues.

The default Raspberry Pi wireless adaptor cannot act as both an access point and in monitor mode; so the remainder of the bettercap tooling may not be useful without the addition of further hardware. 