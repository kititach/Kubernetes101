# Module 06: Scheduling

เป้าหมาย: ควบคุมว่า Pod ตัวไหน ควรไปรันที่ Worker Node เครื่องไหน

## ไฟล์ใน Module นี้
- [nodeselector-pod.yaml](nodeselector-pod.yaml): Pod ที่จำกัดให้ไปรันที่ Node ที่มี Label `disktype=ssd` เท่านั้น
- [toleration-pod.yaml](toleration-pod.yaml): Pod ที่มี Toleration สำหรับ Node ที่มี Taint `gpu=true:NoSchedule`

## 1. Node Selector (ระดับง่ายสุด)
ผูก Pod ให้วิ่งไปหา Node ที่มี Label ตรงกันเท่านั้น
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### 🛠️ ขั้นตอนการทำ Lab: Node Selector
1. **สร้าง Pod:**
   ```bash
   kubectl apply -f nodeselector-pod.yaml
   ```
2. **ตรวจสอบสถานะ:**
   ```bash
   kubectl get pods ssd-pod
   ```
   *จะเห็นสถานะเป็น `Pending` เนื่องจากเครื่อง Node minikube ยังไม่มี Label `disktype=ssd`*
3. **แปะ Label ให้กับ Node:**
   ```bash
   kubectl label nodes minikube disktype=ssd
   ```
4. **ตรวจสอบสถานะอีกครั้ง:**
   ```bash
   kubectl get pods ssd-pod -w
   ```
   *Pod จะเปลี่ยนสถานะเป็น `Running` ทันที*
5. **เคลียร์สภาพแวดล้อม:**
   ```bash
   kubectl delete pod ssd-pod
   kubectl label nodes minikube disktype-
   ```

---

## 2. Taints และ Tolerations (การกีดกัน / อนุญาต)
- **Taint (พ่นยาไล่):** ทำบน Node เพื่อห้าม Pod ทั่วไปมารัน
- **Toleration (ฉีดวัคซีน):** ใส่ใน Pod เพื่อให้ทนต่อ Taint ของ Node นั้นๆ ได้

### 🛠️ ขั้นตอนการทำ Lab: Taints และ Tolerations
1. **ติด Taint ให้กับ Node (พ่นยาไล่):**
   ```bash
   kubectl taint nodes minikube gpu=true:NoSchedule
   ```
2. **ทดสอบสร้าง Pod ปกติ (ไม่มี Toleration):**
   ```bash
   kubectl run test-normal-pod --image=nginx:alpine
   kubectl get pods test-normal-pod
   ```
   *จะค้างที่สถานะ `Pending` เพราะไม่มี Toleration ป้องกัน Taint `gpu=true:NoSchedule`*
3. **สร้าง Pod ที่มี Toleration (มีวัคซีน):**
   ```bash
   kubectl apply -f toleration-pod.yaml
   ```
4. **ตรวจสอบสถานะ:**
   ```bash
   kubectl get pods gpu-pod
   ```
   *`gpu-pod` จะสามารถรันขึ้นมาเป็น `Running` ได้*
5. **เคลียร์สภาพแวดล้อม (ถอน Taint และลบ Pod ทดสอบ):**
   ```bash
   kubectl taint nodes minikube gpu=true:NoSchedule-
   kubectl delete pod test-normal-pod
   kubectl delete pod gpu-pod
   ```

---

## 3. Node Affinity / Anti-Affinity (ระดับซับซ้อน)
ใช้สำหรับความต้องการเชิงลึก เช่น "อยากให้อยู่ Node นี้ แต่ถ้าไม่ได้ก็ไม่เป็นไร" (PreferredDuringScheduling) หรือ "ต้องอยู่ Node นี้เท่านั้น" (RequiredDuringScheduling) ซึ่งจะยืดหยุ่นกว่า Node Selector (ศึกษาเพิ่มเติมรายละเอียดได้จาก K8S-GUIDE)
