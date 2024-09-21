### Deploying Metric Server

Metrics server sebuah resource metrics monitoring tool untuk kubernetes. Metrics server disini digunakan untuk mengukur penggunaan CPU dan Memory Pods atau Nodes dalam cluster. Metrics server juga bersifat scalable dan efisien.

**Jalankan di Node Control Plane**

1. Download manifest lalu edit

   ```bash
   # Download metrics-server manifest
   $ wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
   
   # Edit metrics-server manifest
   $ nano high-availability-1.21+.yaml
   ```

2. Di bagian deployment metrics-server, tambahkan `args` baru yakni `--kubelet-insecure-tls` untuk mendisable certificate verification.

   ```yaml
                   k8s-app: metrics-server
               namespaces:
               - kube-system
               topologyKey: kubernetes.io/hostname
         containers:
         - args:
           - --cert-dir=/tmp
           - --secure-port=10250
           - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
           - --kubelet-use-node-status-port
           - --metric-resolution=15s
           - --kubelet-insecure-tls 	<<--- add this
   ```

3. Apply manifest & cek status deployment  metrics-server

   ```bash
   # Apply manifest
   $ kubectl apply -f high-availability-1.21+.yaml
   
   # Check metrics server deployment
   $ kubectl get deployments -n kube-system
   NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
   calico-kube-controllers   1/1     1            1           124m
   coredns                   2/2     2            2           169m
   metrics-server            2/2     2            2           18m
   ```

4. Verifikasi dengan menjalankan `kubectl top`

   ```bash
   # Check nodes usage
   $ kubectl top nodes
   NAME              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
   control-plane-1   148m         7%     1295Mi          70% 
   control-plane-2   125m         6%     994Mi           54%
   control-plane-3   135m         6%     1009Mi          55%
   worker-1          68m          3%     908Mi           49%
   worker-2          60m          3%     827Mi           45%
   
   # Check pods usage
   $ kubectl top pods -A
   NAMESPACE     NAME                                       CPU(cores)   MEMORY(bytes)   
   kube-system   calico-kube-controllers-7fbd86d5c5-2bzzc   3m           12Mi            
   kube-system   calico-node-4l5m5                          40m          125Mi           
   kube-system   calico-node-fghjs                          38m          129Mi           
   kube-system   calico-node-m9jmx                          35m          119Mi           
   kube-system   calico-node-pkr4h                          38m          135Mi           
   kube-system   calico-node-qg97m                          39m          116Mi 
   ```

Referensi: 

- https://github.com/kubernetes-sigs/metrics-server
- https://medium.com/google-cloud/metrics-server-in-kubernetes-d190410a8e30
