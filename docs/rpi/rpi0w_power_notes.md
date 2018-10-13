## Some notes on Raspberry Pi Zero W power consumption

Tested with an ordinary "USB Safety Tester" USB power monitor, Raspberry Pi Zero W, 
with an rpi camera attached and a PIR sensor attached.

### Normal operation

With WiFi operating, but operating headless (no HDMI output), the rpi's power consumption is about 100 milliamps (0.10A)

### HALT mode

When the Pi Zero W is shut down with something like `sudo shutdown -h now`
it switches into a low-power mode: 
the power consumption is about 20 milliamps (0.02A).
The rpi can subsequently be rebooted ("wake-from-halt").
