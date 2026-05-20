# Module 00: Setup & Installation

เป้าหมายของโมดูลนี้คือการเตรียมเครื่องมือให้พร้อมสำหรับการทำแล็บ Kubernetes

## สิ่งที่ต้องติดตั้ง

1. **kubectl**
   เครื่องมือ Command Line สำหรับคุยกับ K8s Cluster
   - *วิธีติดตั้ง (Linux):*
     ```bash
     curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
     sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
     ```

2. **Minikube**
   เครื่องมือสร้าง Local K8s Cluster สำหรับการเรียนรู้ (แนะนำสำหรับผู้เริ่มต้น)
   - *วิธีติดตั้ง (Linux):*
     ```bash
     curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
     sudo install minikube-linux-amd64 /usr/local/bin/minikube
     ```

## การเริ่มใช้งาน (Start Cluster)

```bash
# เริ่มต้น cluster (ต้องรัน docker หรือ virtualbox ไว้ก่อน)
minikube start

# ตรวจสอบสถานะ
minikube status
kubectl get nodes
```

## ทดสอบว่าพร้อมทำงาน

```bash
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube
```

*(ลบเมื่อทดสอบเสร็จ)*
`kubectl delete service hello-minikube`
`kubectl delete deployment hello-minikube`
