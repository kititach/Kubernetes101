# Module 02: Workloads

เป้าหมาย: ทำความเข้าใจ Workload ประเภทต่างๆ ที่ใช้ในการรันแอปพลิเคชันจริงบน Production

## Deployment และ ReplicaSet

Deployment คือตัวจัดการเวอร์ชันของแอปพลิเคชัน และทำงานร่วมกับ ReplicaSet เพื่อรับประกันจำนวน Pod

สร้างไฟล์ `nginx-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:  # <--- Pod Blueprint
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24-alpine
        ports:
        - containerPort: 80
```

### คำสั่งทดสอบ
```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get rs
kubectl get pods

# ทดลองจำลอง Pod พัง (ลบ 1 ตัว K8s จะสร้างใหม่ทันที)
kubectl delete pod <pod-name>
```

### การทำ Rolling Update
ลองแก้เวอร์ชัน Image ในไฟล์เป็น `nginx:1.25-alpine` แล้ว apply ใหม่
```bash
# ดูสถานะการอัปเดต
kubectl rollout status deployment/nginx-deployment

# ดูประวัติ
kubectl rollout history deployment/nginx-deployment

# ถอยเวอร์ชันกลับ (Rollback)
kubectl rollout undo deployment/nginx-deployment
```

---

---

## Resource Requests & Limits

กำหนดขอบเขต CPU/Memory ของ Container — ถ้าไม่กำหนด Pod อาจกิน Resource ของทั้ง Node

- **Requests:** ปริมาณ minimum ที่ K8s จะจองให้ (ใช้ในการเลือก Node)
- **Limits:** ปริมาณสูงสุดที่ยอมให้ใช้ (เกินแล้วถูก throttle/kill)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "200m"
            memory: "128Mi"
```

*(100m CPU = 0.1 core, 64Mi = 64 Mebibytes)*

---

## Liveness & Readiness Probes

- **Liveness:** Pod ยังทำงานปกติไหม? ถ้า fail → K8s จะ Restart Container
- **Readiness:** Pod พร้อมรับ Traffic ไหม? ถ้า fail → K8s จะหยุดส่ง Request มา (แต่ไม่ Restart)

```yaml
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5   # รอให้แอปเริ่มก่อน
          periodSeconds: 10         # ตรวจทุก 10 วินาที
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
```

### ทดสอบ
```bash
kubectl apply -f nginx-deployment.yaml
kubectl describe pod <pod-name> | grep -A10 "Liveness\|Readiness"
```

---

## Workload ประเภทอื่นๆ (ทฤษฎี)
- **StatefulSet:** ใช้กับ Database เช่น MySQL, MongoDB เพราะรับประกันลำดับชื่อ `pod-0`, `pod-1` และการเกาะกับดิสก์ (PVC)
- **DaemonSet:** กระจาย Pod ไปยังทุก Node โดยอัตโนมัติ (เช่น Fluentd สำหรับเก็บ Log)
- **Job / CronJob:** สำหรับงานประมวลผลที่ทำแล้วจบไป (ไม่รันตลอดเวลา)
