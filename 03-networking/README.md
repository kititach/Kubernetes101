# Module 03: Networking & Services

เป้าหมาย: ให้ Pods สามารถสื่อสารกันเองได้ และอนุญาตให้ Traffic จากภายนอกเข้ามายัง Pods ภายใน Cluster ได้ผ่านช่องทางต่าง ๆ

## แนวคิดหลัก

เนื่องจาก Pod ใน Kubernetes เกิดง่ายตายเร็ว (Ephemeral) ทำให้ IP Address ของ Pod เปลี่ยนแปลงได้ตลอดเวลา เราจึงไม่ควรเรียกใช้งาน Pod ผ่าน IP ของมันโดยตรง แต่จะใช้ **Service** มาครอบเป็นหน้าด่าน โดย Service จะมี IP คงที่ (ClusterIP) และทำหน้าที่กระจาย Traffic ไปหา Pods ที่อยู่เบื้องหลังให้โดยอัตโนมัติ ผ่านการจับคู่ **Label Selector**

### ประเภทของ Service
1. **ClusterIP (Default):** เปิดช่องทางสื่อสารเฉพาะภายใน Cluster เท่านั้น (เช่น App คุยกับ Database)
2. **NodePort:** เปิดพอร์ตบนทุก ๆ Worker Node (พอร์ตช่วง 30000-32767) เพื่อให้ภายนอกเข้าถึงผ่าน `NodeIP:NodePort` ได้
3. **LoadBalancer:** สร้าง Load Balancer จริง ๆ บนระบบ Cloud (เช่น AWS ALB, GCP LB) เพื่อให้ได้ Public IP ของ Cloud ผูกเข้าหา Service

---

## ขั้นเตรียมความพร้อม (Prerequisite)

ก่อนจะเริ่มทำระบบเครือข่าย เราต้องรันตัวแอปพลิเคชัน (Pod/Deployment) ขึ้นมาก่อน เพื่อรอรับ Traffic ที่เราจะทดสอบส่งเข้าไป:

```bash
# 1. รัน Nginx Deployment จาก Module 02 (มี Label: app: nginx)
kubectl apply -f ../02-workloads/nginx-deployment.yaml

# 2. ตรวจสอบว่า Pods ของ Nginx รันครบและมีสถานะ Running
kubectl get pods -l app=nginx -o wide
```

---

## ภาคปฏิบัติ (Hands-on)

### 1. ClusterIP (สื่อสารภายใน Cluster)

เราจะสร้าง ClusterIP Service เพื่อเป็นหน้าด่านให้กับ Nginx Pods ของเรา

#### โค้ดใน [clusterip-svc.yaml](file:///home/demo/lab/k8s/03-networking/clusterip-svc.yaml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx      # ชี้ไปยัง Pod ที่มี label "app: nginx"
  ports:
  - port: 80        # Port ของ Service ที่เราจะเรียกใช้
    targetPort: 80  # Port ของ Container Nginx ด้านใน
```

#### วิธีทดสอบ:
```bash
# 1. Apply ไฟล์ Service
kubectl apply -f clusterip-svc.yaml

# 2. ตรวจสอบสถานะ Service และดูว่า Endpoints ชี้ไปที่ IP ของ Pods จริงไหม
kubectl get svc nginx-clusterip
kubectl describe svc nginx-clusterip

# 3. ทดสอบเรียกใช้ (เนื่องจากเป็น ClusterIP จะเรียกจากเครื่องเราตรง ๆ ไม่ได้)
# วิธีที่ 3.1: รัน temporary Pod เข้าไป curl ข้างใน Cluster
kubectl run curl-test --image=curlimages/curl -i --rm --restart=Never -- curl -s http://nginx-clusterip

# วิธีที่ 3.2: ทดสอบทำ Port-Forward มาที่เครื่องตัวเอง
kubectl port-forward svc/nginx-clusterip 8081:80
# จากนั้นเปิดอีก Terminal หรือ Browser ลองเข้า http://localhost:8081 (กด Ctrl+C เพื่อยกเลิก)
```

---

### 2. NodePort (เข้าถึงจากเครื่องภายนอก)

เราจะเปิด NodePort เพื่อให้เครื่องเรา (Host OS) สามารถเข้าถึงเว็บ Nginx ด้านในคลัสเตอร์ได้โดยตรงผ่านพอร์ตเฉพาะเจาะจง

#### โค้ดใน [nodeport-svc.yaml](file:///home/demo/lab/k8s/03-networking/nodeport-svc.yaml):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx      # ชี้ไปยัง Pod ที่มี label "app: nginx"
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080 # พอร์ตที่เปิดบนเครื่อง Node ของมินิคูป (ต้องอยู่ในช่วง 30000-32767)
```

#### วิธีทดสอบ:
```bash
# 1. Apply ไฟล์ Service
kubectl apply -f nodeport-svc.yaml

# 2. ตรวจสอบพอร์ตที่เปิด
kubectl get svc nginx-nodeport

# 3. ทดสอบเข้าถึงเว็บผ่านเบราว์เซอร์หรือ curl
# สำหรับ Minikube: รันคำสั่งนี้เพื่อให้ระบบเปิด URL ของ Service ให้โดยอัตโนมัติ
minikube service nginx-nodeport --url

# หรือดึง IP ของ Minikube มาใช้ร่วมกับพอร์ต 30080:
curl http://$(minikube ip):30080
```

---

### 3. Ingress (การทำ Web Routing ด้วย Domain Name)

แทนที่จะเปิด NodePort สำหรับทุก ๆ แอปพลิเคชัน ซึ่งสิ้นเปลืองพอร์ตและตั้งค่า SSL ลำบาก เราสามารถใช้ **Ingress** เป็นประตูด่านหน้าตัวเดียวที่ทำงานบนพอร์ต HTTP/HTTPS (80/443) และทำ Routing กระจายตามชื่อ Domain หรือ Path ได้

#### โค้ดใน [ingress.yaml](file:///home/demo/lab/k8s/03-networking/ingress.yaml):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: myapp.local      # ชื่อ Domain ที่จะจำลองขึ้นมาใช้งาน
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-clusterip # ชี้เป้าไปหา ClusterIP Service ที่สร้างไว้ในข้อ 1
            port:
              number: 80
```

#### วิธีทดสอบ:
```bash
# 1. เปิดใช้ Addon Ingress ใน Minikube
minikube addons enable ingress

# 2. (ข้อควรระวัง) รอจนกระทั่ง ingress-nginx controller pod ทำงานเสร็จสมบูรณ์
kubectl get pods -n ingress-nginx

# 3. Apply ไฟล์ Ingress
kubectl apply -f ingress.yaml

# 4. ตรวจสอบสถานะและคอยจนกว่าจะมีค่า IP ปรากฏในคอลัมน์ ADDRESS (อาจใช้เวลา 10-30 วินาที)
kubectl get ingress nginx-ingress

# 5. ตั้งค่า DNS จำลองบนเครื่องของคุณ
# ดึง IP ของ Minikube ออกมา:
minikube ip
# ตัวอย่างเช่น ได้ IP เป็น 192.168.49.2
# ให้นำไปเพิ่มบรรทัดนี้ลงในไฟล์ /etc/hosts (Linux/macOS) หรือ C:\Windows\System32\drivers\etc\hosts (Windows)
# <MINIKUBE_IP> myapp.local
# ตัวอย่าง: 192.168.49.2 myapp.local

# 6. ทดสอบเข้าผ่าน Domain
curl http://myapp.local
```

---

## การเก็บกวาด (Cleanup)

เมื่อทำแล็บเสร็จแล้วและต้องการเคลียร์ทรัพยากร ให้ใช้คำสั่งเหล่านี้:

```bash
kubectl delete -f ingress.yaml
kubectl delete -f nodeport-svc.yaml
kubectl delete -f clusterip-svc.yaml
kubectl delete -f ../02-workloads/nginx-deployment.yaml
```
