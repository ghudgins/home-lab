
# Inventory 

**msmiller** - dc:a6:32:bf:5d:15 - 192.168.86.33

Understudies (HA):
**peggy** - dc:a6:32:39:82:03 - 192.168.86.36
**jerry** - dc:a6:32:5c:71:fd - 192.168.86.37

workers:
**richard** - dc:a6:32:bf:5d:39 - 192.168.86.34
**frank** -  dc:a6:32:bf:5c:32 - 192.168.86.35
**peggy** - b8:27:eb:d3:71:fd - 192.168.86.36


order on rack (top to bottom):
 - jerry
 - frank
 - richard
 - msmiller

 clear enclosure:
 - peggy


### prep OS install

use etcher & the ubntu light image

### Put the card in the pi

### Watch it boot on your router (MASTER)

assign `192.168.86.33` static IP to the new device on router

restart 

For the workers you can see it boot up from the ssh on the master
`cat /var/lib/misc/dnsmasq.leases`


### ssh-copy-id for speed connecting

_**id_rsa_rak8s.pub** is a key that does not contain a password. for security reasons remove this after you complete the setup and use a key that requires a password._

`ssh-copy-id -i ~/.ssh/id_rsa_rak8s.pub ubuntu@192.168.86.33`

### change hostname

raspbian:
`sudo raspi-config`

ubuntu:
```
sudo hostnamectl set-hostname peggy
```

then add yourself to 
```
sudo vim /etc/hosts
```

reboot


### disable swap or be in a world of pain later

ubuntu:
```
sudo swapoff -a 
```


