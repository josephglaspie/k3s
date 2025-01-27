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
sudo vi /etc/dhcpcd.conf

interface wlan0
static ip_address=192.168.1.34/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8

sudo systemctl restart dhcpcd

ip addr show wlan0 #matches and looks good

## Install k3s on Raspberry Pi
```
sudo nano /boot/cmdline.txt
# add this to end of line don't add new line
cgroup_memory=1 cgroup_enable=memory
sudo reboot

sudo curl -sfL https://get.k3s.io | sh -
sudo systemctl status k3s
sudo kubectl get nodes

# get K3S_TOKEN
sudo cat /var/lib/rancher/k3s/server/token

# add worker node
sudo curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.34:6443 K3S_TOKEN=K105f7ecab152bc9c8fb9ce76fe137c56548d5d11567514a73573b5e1bd76135b11::server:fbe43fed0456bbf0ac08a359700938cd sh -

```