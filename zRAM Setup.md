- ZRAM Mod for Pi and other PCs
- Compress Section of Ram
- Use Compressed Ram as Swap

Disable existing swap file (pi)
- nano /etc/dphys-swapfile
- set CONF_SWAPSIZE=0

Create Systemd Unit file to run a zram script:

=========
Unit File
=========
Create File
- cd /etc/systemd/system/
- touch zram.service

```
[Unit]
Description=Run zRAM script

[Service]
Type=oneshot
ExecStart=/usr/bin/zram start
ExecStop=/usr/bin/zram stop
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```  
======
Script
======

Put file
- cd /usr/bin
- touch zram
- make executable! > chmod a+x zram  

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          zram
# Required-Start:    $local_fs
# Required-Stop:     $local_fs
# Default-Start:     S
# Default-Stop:      0 1 6
# Short-Description: Use compressed RAM as in-memory swap
# Description:       Use compressed RAM as in-memory swap
### END INIT INFO

# Author: Antonio Galea <antonio.galea@gmail.com>
# Thanks to Przemysław Tomczyk for suggesting swapoff parallelization
# Distributed under the GPL version 3 or above, see terms at
#      https://gnu.org/licenses/gpl-3.0.txt
FRACTION=75

MEMORY=`perl -ne'/^MemTotal:\s+(\d+)/ && print $1*1024;' < /proc/meminfo`
CPUS=`nproc`
SIZE=$(( MEMORY * FRACTION / 100 / CPUS ))

case "$1" in
  "start")
    param=`modinfo zram|grep num_devices|cut -f2 -d:|tr -d ' '`
    modprobe zram $param=$CPUS
    for n in `seq $CPUS`; do
      i=$((n - 1))
      echo $SIZE > /sys/block/zram$i/disksize
      mkswap /dev/zram$i
      swapon /dev/zram$i -p 10
    done
    ;;
  "stop")
    for n in `seq $CPUS`; do
      i=$((n - 1))
      swapoff /dev/zram$i && echo "disabled disk $n of $CPUS" &
    done
    wait
    sleep .5
    modprobe -r zram
    ;;
  *)
    echo "Usage: `basename $0` (start | stop)"
    exit 1
    ;;
esac
```  

==========================================

Enable Unit file
- systemctl enable zram.service
- Should come back saying symlink created

Start Service
- systemctl start zram.service

Verify zram created
- htop
- /dev will have zram devices (4 of them)

- Works!