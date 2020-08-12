# kubernetes-for-beginners

# Pre-requsites

Modify your Virtualbox settings based on your network preference in the *Vagrantfile* in the repository

```
NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2

IP_NW = "192.168.56."
MASTER_IP_START = 50
NODE_IP_START = 51
```

# Install Kubernetes
install on these steps on master and worker nodes

- Add kubernetes signing key
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

- Add software repositories
```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

- Install kubenetes tools
```
sudo apt-get install kubeadm kubelet kubectl
```

- check for kubernetes version
```
kubeadm version
```

# Kubernetes Deployment
- install these only on the master node
```
kubeadm init --apiserver-advertise-address=192.168.56.51 --pod-network-cidr=192.168.0.0/16
```

Once this command finishes, it will display a kubeadm join message at the end. Make a note of the whole entry. This will be used to join the worker nodes to the cluster.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Deploy Pod Network to Cluster
```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl get pods --all-namespaces
```

# join worker node to cluster
execute this command in the workernodes of the kubernetes cluster so they would bootstrap to masternode
```
kubeadm join 192.168.56.51:6443 --token 7kk6uh.n6ojcbz90f6rqgx5 \
    --discovery-token-ca-cert-hash sha256:1ab498d5becd038bf9bda579c51a1227908c0ff65cf9c218fe0f8bb7b934f279
```

Once, above nodes are joined to master, verify you are able to get the nodes

```
kubemaster $ kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kubemaster   Ready    master   15m    v1.18.6   192.168.56.51   <none>        Ubuntu 18.04.3 LTS   4.15.0-72-generic   docker://19.3.12
kubenode01   Ready    <none>   10m    v1.18.6   192.168.56.52   <none>        Ubuntu 18.04.3 LTS   4.15.0-72-generic   docker://19.3.12
kubenode02   Ready    <none>   9m8s   v1.18.6   192.168.56.53   <none>        Ubuntu 18.04.3 LTS   4.15.0-72-generic   docker://19.3.12
kubemaster $
```
