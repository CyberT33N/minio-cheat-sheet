# MinIO Cheat Sheet







<br><br>
<br><br>
___________________________________________
___________________________________________
<br><br>
<br><br>

## Minikube
- minio-dev.yaml
```yaml
# Deploys a new Namespace for the MinIO Pod
apiVersion: v1
kind: Namespace
metadata:
  name: minio-dev # Change this value if you want a different namespace name
  labels:
    name: minio-dev # Change this value to match metadata.name
---
apiVersion: v1
kind: Secret
metadata:
  name: minio-secret
  namespace: minio-dev
type: Opaque
data:
  MINIO_ROOT_USER: dGVzdDY5Njk2OTY5  # Base64-kodierter Wert für "test69696969"
  MINIO_ROOT_PASSWORD: dGVzdDY5Njk2OTY5 # Base64-kodierter Wert für "test69696969"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: minio-pv
spec:
  capacity:
    storage: 10Gi # Ändere dies nach Bedarf
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data # Der Pfad auf deinem Minikube-Host
  storageClassName: standard
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-pvc
  namespace: minio-dev # Stelle sicher, dass dies mit dem Namespace übereinstimmt
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi # Sollte gleich oder kleiner als der PV-Speicher sein
  storageClassName: standard
---
# Deploys a new Namespace for the MinIO Pod
apiVersion: v1
kind: Namespace
metadata:
  name: minio-dev # Change this value if you want a different namespace name
  labels:
    name: minio-dev # Change this value to match metadata.name
---
# Deploys a new MinIO Pod into the metadata.namespace Kubernetes namespace
#
# The `spec.containers[0].args` contains the command run on the pod
# The `/data` directory corresponds to the `spec.containers[0].volumeMounts[0].mountPath`
# That mount path corresponds to a Kubernetes Persistent Volume Claim
# 
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: minio
  name: minio
  namespace: minio-dev # Change this value to match the namespace metadata.name
spec:
  containers:
  - name: minio
    image: quay.io/minio/minio:latest
    command:
    - /bin/bash
    - -c
    args: 
    - minio server /data --console-address :9001
    env:
    - name: MINIO_ROOT_USER
      valueFrom:
        secretKeyRef:
          name: minio-secret
          key: MINIO_ROOT_USER
    - name: MINIO_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: minio-secret
          key: MINIO_ROOT_PASSWORD
    volumeMounts:
    - mountPath: /data
      name: minio-storage
  nodeSelector:
    kubernetes.io/hostname: minikube # Correct node hostname
  volumes:
  - name: minio-storage
    persistentVolumeClaim:
      claimName: minio-pvc # Refer to the PVC created above
  - name: secret-volume
    secret:
      secretName: minio-secret
---
apiVersion: v1
kind: Service
metadata:
  name: minio-service
  namespace: minio-dev
spec:
  type: NodePort
  selector:
    app: minio
  ports:
  - name: api-port      # Name des API-Ports
    protocol: TCP
    port: 9000         # Der Port, auf dem der Service intern läuft (API-Port)
    targetPort: 9000   # Der Port des Containers, auf den der Service weiterleitet
    nodePort: 30000    # Der NodePort, der auf jedem Knoten verfügbar sein wird
  - name: webui-port    # Name des WebUI-Ports
    protocol: TCP
    port: 9001         # Der Port für die WebUI
    targetPort: 9001   # Der Port des Containers, auf den der Service weiterleitet
    nodePort: 30001    # Der NodePort für die WebUI

```




### Links
- http://192.168.49.2.nip.io:30001/login
- User:test69696969 | Password:test69696969

<br><br>
<br><br>


### Uninstall

<br><br>

#### Full
```shell
# Delete the MinIO namespace and all resources within it
kubectl delete namespace minio-dev
```

<br><br>

#### Single steps
```shell
# Lösche den Pod
kubectl delete pod minio -n minio-dev

# Lösche das Secret
kubectl delete secret minio-secret -n minio-dev

# Lösche den PersistentVolumeClaim
kubectl delete pvc minio-pvc -n minio-dev

# Lösche den PersistentVolume
kubectl delete pv minio-pv

# Lösche den Service
kubectl delete service minio-service -n minio-dev

# Lösche den Namespace
kubectl delete namespace minio-dev
```


<br><br>
<br><br>

### Install
```shell
bash ./minio/setup.sh
```

<br><br>
<br><br>

### Re-install
```shell
bash ./reinstall.sh --minio
```

<br><br>
<br><br>

### Ugprade
```shell
bash ./minio/setup.sh
```
- For most cases you can just re-run it and it will detect if there are any changes. But in other cases like e.g. where we want to change credentials you have to delete the pod. However, for local environemnts you can just run the reinstall script above. This will be the easieast way when yua re doing some changes


