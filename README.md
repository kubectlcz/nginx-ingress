#  Kebernetes instalace nginx-ingress **(WIP)**

# Nastaveni nginx-ingress pro k8s kubeadm
Nasledujici dokument popisuje jak nainstalovat na custom k8s cluster (kudeadm) nginx-ingress.

Pro tento dokument mam cluster kde je jeden master, tri worker nodes a jeden samostatny load balancer (LB) kde pouzivam HAProxy.

Vsude mam nainstalovany __Ubuntu 20.04 LTS__.

Vsechny prikazy ktere budu pouzivat jsou na Linuxu pripade Macu. __Windows nastaveni neni obsahem tohoto dokumentu !!__

## Prostredi - CPU a RAM je doporucene
|Role|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Load Balancer|172.16.16.100|Ubuntu 20.04|1G|1|
|Master|172.16.16.101|Ubuntu 20.04|2G|2|
|Worker|172.16.16.201|Ubuntu 20.04|2G|2|
|Worker|172.16.16.202|Ubuntu 20.04|1G|1|

> * Potrebujete **root** ucet na vsech serverech kde je **kubeadmin**
> * Pro vsechny prikazy ktere se zde budou pouzivat bych pouzil **root** 

## Pozadavky
Pokud chcete toto zkusit na virtualizovanem prostredi budete potrebovat
* Virtualbox installed
* Host stroj by mel mit minimalne 8 cores
* Host stroj by mel mit minimalne 8G memory

## Nastaveni load balanceru
Prihlaste se k load balanceru
```
ssh root@172.16.16.100
```


##### Instalace HAProxy
```
apt update && apt install -y haproxy
```
##### Konfigurace haproxy
Nasledujici snippet pridejde do **/etc/haproxy/haproxy.cfg** viz **[konfigurace](/haproxy/haproxy.cfg)`**
```
frontend http-frontend
    bind *:80
    mode http
    default_backend http-backend

backend http-backend
    mode http
    balance roundrobin
    server worker-node-1 172.16.16.201:80 check fall 3 rise 2
    server worker-node-2 172.16.16.202:80 check fall 3 rise 2
```
##### Restart haproxy service
```
systemctl restart haproxy
```
V tuto chvili je nainstalovany a nakonfigurovany load balancer. Jedna se o jednoduchou konfiguraci kdy je potreba mit nadefinovany __frontend__ a __backend__

## frontend
__http-frontend__ - je nazev frontendu a je jedno co tam date.

__bind__ - rikame cokoliv prijde na load balancer na port 80

__mode__ - jaky protokol se pouzije

__default_backend__ - rikame kdo bude pozadavky ktere prijde load balancovat. Nazev muzi odpovidat nazvu __backend_backend http-backend__ a __default_backend http-backend__

```
frontend http-frontend
    bind *:80
    mode http
    default_backend http-backend
```

## backend
__http-backend__ - je nazev backendu ktery musi byt stejny jaky je uveden ve __frontendu__ a to konkretne __default_backend__.

__balance__ - rika se jaky typ balanci requestu ma load balancer pouzit v tomto pripade to je typ __roundrobin__ coz je prijde jeden request tak jde na server1 prijde druhy jde na server2 a dalsi request, ktery prijde pujde na server1 a atd.

__server__ oznacuje na jaky server ma frontend balancovat. Zde muze byt kolik jen serveru chcete v mem pripade to jsou dva. __worker-node-1__ __worker-node-2__ je jen pojmenovani serveru a pak informaci o tom na jake ip adresu se ma balancovat request

```
backend http-backend
    mode http
    balance roundrobin
    server worker-node-1 172.16.16.201:80 check fall 3 rise 2
    server worker-node-2 172.16.16.202:80 check fall 3 rise 2

```

Muzete se ze load balanceru odhlasit v tuto chvili je LB nastaven.


## Nastaveni nginx-ingress



## Deploy test app


## Nastaveni DNS domeny v /etc/hosts
##### Disable Firewall
```
ufw disable
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update && apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```
### Kubernetes Setup
##### Add Apt repository
```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```
##### Install Kubernetes components
```
apt update && apt install -y kubeadm=1.19.2-00 kubelet=1.19.2-00 kubectl=1.19.2-00
```
## On any one of the Kubernetes master node (Eg: kmaster1)
##### Initialize Kubernetes Cluster
```
kubeadm init --control-plane-endpoint="172.16.16.100:6443" --upload-certs --apiserver-advertise-address=172.16.16.101 --pod-network-cidr=192.168.0.0/16
```
Copy the commands to join other master nodes and worker nodes.
##### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.15/manifests/calico.yaml
```

## Join other nodes to the cluster (kmaster2 & kworker1)
> Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

> IMPORTANT: You also need to pass --apiserver-advertise-address to the join command when you join the other master node.

## Downloading kube config to your local machine
On your host machine
```
mkdir ~/.kube
scp root@172.16.16.101:/etc/kubernetes/admin.conf ~/.kube/config
```
Password for root account is kubeadmin (if you used my Vagrant setup)

## Verifying the cluster
```
kubectl cluster-info
kubectl get nodes
kubectl get cs
```

Have Fun!!
