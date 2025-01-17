## At master nodes

Give root power to user -
``` 
sudo su - root
``` 

Install docker
``` 
yum install docker -y
``` 

Enable docker
``` 
systemctl enable docker --now
``` 

Configure YUM 
``` 
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
``` 

Install kubadm and kubelet
``` 
yum install kubeadm 
yum install kubelet
``` 

Enable kubelet
``` 
systemctl enable --now kubelet
``` 

Pull the image from docker hub
``` 
kubeadm config images pull
``` 

Check driver of docker
``` 
docker info | grep -i cgroup
Cgroup Driver: cgroupfs
``` 

Change the driver from cgroupfs  to  systemd
``` 
cat  /etc/docker/daemon.json
{
   "exec-opts": ["native.cgroupdriver=systemd"]
}
``` 
 

Now, restart docker
``` 
systemctl restart docker
``` 

Now, check
``` 
docker info | grep -i cgroup
o/p -  Cgroup Driver: systemd
``` 

For routing       
``` 
yum install iproute-tc
``` 

Set the bridge routing to 1 using
``` 
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
``` 

As we are using t2-micro of aws which gives 1CPU and 1GB ram...Now, we can initialize master by ignoring CPU and Mem       
``` 
kubeadm init --pod-network-cidr=10.240.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem
``` 

After initializing, it give us some commands that we need to run for making the nodes as worker/slave....
``` 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
``` 

## At worker nodes

Give root power to user -
``` 
sudo su - root
``` 

Install docker
``` 
yum install docker -y
``` 

Enable docker
``` 
systemctl enable docker --now
``` 

Configure YUM 
``` 
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
``` 

Install kubadm and kubelet
``` 
yum install kubeadm 
yum install kubelet
``` 

Enable kubelet
``` 
systemctl enable --now kubelet
``` 

Check driver of docker
``` 
docker info | grep -i cgroup
- Cgroup Driver: cgroupfs
``` 

Change the driver from cgroupfs  to  systemd
``` 
cat  /etc/docker/daemon.json
{
   "exec-opts": ["native.cgroupdriver=systemd"]
}
``` 
 

Now, restart docker
``` 
systemctl restart docker
``` 

Now, check
``` 
docker info | grep -i cgroup
Cgroup Driver: systemd
``` 

For Routing
``` 
yum install iproute-tc
``` 

Change the iptable 
``` 
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
``` 

Start the system  
``` 
sysctl --system
``` 

## At Master node

we have to create a token so that worker nodes can join the master node.
``` 
kubeadm token create  --print-join-command
``` 


Now if we run kubectl get nodes , then we will find nodes are not ready....so we need to make overlay connection between master and worker. Now, we use Flannel plugin .
``` 
kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
``` 

## At workers nodes
Use the join cmd given by master node so that worker can connect....

## At Master node

Now, run the command and get the nodes....
``` 
kubectl get nodes
``` 

💥 Now, we have to launch the wordpress with mysql data base. So i am launching wordpress and mysql inside a pod… <br>

To launch the wordpress use this command
```
kubectl run mywp — image=wordpress:5.1.1-php7.3-apache
```
To launch mysql pod
```
kubectl run mydb — image=mysql:5.7 — env=MYSQL_ROOT_PASSWORD=redhat — env=MYSQL_DATABASE=wpdb — env=MYSQL_USER=anuja — env=MYSQL_PASSWORD=redhat
```

Exposing Wordpress pod:
```
kubectl expose pods mywp1 — type=NodePort — port=80
```

To access the wordpress in public world we need to expose. Here, i am exposing on NodePort as the the client is in the public world.
```
kubectl get svc
```

Now you can take the public of any node weather master or slave with the exposed port and type on browser ….. you will landed to the wordpress login page and then enter password and username of the mysql database and hit the run installation button. your wordpress application is ready !!
