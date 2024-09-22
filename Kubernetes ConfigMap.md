## ConfigMaps

ConfigMap adalah API Object yang digunakan untuk menyimpan non-confidential (tidak rahasia) data dalam key-value pairs. Pods dapat menggunakan ConfigMap sebagai variabel environment, command-line arguments, atau sebagai configuration files didalam sebuah volume. Kegunaan utama dari ConfigMap adalah untuk memisahkan detail konfigurasi dari container image yang dimana dapat meningkatkan fleksibilitas container tersebut. Konteks nyatanya biasanya ConfigMaps digunakan untuk khusus ke environment yang berbeda seperti development, testing dan production

ConfigMap mempermudah dalam menkonfiugrasi dan management aplikasi. Sehingga tidak lagi memerlukan hardcode data konfigurasi satu per satu ke kontainer.

![image](https://images.prismic.io/qovery/655e12b4531ac2845a255234_unnamed-6-.png?auto=format,compress)

### Examples

Contoh penggunakan ConfigMap

1. Key-Value based

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: example-configmap
   data:
     config.json: |    
       { "key": "value",
         "service": "example"
       }
   ---
   ## Using value from config map example
   apiVersion: v1
   kind: Pod
   metadata:
     name: example-pod-1
   spec:
     containers:
     - name: example-container
       image: example-image
       env:
       - name: SPECIAL_CONFIG
       valueFrom: 
         configMapKeyRef:
           name: example-configmap
           key: config.json
   ```

2. Mount ConfigMap as Volume

   ```bash
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: nginx-conf
   data:
     nginx.conf: |
       worker_processes 1;
       http {
         server {
           listen       80;
           server_name  localhost;
           location /testpath {
             alias /var/www/testpath/;
             index index.html;
           }
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     labels:
       app: nginx-test
   spec:
     containers:
     - name: nginx
       image: nginx
       volumeMounts:
       - name: nginx-conf
         mountPath: /etc/nginx/nginx.conf
         subPath: nginx.conf
         readOnly: true
     volumes:
     - name: nginx-conf
       configMap:
         name: nginx-conf
         items:
         - key: nginx.conf
           path: nginx.conf
   ```

References:

- https://www.qovery.com/blog/kubernetes-configmap-our-complete-guide/
- https://kubernetes.io/docs/concepts/configuration/configmap/
- https://kubernetes.io/docs/tutorials/configuration/updating-configuration-via-a-configmap/
