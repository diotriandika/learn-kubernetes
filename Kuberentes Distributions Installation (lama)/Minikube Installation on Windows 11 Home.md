## Instalasi Minikube di Windows (Home Edition)

Minikube bergantung pada sebuah virtual machine manager untuk mendeploy Cluster Kubernetes. Kita bisa menggunakan Docker Desktop untuk membuat Cluster Kubernetes, namun karena Docker Desktop perlu berjalan di Type-1 Hypervisor *(Hyper-V)* untuk dapat membuat kubernetes cluster dan mengingat disini saya memiliki Windows Home Edition maka saya akan menggunakan Minikube yang mendukung Type-1 & Type-2 Hypervisor untuk menjalankannya. [*Read reference here*](https://medium.com/containers-101/local-kubernetes-for-windows-minikube-vs-docker-desktop-25a1c6d3b766#:~:text=Windows%20considerations)

### Step 1 - Minikube Installation

Minikube menyediakan beragam opsi dalam instalasinya, disini saya akan menggunakan Windows Package Manager untuk menjalankan instalasi Minikube.

```powershell
PS C:> winget install minikube
```

### Step 2 - Kubectl Installation

Agar dapat berinteraksi dengan kubernetes cluster, diperlukan tool kubectl. Instalasi kubectl dapat dilakukan dengan menambahkan binary file kubectl kedalam windows system path atau dengan menggunakan package manager. [*Read reference here*](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

```powershell
PS C:> winget install -e --id Kubernetes.kubectl
```

 Verifikasi instalasi kubectl dengan mengecek versi client kubectl

```powershell
PS C:> kubectl version --client
```

### Creating Kubernetes Cluster using Minikube

Sekarang kita dapat membuat cluster kubernetes dengan menggunakan minikube. Sebagai contoh dibawah saya akan membuat sebuah single node cluster kubernetes dengan driver virtualbox

```powershell
PS C:> minikube start --driver=virtualbox --memory=2000m --cpus=2 --disk-size="15000mb" --no-vtx-check
```

Menjalankan perintah diatas akan membuat sebuah single node kubernetes cluster dengan spesifikasi node 2gb ram, 2cpu & 15gb disk capacity divirtualbox. Kalian juga bisa menambahkan option `--no-vtx-check` untuk melewati validasi virtualization support yang biasanya tidak terdeteksi karena digunakan WSL2.

Verifikasi dengan mengecek nodes yang tersedia menggunakan kubectl dan cek vm yang dibuat oleh minikube di virtualbox.

```powershell
PS C:> kubectl get nodes -A
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   2m14s   v1.30.0
```

![Screenshot 2024-07-02 130322](C:\Users\Dio Tri Andika\Pictures\Screenshots\Screenshot 2024-07-02 130322.png)

Kita juga bisa membuat multi-node kubernetes cluster dengan menambahkan parameter `--nodes=<number-of-nodes>`

```powershell
PS C:> minikube start --driver=virtualbox --memory=2000m --cpus=2 --disk-size="15000mb" --nodes=5 --no-vtx-check
```

Full References:

- https://www.getambassador.io/blog/kubernetes-with-minikube-on-windows-home
- https://medium.com/containers-101/local-kubernetes-for-windows-minikube-vs-docker-desktop-25a1c6d3b766
- https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
- https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download
- https://github.com/kubernetes/minikube/issues/3900
- https://github.com/kubernetes/minikube/issues/16788

