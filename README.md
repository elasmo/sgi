# IRIX

This wiki is a bit of a mess at the moment, but, hey, better than nothing!
This should be split up into several pages, rather than this big one...

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

Display network interface information:

`netstat -ian`

Change the primary network interface:
```
vi /etc/config/netif.options
...
if1name=ec3
...
```
## Various IRIX Commands
| Command | Example | Description |
|---------|---------|-------------|
| `uname` | `uname -R` | Get release name |
| `hinv` | `hinv` | List hardware inventory. Use `-vv` for details |
| `nvram` | `nvram` | List NVRAM variables (including MAC address) |

## Packages
### Inst
`inst` is a command-line tool for managing software on IRIX. Packages ("distributions") come in the form of files ending with the `.tardist` extension. These files are `tar` archives which contain specially crafted binary files, along with a human-readable manifest. The `inst` program can be invoked without arguments to present a selection menu, or can be used directly from the CLI. 
One can for example directly install a `tardist` file either by:
```
inst -f <foo.tardist>
Inst > go
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
* You want a DIN-8 to DB9 adapter to connect an Indy or Challenge S to a serial terminal

# Bootup & Command Monitor (PROM)
SGI boxen boot into something called the Command Monitor, which is equivalent to Sun's OpenBoot, which itself was a precursor to EFI. The equivalent on a PCI before EFI would be the BIOS. As with these systems, the purpose of the command monitor is to configure the basic system, and boostrap an operating system from disk or network. You can recognise the command monitor: it appears as a single terminal in the middle of the screen during bootup. You can enter the command monitor startup menu by pressing `ESC` at the start of bootup, or, on some PROMs, click on the `Stop for Maintenance` button during the `Starting up the system...` popup. You will then be presented with a  number of options, such as

* Start System
* Install System Software
* Run Diagnostics
* Recover System
* Enter Command Monitor
* Select Keyboard Layout

By selecting *Enter Command Monitor*, you will land in the PROM CLI, which allows you to set up boot partitions, debug the system, and program NVRAM.

## NVRAM MAC Recovery
TODO

# Customisation

## Timezone
The simplest way to configure the timezone is to run the language configuration program from the *toolchest*. It can be found under *Desktop->Customize->Language*. This will also take daylight savings into account, and the resulting timezone string in `/etc/TIMEZONE` will be impressively complex.

You can of course edit the `TZ` variable in `/etc/TIMEZONE` by hand, but let me tell you, the syntax is baffling:
```
stdoffset<b>[dst<b>[offset<b>],[start<b>[/time<b>],end<b>[/time<b>]]]
```
Where 
* `std` and optionally `dst` is the timezone abbreviation for standard time (e.g `CET`, `BST` and so on) and daylight savings time respectively.
* `offset` is the number of hours we're away from [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), be it positive or negative, and can contain hours and optionally minutes and secondds (`hh<b>[:mm<b>[:ss<b>]]`)
* `start/time,end/time` specifies when to switch to and switch back from daylight savings, in three different formats. See the `environ(5)` manpage.

The string produced by the language configuration program for Sweden is `CET-1CEST-2,M3.5.0/2,M10.5.0/3`. I read this as
* Standard time = `CET` (Central European Time) which is `UTC` minus 1 hour
* Daylight Savings = `CEST` (Central European Standard Time) which `UTC` minus two hours
* Daylight savings starts on the first (0th) weekday of the last (5th) week of the third month (March)
* Daylight savings ends on the first (0th) weekday of the last (5th) week of the tenth month (October)
* I have no idea what `/2` and `/3` mean, haha!

I encourage you to explore the timezone files in `/usr/lib/locale/TZ/`, they're actually full of fascinating source material and historical information about how different countries came to have the time schemes that they have. See also the last reference ("Guide for setting up TIMEZONE") for more excruciating details.

#### References

* `/usr/lib/locale/TZ/` timezone files
* [IRIX (R) Admin System Configuration and Operation (008-2859-005)](https://irix7.com/techpubs/007-2859-005.pdf])
* [Guide for setting up TIMEZONE settings in IRIX 6.x](http://cspry.co.uk/computing/Indy_admin/TIMEZONE.html)
* [environ(5)](http://nixdoc.net/man-pages/IRIX/man5/environ.5.html) manpage

## Toolchest
Toolchest is the name of the main menu program on the [IRIX Interactive Desktop](https://en.wikipedia.org/wiki/IRIX_Interactive_Desktop). It's located on the upper left corner of the standard desktop, and consists of a number of menus and submenus.

### Customising The Toolchest
The `toolchest` program is automatically launched from FOO, and reads a number of configuration files:

|Path|Purpose|
|----|-------|
|`/usr/lib/X11/system.chestrc`|The system-wide default menu|
|`/usr/lib/X11/app-chests/*.chest`|Additional configuration files which are added to the system-default menu|
|`$HOME/.chestrc`|User-specific default menu, which *overrides* the aforementioned system-wide configuration|
|`$HOME/.auxchestrc`|User-specific configuration, which is *added* to the aforementioned configurations|
|`/usr/lib/X11/remote.chestrc|`|Alternate configuration file used by remote logins (via `/usr/sbin/accessworkstation` or `Toolchest->Desktop->Accessfiles`)|
|`/usr/lib/X11/app-defaults/Toolchest`|System-wide `.resource` file|

In practice, you probably just want to edit `/usr/lib/X11/system.chestrc` to get rid of unwanted defaults, and then add your own menus via `$HOME/.auxchestrc`. The configuration file format is quite flexible and won't seem that outlandish if you've used something like *fluxbox* or *pekwm*. Editing the main configuration file should give you a good idea on how to customize your menu, but here's an example:

```
# Create a menu containing some items. This will be automatically added to the top-level menu, ToolChest
Menu FooBar
{   
	# Invoke xmessage when you click on 'Say Hello'
	"Say Hello"  f.checkexec.sh "/usr/bin/X11/xmessage Hello"
    
    # Add a separator
	no-label  f-separator
    
	# Same as above, but show an animation (le = launch effect) when launched
    "Say Hello With Effect"  f.checkexec.sh.le "/usr/bin/X11/xmessage Hello" 
    
    # Add a separator    
    no-label  f-separator
    
    # Add a reference to a submenu
    "Hax" f.menu Hax
}

Menu Hax
{
    "XLoad" f.checkexec.sh.le "/usr/bin/X11/xload"
}

# Add a simple button to the end of the top-level menu
Menu ToolChest
{
	"Ladida" f.checkexec.sh.le "/usr/bin/X11/xmessage Ladida" 
}
```

I've not yet figured out *what* actually launches `toolchest` (or anyt other  default components of the desktop to be honest), so that's a pending task. Having a version of `grep` that supports the `-r` (recursive) option sure would be helpful ;)
The *resource* file can probably be used to modify some standard options (see the manpage) as well.


#### References

* [IRIX Interactive Desktop](https://en.wikipedia.org/wiki/IRIX_Interactive_Desktop) on wikipedia
* [Toolchest(1X)](http://nixdoc.net/man-pages/IRIX/man1/toolchest.1.html) manpage#

# Machine specifics

## Challenge S
Basically a server version of an Indy. Does not have an installed graphics card.
Connect to the machine by using a serial terminal: `cu -s 9600 -l /dev/cuaU0`.

To install this machine over the network you'll likely want an ethernet transducer
to connect the built-in AUI interface to a modern network. The machine does not
support booting using a secondary network card.

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

# Poorly Sorted References

### Command Monitor (PROM)
* http://csweb.cs.wfu.edu/~torgerse/Kokua/SGI/007-2859-021/sgi_html/ch09.html
* [Indy NVRAM reprogramming](https://wiki.preterhuman.net/Indy_NVRAM_reprogramming)

### SGI
* [SGI FAQ](https://www.opennet.ru/docs/FAQ/OS/SGI/graphics.html)
* [SGI in Hollywood and Gaming](http://www.sgistuff.net/funstuff/hollywood/index.html)
* [SGI Advice and Technical Data](http://www.sgidepot.co.uk/sgi.html)
* [http://www.computinghistory.org.uk/det/11261/SGI-Indy/](SGI Indy Information) on computinghistory.org
* [3D glasses for SGI](https://www.geektechnique.org/projectlab/851/making-3d-glasses-for-a-silicon-graphics.html)
* [The IRIX Network](https://irix.cc) - community-driven forum (still active in of 2019)
* [SGI graphics Frequently Asked Questions (FAQ)](https://www.opennet.ru/docs/FAQ/OS/SGI/graphics.html)

### IRIX

#### General (unsorted)
* [IRIX 6.2 Technical Report](http://www.sgistuff.net/software/irixintro/documents/irix6.2TR.html) (specs, ABI, standards, filesystems, ...)
* [Helpful commands in IRIX](https://wiki.preterhuman.net/Helpful_commands_in_IRIX)
* [MOTIF FAQ](https://babbage.cs.qc.cuny.edu/courses/GUIDesign/motif-faq.html#130) (useful for build problems relating to [Motif](https://en.wikipedia.org/wiki/Motif_(software)))
* [MIPSPro Compiler Suite](https://web.archive.org/web/20180512081605/http://www.nekochan.net/wiki/MIPSpro) wiki entry on NekoChan
* [IRIS/IRIX System Administration Environment Variables](http://retrogeeks.org/sgi_bookshelves/SGI_EndUser/books/IRIX_EnvVar/sgi_html/ch02.html)
* [Guide for configuring an SGI Indy with IRIX 6.5](https://cspry.co.uk/computing/Indy_admin/configuration.html) ("Idiot's guide")
* [Denag](https://github.com/dcantrell/denag) - get rid of the MIPSPro license nag screen
* [Use Custom Background on Irix](https://wiki.preterhuman.net/Use_Custom_Background_on_Irix)
* [IRIX Interactive Desktop](https://en.wikipedia.org/wiki/IRIX_Interactive_Desktop) on Wikipedia
* [XLI package description](https://web.archive.org/web/20100120130550/http://freeware.sgi.com/Installable/xli-1.16-sgipl1.html)
* [Building IRIX tardists](https://vanalboom.org/node/7)
* [xli-1.16: description + notes](https://web.archive.org/web/20100120130550/http://freeware.sgi.com/Installable/xli-1.16-sgipl1.html)
* [BACKGROUND(1)](http://nixdoc.net/man-pages/IRIX/man1/background.1.html)
* [Use Custom Background on Irix](https://wiki.preterhuman.net/Use_Custom_Background_on_Irix)
* [Using the Command (PROM) Monitor](http://csweb.cs.wfu.edu/~torgerse/Kokua/SGI/007-2859-021/sgi_html/ch09.html)
* [prom(1) manual page](https://nixdoc.net/man-pages/IRIX/man1/prom.1.html)
* [Indy NVRAM MAC Programming](https://wiki.preterhuman.net/Indy_NVRAM_reprogramming)
* [Fsn file manager](https://en.wikipedia.org/wiki/Fsn_(file_manager)) on Wikipedia
* [MIPS R5000: fast, Affordable 3-D](http://www.sgidepot.co.uk/byte.html) (Byte Magazine aricle, May 1996)


#### Install
* [irixboot](https://github.com/halfmanhalftaco/irixboot) debian-based [Vagrant](https://en.wikipedia.org/wiki/Vagrant_(software)) box with bootp + tftpd
* [Remote, network installation of SGI IRIX 6.5](http://techpubs.spinlocksolutions.com/irix/remote-irix-6.5-installation-from-linux.html) from a GNU/Linux install server
* [Installation Guide for SGI IRIX](http://rmni.iqfr.csic.es/guide/man/instguide/contents.htm)
* [Installing IRIX over the network](https://software.majix.org/irix/install-network.shtml)
* [Booting IRIX Installation from CD](https://software.majix.org/irix/install-bootcd.shtml)
* [Installation Guide](http://www.sgistuff.net/software/tipstricks/guide_install.html) Tips & Tricks
* [Getting an Indy Desktop](https://blog.pizzabox.computer/posts/getting-an-indy-desktop/) (replacing disk with SD card, partitioning, netboot, install)
* [Installing IRIX 6.5](http://www.futuretech.blinkenlights.nl/6.5inst.html) ([mirror](http://www.sgidepot.co.uk/6.5inst.html))
* [Scratch install of IRIX 6.5 Base Release](http://rmni.iqfr.csic.es/guide/man/instguide/chap3-5.htm)
* [Scratch install of IRIX 6.5 Base + 6.5.3 Intermediate Release](http://rmni.iqfr.csic.es/guide/man/instguide/chap3-6.htm)
* [Installing IRIX 6.5 Across a Network](http://www.futuretech.blinkenlights.nl/netboot.html) 
* [Installing IRIX 6.5](http://www.futuretech.blinkenlights.nl/6.5inst.html)

##### Software
* [SGI/IRIX CDs](https://jrra.zone/sgi/)
* [IRIX File Archive](http://retrogeeks.org/sgi/files) (mostly production software like Adobe Photoshop)
* [Index of /nekoware/nekoware-mips3/current/](http://nekofiles.irixnet.org/nekoware/nekoware-mips3/current/) NekoWare tardist mirror
* [efs2tar](https://github.com/sophaskins/efs2tar/tree/master/efs), a utility for converting efs images (i.e `.img`-files found on archive.org) to tarballs. See also [A yak shave with SGI's EFS](https://blog.pizzabox.computer/posts/sgi-efs-yakshave/)
* [ports.sgi.sh](http://ports.sgi.sh/) - some useful ported software, such as [GCC](https://en.wikipedia.org/wiki/GNU_Compiler_Collection) along with binutils, and python3.5. See also the [SGIDEV Network Wiki](http://www.sgidev.org/wiki/IRIX_Binaries.html)
* [Simple ports system for IRIX that's designed to be able to run on a fresh install.](https://github.com/larb0b/irixports)
* [WinWorld IRIX Software Mirror](https://winworldpc.com/product/irix/65) (IRIX install CD's and patches)

##### Parts
* [Ian's SGI Depot](http://www.sgidepot.co.uk/sgidepot/partsspares.html#INDY)

