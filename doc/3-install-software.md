### Install nginx-controller

 - install using the bare metal instructions maybe https://kubernetes.github.io/ingress-nginx/deploy/
 - `kubectl edit service/ingress-nginx-controller -n ingress-nginx` and change the type of the service to `LoadBalancer` so metallb will grant an external IP https://stackoverflow.com/questions/64193967/k8s-nginx-ingress-take-my-node-ip-as-address
 - deploy the ingress and customize it further if necessary https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/


### Install cert manager

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.3/cert-manager.yaml (check for latest)

### Limit size of syslog on the masters? /shrug

```
sudo vim /etc/logrotate.d/rsyslog
```

```
/var/log/syslog
{
        rotate 7
        size 100k
        daily
        missingok
        notifempty
        delaycompress
        compress
        postrotate
                /usr/lib/rsyslog/rsyslog-rotate
        endscript
}

/var/log/mail.info
/var/log/mail.warn
/var/log/mail.err
/var/log/mail.log
/var/log/daemon.log
/var/log/kern.log
/var/log/auth.log
/var/log/user.log
/var/log/lpr.log
/var/log/cron.log
/var/log/debug
/var/log/messages
{
        rotate 4
        size 100k
        weekly
        missingok
        notifempty
        compress
        delaycompress
        sharedscripts
        postrotate
                /usr/lib/rsyslog/rsyslog-rotate
        endscript
}
```
### Install duckdns adapter in the kube-system namespace

Generate base64 secret
```
echo 'token-from-duckddns' | base64
```

...add that to manifest temporarily
```
kubectl create -f duckdns-secret.yaml -n kube-system
```

then make the pod
```
kubectl create -f duckdns-docker.yaml -n kube-system
```


### Rook-Ceph Install

prep the hardware
```
sudo sgdisk --zap-all /dev/sdb
```

taint a non-primary master node so we have 3 workers (or go buy a new pi so we have 3x3 instead of 3x2)

```
kubectl taint node nodename node-role.kubernetes.io/master:NoSchedule-
```
- install common.yaml
- install operator.yaml
  - there are modifications to this file checked into github especially for raspberry pi
- install cluster.yaml
- install the ceph tools and run `ceph status` to see it running

### Create storageclasses

Create both in directory

set the default after you apply

```
kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Monitoring (WIP)


Deploy dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

Make a service user
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

generate token
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

### Protonmail-Bridge

 - Deploy the PVC
 - Deploy the protonmail bridge
 - go interactive 
```
kubectl exec --stdin --tty protonmail-bridge-deployment-6c79fd7f84-ftwcw -- /bin/bash
```
- set yourself up
```
bash /protonmail/entrypoint.sh init
```
- login
- change mode
- copy the notifications info
- re-deploy the protonmail bridge
- deploy the service, take note of the node port IP

Mail settings:
 - nodeport host
 - port 25
 - no encryption
 - username / password from info 


### Pihole
 
- install via helm (I also had to make the storage default)
``` 
helm install --name-template pihole mojo2600/pihole -f pihole/helm-values.yaml
```
- sync lists from https://firebog.net/
- set up DNS server to 192.168.86.50 on machines of your choice

### Prometheus & Grafana

Install prometheus operator
https://github.com/prometheus-operator/prometheus-operator

Select a release and apply the bundle.yaml

You may have to do this delete business but probably not if you don't mess around installing old releases with helm beforehand 
https://github.com/prometheus-community/helm-charts/issues/250#issuecomment-717159734


### Redis HA Cluster

lol - how do I even describe this.
 - deploy the deployment-large.yaml
 - get a list of ports using: 
 ```
 kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '
 ```
 - delete the blank port-only entry at the end of that list (delete this -> ` :6379`)
 - build the below using what you have the above data 
```
redis-cli --cluster create --cluster-replicas 1 0.244.3.186:6379 10.244.2.209:6379 10.244.3.187:6379 10.244.2.210:6379 10.244.3.188:6379 10.244.2.211:6379
```
 - get to CLI on a redis pod run it and say "yes"

### Nextcloud

Make the maria DB
 - follow order in kustomize

