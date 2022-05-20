## ISO default server installation 18.04 

if you have to install adly server (adyoulike server) with **idrac**, you can use iso image 18.04 with the following preseed:


```bash
auto-inst-adly.seed
```

you can find at the root project the **isolinux** txt.cfg  and the default grub.cfg to boot on automated installation.

```bash
txt.cfg
grub.cfg
```

### How to build the adly-default-server dedicated iso

### Prerequisite

You have to download the default Ubuntu images from canonical Web site 

```bash
wget http://cdimage.ubuntu.com/releases/18.04.3/release/ubuntu-18.04.3-server-amd64.iso
```

Unzip the iso 

```bash
7z -y x ubuntu-18.04.6-server-amd64.iso -osource-files
```

### Modification to adapte the iso for dedicated installation

edit **auto-inst.seed**

```bash
27 # Static network configuration. 
 28 # IPv4 
 29 d-i netcfg/get_ipaddress string 192.168.2.119  <---# you
 30 d-i netcfg/get_netmask string 255.255.255.0        # can set an IP from the default pool IP 
 31 d-i netcfg/get_gateway string 192.168.2.1          #   
 32 d-i netcfg/get_nameservers string 192.168.2.5
 33 d-i netcfg/confirm_static boolean true
 34 
 35 
 36 # Adjust Hostname 
 37 d-i netcfg/hostname string ubuntu-server-18.4-default-installation  <--- change the hostname
```

erase the default [BOOT] folder and copy the default configuration file in folder in **source-files**

```bash
 rm -rf source-files/\[BOOT\] && cp  grub.cfg source-files/boot/grub/ && cp txt.cfg source-files/isolinux/ && cp auto-inst.seed source-files/preseed
```


generate a new md5 for the iso ubuntu-installation **(at the root folder of source-files)**

```bash
find . -path source-files/isolinux -prune -o -type f -not -name md5sum.txt -print0 | xargs -0 md5sum | tee source-files/md5sum.txt
```

create source-files in an iso image

```bash
genisoimage -quiet -D -r -V "ubuntu-autoinstall-testing" -cache-inodes -J -l -joliet-long -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -o  ../autoinstall-adly-18.04.iso .
```
