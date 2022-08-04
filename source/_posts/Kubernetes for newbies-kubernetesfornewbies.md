---
title: Kubernetes for newbies
date:  2020-07-13 12:00:00
tags:
- k8s
---
### What is Kubernetes and what does it do	
Kubernetes is a container orchestration tool for the automated management of conatinerized applications.

Three major benefits of a orchestration tool
 * help multiple instances of a piece of software to be spread across multiple servers (high availability)
 * roll out new code changes accross the entire cluster (rolling update)
 * automatically create containers to handle additional load (scale up)

### Typical Kubernetes cluster architecture

A kubernetes cluster has one or more control servers which manage and control the cluster and host Kubernetes API. Here, the master node is actually the control server. Worker nodes are responsible for running your actual application workloads.

![image.png](/img/2020/07/image-a8f801adef9147aebe72b28b631ce3a7.png)

* etcd: provides distributed, synchronized data storage for the cluster state
* kube-apiserver: servers the kubernetes API, the primary interface for the cluster
* kubelet: an agent that manages the process of running containers on each node between kube-apiserver and docker runtime
* kubectl: a command line tool that interact with the cluster
* kube-controller-manager: a system state regulator that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state. In short, it is a kind of backend interface that bundles servral components into one package
* kube-scheduler: a policy-rich, topology-aware, workload-specific function that balances and distributes available resources by taking into account individual and collective resource requirements, quality of service requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, deadlines, and so on
* kube-proxy: handles network communication between nodes by adding firewall routing rules

Useful commands:
```bash
kubectl get pods -n kube-system
```

### Containers and Pods

Pods: the smallest and most basic building block of the kubernetes model, which consists of one or more containers, share the same storage resources and a unique IP address in the Kubernetes cluster network

Some useful commands:
```bash
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF

kubectl get pods -n kube-system
kubectl describe pod nginx
kubectl delete pod nginx
```

### Networking in Kubernetes

The kubernetes networking model involves creating a virtual network accross the whole cluster. This means that every pod on the cluster has a unique IP address, and can communicate with any other pod in cluster, even if that other pod is running on a different node.

Useful commands:
```bash
kubectl get pods -o wide
```

### Kubernetes Deployment

Deployment: a great way to automate the management of pods, which allows you to specify a desired state for a set of pods. The cluster will then constantly work to maintain that desired state

* scaling
* rolling update
* self-healing

```yaml
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
EOF
```

### Kubernetes Service
Service allow you to dynamically access a group of reploca pods. Replica pods are often being created and destroyed. The service creates an abstraction layer on top of a set of replica pods so that you can get access to the service rather than the pods.

In the below example, we are actually exposing the nginx deployment to external source. But it is not usally a good idea. In most cases, we will use ClusterIP to declare an internal service which will be accessible for all pods in the cluster.

```
cat << EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
EOF

# Get a list of services in the cluster
kubectl get svc

# Since this is a NodePort service, you should be able to access it using port 30080 on any of your cluster's servers. 
```


### Building a Kubernetes Cluster on Ubuntu 8

1. Docker
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu

sudo apt-mark hold docker-ce
```

2. kubelet kubeadm kubectl

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00

sudo apt-mark hold kubelet kubeadm kubectl
```

3. Init master node

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

4. Set up the local kubeconfig

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl version
```

5. Join master node from worker node
```bash
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
kubectl get nodes
```

6. Setup Flannel network

```bash
# On all three nodes
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Master node 
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

# Verify that all the nodes now have a STATUS of Ready
kubectl get nodes
# verify that the Flannel pods are up and running.
kubectl get pods -n kube-system
```

### Microservice with Kubernetes

Last but not the least, a very interesting microservice application which could be deployed by kubernetes:
https://github.com/linuxacademy/robot-shop

```bash
git clone https://github.com/linuxacademy/robot-shop.git
kubectl create namespace robot-shop
kubectl -n robot-shop create -f ~/robot-shop/K8s/descriptors/
kubectl get pods -n robot-shop -w
http://$kube_server_public_ip:30080
```