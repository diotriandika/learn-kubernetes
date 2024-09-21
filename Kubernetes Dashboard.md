### Deploying Kuberentes Dashboard

Kubernetes menyediakan sebuah web-based dashboard yang dapat digunakan untuk membuat, monitor hingga memanage sebuah cluster.

**Jalankan di Node Control Plane**

1. Install Helm untuk mempermudah instalasi Kubernetes Dashboard. 

   ```bash
   # Install Helm
   $ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   
   # Verify installation
   $ helm version
   version.BuildInfo{Version:"v3.16.1"
   ```

2. Tambahkan Kubernetes Dashboard helm repository.

   ```bash
   $ helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
   ```

3. Deploy chart kuberentes dashboard dengan nama `kubernetes-dashboard` beserta namespacenya.

   ```yaml
   $ helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
   ```

   > Secara default k8s dashboard web type servicenya adalah ClusterIP, yang berarti hanya bisa diakses didalam cluster saja. Maka dari itu kita perlu mengekspos menggunakan NodePort atau Ingress.

4. Rubah service `kubernetes-dashboard-kong-proxy` ke NodePort

   ```bash
   # Create a file named values.yaml with the following content
   $ cat << EOF | tee values.yaml
   kong:
     proxy:
       type: NodePort
     http:
       enabled: true
   EOF
   ```

5. Upgrade helm release kubernetes dashboard dengan file yaml sebelumnya.

   ```bash
   $ helm upgrade kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard -f values.yaml -n kubernetes-dashboard
   ```

6. Cek service `kubernetes-dashboard-kong-proxy` dan expose portnya

   ```bash
   $ kubectl get svc -n kubernetes-dashboard
   NAME                                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
   kubernetes-dashboard-api               ClusterIP   10.104.2.57      <none>        8000/TCP                        13m
   kubernetes-dashboard-auth              ClusterIP   10.102.227.161   <none>        8000/TCP                        13m
   kubernetes-dashboard-kong-manager      NodePort    10.111.242.82    <none>        8002:31487/TCP,8445:32011/TCP   13m
   kubernetes-dashboard-kong-proxy        NodePort    10.98.122.251    <none>        443:30183/TCP                   13m
   kubernetes-dashboard-metrics-scraper   ClusterIP   10.107.59.165    <none>        8000/TCP                        13m
   kubernetes-dashboard-web               ClusterIP   10.108.251.243   <none>        8000/TCP                        13m
   ```

7. Coba akses ke web browser dengan format `https://<virtual-ip>:<exposed-port`

   ![image-20240921170225143](https://github.com/user-attachments/assets/72495f3e-1139-4604-b9ca-d494caee998f)



8. Untuk dapat login, kita memerlukan sebuah bearer token yang dapat digenerate melalui sebuah service account. Untuk itu buat manifest untuk user account yang akan digunakan lalu apply.

   ```bash
   $ nano admin-dashboard.yaml
   ```

   ```yaml
   ## Define ServiceAccount
   apiVersion: v1 
   kind: ServiceAccount
   metadata:  
     name: admin-user  
     namespace: kubernetes-dashboard
   
   ---
   ## Define ClusterRoleBinding
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRoleBinding
   metadata:  
     name: admin-user
   roleRef:
     apiGroup: rbac.authorization.k8s.io  
     kind: ClusterRole  
     name: cluster-admin
   subjects: 
     - kind: ServiceAccount  
       name: admin-user  
       namespace: kubernetes-dashboard
   
   ---
   ## Define Secret
   apiVersion: v1
   kind: Secret
   metadata:  
     name: admin-user  
     namespace: kubernetes-dashboard  
     annotations:    
       kubernetes.io/service-account.name: "admin-user"
   type: kubernetes.io/service-account-token
   ```

   ```bash
   $ kubectl apply -f admin-dashboard.yaml 
   serviceaccount/admin-user created
   clusterrolebinding.rbac.authorization.k8s.io/admin-user created
   secret/admin-user created
   ```

9. Decode access token dari secret yang sudah dibuat.

   ```bash
   $ kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
   eyJhbGciOiJSUzI1NiIsImtpZCI6IkRMV2NDQmtBWVo4X2hvaF9Ianhab1h5ZGJIblNSc2lfbU5FSFdDbEdpLWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwZTlkNGI5ZS00MDRiLTRjMWEtYmIyNi04YTk3MWQzY2YyNTgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.vO4ninblQTK8p6oDebZhK-WN9UlfM0miOLucYhqnjBOr1KDwiTJ0TpuPCt0ypsZ_thHMvwVerp32o2VkeQavP14_ng_mWRPwAHt5EqxX59rahNvhz5r2f4dmLnqlClEr7qdCmBpmkjsiUXgEw2sC2El4mvT2VMjGc64pfnfd3vHtZ0XukN3EL-PJJwb2rqBjX0C9vg6n6RNffb7ZvWwk1voISC_bAqBU08WZ7RjgVh2RrkXXoplLLje97UZT81509QTt0OwmUd_hM-XoUz8tBhutIHGyP19olK-dhxzb1N3XlaVTj_D0cf2swYGXPu9ejyuaJMaD8oINyU1EA8VzVw
   ```

   > akan muncul random text seperti diatas, ini adalah bearer token dari admin-user.

10. Gunakan token diatas untuk login ke dashboard

    ![image-20240921171401999](https://github.com/user-attachments/assets/7b3fc309-9686-424c-8d96-1812d6ceb8da)


Referensi:

- https://www.kerno.io/learn/kubernetes-dashboard-deploy-visualize-cluster
- https://github.com/kubernetes/dashboard
