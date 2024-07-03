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



