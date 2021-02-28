.
# Inventory if you aren't doing the subnet thing

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

# For the love of good do Ubuntu not Raspbian for all the following

### prep OS install

use etcher & the ubntu light image

### Raspbian only: with card still in desktop, add ssh to root of boot

`touch /Volumes/boot/ssh`

ubuntu - skip

### Put the card in the pi :-)

### Watch it boot on your router (MASTER)

assign `192.168.86.33` static IP to the new device on router
restart like a mug if it's ubuntu

For the workers you can see it boot up from the ssh on the master
`cat /var/lib/misc/dnsmasq.leases`

### reset password of `pi` or `ubuntu` user

`passwd`

*it just happens on ubuntu* 

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
```
sudo reboot 
```


### disable swap or be in a world of pain later

raspbian:
```
sudo dphys-swapfile swapoff && \
  sudo dphys-swapfile uninstall && \
  sudo update-rc.d dphys-swapfile remove
```
and then...
`sudo systemctl disable dphys-swapfile`

ubuntu:
```
sudo swapoff -a 
```


