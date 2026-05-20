# Module 03: Networking & Services

เป้าหมาย: ให้ Pods สามารถสื่อสารกันเองได้ และอนุญาตให้ Traffic ภายนอกเข้ามาที่ Pods ได้

## แนวคิดหลัก

Pod ตายแล้ว IP เปลี่ยน เราจึงใช้ **Service** มาเป็นหน้าด่าน โดยมี IP คงที่ และส่ง Traffic ไปหา Pods โดยอัตโนมัติ (ผ่าน Label Selector)

### ประเภทของ Service
1. **ClusterIP:** เปิดให้สื่อสารภายใน Cluster เท่านั้น (Default)
2. **NodePort:** เปิดพอร์ตบนทุกๆ Worker Node (พอร์ต 30000+)
3. **LoadBalancer:** สร้าง Load Balancer จริงๆ บน Cloud (เช่น AWS ALB)

---

## ภาคปฏิบัติ (Hands-on)

### 1. ClusterIP (คุยภายใน)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend   # ชี้ไปที่ Pod ที่มี label นี้
  ports:
  - port: 80       # Port ของ Service
    targetPort: 8080 # Port ของ Container
```

### 2. NodePort (เข้าจากภายนอก)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    nodePort: 30080
```

---

## Ingress (การทำ Routing ชั้นสูง)

แทนที่จะเปิด NodePort สำหรับทุกแอป เราใช้ Ingress 1 ตัวรับ Traffic (พอร์ต 80/443) แล้วกระจายตามชื่อ Domain หรือ Path

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```
*(ต้องเปิด Ingress addon ใน Minikube ก่อน: `minikube addons enable ingress`)*
