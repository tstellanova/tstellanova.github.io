## Purpose

Setup your [Raspberry Pi 0 w](http://amzn.to/2oH8VcR) to develop with the Rust language.

## Initial configuration
- Ensure your rpi0 has a network connection so it can download and install packages
- Install Raspbian based on Debian Stretch

## Configure `pigpio`
If you're working with the rpi0's GPIO, the `pigpio` library can be very handy.
By default the pigpio library is now included with Raspbian, but you may need to enable it

Check:
`sudo service pigpiod status`
Should show `Active: active (running) ` -- if it does not:

### Setup pigpiod to run at boot
- `sudo systemctl enable pigpiod`
- After reboot, verify with: ` sudo service pigpiod status`

### Optional: Install python pigpio access
Some pigpio examples are provided in the Python language: In order to run these you'll need to have these packages installed:
```
sudo apt-get update
sudo apt-get install python-pigpio python3-pigpio
```

## Install rustup
- From the rpi3 command line: `curl https://sh.rustup.rs -sSf | sh`
- You can uninstall rustup at any time with `rustup self uninstall`
- Installing rust can take some time, especially on slow wifi connection



