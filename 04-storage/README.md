# Module 04: Storage

เป้าหมาย: วิธีเก็บข้อมูลถาวร (Persistent Data) ใน Kubernetes

## Persistent Volume (PV) และ Persistent Volume Claim (PVC)

- **PV:** ทรัพยากร Storage ของ Cluster (Admin เป็นคนสร้าง หรือให้ Cloud สร้างอัตโนมัติผ่าน StorageClass)
- **PVC:** ใบขอพื้นที่ Storage (Developer เป็นคนสร้าง)

### 1. สร้าง PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 2. นำ PVC ไปผูกกับ Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```
