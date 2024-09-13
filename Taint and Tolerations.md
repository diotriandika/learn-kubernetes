## Taints and Tolerations Overview

Dalam kuberentes, Taints and Tolerations digunakan untuk mengontrol penjadwalan Pod ke nodes dalam cluster. Kedua hal tersebut membantu untuk memastikan bahwa hanya Pod tertentu yang akan diletakan di node tertentu yang bergantung juga dari kondisi serta kebutuhan yang spesifik. Dengan begitu, Taints dan Tolerations berkerja bersama untuk memastikan Pods tidak dischedule pada nodes yang tidak tepat. **Taints** ditambahkan pada node sedangkan **Tolerantions** didefiniskan dalam PodSpec. 

Sebagai contoh, Kubernetes secara otomatis menambahkan **Taint** di master-nodes jadi hanya Pod khusus yang memanage control plane  yang akan dischedule di pada master-node tersebut dan tidak pada node lain seperti worker node. Ini dapat memastikan master-node menjalankan pod-pod control plane.

![image](https://www.densify.com/wp-content/uploads/article-k8s-capacity-taint-tollerations.svg)

### Taints

Taints ditambahkan ke node untuk memastikan bahwa pod tidak akan dijadwalkan/dischedule ke node tersebut, kecuali jika pod memiliki toleration yang sesuai. Taints digunakan untuk menandai node dengan atribut tertentu atau dengan limitasi, seperti hanya menggunakan nodes tertentu untuk workloads tertentu, atau mencegah Pod diletakan pada node yang memiliki karakteristik tertentu. Setiap taint  terdiri dari 3 bagian, yakni:

- **Key**: mewakili nama dari taints untuk mengidentifikasinya secara unique.
- **Value**: disini bersifat opsional yang dimana terkait dengan **Key**.
- **Effect**: bertugas untuk menentukan bagaimana taint akan mempengaruhi penjadwalan Pod.

Terdapat 3 values yang dapat digunakan dalam `Effect`, yakni:

- **NoSchedule**: Tidak akan ada Pod yang dischedule pada node yang ditaint, terkecuali Pod tersebut memiliki `toleration` yang sesuai.
- **PreferNoSchedule**: adalah sebuah "preferensi" atau versi yang lebih "lembut" dari effect `NoSchedule`. Control plane akan *mencoba* menghindari meletakan Pod yang tidak mentoleransi taint pada node, tapi hal tersebut tidak terjamin/guaranteed seperti halnya `NoSchedule`.
- **NoExecute**: Mematikan semua pod yang tidak mentoleransi adanya taint pada node.

Contohnya jika kita ingin menghindari pod dijadwalkan pada node tertentu terkecuali pod tersebut memiliki toleration yang sesuai. Dibawah saya menambahkan taint pada node yang memiliki GPU, jadi misal saya ingin menjadwalkan Pod yang membutuhkan workload gpu maka akan berjalan di node dengan toleration yang sesuai dan tidak akan dijadwalkan pada Node lain. Ini sesuai dengan sifat dari effect NoSchedule.

```bash
kubectl taint nodes <nama-node> gpu=true:NoSchedule
```

Taint diatas akan membuat node tidak akan bisa dischedule/dijadwalkan pod apapun terkecuali pod tersebut memiliki toleration yang cocok dengan `gpu=true`.

### Toleration

Untuk memungkinkan pod dischedule ke node yang memiliki taint, Pod tersebut perlu menambahkan toleration di PodSpecnya. Dengan menyesuaikan toleration, kita dapat memastikan hanya pod tertentu yang dapat dischedule pada node yang memiliki kecocokan key, value dan effect dari taint.

Berikut adalah contoh menggunakan toleration untuk memungkinkan pod dischedule pada node yang memiliki taint gpu=true

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-gpu-intensive
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

Pada contoh diatas kita membuat Pod dengan toleration untuk node yang memiliki taint gpu yang memungkinkannya untuk berjalan pada node tersebut.

Default value dari `operator` adalah `Equal`. Toleration mencocokan sebuah taint jika key dan effectnya sama. Namun ada beberapa case yang membutuhkan seleksi lebih akurat. Berikut adalah value dari `operator`:

- `Exists` jika hanya ingin mencocokan key taint dan tidak memperdulikan value dari key tersebut.
- `Equal` jika ingin menseleksi value dari taint key. Jadi jika sebuah key memiliki beberapa value, maka Pod tidak akan dischedule pada node yang memiliki value berbeda.

```bash
# Make sure the pod running in nodes that matches the toleration
$ k get pod -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
pod-gpu-intensive   1/1     Running   0          72s   192.168.140.22   k8s-worker-2   <none>           <none>

# Checking node taints
$ k describe nodes k8s-worker-1 | grep Taints:
Taints:             gpu=false:NoSchedule
$ k describe nodes k8s-worker-2 | grep Taints:
Taints:             gpu=true:NoSchedule
```

Kita juga bisa menggunakan taint and toleration untuk menschedule pod kemaster-node. Yang dimana ini tidak mungkin jika kita menggunakan NodeSelector.

Referensi:

- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
- https://medium.com/@prateek.malhotra004/demystifying-taint-and-toleration-in-kubernetes-controlling-the-pod-placement-with-precision-d4549c411c67
- https://www.densify.com/kubernetes-autoscaling/kubernetes-taints/
