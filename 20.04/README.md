## ISO default server  installation 20.04

if you have to install server (adyoulike server) with **idrac**, you can use iso image 20.04 build with cloud-init:

**Note**: Since 20.04 Canonical use **cloud-init**. On this same repo, you have the old-way, 18.04 personnal iso with debian preseed package.

To use **cloud-init**, you have to download cloud-init package on Official repo from Canonical:

```bash
sudo apt-get install cloud-init
```

#### How to build 20.04 image 

When you clone this repo, you can find the following file and folder

```bash
.
├── grub.cfg <-- Default configuration of the bootLoader
├── meta-data
├── Readme.md
├── txt.cfg <--- default Isolinux boot
└── user-data <--- ISO configuration 
```

### Prerequisite

You have to download the default Ubuntu images from canonical Web site 

```bash
wget -N https://cdimage.ubuntu.com/ubuntu-server/focal/daily-live/current/focal-live-server-amd64.iso
```

Unzip the iso 

```bash
mkdir source-files && 7z -y x  focal-live-server-amd64.so -osource-files
```

### Modification to adapte the iso for dedicated installation

```bash
   network:
    version: 2
    renderer: networkd
    ethernets:
        enp65s0f0:
            addresses: []
            dhcp4: false
            dhcp6: false
        enp65s0f1:
            addresses: []
            dhcp4: false
            dhcp6: false
        eno3:
            addresses: []
            dhcp4: false
            dhcp6: false
            optional: true
        eno4:
            addresses: []
            dhcp4: false
            dhcp6: false
            optional: true
    bonds:
        bond0:
            addresses: []
            interfaces:
            - enp65s0f0
            - enp65s0f1
            parameters:
                mode: active-backup
                mii-monitor-interval: 100
    vlans:
        bond0.2000:
            addresses: [ ]  <-- You have to modify here before to compil in new image ISO a
            gateway4: 
            id: 2000
            nameservers:
                search: [ domain.tld ]
                addresses:
                    - " "
                    - " "
            link: bond0
```

Check you feature iso configuration 

```bash
cloud-init devel schema --config-file user-data
```

if the config is valid

```bash
 cloud-init devel schema --config-file user-data
Valid cloud-config: user-data
```

Prepare the futur ISO

```bash
 rm -rf source-files/\[BOOT\] && cp  grub.cfg source-files/boot/grub/ && cp txt.cfg source-files/isolinux/ && mkdir source-files/server && cp user-data source-files/server && cp meta-data source-files/server
``` 

You  have to regenerate a new md5sum to avoid the check integrity

```bash
find . -path source-files/isolinux -prune -o -type f -not -name md5sum.txt -print0 | xargs -0 md5sum | tee source-files/md5sum.txt
```

create an ISO image at the root folder of **source-files**

```bash
genisoimage -quiet -D -r -V "ubuntu-autoinstall" -cache-inodes -J -l -joliet-long -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -o  ../autoinstall-adly-20.04.iso .
```


