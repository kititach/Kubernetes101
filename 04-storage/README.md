# Module 04: Storage

เป้าหมาย: เรียนรู้วิธีการเก็บข้อมูลแบบถาวร (Persistent Data) ใน Kubernetes เพื่อป้องกันไม่ให้ข้อมูลสูญหายเมื่อ Pod ตายหรือถูกลบ

## Persistent Volume (PV) และ Persistent Volume Claim (PVC)

*   **PV (Persistent Volume):** พื้นที่เก็บข้อมูลจริงในคลัสเตอร์ (สร้างโดย Admin หรือสร้างอัตโนมัติผ่าน StorageClass)
*   **PVC (Persistent Volume Claim):** ใบเบิก/คำร้องขอใช้งาน Storage ที่นักพัฒนา (Developer) สร้างขึ้นเพื่อระบุขนาดและสิทธิ์ที่ต้องการ จากนั้น K8s จะนำ PVC นี้ไปจับคู่กับ PV ที่เหมาะสม
*   **StorageClass:** ตัวช่วยสร้าง PV ให้แบบอัตโนมัติ (Dynamic Provisioning) ใน Minikube จะมี default StorageClass ชื่อ `standard` มาให้ ทำให้เราสร้าง PVC แล้วได้ PV มาใช้งานทันทีโดยไม่ต้องสร้าง PV รอไว้ก่อน

---

## ภาคปฏิบัติ (Hands-on)

### ขั้นตอนที่ 1: สร้าง PVC (คำร้องขอพื้นที่เก็บข้อมูล)

เราจะสร้าง PVC ขนาด 1Gi เพื่อขอพื้นที่เก็บข้อมูลมาใช้งาน

#### โค้ดใน [pvc.yaml](file:///home/demo/lab/k8s/04-storage/pvc.yaml):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce # อนุญาตให้เขียนอ่านได้ทีละ 1 Node ณ ช่วงเวลานั้น ๆ
  resources:
    requests:
      storage: 1Gi  # ขนาดความจุที่ขอใช้งาน
```

#### วิธีดำเนินการ:
```bash
# 1. Apply ไฟล์ PVC
kubectl apply -f pvc.yaml

# 2. ตรวจสอบสถานะของ PVC
kubectl get pvc my-pvc
# สังเกตคอลัมน์ STATUS จะเปลี่ยนเป็น "Bound" อย่างรวดเร็ว (เพราะ Minikube สร้าง PV มาผูกให้ทันที)

# 3. ตรวจสอบ PV ที่ระบบสร้างขึ้นมาอัตโนมัติ
kubectl get pv
```

---

### ขั้นตอนที่ 2: สร้าง Pod และเชื่อมต่อกับ PVC

เราจะนำ PVC ที่สร้างไว้ในข้อ 1 มา Mount เข้ากับไดเรกทอรีของ Nginx Container เพื่อเก็บข้อมูลเว็บ

#### โค้ดใน [storage-pod.yaml](file:///home/demo/lab/k8s/04-storage/storage-pod.yaml):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
    - mountPath: "/usr/share/nginx/html" # ไดเรกทอรีที่ต้องการเก็บข้อมูลถาวร
      name: my-storage
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc                  # ผูกกับ PVC ชื่อ my-pvc ที่เราสร้างไว้ก่อนหน้า
```

#### วิธีดำเนินการ:
```bash
# 1. Apply ไฟล์ Pod
kubectl apply -f storage-pod.yaml

# 2. ตรวจสอบสถานะของ Pod จนกระทั่งเป็น Running
kubectl get pod storage-pod
```

---

### ขั้นตอนที่ 3: ทดสอบการทนทานของข้อมูล (Data Persistence Test)

เราจะพิสูจน์ว่าเมื่อเขียนไฟล์ลงใน Disk แล้วลบ Pod ทิ้ง ข้อมูลที่เก็บไว้จะไม่สูญหาย

```bash
# 1. เขียนไฟล์ index.html เข้าไปที่พาธที่ถูก Mount ไว้ใน Pod
kubectl exec storage-pod -- sh -c "echo 'Hello from PVC Storage!' > /usr/share/nginx/html/index.html"

# 2. ลองทดสอบอ่านข้อมูลใน Pod ดูว่าเขียนสำเร็จจริงไหม
kubectl exec storage-pod -- cat /usr/share/nginx/html/index.html
# ผลลัพธ์ควรแสดง: Hello from PVC Storage!

# 3. จำลองสถานการณ์ Pod พัง หรือถูกอัปเดต ด้วยการสั่งลบ Pod ทิ้ง
kubectl delete pod storage-pod

# 4. สั่งสร้าง Pod ขึ้นมาใหม่
kubectl apply -f storage-pod.yaml

# 5. รอจน Pod รันสำเร็จ (Running)
kubectl get pod storage-pod --watch

# 6. ลองเรียกดูข้อมูลไฟล์เดิมใน Pod ตัวใหม่
kubectl exec storage-pod -- cat /usr/share/nginx/html/index.html
# ผลลัพธ์ควรยังคงแสดง: Hello from PVC Storage! แม้ว่าตัว Pod จะถูกลบและสร้างใหม่แล้วก็ตาม
```

---

## การเก็บกวาด (Cleanup)

เมื่อทดสอบเสร็จสิ้นแล้ว สามารถเคลียร์ทรัพยากรทั้งหมดได้ด้วยคำสั่งนี้:

```bash
kubectl delete -f storage-pod.yaml
kubectl delete -f pvc.yaml
```
