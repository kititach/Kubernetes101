# Module 05: Configuration

เป้าหมาย: การส่งค่า Config และ Password (Secrets) เข้าไปใน Pod โดยไม่ต้องแก้โค้ดหรือ Hardcode ใน Image

## ไฟล์ใน Module นี้
- [configmap.yaml](configmap.yaml): กำหนดค่า Config ทั่วไป
- [app-pod.yaml](app-pod.yaml): Pod ที่รับค่าจาก ConfigMap และ Secret มาใช้งาน

## 1. ConfigMap (สำหรับค่าทั่วไป)
ใช้สำหรับเก็บค่า Configuration ทั่วไปที่ไม่ใช่ความลับ
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
```

## 2. Secret (สำหรับความลับ)
ใช้เก็บข้อมูลสำคัญ เช่น Password หรือ API Key ซึ่งจะเข้ารหัสแบบ Base64 ในระดับ K8s
> [!WARNING]
> การเข้ารหัสใน Secret ของ Kubernetes เป็นเพียงการแปลงเป็น Base64 เท่านั้น **ไม่ใช่การเข้ารหัสลับ (Encryption)** ความปลอดภัยที่แท้จริงต้องอาศัยการจำกัดสิทธิ์การเข้าถึงข้อมูลด้วย RBAC เพื่อป้องกันไม่ให้บุคคลที่ไม่เกี่ยวข้องสามารถดูหรือถอดรหัสได้

สร้าง Secret จาก Command Line:
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=P@ssw0rd123
```

## 3. การนำไปใช้ใน Pod
ดึงค่าจาก ConfigMap และ Secret เข้าไปเป็น Environment Variable ภายใน Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: nginx:alpine
    env:
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    envFrom:
    - configMapRef:
        name: app-config
```

## 🛠️ ขั้นตอนการทำ Lab และตรวจสอบความถูกต้อง
1. **สร้าง ConfigMap:**
   ```bash
   kubectl apply -f configmap.yaml
   ```
2. **สร้าง Secret (บน CLI):**
   ```bash
   kubectl create secret generic db-secret \
     --from-literal=username=admin \
     --from-literal=password=P@ssw0rd123
   ```
3. **สร้าง Pod:**
   ```bash
   kubectl apply -f app-pod.yaml
   ```
4. **ตรวจสอบความถูกต้อง:**
   ทดสอบเรียกดู Environment Variables ใน Pod:
   ```bash
   kubectl exec config-pod -- env | grep -E "APP_|DB_"
   ```
   *ผลลัพธ์ที่ได้ควรแสดง:*
   * `APP_ENV=production`
   * `APP_PORT=8080`
   * `DB_PASS=P@ssw0rd123`

---

## 🔄 วิธีการอัปเดต ConfigMap และข้อควรระวัง (ConfigMap Update Behavior)

เมื่อมีการแก้ไขค่าในไฟล์ `configmap.yaml` (เช่น เปลี่ยน `APP_PORT` เป็น `"9090"`) วิธีการส่งไปอัปเดตและพฤติกรรมของระบบมีดังนี้:

### 1. การอัปเดตทรัพยากรบน Cluster
ใช้คำสั่ง apply ปกติเพื่อส่งค่าใหม่ไปอัปเดตที่ API Server:
```bash
kubectl apply -f configmap.yaml
```

### 2. ผลลัพธ์ต่อ Pod ที่รันอยู่ (สำคัญมาก)
> [!WARNING]
> เนื่องจากในแลปนี้ Pod รับค่า ConfigMap ผ่านรูปแบบ **Environment Variables (`envFrom`)** ตัว Kubernetes จะทำการ Inject ค่าเข้าเครื่องตอนเริ่มต้นสร้าง Pod (Startup) เท่านั้น 
> **ดังนั้น การเปลี่ยนค่าและ apply ConfigMap ใหม่ จะไม่ส่งผลให้อัปเดตค่าตัวแปรใน Container ที่รันอยู่โดยอัตโนมัติ**

### 3. วิธีการอัปเดตให้ Pod รับค่าใหม่
* **สำหรับ Pod ทั่วไป (เดี่ยว/ไม่มี Controller):** ต้องทำการลบและสร้าง Pod ใหม่เท่านั้น
  ```bash
  kubectl delete pod config-pod && kubectl apply -f app-pod.yaml
  # หรือใช้คำสั่งบังคับสร้างใหม่:
  kubectl replace --force -f app-pod.yaml
  ```
* **สำหรับ Pod ที่ถูกดูแลโดย Deployment:** ให้สั่งทำ Rollout Restart เพื่อสร้าง Pod ใหม่ตามลำดับ
  ```bash
  kubectl rollout restart deployment <deployment-name>
  ```

---

## 💡 ข้อมูลเพิ่มเติม: หากต้องการอัปเดตโดยไม่ต้องรีสตาร์ท Pod?
หากต้องการอัปเดตค่าแบบ Dynamic (Auto-Reload) โดยไม่ต้องรีสตาร์ท Pod คุณต้องหลีกเลี่ยงการใช้ `env` หรือ `envFrom` และหันไปใช้ **Mount ConfigMap เป็น Volume (ไฟล์)** แทน:
- Kubelet จะทำการ Sync อัปเดตไฟล์ข้อมูลภายใน Container ให้อัตโนมัติ (ปกติจะใช้เวลาดีเลย์ประมาณ 1-2 นาที)
- โค้ดของแอปพลิเคชันต้องรองรับการอ่านไฟล์ซ้ำแบบ Dynamic (Hot-reload) เมื่อไฟล์มีการเปลี่ยนแปลง

