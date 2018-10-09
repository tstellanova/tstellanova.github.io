## Setup OctoPrint 3D Print Server on Raspberry Pi Zero W

Make Prusa i3 MK2 3D printer available for remote printing via OctoPrint running on a Raspberry Pi Zero W, connected with a USB OTG cable, using Slic3r as the client. 

## Summary

- Setup rpi0 with latest raspbian
- Install and configure avahi on rpi0 (for dhcp)
- Install octoprint on rpi0
- Connect to Prusa via usb cables
- Install haproxy on rpi0 (maps port 5000 default -> port 80) or configure octoprint to sit on port 80
- Configure slic3r to connect to the local hostname of rpi0 (will only connect on port 80)

## History / Context

- [Setting up OctoPrint on Raspbian](https://discourse.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspbian/2337)
- [Making OctoPrint available from public Internet](https://discourse.octoprint.org/t/i-want-to-access-my-octoprint-installation-from-the-internet-how-do-i-do-that/221)
- [OctoPrint API](http://docs.octoprint.org/en/master/api/general.html)


## OctoPrint Setup

- Install required tools:
```
sudo apt-get install python-pip python-dev python-setuptools python-virtualenv git libyaml-dev build-essential
```
- Install octoprint:
```
mkdir -p ~/Documents/proj
cd ~/Documents/proj
mkdir OctoPrint && cd OctoPrint
virtualenv venv
source venv/bin/activate
pip install pip --upgrade
pip install https://get.octoprint.org/latest
```
- Install octoprint as a system service:
```
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.init && sudo mv octoprint.init /etc/init.d/octoprint
wget https://github.com/foosel/OctoPrint/raw/master/scripts/octoprint.default && sudo mv octoprint.default /etc/default/octoprint
sudo chmod +x /etc/init.d/octoprint
```

- `sudo vi /etc/default/octoprint` and modify to match your setup:
```
# Path to the OctoPrint executable, you need to set this to match your installation!
DAEMON=/home/pi/Documents/proj/OctoPrint/venv/bin/octoprint
```

- Setup auto start of octoprint service:
```
 sudo update-rc.d octoprint defaults
 sudo service octoprint start
```

- Verify service status: 
```
sudo service octoprint status
```
- At this point OctoPrint should be running and available on port 5000; however, depending on your firewall setup you might not be able to access it on that port. 
- You might have already installed a firewall such as `ufw` (check: `sudo ufw status`), in which case you can temporarily allow port 5000 access with `sudo ufw allow 5000`.  Before installing HAProxy you may want to:
```
sudo ufw disable
sudo apt-get remove ufw
```

## Install HAProxy

- `sudo apt-get install haproxy`
- We'll install and configure HAProxy as a reverse proxy to forward local rpi0 port 5000 to port 80 so that the octoprint API endpoint will be accessible to slic3r (and to ensure that we can access the web interface on the local network). Be warned that _you should not expose this to the public internet_ by, for example, making this available on an rpi0 that has a direct connection to the internet. If you do that, anyone can access your printer and potentially damage it. 

- `sudo vi /etc/haproxy/haproxy.cfg`
- Add the following: 
```
frontend public
        bind :::80 v4v6
        default_backend octoprint

backend octoprint
        reqrep ^([^\ :]*)\ /(.*)     \1\ /\2
        option forwardfor
        server octoprint1 127.0.0.1:5000
```
- This will forward the default OctoPrint port (5000) to port 80
- Verify service status: `sudo service haproxy status`
- At this point you should be able to connect to the OctoPrint web interface on your local network from any browser connected to the same local network via [http://mypi.local](http://mypi.local) 

## Physical Prusa i3 MK2 Connection
- The MK2 3D printer has a USB2 Type B female connector (commonly found on various printers and external drives)
- You need to connect the USB2 micro port on the rpi0 with the MK2.
- I used a very common USB2 micro-B male to USB2 Type A cable, combined with a USB2 Type A female to USB2 Type B male ("print male")
- You can also use something like a "Micro USB OTG to Standard B Type Printer Scanner Hard Disk Cable" -- Micro-B USB2 male on one side and standard USB2 Type B male on the other side. 

## Configure Slic3r
- Slic3r `Preferences` -> `Printer Settings` -> `General` -> `Printer host upload` choose Host Type: Octoprint
- Obtain the API key from OctoPrint via the web interface (wrench `Settings` icon -> `API` -> `API Key`)
- Paste the key into Slic3r `Printer host upload` -> `API Key`
- In Slic3r `Printer host upload` -> `Hostname` set the octoprint hostname, eg `mypi.local` and test to confirm
