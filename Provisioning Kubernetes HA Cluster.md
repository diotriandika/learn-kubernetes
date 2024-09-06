## Provisioning Kubernetes HA Cluster

Sebuah Cluster HA (High Availability) terdiri dari kumpulan (minimal 3) control plane (master nodes), worker nodes dan sebuah loadbalancer. Control Plane atau Master Nodes adalah inti dari sebuah Kubernetes cluster yang memanage semua operasi di cluster tersebut. Ada kalanya master nodes tersebut down dan jika kita menggunakan Single-Node Control Plane, sudah dipastikan cluster tersebut juga akan mati berserta semua aplikasi yang berjalan didalamnya atau worst-casenya data hilang. Maka dari itu sangatlah penting membuat infrastruktur kubernetes cluster yang menggunakan High Availability atau Multi-Node Control Plane untuk meminimalisir terjadinya downtime. 

## Infrastructure

Specs:

| Requirements  | Specs       |
| ------------- | ----------- |
| Load Balancer | 1 Nodes     |
| Control Plane | 3 Nodes     |
| Worker        | 2 Nodes     |
| RAM           | 4096Mi each |
| CPUs          | 2           |
| Storage       | 20G         |

Topology: 

![image](https://github.com/user-attachments/assets/dbb21e5e-14f4-49d7-a355-8f42dce666e6)

## Setup Load Balancer

Load balancer disini diperlukan untuk memproses semua traffic masuk kedalam cluster. Dalam konteks untuk ini, saya menggunakan HAProxy sebagai loadbalancer untuk membagi traffic ke semua control planes. Dan jika salah satu control plane mati, aplikasi tetap bisa diakses karena HAProxy secara otomatis mengalihkan semua traffic ke available server.

### Step 1 - Update Repository & Install HAProxy 

Update packages repository untuk memastikan semua package dalam versi terbaru, kemudian install haproxy di **Node Load Balancer.**

```bash
$ sudo apt-get update && sudo apt-get install -y
```

Edit configuration file `/etc/haproxy/haproxy.cfg` dan sesuaikan seperti dibawah (letakan dibawah option defaults)

```bash
$ sudo nano /etc/haproxy/haproxy.cfg
---
defaults
        ...
frontend k8s-master-frontend
        bind :6443
        mode tcp
        option tcplog
        default_backend k8s-master-backend

backend k8s-master-backend
        mode tcp
        option tcp-check
        balance roundrobin
        default-server inter 10s downinter 5s
        server k8s-master-1 10.20.1.10:6443 check
        server k8s-master-2 10.20.1.11:6443 check
```

> Notes: Port `6443` merupakan port dari Kubernetes API Server. [Reference here](https://kubernetes.io/docs/concepts/security/controlling-access/)

Reload konfigurasi dengan merestart service HAProxy & enable haproxy startup at boot.

```bash
$ sudo systemctl restart haproxy && sudo systemctl enable haproxy
```

Referensi:

- https://facsiaginsa.com/kubernetes/haproxy-for-k8s-load-balancer

## Setup Kubernetes Cluster

Pertama pastikan seluruh Node sudah saling terhubung satu sama lain dan jika perlu bisa lakukan `ssh-copy-id` disetiap node untuk mempermudah akses antar vm

### Step 1 - Setup Containerd as CRI

CRI diperlukan disetiap node dalam cluster sehingga Pod dama berjalan diatasnya. Install containerd disemua master dan worker node sebagai CRI (Container Runtime Interface) mengingat docker sudah mendapat deprecation warning untuk versi 1.20 keatas, [baca disini.](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

#### Jalankan di semua Master dan Worker Node!!

Load module `br_netfilter` & `overlay` yang mana digunakan untuk networking nantinya.

```
# Load overlay & br_netfilter module into running kernel.
$ sudo modprobe overlay && sudo modprobe br_netfilter

# Load on boot modules for containerd
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

Konfigurasi `sysctl` yang diperlukan dan pastikan persisten setiap system reboot.

```bash
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Apply semua perubahan parameter `sysctl` tanpa reboot.

```bash
$ sudo sysctl --system
```

Selanjutnya install package-package yang diperlukan untuk menggunakan repository melaui HTTPS dan tambahkan apt-key serta repository yang dibutuhkan untuk instalasi containerd.

```bash
# Add Docker's offical GPG key: (the same repo as containerd used.)
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg apt-transport-https -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update current repository & install containerd
$ sudo apt-get update & sudo apt-get install containerd.io
```

Buat configuration file default dari containerd

```bash
$ sudo su -
$ mkdir -p /etc/containerd
$ containerd config default | sudo tee /etc/containerd/config.toml
```

Selanjutnya set containerd configuration file diatas agar  `cgroupDriver` untuk runc diset ke systemd yang mana nantinya diperlukan oleh kubelet.

```bash
$ sed -i '/SystemdCgroup/s/false/true/g' /etc/containerd/config.toml
```

Restart containerd dan cek apakah containerd sudah berjalan atau tidak serta enable startup to system boot.

```bash
# restart containerd service
$ sudo systemctl restart containerd

# check containerd status & enable automatic startup while system boot
$ sudo systemctl status containerd && sudo systemctl enable --now containerd
```

> Quick Info: CRI atau Container Runtime Interface adalah software yang bertugas untuk menjalankan container dikubernetes. Saat ini kubernetes mendukung beberapa CRI seperti containerd, CRI-O dan juga Docker. [Lengkapnya disini](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-versions)

Referensi: 

- https://medium.com/@sven_50828/setting-up-a-high-availability-kubernetes-cluster-with-multiple-masters-31eec45701a2
- https://blog.assistanz.com/how-to-create-the-k8s-cluster-with-containerd/
- https://www.nocentino.com/posts/2021-12-27-installing-and-configuring-containerd-as-a-kubernetes-container-runtime/
- https://saurabhkharkate05.medium.com/kubernetes-cluster-setup-with-containerd-945214a0d02c

### Step 2 - Kubernetes Installation

#### Jalankan di semua Master dan Worker Node!!

Dalam menginisasi/memulai sebuah cluster kubernetes diperlukan 3 tool yang wajib diinstal disetiap node, yakni kubeadm, kubelet dan kubectl

> Quick Info: 
>
> - Kubeadm adalah sebuah tool yang digunakan untuk membuat cluster kubernetes, terdapat 2 command krusial yang digunakan yaitu `kubeadm init` yang digunakan untuk menginisiasi sebuah cluster dan `kubeadm join` digunakan untuk bergabung ke cluster yang sudah dibuat. [Lengkapnya disini.](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
> - Kubectl adalah sebuah command-line tool yang digunakan untuk berkomunikasi dengan Kubernetes Cluster  Control Plane. [Lengkapnya disini.](https://kubernetes.io/docs/reference/kubectl/)
> - Kubelet adalah sebuah `node-agent` yang nantinya berjalan disetiap worker-node dalam cluster. Kubelet memastikan container beroperasi didalam Pod. [Lengkapnya disini.](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

Untuk melakukan instalasti ketiga tools diatas, download public signing key untuk package repository kubernetes kemudian tampahkan apt repository kubernetes.

```bash
# Download public signing key kubernetes
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes apt repository
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update current repository lalu install `kubectl`, `kubeadm` dan `kubelet`.

```bash
# Update current repository packages
$ sudo apt-get update

# Install kubectl, kubelet and kubeadm
$ sudo apt-get install kubectl kubelet kubeadm -y

# Prevent the package for being automatically updated
$ sudo apt-mark hold kubectl kubeadm kubelet
```

Enable service kubelet

```bash
$ sudo systemctl enable --now kubelet
```

Referensi : 

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

### Step 3 - Initializing High Availability Kubernetes Cluster

Untuk dapat menginisiasi sebuah cluster kubernetes, pastikan terlebih dahulu bahwa semua host dapat menagkses images dari kubernetes container registry `registry.k8s.io` .

> Note: Matikan swap pada setiap node, karena hingga saat ini kubernetes tidak mensupport system yang menggunakan swap. Jika ingin tetap menggunakan swap system, bisa [baca disini.](https://stackoverflow.com/questions/47094861/error-while-executing-and-initializing-kubeadm#:~:text=To%20install%20kubeadm%20with%20swap%20enabled%3A)

Ada juga prequisites yang perlu diingat sebelum menginisasi high available cluster kubernetes, bisa [baca disini.](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#:~:text=for%20both%20methods-,Create%20load%20balancer%20for%20kube%2Dapiserver,-Note%3A)

#### Execute di k8s-master-1! (run as root)

Initialize k8s-master-node1 untuk mulai membuat cluster, 

```bash
$ kubeadm init --control-plane-endpoint "load-balancer.local:6443" --upload-certs
```

> `load-balancer.local` merupakan dns dari load balancer. Kita bisa menggunakan IP langsung, namun direkomendasikan untuk menggunakan dns terutama dienvironment cloud. [Lengkapnya disini.](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#:~:text=It%20is%20not%20recommended%20to%20use%20an%20IP%20address%20directly%20in%20a%20cloud%20environment.)

Jika berjalan dengan normal, akan terdapat output panjang. Namun yang perlu diperhatikan adalah list dibawah.

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:
========================
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
========================
Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf
  
# Token for Control Plane
========================
You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join load-balancer.local:6443 --token rwarnf.sexqk4fch679te97 \
        --discovery-token-ca-cert-hash sha256:37efc6521fedbd57d7773696d789441e0d0fed5a7a801d66ffeb11bc4890bd05 \                                                                                                    --control-plane --certificate-key 9c2b6f375b12e03588e3bbe48de4cee92f15979a037470297ab9dca7ef9fc052
=========================

# Token for Worker Nodes
=========================
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join load-balancer.local:6443 --token rwarnf.sexqk4fch679te97 \
        --discovery-token-ca-cert-hash sha256:37efc6521fedbd57d7773696d789441e0d0fed5a7a801d66ffeb11bc
 ========================
```

Sesuai petunjuk diatas, untuk memulai cluster dengan user regular (non-root user) kita perlu mengeksekusi perintah dibawah

```bash
# run as non-root user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Perintah diatas bertujuan untuk menyalin `kubectl` config file ke home directory agar kita bisa mengontrol cluster kubernetes dengan tool kubectl.

Verifikasi dengan menjalankan 

```bash
$ kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
k8s-master-1   NotReady   control-plane   3m45s   v1.31.0
```

Selanjutnya terdapat 2 hal penting lagi untuk membangun cluster, yakni section `kubeadm join`. Terdapat dua jenis member untuk bergabung ke cluster, yakni member dengan roles control plane dan worker node. 

Perhatikan **output diterminal**, akan terdapat output mirip seperti dibawah. **Eksekusi** output tersebut sesuai dengan infrastructure masing-masing. 

```bash
# Run this on the other Master-Nodes instantes (k8s-master-2) to join the cluster as Control-Plane.
$ sudo kubeadm join load-balancer.local:6443 --tokenrwarnf.sexqk4fch679te97 --discovery-token-ca-cert-hash sha256:37efc6521fedbd57d7773696d789441e0d0fed5a7a801d66ffeb11bc4890bd05 --control-plane --certificate-key 9c2b6f375b12e03588e3bbe48de4cee92f15979a037470297ab9dca7ef9fc052
       
       
# Run this on each Worker-Nodes available (k8s-worker-1 & k8s-worker-2) to join the cluster as worker nodes.
$ sudo kubeadm join load-balancer.local:6443 --token rwarnf.sexqk4fch679te97 --discovery-token-ca-cert-hash sha256:37efc6521fedbd57d7773696d789441e0d0fed5a7a801d66ffeb11bc
```

> Note: jika terdapat spasi karena melakukan copy melalui terminal, perlu diperhatikan untuk memperbaiki spacing tersebut untuk meminimalisir terjadinya error.

Jika sudah maka akan terdapat output seperti dibawah, dan jika kita melist node dalam cluster akan terlihat seluruh nodes yang sudah digabungkan sebelumnya.

![image-20240906180006232](https://github.com/user-attachments/assets/2a5f23fd-a39f-48d5-b7b8-496149d28f83)

> Notes : Ingat untuk menjalankan command dibawah setelah berhasil join pada `k8s-master-2` agar dapat menggunakan tools `kubectl`.
>
> ```bash
> # run as non-root user
> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
> ```

```bash
# Use kubectl to list available nodes on the current cluster (exec on control plane)
$ kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
k8s-master-1   NotReady   control-plane   4m56s   v1.31.0
k8s-master-2   NotReady   control-plane   3m41s   v1.31.0
k8s-worker-1   NotReady   <none>          81s     v1.31.0
k8s-worker-2   NotReady   <none>          61s     v1.31.0
```

Kita lihat diatas bahwa semua nodes dalam status NotReady, ini dikarenakan dalam cluster tersebut belum ada CNI yang berjalan. 

```bash
$ kubectl get pod -A -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
kube-system   coredns-6f6b679f8f-42zgx               0/1     Pending   0          10m     <none>       <none>         <none>           <none>
kube-system   coredns-6f6b679f8f-zt789               0/1     Pending   0          10m     <none>       <none>         <none>           <none>
kube-system   etcd-k8s-master-1                      1/1     Running   4          10m     10.20.1.10   k8s-master-1   <none>           <none>
kube-system   etcd-k8s-master-2                      1/1     Running   2          9m29s   10.20.1.11   k8s-master-2   <none>           <none>
kube-system   kube-apiserver-k8s-master-1            1/1     Running   4   
...
```

Referensi :

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
- https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
- https://github.com/ruzickap/multinode_kubernetes_cluster/blob/master/docs/source/01-k8s-installation.rst

### Step 4 - Installing CNI Plugin

Dikutip dari [kubernetes.io](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/), CNI atau Container Network Interface dalam konteks networking adalah sebuah daemon dalam pada node yang dikonfigurasi untuk menyediakan CRI Services untuk kubelet. Khususnya, Container Runtime harus dikonfigurasi untuk memuat Plugins CNI yang diperlukan untuk dapat mengimplementasikan [Kubernetes Network Model](https://kubernetes.io/docs/concepts/services-networking/#the-kubernetes-network-model). 

Kubernetes menggunakan Plugin CNI untuk mengelola network di cluster Kubernetes. CNI bertugas untuk mengelola pengalamatan alamat IP ke Pod, network routing antara Pod, Kubernetes service routing, dan lainnya.

Disini saya menggunakan [Calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises) sebagai CNI karena proses pemasangannya & managemennya terbilang mudah. Calico memiliki 2 cara dalam proses intsalasinya, yakni Calico Operator yang menggunakan custom resources untuk mengatur lifecycle dari calico itu sendiri, dan satunya adalah Manifest yang dimana Calico akan diinstall secara langsung sebagai resources Kubernetes. [Lengkapnya disini](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)

Masuk ke salah satu control-plane, lalu jalankan perintah dibawah

```bash
# Download calico networking manifest for the Kubernetes API
$ curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/calico.yaml -O
```

Gunakan `kubectl apply` atau `kubectl create` untuk membuat resources calico.

```bash
$ kubectl create -f calico.yaml
```

Tunggu hingga semua resource dan object calico berjalan, kita bisa melihat secara realtime dengan menggunakan `watch`

```bash
$ watch kubectl get all --all-namespaces
```

Jika sudah maka seharusnya semua Pod saat ini dalam status running

```bash
$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-7fbd86d5c5-tllzj   1/1     Running   0          12m
kube-system   calico-node-5mxws                          1/1     Running   0          12m
kube-system   calico-node-95t2f                          1/1     Running   0          12m
kube-system   calico-node-gs4cj                          1/1     Running   0          12m
kube-system   calico-node-m2ztj                          1/1     Running   0          12m
kube-system   coredns-6f6b679f8f-42zgx                   1/1     Running   0          89m
kube-system   coredns-6f6b679f8f-zt789                   1/1     Running   0          89m
```

dan juga nodes dalam keadaan ready

```bash
$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
k8s-master-1   Ready    control-plane   90m   v1.31.0
k8s-master-2   Ready    control-plane   89m   v1.31.0
k8s-worker-1   Ready    <none>          86m   v1.31.0
k8s-worker-2   Ready    <none>          86m   v1.31.0
```

Referensi :

- https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises
- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
- https://learn.microsoft.com/en-us/azure/aks/concepts-network-cni-overview



Referensi global:

- https://medium.com/@sven_50828/setting-up-a-high-availability-kubernetes-cluster-with-multiple-masters-31eec45701a2
- https://overcast.blog/disaster-recovery-and-high-availability-in-kubernetes-a-guide-21996d80ac39
- https://avinetworks.com/glossary/kubernetes-load-balancer/
- https://facsiaginsa.com/kubernetes/haproxy-for-k8s-load-balancer
- https://kubernetes.io/docs/concepts/security/controlling-access/
- https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

