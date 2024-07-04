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



