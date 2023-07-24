# dynamic-nfs-provisioing
#### Dynamic volume provisioning in Kubernetes automates the creation and management of NFS storage volumes on-demand, simplifying storage management, improving resource utilization, enabling scalability, and providing standardized and portable storage solutions.
## Kubernetes NFS Subdir External Provisioner
#### NFS subdir external provisioner is an automatic provisioner that use your existing and already configured NFS server to support dynamic provisioning of Kubernetes Persistent Volumes via Persistent Volume Claims.

### Setup a provisioner for dynamic nfs provisioning in Kubernetes
#### For Ths, we will be using helm chart to deploy the provisioner
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner 
```
```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=nfs_ip  --set nfs.path=nfs_path
```
##### For example:
```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=198.58.104.162  --set nfs.path=/srv/nfs/kubedata
```
## You can change the nfs name and set it as default using :
```
helm upgrade --install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=fs-0e6c8b8e20cd5356f.efs.us-east-2.amazonaws.com  --set nfs.path=/ --set storageClass.defaultClass=default --set storageClass.name=second-nfs-client
```

###### Please change the ip addres and the path
##### On AWS EFS , here it is
####### Create a EFS and then copy the DNS name of the EFS
![image](https://github.com/thedevopsguru1/dynamic-nfs-provisioing/assets/126810742/c2136ea6-eb7e-4f4c-bf39-3eed8f5ff764)

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=fs-0e4070b4c27a59f48.efs.us-east-2.amazonaws.com  --set nfs.path=/
```
### Let's test it by deploying a PVC and Pod 
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: test-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage  
          
   ```
   ```
   kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
   
   ```
