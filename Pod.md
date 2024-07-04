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

