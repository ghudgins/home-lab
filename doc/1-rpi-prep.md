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


### prep OS install

use etcher & the ubntu light image

### with card still in desktop, add ssh to root of boot
raspian
`touch /Volumes/boot/ssh`

ubuntu
(don't do this)

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


### disable swap 

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


# OLD DOC IS BELOW READ ONLY IF YOU WANT TO DO PRIVATE NETWORK


### Manually setup private subnet

follow this guide: https://downey.io/blog/create-raspberry-pi-3-router-dhcp-server/

- define a new network for the private cluster https://downey.io/blog/create-raspberry-pi-3-router-dhcp-server/
- install dns mask 
- configure it...including mac addresses & static IPs for the other pis
  
**msmiller** - dc:a6:32:bf:5d:16 (FYI you won't be using this here)

**frank** - dc:a6:32:bf:5c:34 --> 10.0.0.50
**richard** - dc:a6:32:bf:5d:39 --> 10.0.0.51

### Observe and fix the now failing coredns pods - this step is hacky
`kubectl -n kube-system edit configmap coredns `

remove/comment `loop`

delete the coredns pods so they rebuild
`kubectl -n kube-system delete pod -l k8s-app=kube-dns `

_more info here https://www.reddit.com/r/kubernetes/comments/9fl7cg/coredns_crashloopbackoff_on_fresh_install/_ 

### Do setup for workers

(follow the top of this doc)
 - ssh only (no wifi)
 - password
 - hostname
 - culture
 - disable swap 

### on workers, edit the boot commands

`sudo apt install vim`

`sudo vim /boot/cmdline.txt`

add this to the end of the top line
`cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`

YOU MUST REBOOT
`sudo reboot`

### Install docker
`curl -sSL get.docker.com | sh`

then add yourself to the docker group
`sudo usermod pi -aG docker`

then refresh groups
`newgrp docker`

```
TODO Address docker error in k8s 
```

### Install k8s manually y'all

install 1.17.0 that rak8s uses (check this though)

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubelet=1.17.0-00 kubectl=1.17.0-00 kubeadm=1.17.0-00
```

in case you need it, here's how to print versions
`curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep Version | awk '{print $2}'`


pull kubeadm
`sudo kubeadm config images pull -v3`

### Join the existing cluster

Generate new token from master
`kubeadm token create --print-join-command`

From the master generate the token

(you can see others available but will be missing cert hash)
`kubeadm token list`

generate new
`sudo kubeadm join --token udy29x.ugyyk3tumg27atmr 192.168.86.33:6443`

copy the join command like
`kubeadm join 192.168.86.33:6443 --token vf4gvr.5a9vjyjbyvz68gxg     --discovery-token-ca-cert-hash sha256:917a7c3f7e86322dcf138844ef6d9ca2e3315c216a7c7c5557f5a85b6295e757`

### set bridge tables
`sudo sysctl net.bridge.bridge-nf-call-iptables=1`

### Setup reverse ssh tunnel so we can ssh from desktop to worker

autossh -o StrictHostKeyChecking=no -i /home/pi/.ssh/id_rsa -fN -R <port>:localhost:22 <user>@<remotehost>

autossh -o StrictHostKeyChecking=no -i /home/pi/.ssh/id_rsa -fN -R 13003:localhost:22 ghudgins@192.168.86.189


### Install docker and k8's on workers





kubeadm join 192.168.86.33:6443 --token iv6nxx.j015mf7fb1rm3gt2 \
    --discovery-token-ca-cert-hash sha256:14e39374832fe2fa9f8acc5e636a22349be988f62a9eedd1a3cdd47ef372ee2a



curl -4 http://192.168.86.33:31118 -d "# test"
curl -4 "http://192.168.86.33:8080" -d "# test"
curl -4 http://192.168.86.33:32598 -d "# test"
32598/TCP
for ep in 10.244.1.6; do
    wget -qO- $ep
done

  helm install stable/traefik --generate-name --set dashboard.enabled=true,serviceType=NodePort,dashboard.domain=dashboard.traefik,rbac.enabled=true,externalIP=192.168.86.33 --namespace kube-system



 kubectl describe node

    Type     Reason                                                       Age                 From                    Message
  ----     ------                                                       ----                ----                    -------
  Warning  listen tcp4 192.168.86.33:443: bind: address already in use  36m (x7 over 119m)  kube-proxy, kubemaster  can't open "externalIP for kube-system/traefik-1598809726:https" (192.168.86.33:443/tcp), skipping this externalIP: listen tcp4 192.168.86.33:443: bind: address already in use
  
  Warning  listen tcp4 192.168.86.33:80: bind: address already in use   36m (x7 over 119m)  kube-proxy, kubemaster  can't open "externalIP for kube-system/traefik-1598809726:http" (192.168.86.33:80/tcp), skipping this externalIP: listen tcp4 192.168.86.33:80: bind: address already in use


       "Network": "10.244.0.0/16",



       kubectl edit cm -n kube-system kube-flannel-cfg




sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 10.96.143.42


sudo iptables -t nat -A POSTROUTING -p tcp -d 10.96.143.42 --dport 80 -j SNAT --to-source 192.168.86.33