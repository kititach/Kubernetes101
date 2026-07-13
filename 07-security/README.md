# Module 07: Security

เป้าหมาย: การจำกัดสิทธิ์ในระดับ K8s (RBAC) และความปลอดภัยระดับ Container (Security Context)

## ไฟล์ใน Module นี้
- [rbac.yaml](rbac.yaml): กำหนด ServiceAccount, Role และ RoleBinding เพื่อจำกัดสิทธิ์ใน K8s Cluster
- [secure-pod.yaml](secure-pod.yaml): Pod ที่มีการตั้งค่าจำกัดสิทธิ์ใน Container (Non-root, No Privilege Escalation)

---

## 1. RBAC (Role-Based Access Control)
ควบคุมสิทธิ์การทำธุรกรรมกับ K8s API Server โดยกำหนดว่า "ใคร (Subject)" ทำ "อะไร (Verb)" กับ "ทรัพยากรไหน (Resource)"

### ส่วนประกอบของ RBAC:
1. **ServiceAccount:** บัญชีประจำตัวของ Pod สำหรับเรียกใช้ API ภายใน Cluster
2. **Role:** ประกาศสิทธิ์การทำกิจกรรมต่างๆ ภายใน Namespace ที่ระบุ
3. **RoleBinding:** การเชื่อมโยง ServiceAccount เข้ากับ Role เพื่อรับสิทธิ์

### 🛠️ ขั้นตอนการทำ Lab: RBAC
1. **สร้างทรัพยากร RBAC:**
   ```bash
   kubectl apply -f rbac.yaml
   ```
2. **ทดสอบตรวจสอบสิทธิ์ด้วยคำสั่ง `kubectl auth can-i`:**
   - ทดสอบว่า ServiceAccount `my-app-sa` มีสิทธิ์ดึงข้อมูล Pod หรือไม่:
     ```bash
     kubectl auth can-i list pods --as=system:serviceaccount:default:my-app-sa
     ```
     *(ควรตอบกลับมาเป็น `yes`)*
   - ทดสอบว่ามีสิทธิ์ลบหรือสร้าง Pod หรือไม่:
     ```bash
     kubectl auth can-i create pods --as=system:serviceaccount:default:my-app-sa
     ```
     *(ควรตอบกลับมาเป็น `no`)*

---

## 2. Security Context (ความปลอดภัยของ Container)
การควบคุมพฤติกรรมการทำงานและความปลอดภัยในระดับรันไทม์ภายใน Container เช่น การไม่ใช้งานสิทธิ์ root หรือสิทธิ์แอดมินของเครื่อง Host

### 🛠️ ขั้นตอนการทำ Lab: Security Context
1. **สร้าง Pod ที่เปิดใช้ Security Context:**
   ```bash
   kubectl apply -f secure-pod.yaml
   ```
2. **ทำความเข้าใจจุดสำคัญ (Gotchas):**
   > [!IMPORTANT]
   > Pod นี้รันโดยระบุ `runAsUser: 1000` (ไม่ใช่ root) และใช้ภาพ `nginxinc/nginx-unprivileged:alpine` แทน `nginx:alpine` ทั่วไป
   > - หากใช้อิมเมจปกติ ตัว Nginx จะพยายามรันด้วยสิทธิ์ root เพื่อเปิด Port 80 และเขียนไฟล์ Cache ในไดเรกทอรีส่วนของระบบ ส่งผลให้เกิดสถานะ `CrashLoopBackOff` ทันที
   > - อิมเมจ `nginx-unprivileged` ได้รับการออกแบบให้รันบนสิทธิ์ User ปกติ (รันที่พอร์ต 8080)
3. **ตรวจสอบว่ารันด้วยสิทธิ์ปกติจริงหรือไม่:**
   ```bash
   kubectl exec secure-pod -- id
   ```
   *(ผลลัพธ์ควรแสดง UID เป็น `1000` ซึ่งไม่ใช่ root)*
4. **เคลียร์สภาพแวดล้อม:**
   ```bash
   kubectl delete -f rbac.yaml
   kubectl delete -f secure-pod.yaml
   ```
