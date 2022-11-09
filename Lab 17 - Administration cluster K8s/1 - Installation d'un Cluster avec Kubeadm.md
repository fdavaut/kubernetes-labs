# Installation d'un cluster Kubernetes sur des nodes Ubuntu 20.04 avec kubeadm

[Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) est un outil fourni avec Kubernetes pour aider les utilisateurs à installer un cluster Kubernetes prêt pour la production en appliquant les meilleures pratiques.

## Environnement Lab

Accédez au répertoire `0 - Lab Setup` en ligne de commande.
Lancez l'environment Vagrant

```bash
vagrant up
vagrant status
vagrant ssh k8s-master-1
```

## Operation commune à tous les serveurs

Cette section concerne tous les nodes

### Install de kubelet, kubeadm and kubectl

Configuration des Repos

```bash
sudo apt update

sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Installation des packages

```bash
sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Test

```bash
kubectl version --client
kubeadm version
```

### Desactivation du Swap


```bash
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a

# Verification

sudo mount -a
free -h
```

### Modules Kernel a activer

```bash

# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```

### Installation du Contenair Runtime

Pour exécuter des conteneurs dans des Pods, Kubernetes utilise un runtime de conteneur. Les runtimes de conteneurs pris en charge sont les suivants :
* Docker
* CRI-O
* Containerd

Nous faisons le choix d'installer que Containerd

```bash
# Configure persistent loading of modules
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

# Load at runtime
sudo modprobe overlay
sudo modprobe br_netfilter

# Ensure sysctl params are set
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload configs
sudo sysctl --system

# Install required packages
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# Add Docker repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install containerd
sudo apt update
sudo apt install -y containerd.io

# Configure containerd and start service
sudo su -
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml

# restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status  containerd
```

## Operation sur Un master Node

### Initialisation du cluster

```bash
lsmod | grep br_netfilter

# demarrage par defaut de l'agent Kubelet
sudo systemctl enable kubelet
```

Nous voulons maintenant initialiser la machine qui exécutera les composants du plan de contrôle, notamment etcd (la base de données du cluster) et le serveur API.

```bash
sudo kubeadm config images pull
```

```bash
sudo kubeadm config images pull --cri-socket unix:///run/containerd/containerd.sock
```
Quelques options de base de `kubeadm init` qui sont utilisées pour amorcer le cluster.
* `--control-plane-endpoint` :  définit le point de terminaison partagé pour tous les nœuds du plan de contrôle. Peut être DNS/IP
* `--pod-network-cidr` : Utilisé pour définir un CIDR du réseau des Pod
* `--cri-socket` : A utiliser si vous avez plus d'un conteneur runtime pour définir le chemin de la socket runtime.
* `--apiserver-advertise-address` : Définir l'adresse de publication pour l'API Server du plan de contrôle

Pour containerd ça donne

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --cri-socket unix:///run/containerd/containerd.sock \
  --upload-certs \
  --control-plane-endpoint=<IP de votre Host> # Idealement IP ou DNS du Loadbalancer partagé par les nodes du control-plane
```

La bonne exécution de  la commande précédente devrait afficher un resultat similaire suivant 

<details><summary>Output de la commande précédente</summary>

```bash
....
[init] Using Kubernetes version: v1.24.3
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Using existing ca certificate authority
[certs] Using existing apiserver certificate and key on disk
[certs] Using existing apiserver-kubelet-client certificate and key on disk
[certs] Using existing front-proxy-ca certificate authority
[certs] Using existing front-proxy-client certificate and key on disk
[certs] Using existing etcd/ca certificate authority
[certs] Using existing etcd/server certificate and key on disk
[certs] Using existing etcd/peer certificate and key on disk
[certs] Using existing etcd/healthcheck-client certificate and key on disk
[certs] Using existing apiserver-etcd-client certificate and key on disk
[certs] Using the existing "sa" key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/scheduler.conf"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0611 22:34:23.276374    4726 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0611 22:34:23.278380    4726 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 8.008181 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.21" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master01.computingforgeeks.com as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master01.computingforgeeks.com as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: zoy8cq.6v349sx9ass8dzyj
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc
```

</details>

Vous avez donc les deux commandes pour les autres nodes de rejoindre le Cluster
* Rejoindre en tant que master :
  
  ```bash
  kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc \
    --control-plane
  ```

* Rejoindre en tant que worker :
  
  ```bash
  kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc
  ```

### Configurer le fichier KUBECONFIG pour Kubectl

Créer la config de `kubectl`

```bash
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verifier
kubectl cluster-info
```
## Operation sur les autres master Node

les autres master node rejoignent avec la commande générée précédemment 

```bash
kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc \
    --control-plane
```

## Operation sur les Worker Node

Les  Worker node également rejoignent avec la commande générée précédemment

```bash
kubeadm join <IP de votre Server>:6443 --token sr5l1l.hzy6ji90ly5hzy7q \
    --discovery-token-ca-cert-hash sha256:da39a3ee5e6b4b0d3255bfef95601890afd807092c5af807671077b047e15ddc
```

Confirmez la bonne installation du cluster

```bash
kubectl get nodes -o wide
```
## Installation d'un Plugin Reseau : Cas de Calico

Autres [CNI plugins supportés](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

```bash
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml 
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

```bash

```

```bash

```

```bash

```

```bash

```
```bash

```

```bash

```

```bash

```
```bash

```

```bash

```

```bash

```
```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```