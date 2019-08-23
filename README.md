# IRIX
## Directories
| What | Where | Description |
|------|-------|-------------|
| Home directories | `/usr/people/` |
| Logs | `/var/adm/` | Contains files such as `SYSLOG`, `lastlog`, `wtmp` and `crash` | 

## Networking
Set hostname:

`/etc/sys_id`

Set DNS:

`/etc/resolv.conf`

List enabled and disabled services and daemons:

`chkconfig`

## Various IRIX Commands
| Command | Example | Description |
|---------|---------|-------------|
| `uname` | `uname -R` | Get release name |
| `hinv` | `hinv` | List hardware inventory |
| `nvram` | `nvram` | List NVRAM variables (including MAC address) |

## Packages
### Inst
`inst` is a command-line tool for managing software on IRIX. Packages ("distributions") come in the form of files ending with the `.tardist` extension. These files are `tar` archives which contain specially crafted binary files, along with a human-readable manifest. The `inst` program can be invoked without arguments to present a selection menu, or can be used directly from the CLI. 
One can for example directly install a `tardist` file either by:
```
inst -f <foo.tardist>
``` 
Or by invoking:
```
inst
Inst > from foo.tardist
Inst > go
```
## Other commands
There seem to be a plethora of other commands, most of which we haven't tested yet, but have seen mentioned elsewhere:
`versions`, `showfiles`, `showprods`.

## Random jazz
* IRIX 6.5.22 uses a crypt() implementation from 1988 including support for DES and Enigma
* You'l likely want a DIN-8 to DB9 adapter to connect an Indy or Challenge S to a serial terminal

# Bootup & Command Monitor (PROM)
SGI boxen boot into something called the Command Monitor, which is equivalent to Sun's OpenBoot, which itself was a precursor to EFI. The equivalent on a PCI before EFI would be the BIOS. As with these systems, the purpose of the command monitor is to configure the basic system, and boostrap an operating system from disk or network. You can recognise the command monitor: it appears as a single terminal in the middle of the screen during bootup. You can enter the command monitor startup menu by pressing `ESC` at the start of bootup, or, on some PROMs, click on the `Stop for Maintenance` button during the `Starting up the system...` popup. You will then be presented with a  number of options, such as

* Start System
* Install System Software
* Run Diagnostics
* Recover System
* Enter Command Monitor
* Selct Keyboard Layout

By selecting *Enter Command Monitor*, you will land in the PROM CLI, which allows you to set up boot partitions, debug the system, and program NVRAM.

## NVRAM MAC Recovery
TODO

# Machine specifics

## Challenge S
TODO: Install OS

Basically a server version of an Indy. Does not have an installed graphics card.
Connect to the machine by using a serial terminal: `cu -s 9600 -l /dev/cuaU0`.

```
>> hinv
                   System: IP22
                Processor: 175 Mhz R4400, with FPU
     Primary I-cache size: 16 Kbytes
     Primary D-cache size: 16 Kbytes
     Secondary cache size: 1024 Kbytes
              Memory size: 192 Mbytes
                SCSI Disk: scsi(0)disk(1)
```

## Indy
TODO: Fix Dallas battery, install OS

Defective dallas chip (i.e. doesn't remember MAC address etc.)

```
>> setenv netaddr 10.0.0.2
>> bootp():
ec0: bad ethernet address ff:ff:ff:ff:ff:ff
Unable to execute bootp():
```

```
>> hinv
                   System: IP22
                Processor: 134 Mhz R4600, with FPU
     Primary I-cache size: 16 Kbytes
     Primary D-cache size: 16 Kbytes
     Secondary cache size: 512 Kbytes
              Memory size: 160 Mbytes
                 Graphics: Indy 8-bit
                    Audio: Iris Audio Processor: version A2 revision 4.1.0
```

## Indy 2nd
The most high end of all the Indy's.

Probably a defective Dallas chip, the MAC address shows up as 'bad'.

```
>> hinv
                   System: IP22
                Processor: 180 Mhz R5000, with FPU
     Primary I-cache size: 32 Kbytes
     Primary D-cache size: 32 Kbytes
     Secondary cache size: 512 Kbytes
              Memory size: 256 Mbytes
                 Graphics: Indy 24-bit
                SCSI Disk: scsi(0)disk(1)
                    Audio: Iris Audio Processor: version A2 revision 4.1.0
```

Boot from console
```
>> auto


                           Starting up the system...

Loading scsi(0)disk(1)rdisk(0)partition(8)/sash: 136336+22752+3248+341792+49344d+4620+6880 entry: 0x97fa5ee0
IRIX Release 6.5 IP22 Version 10070055 System V
Copyright 1987-2003 Silicon Graphics, Inc.
All Rights Reserved.

ec0: machine has bad ethernet address: 04:91:a2:91:c3:23
The system is coming up.

network: WARNING: Failed to configure ec0 as indy1.
network: WARNING: Cannot access primary interface, ec0.
Using standalone network mode.
ifconfig: ioctl (SIOCGIFFLAGS): no such interface
Warning:  Internet Gateway web server running as root.
          Use "chkconfig webface_apache off" to disable.
inst: 
inst: Software installation has installed new configuration files and/or saved
inst: the previous version in some cases.  You may need to update or merge
inst: old configuration files with the newer versions.  See the "Updating
inst: Configuration Files" section in the versions(1M) manual page for details.
inst: The shell command "versions changed" will list the affected files.
inst: 
inst: These directories were unable to be moved properly during the
inst: installation process.  Check for any user-modified files, then
inst: delete the directories.
inst:    /usr/include/Vk.O
Privilege separation user sshd does not exist



indy1 console login: root
IRIX Release 6.5 IP22 indy1
Copyright 1987-2003 Silicon Graphics, Inc. All Rights Reserved.
Last login: Sun Nov  8 03:12:54 CET 2009 on :0
TERM = (vt100)
indy1 5# uname -aR
IRIX indy1 6.5 6.5.22m 10070055 IP22
...
```

# Resources
* http://www.sgidepot.co.uk/sgi.html
* https://irix.cc
* [Building IRIX tardists](https://vanalboom.org/node/7)
* [xli-1.16: description + notes](https://web.archive.org/web/20100120130550/http://freeware.sgi.com/Installable/xli-1.16-sgipl1.html)
* [BACKGROUND(1)](http://nixdoc.net/man-pages/IRIX/man1/background.1.html)
* [Use Custom Background on Irix](https://wiki.preterhuman.net/Use_Custom_Background_on_Irix)
* [SGI graphics Frequently Asked Questions (FAQ)](https://www.opennet.ru/docs/FAQ/OS/SGI/graphics.html)
* [Using the Command (PROM) Monitor](http://csweb.cs.wfu.edu/~torgerse/Kokua/SGI/007-2859-021/sgi_html/ch09.html)
* [Indy NVRAM MAC Programming](https://wiki.preterhuman.net/Indy_NVRAM_reprogramming)
* http://www.sgidepot.co.uk/6.5inst.html
* http://www.sgidepot.co.uk/byte.html
* [Fsn file manager (Wikipedia)](https://en.wikipedia.org/wiki/Fsn_(file_manager))
* [3D glasses for a SGI](https://www.geektechnique.org/projectlab/851/making-3d-glasses-for-a-silicon-graphics.html)
* http://www.sgistuff.net/hardware/systems/documents/007-9804-060-indy.pdf
* http://www.sgistuff.net/hardware/systems/documents/007-2314-005-challenges.pdf
* http://www.computinghistory.org.uk/det/11261/SGI-Indy/

