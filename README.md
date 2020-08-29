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

# Test your Kubernetes Cluster 
if you have created a docker image, you can update in the image section of the deployment. Here, I am taking an example of using the application deploymnent using *nginx*

*nginx.yml*
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```
kubectl create -f nginx.yml
kubectl get deployments
kubectl expose deployment nginx type=NodePort --port=80
kubectl get services
kubectl get pods -o wide
kubectl get deployments
```

From services you could see that deployment of the port:hostport containers are already mapped. so you would need to check those containers are scheduled on the which nodes using *kubectl get pods -o wide*
now, 

```
curl http://<node_where_container_scheduled>:<hostport>
```

*examples* 

```
kubemaster $ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        8h
nginx        NodePort    10.111.249.242   <none>        80:31063/TCP   22m
kubemaster $
kubemaster $ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-585449566-jnqcg   1/1     Running   0          24m   192.168.1.9   kubenode01   <none>           <none>
nginx-585449566-x92v4   1/1     Running   0          24m   192.168.1.8   kubenode01   <none>           <none>
kubemaster $
```
check you are able to access the application

```
kubemaster $ curl http://kubenode01:31063
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
kubemaster $
```
