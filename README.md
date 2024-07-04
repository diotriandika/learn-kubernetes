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

- 
