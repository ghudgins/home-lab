### Manually install container.d & kubeadm / kubectl / kubelet

##### Install container.d

Follow these configuration AND installation instructions
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

Then install it
`sudo apt-get update -y`
`sudo apt-get install -y containerd`

then start it 
`sudo systemctl enable --now containerd`

_todo: you might have to do something to add it to system start_


##### SKIP: Docker steps for 
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### SKIP: Set docker to use systemd

```
# Set up the Docker daemon
sudo vim /etc/docker/daemon.json

EOF>>>
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

and then reboot

`sudo systemctl daemon-reload`
`sudo systemctl restart docker`


##### CGROUPS settings in boot

Ubuntu:
```
sudo vim /boot/firmware/cmdline.txt
```

...add to end of the line
```
 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

reboot

### Install k8s


Install the version you want

For specific versions use this guide https://stackoverflow.com/questions/49721708/how-to-install-specific-version-of-kubernetes

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubelet=1.19.3-00 kubectl=1.19.3-00 kubeadm=1.19.3-00
```


### HA (Primary nodes only)

* install keepalived
* install haproxy

**enable a floating IP**
```
sudo vim /etc/sysctl.conf
```

...add to top of file
```
net.ipv4.ip_nonlocal_bind=1
```

...make it so
```
sudo sysctl -p
```

**edit keepalived conf on primary**

```
sudo vim /etc/keepalived/keepalived.conf
```

```
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 103
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        192.168.86.39
    }
    track_script {
        check_apiserver
    }
}
```

**slightly different configuration for master node 2 (priority is lower)**

```
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 102
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        192.168.86.39
    }
    track_script {
        check_apiserver
    }
}
```

**slightly different configuration for master node 3 (priority is even lower)**

```
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 101
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        192.168.86.39
    }
    track_script {
        check_apiserver
    }
}
```
**Check script**
```
sudo vim /etc/keepalived/check_apiserver.sh
```

```
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:192.168.86.39/ -o /dev/null || errorExit "Error GET https://localhost:6443/"
if ip addr | grep -q 192.168.86.39; then
    curl --silent --max-time 2 --insecure https://192.168.86.39:6443/ -o /dev/null || errorExit "Error GET https://192.168.86.39:6443/"
fi
```

**Config haproxy**
```
sudo vim /etc/haproxy/haproxy.cfg
```

```
# /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s

#---------------------------------------------------------------------
# apiserver frontend which proxys to the masters
#---------------------------------------------------------------------
frontend apiserver
    bind *:6443
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server msmiller 192.168.86.33:7300 check
        server peggy 192.168.86.36:7300 check
        server jerry 192.168.86.37:7300 check
```


### Initialize primary (once) 

```
kubeadm init \
--apiserver-advertise-address=192.168.86.39 \
--control-plane-endpoint=192.168.86.39:6443 \
--pod-network-cidr=10.244.0.0/16 \
--upload-certs
--dry-run
```


### Apply network config (flannel)
```
 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

One time we had to delete the interface when the pods wouldn't start. didn't happen the latest time through this doc
```
sudo ip link delete flannel.1
```

### Join as masters or workers
(copy the output from above step for the relevant actor)


Workers, replace the tokens by printing a join command from another node

`kubeadm token create --print-join-command`

add this to the end of what the join command prints
```
--cri-socket /run/containerd/containerd.sock
```
### Copy config so you can kubectl from your desktop

```
scp -r ubuntu@msmiller:~/.kube /Users/ghudgins/
```



