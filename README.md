# Apa itu Kubernetes?

Kubernetes adalah platform Orkestrasi Container opensource yang dapat membantu dalam managemen containerized workloads dan services. Kubernetes dapat mengotomasikan pekerjaan seperti menyebarkan, mengubah specs, dan mengelola aplikasi yang berjalan dicontainer.

Kubernetes adalah aplikasi untuk automation deployment, scaling, dan manajemen aplikasi berbasis container.

### Kubernetes Master

- kube-api-server betugas sebagai API yang digunakan untuk berinteraksi dengan Kubernetes Cluster
- etcd betugas untuk sebagai database untuk menyimpan data Kubernetes Cluster
- kube-scheduler bertugas memperhatikan aplikasi yang akan kita jalankan dan meminta Node untuk menjalankan aplikasi tersebut.
- kube-controller-manager bertugas melakukan kontrol terhadap node-node Kubernetes Cluster.
- cloud-controller-manager bertugas untuk melakukan kontrol terhadap interaksi dengan cloud provider.

### Kubernetes Worker/Nodes

- kubelet berjalan di setiap Node dan betugas untuk memastikan bahwa aplikasi kita berjalan di Node.
- kube-proxy berjalan di setiap Node dan bertugas sebagai proxy terhadap arus network yang masuk ke aplikasi kita dan sebagai load balancer juga.
- container-manager bejalan di setiap Node dan bertugas sebagai container manager. Kubernetes mendukung beberapa container manager seperti Docker, containerd, cri-o, rklet, dan yang lainnya.



## Apa itu Node?

- Node adalah worker machine di Kubernetes, sebelumnya ada yang menyebutnya Minion.
- Node bisa saja dalam bentuk VM atau Mesin Fisik
- Di dalam Node selalu terdapat Kubelet, kube-proxy dan container manager.

## Apa itu Pod?

- Pod adalah unit terkecil yang bisa dideploy di Kubernetes Cluster

- Pod berisi satu atau lebih container

- Secara sederhana Pod adalah aplikasi kita yang running di Kubernetes Cluster.

- Pod tidak bisa running di Node yang berbeda

  > ![image-20240704185919619](C:\Users\Dio Tri Andika\AppData\Roaming\Typora\typora-user-images\image-20240704185919619.png)

### Labels

Kenapa butuh label?

- Untuk memberi tanda pada Pod
- Untuk mempermudah orgranisir Pod
- Memberi infromasi tambahan pada Pod\
- Labl tidak hanya bisa digunakan pada pod, tapi pada semua resource di Kubernetes, seperti Replication Controller, Replicas Set, Service, dan lain lain.

Contoh menggunakan Label pada Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-labels
  labels:
    name: nginx-labels
    environment: testing
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Kita bisa cek labels pada Pod dengan

```bash
$ kubectl get pod --show-labels
NAME               READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod          1/1     Running   0          39m   <none>
nginx-pod-labels   1/1     Running   0          13s   environment=testing,name=nginx-labels
```

Salah satu kegunaan label biasanya digunakan untuk mencari Pod, sebagai contoh saya ingin mencari Pod dengan key environment productions

```bash
$ kubectl get pod --show-labels
NAME                READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod           1/1     Running   0          60m   <none>
nginx-pod-labels    1/1     Running   0          21m   environment=testing,name=nginx-labels
nginx-pod-labels2   1/1     Running   0          67s   environment=productions,name=nginx-labels
```

diatas terlihat bahwa pod dengan label `environment=productions` adalah `nginx-pod-labels2`. Selanjutnya saya bisa mencari pod2 dengan label tersebut dengan cara

```bash
$ kubectl get pod -l environment=productions
NAME                READY   STATUS    RESTARTS   AGE
nginx-pod-labels2   1/1     Running   0          3m9s
```

maka yang muncul hanyalah pod yang memiliki label tersebut.

### Annotation

Untuk apa Annotation?

- Annotation mirip dengan Label, hanya saja tidak dapat difilter atau diquery seperti label
- Biasanya annotation digunakan untuk menambahkan informasi tambahan dalam ukuran besar
- Annotation bisa menampung informasi sampai 256kb

Contoh menggunakan Annotation pada Pod:

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx-pod-annotation
  labels:
    team: ceker-ayam
  annotations:
    description: Aplikasi ini dikembangkan oleh tim ceker-ayam
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Kita bisa cek annotations dari sebuah Pod dengan menggunakan `kubectl describe pod <pod-name>`

```bash
$ kubectl describe pod nginx-pod-annotation
Name:             nginx-pod-annotation
Namespace:        default
Priority:         0
Service Account:  default
Node:             kube-pzn-cluster/192.168.59.112
Start Time:       Sat, 20 Jul 2024 13:36:37 +0800
Labels:           team=ceker-ayam
Annotations:      description: Aplikasi ini dikembangkan oleh tim ceker-ayam
Status:           Running
IP:               10.244.0.3
```

Diatas kita bisa melihat Annotations yang kita sisipkan pada pod nignx-pod-annotation. 

Kita juga bisa menambahkan annotation pada Pod dengan menggunakan `kubectl annotate pod <pod-name> key="value" `

### Namespace

Kapan menggunakan Namespace?

- Ketika resources di Kubernetes sudah terlalu banyak
- Ketika butuh memisahkan resources untuk multi-tenant, team atau environment.
- Nama resources bisa sama jika berada di namespace yang berbeda

Untuk melihat namespace kita bisa menggunakan `kubectl get namespaces`

```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   56m
kube-node-lease   Active   56m
kube-public       Active   56m
kube-system       Active   56m
```

