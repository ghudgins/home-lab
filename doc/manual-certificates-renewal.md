# Manual Cert Renewal

You really should upgrade k8s once a year. 

...however if you don't...

On each master node execute
`sudo kubeadm certs check-expiration`

If expired:
`sudo kubeadm certs renew all`

Once you do this the conf in the home directories will be wrong. Replace them with 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

You'll also need to update your local kubectl 
`scp -r ubuntu@msmiller:~/.kube /Users/ghudgins/`
