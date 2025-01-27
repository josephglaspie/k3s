## Raspberry pi project

K8s Raspberry Pi - Kargo 

Create k3s pi cluster
Install Argo
Use argo to install prometheus
Create App and deploy it with Helm
Install Kargo

### Networking:
sudo apt update
sudo apt install dhcpcd5
sudo nano /etc/dhcpcd.conf

interface wlan0
static ip_address=192.168.1.245/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8

sudo systemctl restart dhcpcd

ip addr show wlan0 #matches and looks good

## Install k3s on Raspberry Pi
```bash
sudo nano /boot/firmware/cmdline.txt
# add this to end of line don't add new line
cgroup_memory=1 cgroup_enable=memory
sudo reboot

# if manager node do this next part else skip and go down to # add worker node
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

## Make it accessible from outside
```bash
#check that it's accessible
curl https://192.168.1.34:6443 -k # you should get a 401
# copy over config file
scp joseph@192.168.1.34:/etc/rancher/k3s/k3s.yaml ~/raspberrypi/k3s.yaml # then change the server to the correct ip address

```

#### Resources
https://everythingdevops.dev/step-by-step-guide-creating-a-kubernetes-cluster-on-raspberry-pi-5-with-k3s/

nc -zv 192.168.1.34 6443