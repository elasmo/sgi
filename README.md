# IRIX
## Directories
| What | Where | Description |
|------|-------|-------------|
| Home directories | `/usr/people/` |
| Logs | `/var/adm/` | Contains files such as `SYSLOG`, `lastlog`, `wtmp` and `crash` |
| 

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
IRIX 6.5.22 uses a crypt() implementation from 1988. A 31 years old
rock solid DES and Enigma implementation.

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

# Resources
* http://www.sgidepot.co.uk/sgi.html
* https://irix.cc
* [Building IRIX tardists](https://vanalboom.org/node/7)
* [xli-1.16: description + notes](https://web.archive.org/web/20100120130550/http://freeware.sgi.com/Installable/xli-1.16-sgipl1.html)
* [BACKGROUND(1)](http://nixdoc.net/man-pages/IRIX/man1/background.1.html)
* [Use Custom Background on Irix](https://wiki.preterhuman.net/Use_Custom_Background_on_Irix)
