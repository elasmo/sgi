# Bootp server preperations
## Unpack warezz
```
mount -o loop -t efs IRIX\ 6.5\ Foundation\ 1.img tmp
cp -a tmp f1
umount tmp
mount -o loop -t efs IRIX\ 6.5\ Foundation\ 2.img tmp
cp -a tmp f2
umount tmp
mount -o loop -t efs IRIX\ 6.5\ Development\ Foundation.img tmp
cp -a tmp devf
umount tmp
mount -o loop -t efs IRIX\ 6.5\ Development\ Libraries.img tmp
cp -a tmp devl
umount tmp
unzip "Irix 6.5.30_cdimages.zip"
mount -o loop -t efs Irix\ 6.5.30_cdimages/Applications.image tmp/
cp -a tmp apps
umount tmp
mount -o loop -t efs Irix\ 6.5.30_cdimages/Complementary_Applications.image tmp
cp -a tmp capps
umount tmp
mount -o loop -t efs Irix\ 6.5.30_cdimages/Instalation_Tools_and_Overlays1.image tmp
cp -a tmp ovl1
umount tmp
mount -o loop -t efs Irix\ 6.5.30_cdimages/Overlays2.image tmp
cp -a tmp ovl2
umount tmp
mount -o loop -t efs Irix\ 6.5.30_cdimages/Overlays3.image tmp
cp -a tmp ovl3
umount tmp
```

```
mv devl devf f1 f2 /home/irix/i/
mv apps capps ovl1 ovl2 ovl3 /home/irix/i/30/
```

## Configuration
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

```
apt-get install openbsd-inetd bootp tftpd rsh-redone-server rsh-redone-client
```

Edit /etc/inetd.conf
```
shell		stream	tcp	nowait	root	/usr/sbin/tcpd	/usr/sbin/in.rshd
bootps		dgram	udp	wait	root	/usr/sbin/bootpd	bootpd -i -t 120
tftp		dgram	udp	wait	nobody	/usr/sbin/tcpd	/usr/sbin/in.tftpd /srv/tftp
```

Edit /etc/bootptab
```
challengeip=192.168.2.177
```

```
systemctl stop NetworkManager
ip addr add 192.168.2.5/24
echo 1 > /proc/sys/net/ipv4/ip_no_pmtu_disc
echo "2048 32767" > /proc/sys/net/ipv4/ip_local_port_range
```

```
systemctl enable inetd
systemctl start inetd
```

Connect to SGI Challenge S
```
# cu -s 9600 -l /dev/cuaU0
...
>> setenv netaddr 192.168.2.177
>> bootp():i/30/ovl1/stand/sash64
ec0: ethernet cable problem
ec0: ethernet cable problem
ec0: ethernet cable problem
ec0: ethernet cable problem
No server for :i/30/ovl1/stand/sash64.  
Your netaddr environment variable may be set incorrectly, or
the net may be too busy for a connection to be made.
Unable to execute bootp():i/30/ovl1/stand/sash64
```
