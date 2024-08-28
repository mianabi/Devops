## Deploy Kubernetes Cluster on Ubuntu 22.04 with Containerd:
### 1- Install containerd
To install containerd, follow these steps on both Vms:
Load the br_netfilter module required for networking.
```yml
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
To allow iptables to see bridged traffic, as required by Kubernetes, we need to set the values of certain
fields to 1.
```yml
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
Apply the new settings without restarting:
```yml
sudo sysctl -system
```
Get the apt-key and then add the repository from which we will install containerd.
```yml
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release - cs) stable"
```
### OR:
```yml
apt install containerd -y
```
Set up the default configuration file.
```yml
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Next up, we need to modify the containerd configuration file and ensure that the cgroupDriver is
set to systemd. To do so, edit the following file:
```yml
sudo vim /etc/containerd/config.toml
```
### and search plugins."io.containerd.grpc.v1.cri".containerd.run mes.runc.op ons , then change SystemdCgroup = false to true.
```yml
SystemdCgroup = true
```
Finally, to apply these changes, we need to restart containerd.
```yml
sudo systemctl restart containerd
```



