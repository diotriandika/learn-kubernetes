## Pod

Melihat list pod yang ada

```bash
$ kubectl get pod
```

Melihat pod yang ada di seluruh namespace

```bash
$ kubectl get pod -A
```

Melihat detail sebuah pod

```bash
$ kubectl describe pod <pod-name>
```

### Membuat Pod

Untuk membuat pod kita memerlukan sebuah spec file yang berupa yaml, nantinya file ini yang akan kita kirimkan ke kube-apiserver dan kube-apiserver yang akan membuat seluruh resource yang diperlukan untuk menjalankan pod tersebut.

pod configuration file:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-name
spec:
 containers
 - name: container-name
   image: image-name
   ports
   - containerPort: <container-port>
```

Sebagai contoh disini saya akan membuat sebuah pod nginx dengan nama  file pod-nginx.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
   - name: nginx
     image: nginx
     ports:
      - containerPort: 80
```

lalu submmit file tersebut

```bash
$ kubectl create -f pod-nginx.yaml
```

> `-f` parameter untuk memasukan file

Selanjutnya cek apakah pod sudah berhasil dibuat

```bash
$ kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          5m56s
```

atau agar lebih lengkap

```bash
$ kubectl get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          8m19s   10.244.0.3   kube-pzn   <none>           <none>
```

Kita juga bisa melihat detail dari pod tersebut dengan menggunakan `kubectl describe pod <pod-name>`

```bash
$ kubectl describe pod nginx-pod
```

### Mengakses Pod (hanya untuk testing, selengkapnya ada di Resource Services)

Untuk mengakses pod yang sudah dibuat, disini kita bisa menggunakan `kubectl port-forward` untuk menforward pord yang ditentukan ke port pod tersebut. Saat production ga make ini, jadi coba cari tau kenapa ga make ini di production well belum belajar sampe sana soalnya.

```bash
$ kubectl port-forward <pod-name> <access-port>:<pod-port>
```

untuk kasus pod diatas, nginx secara default mengexpose port 80, dan saya akan melakukan port-forward ke port 8080 untuk mengakses pod tersebut.

```bash
$ kubectl port-forward nginx-pod 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

selanjutnya buka browser dan akses http://localhost:8080

![image-20240704201649489](C:\Users\Dio Tri Andika\AppData\Roaming\Typora\typora-user-images\image-20240704201649489.png)

### Menghapus Pod

Untuk dapat menghapus sebuah pod atau lebih, kita bisa menggunakan `kubectl delete pod <pod-name>` & `kubectl delete pod <pod-name1> <pod-name2> <pod-name3>`. Untuk menghapus semua pod dinamespace default `kubectl delete pod --all`  

```bash
$ kubectl delete pod nginx-pod
pod "nginx-pod" deleted
```

Menghapus Pod yang menggunakan Label

```bash
$ kubectl delete pod --labels label-key1="label-value1"
```

Menghapus semua Pod yang ada di sebuah Namespace

```bash
$ kubectl delete pod --all --namespace <namespace-name>
```

