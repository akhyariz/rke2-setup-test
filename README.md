# RKE2-HA SETUP

## VM Master1

**Disable Firewall**

```
sudo iptables -L
sudo firewall-cmd --list-all
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
sudo iptables -L
```

**Important**: *If your node has NetworkManager installed and enabled, ensure that it is configured to [ignore CNI-managed interfaces](https://docs.rke2.io/known_issues#networkmanager).*

```
sudo vi /etc/NetworkManager/conf.d/rke2-canal.conf
```
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```
```
sudo systemctl reload NetworkManager
```


### Install RKE2-Server
```
curl -sfL https://get.rke2.io -o install.sh
chmod +x ./install.sh
sudo INSTALL_RKE2_TYPE="server" ./install.sh 
```
```
sudo vi /etc/rancher/rke2/config.yaml
```
```
token: K10e33baf29e6cfc7a7b903481d658b17552ce4add96effa20cb904fe556a9b3034::server:ea5189cde8a2a194c0734a79b60a3137
write-kubeconfig-mode: 600
tls-san: 
  - rke2.ranchercluster.com
  - 192.168.1.110
node-name: rke2-master1
```

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```
```
sudo journalctl -u rke2-server -f
```

**Install kubectl**
```
sudo ln -s /var/lib/rancher/rke2/bin/* /usr/local/bin/
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/rke2/rke2.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get nodes
```


## VM HA

**Disable Firewall**
```
sudo iptables -L
sudo firewall-cmd --list-all
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
sudo iptables -L
```

**Install+configure nginx**
```
sudo dnf install -y nginx nginx-mod-stream
sudo vi /etc/nginx/nginx.conf
```
	...
	stream {
		include /etc/nginx/stream.d/*.conf;
	}



```sudo mkdir /etc/nginx/stream.d/
sudo vi /etc/nginx/stream.d/rke2-master.conf
	upstream rke2_master_kubernetes_api{
		server 192.168.1.111:6443;
		server 192.168.1.112:6443;
		server 192.168.1.113:6443;
	}

	server {
		listen 6443;
		proxy_pass rke2_master_kubernetes_api;
		proxy_timeout 10s;
		proxy_connect_timeout 5s;
	}


	upstream rke2_master_supervisor_api{
		server 192.168.1.111:9345;
		server 192.168.1.112:9345;
		server 192.168.1.113:9345;
	}

	server {
		listen 9345;
		proxy_pass rke2_master_supervisor_api;
		proxy_timeout 10s;
		proxy_connect_timeout 5s;
	}
```

**Disable selinux**
```
vi /etc/selinux/config
```
	...
	SELINUX=disabled
	...

```
sudo reboot
```

**Start nginx**

```
sudo systemctl start nginx
```

**Install kubectl**
 ```
 sudo    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

```
mkdir -p $HOME/.kube
scp r00t@rke2-master1:~/.kube/config ~/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get node
```


## VM Master2

**Disable Firewall**

```
sudo iptables -L
sudo firewall-cmd --list-all
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
sudo iptables -L
```

**Important**: If your node has NetworkManager installed and enabled, ensure that it is configured to [ignore CNI-managed interfaces](https://docs.rke2.io/known_issues#networkmanager).

```
sudo vi /etc/NetworkManager/conf.d/rke2-canal.conf
```
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```
```
sudo systemctl reload NetworkManager
```


### Install RKE2-Server
```
curl -sfL https://get.rke2.io -o install.sh
chmod +x ./install.sh
sudo INSTALL_RKE2_TYPE="server" ./install.sh 
```
```
sudo vi /etc/rancher/rke2/config.yaml
```
```
server: https://192.168.1.110:9345
token: K10e33baf29e6cfc7a7b903481d658b17552ce4add96effa20cb904fe556a9b3034::server:ea5189cde8a2a194c0734a79b60a3137
write-kubeconfig-mode: 600
tls-san: 
  - rke2.ranchercluster.com
  - 192.168.1.110
node-name: rke2-master2
```

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```
```
sudo journalctl -u rke2-server -f
```

**Install kubectl**
```
sudo ln -s /var/lib/rancher/rke2/bin/* /usr/local/bin/
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/rke2/rke2.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get nodes
```

# VM Master3

**Disable Firewall**

```
sudo iptables -L
sudo firewall-cmd --list-all
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
sudo iptables -L
```

**Important**: If your node has NetworkManager installed and enabled, ensure that it is configured to [ignore CNI-managed interfaces](https://docs.rke2.io/known_issues#networkmanager).

```
sudo vi /etc/NetworkManager/conf.d/rke2-canal.conf
```
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```
```
sudo systemctl reload NetworkManager
```


### Install RKE2-Server
```
curl -sfL https://get.rke2.io -o install.sh
chmod +x ./install.sh
sudo INSTALL_RKE2_TYPE="server" ./install.sh 
```
```
sudo vi /etc/rancher/rke2/config.yaml
```
```
server: https://192.168.1.110:9345
token: K10e33baf29e6cfc7a7b903481d658b17552ce4add96effa20cb904fe556a9b3034::server:ea5189cde8a2a194c0734a79b60a3137
write-kubeconfig-mode: 600
tls-san: 
  - rke2.ranchercluster.com
  - 192.168.1.110
node-name: rke2-master3
```

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```
```
sudo journalctl -u rke2-server -f
```

**Install kubectl**
```
sudo ln -s /var/lib/rancher/rke2/bin/* /usr/local/bin/
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/rke2/rke2.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get nodes
```

# VM Worker1

**Disable Firewall**

```
sudo iptables -L
sudo firewall-cmd --list-all
sudo systemctl stop firewalld.service 
sudo systemctl disable firewalld.service 
sudo iptables -L
```

**Important**: If your node has NetworkManager installed and enabled, ensure that it is configured to [ignore CNI-managed interfaces](https://docs.rke2.io/known_issues#networkmanager).

```
sudo vi /etc/NetworkManager/conf.d/rke2-canal.conf
```
```
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```
```
sudo systemctl reload NetworkManager
```


### Install RKE2-Server
```
curl -sfL https://get.rke2.io -o install.sh
chmod +x ./install.sh
sudo INSTALL_RKE2_TYPE="agent" ./install.sh 
```
```
sudo vi /etc/rancher/rke2/config.yaml
```
```
server: https://192.168.1.110:9345
token: K10e33baf29e6cfc7a7b903481d658b17552ce4add96effa20cb904fe556a9b3034::server:ea5189cde8a2a194c0734a79b60a3137
node-name: rke2-worker1
```

```
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```
```
sudo journalctl -u rke2-agent -f
```

*To be continued...*
