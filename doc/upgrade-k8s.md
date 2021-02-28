### Upgrade k8s

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.19.3-00 && \
sudo apt-mark hold kubeadm
```


**workers**

on each worker
```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.19.3-00 && \
apt-mark hold kubeadm
```

```
sudo kubeadm upgrade node
```


```
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.19.3-00 kubectl=1.19.3-00 && \
apt-mark hold kubelet kubectl
```