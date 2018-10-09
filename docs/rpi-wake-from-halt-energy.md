## Raspberry Pi Battery Saving with Wake from Halt

Summary: You can potentially save a lot of energy for battery powered applications by forcing the RPi to halt and then reset whenever a triggering event takes place.

## Details

- The bootloader supports a [wake from halt mode](https://elinux.org/RPI_safe_mode#cite_note-1) that is triggerd by grounding GPIO3
- You can eg short pins 5 and 6 together to connect GPIO3 to ground.
- GPIO3 is a special IO pin -- it's normally used for i2c and is pulled high (SCL).
- If the RPi is already in HALT state (reached by doing eg `sudo shutdown -h now` from active state) and theb GPIO3 is grounded, the system will reboot.
- So for long-running tasks where you need to eg conserve battery you can:
  - Perform your active task as soon as the system reboots
  - Force the system to shut down (enter HALT state)
  - Externally trigger GPIO3, causing a reboot
  - Repeat until your batteries die
- The external trigger could be anything from an RTC timer expiring to a PIR sensor firing.  
- Note that grounding GPIO3 momentarily after the RPi is already active should have no effect (which is different than using the RUN header to reset the device)
- Obviously applications are limited by the time it takes to reboot.  Simplify your boot sequence and you may speed up your boot. For latency-sensitive applications you could consider using systemd to trigger your application code _before_ networking configuration finishes.
