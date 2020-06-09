What follows are some sloppy laid out notes taken during an IRIX installation.
It may be somewhat inconsistent, but hopefully helpful enough to reproduce.
I used the IRIX 6.5.22 images for SGI Challenge S. You may want to use something
more recent.

# Bootp server preparations
## Unpacking images

Foundation, part 1 and 2
```
mount -o loop -t efs IRIX\ 6.5\ Foundation\ 1.img tmp
cp -a tmp f1 && umount tmp

mount -o loop -t efs IRIX\ 6.5\ Foundation\ 2.img tmp
cp -a tmp f2 && umount tmp
```

Development foundation and libraries
```
mount -o loop -t efs IRIX\ 6.5\ Development\ Foundation.img tmp
cp -a tmp devf && umount tmp

mount -o loop -t efs IRIX\ 6.5\ Development\ Libraries.img tmp
cp -a tmp devl && umount tmp
```

6.5.30 applications and overlays
```
unzip "Irix 6.5.30_cdimages.zip"
mount -o loop -t efs Irix\ 6.5.30_cdimages/Applications.image tmp/
cp -a tmp apps && umount tmp

mount -o loop -t efs Irix\ 6.5.30_cdimages/Complementary_Applications.image tmp
cp -a tmp capps && umount tmp

mount -o loop -t efs Irix\ 6.5.30_cdimages/Instalation_Tools_and_Overlays1.image tmp
cp -a tmp ovl1 && umount tmp

mount -o loop -t efs Irix\ 6.5.30_cdimages/Overlays2.image tmp
cp -a tmp ovl2 && umount tmp

mount -o loop -t efs Irix\ 6.5.30_cdimages/Overlays3.image tmp
cp -a tmp ovl3 && umount tmp
```

Move directories to the tftpd directory tree
```
mv devl devf f1 f2 /home/irix/i/
mv apps capps ovl1 ovl2 ovl3 /home/irix/i/30/
```

## Configuration
**Install mksh and add irix user**
```
apt install mksh
adduser --home /home/irix --shell /bin/mksh --system --group --disabled-password --gecos 'SGI IRIX' irix
echo '+ root' > /home/irix/.rhosts
mkdir /home/irix/i
chown -R root:root /home/irix
mkdir -p /srv/tftp
ln -sf /home/irix/i /srv/tftp/i
mkdir /home/irix/i/30
```

**Install packages**
```
apt-get install openbsd-inetd bootp tftpd rsh-redone-server rsh-redone-client
```

**Edit /etc/inetd.conf**
```
shell		stream	tcp	nowait	root	/usr/sbin/tcpd	/usr/sbin/in.rshd
bootps		dgram	udp	wait	root	/usr/sbin/bootpd	bootpd -i -t 120
tftp		dgram	udp	wait	nobody	/usr/sbin/tcpd	/usr/sbin/in.tftpd /srv/tftp
```

Important note: I didn't have any success with the `tftpd` package (Debian 10). Using `atftpd`
instead worked fine. Not sure why. It can be setup in the same way as `tftpd`.

**Edit /etc/bootptab**
```
challenge:ip=192.168.2.177
```

**Network configuration**
The server is setup on a temporary local network connected to the same network
switch as the SGI machine.
```
ip address add 192.168.2.5/24
```

SGI Indys can't cope with port numbers greater than 32767 or path-MTU discovery.
```
echo 1 > /proc/sys/net/ipv4/ip_no_pmtu_disc
echo "2048 32767" > /proc/sys/net/ipv4/ip_local_port_range
```

**Enable and start inetd**
```
systemctl enable inetd
systemctl start inetd
```

**Connect to SGI serial terminal**
```
# cu -s 9600 -l /dev/cuaU0
...
>> setenv netaddr 192.168.2.177
>> bootp():i/30/ovl1/stand/sash64
...
System Maintenance Menu

1) Start System
2) Install System Software
3) Run Diagnostics
4) Recover System
5) Enter Command Monitor

Option? 5
Command Monitor.  Type "exit" to return to the menu.
> bootp():/i/22/ovl1/stand/fx.ARCS
Setting $netaddr to 192.168.2.177 (from server tetra.void.lan)
Obtaining /i/22/ovl1/stand/fx.ARCS from server tetra.void.lan
95040+26448+7168+2805248+50056d+5908+8928 entry: 0x93d4aa40
Currently in safe read-only mode.
Do you require extended mode with all options available? (no) yes
SGI Version 6.5 ARCS BE  Oct  6, 2003
fx: "device-name" = (dksc) 
fx: ctlr# = (0) 
fx: drive# = (1) 
...opening dksc(0,1,0)
...drive selftest...OK
Scsi drive type == QUANTUM FIREBALL ST4.3S 0F0C

----- please choose one (? for help, .. to quit this menu)-----
[exi]t             [d]ebug/           [l]abel/           [a]uto
[b]adblock/        [exe]rcise/        [r]epartition/

fx> r

----- partitions-----
part  type        blocks            Megabytes   (base+size)
  7: xfs        4096 + 8467136        2 + 4134 
  8: volhdr        0 + 4096           0 + 2    
 10: volume        0 + 8471232        0 + 4136 

capacity is 8471232 blocks

----- please choose one (? for help, .. to quit this menu)-----
[ro]otdrive           [o]ptiondrive         [e]xpert
[u]srrootdrive        [re]size
fx/repartition> ro

fx/repartition/rootdrive: type of data partition = (xfs) 
Warning: you will need to re-install all software and restore user data
from backups after changing the partition layout.  Changing partitions
will cause all data on the drive to be lost.  Be sure you have the drive
backed up if it contains any user data.  Continue? yes

----- partitions-----
part  type        blocks            Megabytes   (base+size)
  0: xfs      266240 + 8204992      130 + 4006 
  1: raw        4096 + 262144         2 + 128  
  8: volhdr        0 + 4096           0 + 2    
 10: volume        0 + 8471232        0 + 4136 

capacity is 8471232 blocks

----- please choose one (? for help, .. to quit this menu)-----
[ro]otdrive           [o]ptiondrive         [e]xpert
[u]srrootdrive        [re]size
fx/repartition>
fx> exi 
...
Option? 5
Command Monitor.  Type "exit" to return to the menu.
>> hinv
                   System: IP22
                Processor: 175 Mhz R4400, with FPU
     Primary I-cache size: 16 Kbytes
     Primary D-cache size: 16 Kbytes
     Secondary cache size: 1024 Kbytes
              Memory size: 192 Mbytes
                SCSI Disk: scsi(0)disk(1)
>> bootp():/i/22/ovl1/dist/miniroot/unix.IP22
Setting $netaddr to 192.168.2.177 (from server tetra.void.lan)
Obtaining /i/22/ovl1/dist/miniroot/unix.IP22 from server tetra.void.lan
Setting $netaddr to 192.168.2.177 (from server tetra.void.lan)
Obtaining /i/22/ovl1/dist/miniroot/unix.IP22 from server tetra.void.lan
IRIX Release 6.5 IP22 Version 10070055 System V
Copyright 1987-2003 Silicon Graphics, Inc.
All Rights Reserved.

WD95A SCSI controller 4 - differential external, rev 0, min xfer period 100ns
WD95A SCSI controller 5 - differential external, rev 0, min xfer period 100ns
gfe1: missing
gfe0: missing
xpi0: missing from slot 1
xpi0: missing from slot 0
WARNING: Kernname environment variable not set by sash. Runtime symbol table not loaded. Loadable modules will not be registered or loaded.
ALERT: I/O error in filesystem ("/") meta-data dev 0x4a block 0x8132bf
       ("xfs_read_buf") b_error 28 b_bcount 512 b_resid 0
WARNING: initial mount of root device /hw/node/io/gio/hpc/scsi_ctlr/0/target/1/lun/0/disk/partition/1/block failed with errno 7
WARNING: initial mount of root device /hw/node/io/gio/hpc/scsi_ctlr/0/target/1/lun/0/disk/partition/1/block failed with errno 1011
PANIC: vfs_mountroot: no root found
[Press reset to restart the machine.]
```

At this point I have partitioned the disk using fx.ARCS with `auto` and `sync`.
Not sure if the latter is needed.
Loading `/i/22/ovl1/dist/miniroot/unix.IP22` didn't work anyway. I had more success installing
from the system maintainance menu:
```
System Maintenance Menu

1) Start System
2) Install System Software
3) Run Diagnostics
4) Recover System
5) Enter Command Monitor

Option? 2


                         Installing System Software...

                       Press <Esc> to return to the menu.



1) Remote Tape  2) Remote Directory  X) Local CD-ROM  X) Local Tape  

Enter 1-4 to select source type, <esc> to quit,
or <enter> to start: 2


Enter the name of the remote host: tetra
Enter the remote directory: i/22/ovl1/dist


1) Remote Tape  2)[Remote Directory]  X) Local CD-ROM  X) Local Tape  
      *a) Remote directory i/22/ovl1/dist from server tetra.

Enter 1-4 to select source type, a to select the source, <esc> to quit,
or <enter> to start: 


Setting $netaddr to 192.168.2.177 (from server tetra)
Copying installation program to disk.
Setting $netaddr to 192.168.2.177 (from server tetra)
......... 10% ......... 20% ......... 30% ......... 40% ......... 50% 
......... 60% ......... 70% ......... 80% ......... 90% ......... 100% 

Copy complete
Setting $netaddr to 192.168.2.177 (from server tetra)
Setting $netaddr to 192.168.2.177 (from server tetra)
IRIX Release 6.5 IP22 Version 10070055 System V
Copyright 1987-2003 Silicon Graphics, Inc.
All Rights Reserved.

WD95A SCSI controller 4 - differential external, rev 0, min xfer period 100ns
WD95A SCSI controller 5 - differential external, rev 0, min xfer period 100ns
gfe1: missing
gfe0: missing
xpi0: missing from slot 1
xpi0: missing from slot 0
root on /hw/node/io/gio/hpc/scsi_ctlr/0/target/1/lun/0/disk/partition/1/block ; dumpdev on /dev/swap ; boot swap file on /dev/swap swplo 57000
Creating miniroot devices, please wait...

Current system date is Fri Dec 31 16:26:40 PST 1999

Mounting file systems:

/dev/dsk/realroot: Invalid argument
No valid file system found on: /dev/dsk/realroot
This is your system disk: without it we have nothing
on which to install software.


Make new file system on /dev/dsk/realroot [yes/no/sh/help]: yes
...
```

From this point it went roughly like this:
```
from ..
open ..
keep *
install standard
install prereq
conflicts (to solve any conflicts)
go
exit
```

For the system to find the bootloader I neeed to go into the command monitor and:
```
>> setenv OSLoader sash
```
