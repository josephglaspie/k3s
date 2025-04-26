## Raspberry pi project

K8s Raspberry Pi - Kargo 

- Create k3s pi cluster
- Install Argo
- Use argo to install prometheus
- Create App and deploy it with Helm
- Install Kargo

### Networking:
```bash
#setup static ip address
sudo apt update
sudo apt install dhcpcd5
sudo nano /etc/dhcpcd.conf

interface wlan0
static ip_address=192.168.1.245/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8

sudo systemctl restart dhcpcd

ip addr show wlan0 #matches and looks good
```

### Install k3s on Raspberry Pi
```bash
sudo nano /boot/firmware/cmdline.txt
# add this to end of line don't add new line
# enable cgroup memory
cgroup_memory=1 cgroup_enable=memory
sudo reboot

# if manager node do this next part else skip and go down to # add worker node
# create manager node
sudo curl -sfL https://get.k3s.io | sh -
sudo systemctl status k3s
sudo kubectl get nodes

# get K3S_TOKEN
sudo cat /var/lib/rancher/k3s/server/token

# add worker node
sudo curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.34:6443 K3S_TOKEN=K105f7ecab152bc9c8fb9ce76fe137c56548d5d11567514a73573b5e1bd76135b11::server:fbe43fed0456bbf0ac08a359700938cd sh -

#create test pod from main
kubectl run test-pod --image=busybox --restart=Never -- sleep 3600
```

### Make it accessible from outside
```bash
#check that it's accessible
curl https://192.168.1.34:6443 -k # you should get a 401
# copy over config file
scp joseph@192.168.1.34:/etc/rancher/k3s/k3s.yaml ~/raspberrypi/k3s.yaml # then change the server to the correct ip address
```

### Load Balancer
```bash
# First add metallb repository to your helm
helm repo add metallb https://metallb.github.io/metallb
# Check if it was found
helm search repo metallb
# Install metallb
helm upgrade --install metallb metallb/metallb --create-namespace \
--namespace metallb-system --wait
#configure CR in cluster. 192.168.1.200-192.168.1.225 is within my LAN range
cat << 'EOF' | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.200-192.168.1.225
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

### ArgoCD
```bash
k create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
#get admin pw
PW=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo)
# 8gPJCZohhdWnSC5R
#set argo server up with loadbalancer
kubectl patch service argocd-server -n argocd --patch '{ "spec": { "type": "LoadBalancer", "loadBalancerIP": "192.168.1.208" } }'
kubectl get svc -n argocd
argocd login 192.168.1.208 --username admin --password $PW
```


#### Resources
https://everythingdevops.dev/step-by-step-guide-creating-a-kubernetes-cluster-on-raspberry-pi-5-with-k3s/  
https://rpi4cluster.com/k3s-argo-cd/

#### Misc
##### Reboot all pods
```bash
kubectl rollout restart deployment -n argocd
kubectl get svc -n argocd -o yaml | kubectl apply -f -
```
##### Redis Argo
```bash
#get redis pw
kubectl -n argocd get secret argocd-redis -o jsonpath='{.data.auth}' | base64 -d
# Now exec into the pod
kubectl -n argocd exec -it argocd-redis-6ffbf44c94-l962b -- sh
# Inside the pod
redis-cli
# Authenticate
auth <paste password here>
# Now free to run
keys *
info
```