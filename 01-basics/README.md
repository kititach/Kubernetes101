# Module 01: K8s Basics

เป้าหมาย: ทำความเข้าใจและทดลองสร้างส่วนประกอบพื้นฐานที่สุดของ Kubernetes คือ "Pod"

## แนวคิดหลัก

- **Pod:** เหมือนรังไหมที่ห่อหุ้ม Container ไว้ K8s ไม่ได้จัดการ Container โดยตรง แต่จัดการผ่าน Pod
- 1 Pod ควรมี 1 หน้าที่หลัก
- ไม่ควรสร้าง Pod ตรงๆ ใน Production (ควรใช้ Deployment แทน) แต่จำเป็นต้องรู้เพื่อเข้าใจกลไก

## ภาคปฏิบัติ (Hands-on)

### 1. สร้าง Pod ด้วย Imperative Command
```bash
# สั่งสร้างทันที (ไม่แนะนำในชีวิตจริง)
kubectl run my-nginx --image=nginx:alpine

# ดูผลลัพธ์
kubectl get pods
```

### 2. สร้าง Pod ด้วย Declarative YAML (วิธีที่ถูกต้อง)
สร้างไฟล์ `pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx-container
    image: nginx:alpine
    ports:
    - containerPort: 80
```

รันคำสั่ง:
```bash
kubectl apply -f pod.yaml
```

### 3. ตรวจสอบและ Debug
```bash
kubectl get pods -o wide
kubectl describe pod my-first-pod
kubectl logs my-first-pod
```

### 4. ลบทำความสะอาด
```bash
kubectl delete pod my-nginx
kubectl delete -f pod.yaml
```

---

## Namespace

Namespace คือการแบ่ง Cluster ออกเป็นพื้นที่ย่อยๆ — ทุกอย่างที่ทำมาตอนนี้อยู่ใน `default` namespace โดยอัตโนมัติ

### คำสั่งพื้นฐาน
```bash
# ดู namespace ทั้งหมดในระบบ
kubectl get namespaces

# ดู pods ใน namespace ที่ระบุ
kubectl get pods -n kube-system

# ดู pods ทุก namespace
kubectl get pods -A
```

### สร้างและใช้ Namespace แยก
```bash
# สร้าง namespace ใหม่
kubectl create namespace dev
kubectl create namespace staging
```

```yaml
# pod-in-dev.yaml — กำหนด namespace ใน metadata
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:alpine
```

```bash
kubectl apply -f pod-in-dev.yaml

# ต้องระบุ -n เสมอเมื่อดู resource ใน namespace อื่น
kubectl get pods -n dev

# ลบ namespace จะลบทุกอย่างข้างในด้วย
kubectl delete namespace dev
```
