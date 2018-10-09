
## Launch service under `screen` at boot

This will launch a binary executable under `screen` using init.d at boot. 
Tested with Raspbian Scratch and Jessie.

- PUT A FILE in `/etc/init.d`:

```
#! /bin/bash
### BEGIN INIT INFO
# Provides: trappo
# Required-Start:    $all
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Start trappist at boot
### END INIT INFO

su - pi -c "cd ~pi/Documents/proj/trappist; screen -L -dmS trappoland ~pi/Documents/proj/trappist/target/debug/trappist"
```
