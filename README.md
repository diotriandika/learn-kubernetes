### **Apa itu Kubernetes?**
Kubernetes adalah platform orkestrasi container open-source yang digunakan untuk mengotomatisasi deployment, scaling, dan pengelolaan aplikasi yang berjalan dalam container. Dengan Kubernetes, kita dapat dengan mudah mengelola aplikasi yang terdiri dari banyak container, menyediakan platform yang sangat skalabel, efisien, dan dapat dipelihara. Kubernetes memungkinkan kita untuk menjalankan aplikasi dalam lingkungan yang terdistribusi, memantau statusnya, dan mengelola update serta scaling secara otomatis.

### **Komponen Kubernetes Master**
Kubernetes memiliki beberapa komponen utama yang bekerja sama untuk mengelola dan mengontrol cluster kita. Diantaranya adalah:

1. **kube-api-server**  
   API Server adalah titik interaksi utama antara kita (pengguna atau sistem lain) dengan Kubernetes. Semua perintah yang kita berikan ke Kubernetes, baik untuk mendirikan pod, deployment, scaling, dan konfigurasi lainnya, akan dikirim melalui API Server ini. Kube-api-server memastikan komunikasi yang konsisten dan aman di seluruh komponen dalam cluster.

2. **etcd**  
   Etcd adalah database distributed yang menyimpan seluruh data konfigurasi dan state dari Kubernetes cluster kita. Setiap perubahan dalam status atau konfigurasi cluster (seperti pengaturan pod, deployment, atau service) disimpan dalam etcd. Data ini sangat penting karena Kubernetes mengandalkan etcd untuk menjaga konsistensi dan integritas cluster.

3. **kube-scheduler**  
   Kube-scheduler adalah komponen yang bertugas untuk memutuskan di mana pod akan dijalankan dalam cluster. Ia memilih node yang sesuai berdasarkan berbagai faktor, seperti ketersediaan resource, aturan penjadwalan, dan toleransi node. Dengan scheduler ini, kita bisa memastikan aplikasi kita berjalan di tempat yang tepat.

4. **kube-controller-manager**  
   Kube-controller-manager bertugas untuk menjaga agar kondisi yang diinginkan dalam cluster tetap sesuai. Misalnya, jika kita ingin memastikan bahwa ada 3 replika pod yang berjalan pada setiap saat, controller akan terus memonitor dan memastikan bahwa jumlah pod tetap terjaga dengan melakukan penambahan atau pengurangan jika diperlukan.

5. **cloud-controller-manager**  
   Jika kita menjalankan Kubernetes di atas platform cloud (seperti AWS, Azure, atau Google Cloud), cloud-controller-manager mengelola interaksi Kubernetes dengan penyedia layanan cloud. Misalnya, ia bertanggung jawab untuk mengelola node dalam cloud atau integrasi storage dengan cloud provider.

### **Komponen Kubernetes Worker / Nodes**
Node adalah mesin tempat aplikasi (dalam bentuk pod) dijalankan. Setiap node dalam Kubernetes memiliki beberapa komponen yang memastikan bahwa aplikasi kita berjalan dengan baik:

1. **kubelet**  
   Kubelet adalah agen yang berjalan di setiap node. Tugas utamanya adalah memastikan bahwa pod yang didefinisikan dalam Kubernetes cluster berjalan dengan baik di node tersebut. Kubelet memonitor status container dan pod, serta memastikan bahwa semuanya sesuai dengan konfigurasi yang sudah ditentukan.

2. **kube-proxy**  
   Kube-proxy berfungsi untuk menangani lalu lintas jaringan masuk dan keluar dari pod. Ia juga melakukan load balancing untuk memastikan bahwa trafik dialihkan dengan merata ke pod yang tersedia. Misalnya, jika kita memiliki beberapa pod yang menyediakan layanan yang sama, kube-proxy akan mengatur agar trafik yang masuk dapat dibagi dengan baik antara pod-pod tersebut.

3. **container manager**  
   Container manager bertugas mengelola lifecycle container dalam node. Kubernetes mendukung berbagai container manager, seperti **Docker**, **containerd**, dan **CRI-O**. Komponen ini mengurus pembuatan, pengelolaan, dan penghapusan container dalam node.

### **Apa itu Node?**
Node adalah mesin (baik mesin fisik atau virtual) dalam Kubernetes yang menjalankan pod-pod. Node bertanggung jawab untuk menyediakan resource (CPU, memori, penyimpanan) yang dibutuhkan oleh pod untuk berfungsi. Dalam sebuah cluster, kita dapat memiliki beberapa node yang masing-masing memiliki pod yang berbeda.

- Node dibagi menjadi dua jenis: **Master node** (yang mengelola kontrol dan pengaturan cluster) dan **Worker node** (yang menjalankan aplikasi dalam bentuk pod).
- Setiap node selalu memiliki komponen penting seperti kubelet, kube-proxy, dan container manager.

### **Apa itu Pod?**
Pod adalah unit terkecil yang dapat dideploy dalam Kubernetes. Pod bisa berisi satu atau lebih container yang berjalan bersama di dalam satu lingkungan. Container-container dalam pod ini berbagi IP, volume storage, dan namespace, yang memungkinkan mereka berkomunikasi dengan lancar.

- **Pod bisa berisi lebih dari satu container** yang saling berkomunikasi dan berbagi resource.
- **Pod tidak dapat berjalan di node yang berbeda**. Semua container dalam satu pod dijadwalkan untuk berjalan pada node yang sama, yang memungkinkan mereka berbagi konfigurasi dan resource secara langsung

### **Labels dalam Kubernetes**
Labels adalah metadata yang digunakan untuk memberi identifikasi atau tanda pada objek-objek di Kubernetes, seperti pod, service, replication controller, dan lainnya. Dengan menggunakan labels, kita bisa mengelompokkan objek-objek tersebut berdasarkan kriteria tertentu, serta melakukan pencarian atau seleksi.

**Manfaat Labels**:
- Labels memudahkan kita untuk mengorganisir dan mencari pod berdasarkan kriteria tertentu.
- Labels juga memungkinkan kita untuk melakukan kontrol lebih lanjut, seperti memilih pod tertentu untuk dilakukan scaling atau update.

**Contoh penggunaan label pada pod**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-labels
  labels:
    app: nginx
    environment: production
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Untuk menampilkan label pada pod, kita dapat menggunakan perintah berikut:

```bash
$ kubectl get pods --show-labels
```

Untuk mencari pod berdasarkan label:

```bash
$ kubectl get pods -l environment=production
```

---

### **Annotation dalam Kubernetes**
Annotations berfungsi untuk menambahkan metadata atau informasi lebih rinci pada objek, namun tidak seperti labels, annotations **tidak dapat difilter atau dicari**. Biasanya, annotations digunakan untuk menyimpan informasi besar atau deskriptif, seperti metadata aplikasi, versi, atau informasi debug.

**Contoh penggunaan annotation pada pod**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-annotation
  annotations:
    description: Aplikasi ini dikembangkan oleh tim ceker-ayam
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Untuk melihat annotation pada pod, kita bisa menggunakan:

```bash
$ kubectl describe pod nginx-pod-annotation
```

Jika kita ingin menambahkan annotation:

```bash
$ kubectl annotate pod <pod-name> key="value"
```

### **Namespace dalam Kubernetes**
Namespace digunakan untuk mengelompokkan dan memisahkan berbagai resource dalam cluster Kubernetes. Ini sangat berguna terutama ketika kita memiliki banyak tim yang bekerja dalam satu cluster atau ketika kita ingin memisahkan berbagai lingkungan (seperti development, staging, dan production).

**Kapan menggunakan Namespace**:
- Ketika jumlah resource dalam cluster terlalu besar dan perlu dipisahkan untuk organisasi yang lebih baik.
- Untuk memisahkan resource antar tim, aplikasi, atau lingkungan.

**Contoh melihat namespaces**:

```bash
$ kubectl get namespaces
```

Jika kita membuat resources tanpa menentukan namespace, Kubernetes akan menempatkannya pada namespace **default**.

Untuk melihat resource dalam namespace tertentu:

```bash
$ kubectl get pods --namespace <namespace-name>
```

### **Probe dalam Kubernetes**
Probe adalah mekanisme yang digunakan untuk memeriksa status kesehatan aplikasi dalam pod. Tiga jenis probe yang digunakan dalam Kubernetes adalah:

1. **Liveness Probe**  
   Digunakan untuk mengecek apakah aplikasi dalam pod masih berjalan. Jika liveness probe gagal, Kubernetes akan merestart pod tersebut.

2. **Readiness Probe**  
   Menentukan apakah pod siap menerima trafik. Jika readiness probe gagal, Kubernetes akan menghentikan pengiriman trafik ke pod tersebut hingga probe berhasil.

3. **Startup Probe**  
   Startup probe digunakan untuk aplikasi yang membutuhkan waktu lama untuk startup. Jika aplikasi belum siap, Kubernetes tidak akan menjalankan pengecekan liveness atau readiness terlalu cepat.

**Jenis pengecekan probe**:
- **HTTP Get**: Melakukan permintaan HTTP GET ke endpoint aplikasi dalam pod.
- **TCP Socket**: Memeriksa koneksi TCP ke aplikasi dalam pod.
- **Exec**: Menjalankan perintah dalam container untuk memeriksa status aplikasi.
