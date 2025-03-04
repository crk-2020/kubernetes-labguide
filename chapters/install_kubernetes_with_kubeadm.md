
# Setting up Kubernetes Cluster with Kubeadm  

This documents describes how to setup kubernetes from scratch on your own nodes, without using a managed service. This setup uses **kubeadm** to install and configure kubernetes cluster.

## Compatibility

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.

The below steps are applicable for the below mentioned OS


| OS | Version |
| --- | --- |
| **Ubuntu** | ** 18.04 / 20.04 ** |

## Base Setup 

Refer to [base setup document](../base_setup) to set up the nodes if you are configuring the nodes from scratch. 


## Configuring Docker on ALL Nodes

Note: Run this on ALL the nodes. 

Configure docker to use systems as cgroupfs driver. This is needed for Kubernetes to work properly. 

```
cat >  /etc/docker/daemon.json << EOF 
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

And restart docker daemon 
```
 systemctl daemon-reload
 systemctl restart docker

```


## Initializing Master

This tutorial assumes **kube-01**  as the master and used kubeadm as a tool to install and setup the cluster. This section also assumes that you are using vagrant based setup provided along with this tutorial. If not, please update the IP address of the master accordingly.

To initialize master, run this on kube-01 (1st node)

`replace 192.168.56.101 with the actual IP of your node`

```
kubeadm init --apiserver-advertise-address 192.168.56.101 --pod-network-cidr=192.168.0.0/16

```

### Initialization of the Nodes (Previously Minions)

After master being initialized, it should display the command which could be used on all worker/nodes to join the k8s cluster.

e.g.
```
kubeadm join --token c04797.8db60f6b2c0dd078 192.168.12.10:6443 --discovery-token-ca-cert-hash sha256:88ebb5d5f7fdfcbbc3cde98690b1dea9d0f96de4a7e6bf69198172debca74cd0
```
`dont copy above command as is, this is just a sample, use actual`

Copy and paste it on all node.


### Confgure kubectl Client


`on Master Node`

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Validate

```
kubectl get nodes
```

You could also put the above command on a watch to observe the nodes getting ready.

```
watch kubectl get nodes
```

## Configure Kubernetes Networking

Installing overlay network is necessary for the pods to communicate with each other across the hosts. It is necessary to do this before you try to deploy any applications to your cluster.

There are various overlay networking drivers available for kubernetes. We are going to use **Weave Net**.

```

export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```



## e2e Testing

You could validate the status of this cluster, health of pods and whether all the components are up or not by using a few or all of the following commands.

To check if nodes are ready

```
kubectl get nodes

```

[ Expected output ]

```
root@kube-01:~# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
kube-01   Ready     master    9m        v1.8.2
kube-02   Ready     <none>    4m        v1.8.2
kube-03   Ready     <none>    4m        v1.8.2
```


Additional Status Commands

```
kubectl version -o yaml

kubectl cluster-info

kubectl get pods -n kube-system

kubectl get events

```

It will take a few minutes to have the cluster up and running with all the services.


## Set up Visualiser

Fork the repository and deploy the visualizer on kubernetes


```
git clone  https://github.com/schoolofdevops/kube-ops-view
kubectl apply -f kube-ops-view/deploy/

```

Visualiser will run on  **32000** port. You could access it using a URL such as below and  add /#scale=2.0 or similar option where 2.0 = 200% the scale.

`replace <NODE_IP> with actual IP of one of your nodes`

```
http://<NODE_IP>:32000/#scale=2.0
```


![kube-visualizer](images/kube-ops-view.png)

Kubernetes visualiser is a third party application which provides a operational view of your kubernetes cluster. Its very useful tool for learning kubernetes as it demonstrates the state of the cluster as well as state of the pods as you make changes. You could read further about it [at this link](https://kubernetes-operational-view.readthedocs.io/en/latest/).  





## Download the supporting code

Before we proceed further, please checkout the code from the following git repo. This would offer the supporting code for the exercises that follow.

`run this on the host where you have configured kubectl`
```
git clone https://github.com/initcron/k8s-code.git
```
