# Upgrade-Thecus-N2310-N4310-PPC-to-SMB2-TLS1.2
This project describe how to update old Thecus PPC-32bit NAS to with never Samba version that supports SMB2 protocol and other
### HDD Spin Down
To spin down hdd drive in Linux we can use hdparm. https://wiki.archlinux.org/title/Hdparm
``` bash
hdparm -C /dev/sda # check status
hdparm -S 120 /dev/sda # set sping down idle time
#unit is mostly5 second so 120 => 10 minutes
hdparm -y /dev/sda # force spindown

```
 On initial set-up NAS is instaling operating system to HDD, so drives are never spining down. If we force it will spin-up in minute.

 Checking file system show mdadm araiy are used , so technicly we can move system to USB stick 'on-fly' by adding new partitions to arrays.

 ```
[root@N2310 ~]# df | grep md
Filesystem      1K-blocks    Used  Available Use% Mounted on
/dev/md70        10442880 2582976    7859904  25% /
/dev/md50          522944   66240     456704  13% /raidsys/0
/dev/md0       1922317696 2125696 1920192000   1% /raid0
```

First make new GPT partition table on USB disk
```
mdadm -D /dev/md*
mdadm --grow /dev/md70 --raid-devices=1
echo "S_LED 2 0" > /proc/thecus_io
parted /dev/sdv mklabel gpt #make
```