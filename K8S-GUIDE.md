# Kubernetes (K8s) Learning Lab — คู่มือฉบับสมบูรณ์

คู่มือนี้เป็น Reference หลักสำหรับผู้เริ่มต้นจนถึงระดับมืออาชีพที่ใช้งาน Kubernetes (K8s)
เป้าหมายคือเพื่อให้เข้าใจ "ทำไม" และ "อย่างไร" ควบคู่กันไป

---

## สถาปัตยกรรมของ Kubernetes (Architecture)

Kubernetes ทำงานเป็น Cluster ซึ่งประกอบด้วย 2 ส่วนหลัก:

1. **Control Plane (Master Node):** สมองของคลัสเตอร์
   - **kube-apiserver:** ช่องทางเดียวในการคุยกับคลัสเตอร์ (ผ่าน API/kubectl)
   - **etcd:** ฐานข้อมูลแบบ Key-Value เก็บสถานะทุกอย่างของคลัสเตอร์
   - **kube-scheduler:** เลือก Node ที่เหมาะสมที่สุดสำหรับรัน Pod
   - **kube-controller-manager:** ควบคุมสถานะให้เป็นไปตามที่กำหนด (Desired State)

2. **Worker Nodes:** เครื่องที่ใช้รันแอปพลิเคชันจริง
   - **kubelet:** เอเจนต์ที่คอยรับคำสั่งจาก Master เพื่อจัดการ Pod บน Node นั้น
   - **kube-proxy:** ดูแลเรื่อง Network และการทำ Load Balancing ภายใน
   - **Container Runtime:** ตัวรันคอนเทนเนอร์ (เช่น containerd, CRI-O หรือ Docker)

---

## 0. Namespace

Namespace คือการแบ่ง Cluster ออกเป็นพื้นที่ย่อย ทุก resource ที่สร้างโดยไม่ระบุจะอยู่ใน `default` namespace

```bash
kubectl get namespaces          # ดู namespace ทั้งหมด
kubectl get pods -n kube-system # ดู resource ใน namespace ที่ระบุ
kubectl get pods -A             # ดูทุก namespace
kubectl create namespace dev    # สร้าง namespace ใหม่
```

> 💡 ใน Production ควรแยก namespace ตาม team หรือ environment (dev, staging, prod)

---

## 1. Core Concepts (Object พื้นฐาน)

### Pod
- เป็นหน่วยเล็กที่สุดใน K8s ที่เราสามารถสร้างได้
- 1 Pod มักจะมี 1 Container (แต่มีหลายตัวได้ เรียกว่า sidecar)
- Pod เกิดและตายได้ตลอดเวลา (Ephemeral) ห้ามเก็บข้อมูลสำคัญไว้ใน Pod โดยตรง
- เมื่อตาย K8s จะสร้างขึ้นมาใหม่ และ **IP ของ Pod จะเปลี่ยนเสมอ**

### Deployment & ReplicaSet
- **ReplicaSet:** รับประกันว่าจะมี Pod ตามจำนวนที่กำหนด (Replicas) รันอยู่เสมอ
- **Deployment:** ตัวครอบ ReplicaSet อีกที ใช้จัดการเรื่องการอัปเดตเวอร์ชัน (Rolling Update / Rollback) แบบไม่มี Downtime
> 💡 *Best Practice:* ห้ามสร้าง Pod ตรงๆ ให้สร้างผ่าน Deployment เสมอ

### Service
- เป็นตัวทำ Load Balance ให้กับ Pods โดยมี "IP คงที่" เสมอ
- Service จะจับคู่กับ Pod ผ่าน **Labels และ Selectors**
- **ประเภทของ Service:**
  - `ClusterIP` (Default) - เข้าถึงได้เฉพาะภายใน Cluster
  - `NodePort` - เปิดพอร์ตบนทุก Node (พอร์ต 30000-32767) ให้ภายนอกเข้าถึงได้
  - `LoadBalancer` - ขอ Public IP จาก Cloud Provider (AWS, GCP) มาผูกให้

### Ingress
- ตัวจัดการ Routing ภายนอกเข้าสู่ Service ภายใน
- มักจะทำหน้าที่เป็น Reverse Proxy (เช่น Nginx), ทำ SSL/TLS Termination, และ Path-based routing

---

## 2. Kubectl Commands (คำสั่งที่ต้องรู้)

`kubectl` เป็นเครื่องมือหลักที่คุณจะใช้คุยกับ Cluster

### การสร้างและอัปเดต
```bash
# สร้าง object จากไฟล์ yaml
kubectl apply -f deployment.yaml

# ลบ object
kubectl delete -f deployment.yaml
kubectl delete pod my-pod
```

### การตรวจสอบสถานะ (Get / Describe)
```bash
# ดูรายการทั้งหมด (Pods, Services, Deployments, ฯลฯ)
kubectl get pods
kubectl get svc
kubectl get all

# ดูแบบละเอียด (เพิ่ม -o wide)
kubectl get pods -o wide

# ดูข้อมูลเชิงลึกและ Events (มีประโยชน์มากเวลา Debug)
kubectl describe pod <pod-name>
```

### การ Debugging
```bash
# ดู Logs ของ Pod
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # follow logs

# เข้าไปใน Pod (คล้าย docker exec)
kubectl exec -it <pod-name> -- sh

# Forward พอร์ตจาก Pod มาที่เครื่องเรา (เพื่อทดสอบชั่วคราว)
kubectl port-forward pod/<pod-name> 8080:80
```

---

## 2.5 Resource Requests & Limits

กำหนดขอบเขต CPU/Memory ของ Container เสมอ มิเช่นนั้น Pod อาจกิน resource ของทั้ง Node

- **Requests:** ปริมาณ minimum ที่ K8s จองให้ (Scheduler ใช้ตัดสินใจเลือก Node)
- **Limits:** ปริมาณสูงสุดที่ยอมให้ใช้ (เกินแล้วถูก throttle หรือ OOMKilled)

```yaml
resources:
  requests:
    cpu: "100m"      # 100 millicores = 0.1 core
    memory: "64Mi"
  limits:
    cpu: "200m"
    memory: "128Mi"
```

## 2.6 Liveness & Readiness Probes

- **Liveness:** Pod ยังทำงานปกติไหม? ถ้า fail → K8s Restart Container
- **Readiness:** Pod พร้อมรับ Traffic ไหม? ถ้า fail → K8s หยุดส่ง Request มา (ไม่ Restart)

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 5
```

---

## 3. Workloads รูปแบบอื่นๆ

นอกจาก Deployment แล้ว K8s ยังมี Workload แบบอื่นสำหรับงานเฉพาะทาง:

- **StatefulSet:** สำหรับแอปที่ต้องเก็บ State (เช่น Database) รับประกันลำดับการสร้าง และมี Network ID / Storage ที่แน่นอน
- **DaemonSet:** รับประกันว่าจะมี 1 Pod รันอยู่บน "ทุกๆ Node" เสมอ (เหมาะกับ Log Collector เช่น Fluentd หรือ Monitoring agent)
- **Job:** รัน Task แบบครั้งเดียวจบ (เช่น Script ย้ายข้อมูล)
- **CronJob:** เหมือน Job แต่รันตามเวลาที่กำหนดด้วย Cron Expression

---

## 4. Storage (การเก็บข้อมูล)

เนื่องจาก Pod ตายแล้วข้อมูลหาย เราจึงต้องมี Storage Management

- **PersistentVolume (PV):** ก้อนดิสก์จริง (อาจจะเป็น NFS, AWS EBS, หรือโฟลเดอร์บนเครื่อง) ที่แอดมินเตรียมไว้
- **PersistentVolumeClaim (PVC):** ใบขอเบิกพื้นที่จาก PV (Pod จะผูกกับ PVC ไม่ใช่ PV โดยตรง)
- **StorageClass (SC):** ระบบที่ช่วยสร้าง PV ให้อัตโนมัติ (Dynamic Provisioning) ตามที่ PVC ร้องขอ

---

## 5. Configuration & Security

- **ConfigMap:** เก็บการตั้งค่าทั่วไป (Environment Variables, ไฟล์ Config) แบบ Plain text
- **Secret:** เก็บข้อมูลความลับ (Passwords, API Keys, TLS certs) (ใน K8s ถูกเข้ารหัส base64 ควบคุมการเข้าถึงด้วย RBAC)
- **RBAC (Role-Based Access Control):** การจัดการสิทธิ์
  - `Role` - กำหนดสิทธิ์ว่าทำอะไรได้บ้าง (ในระดับ Namespace)
  - `RoleBinding` - ผูก Role เข้ากับ User หรือ ServiceAccount

---

## 6. Advanced Scheduling

เราสามารถควบคุมว่า Pod ควรไปรันที่ Node ไหนได้ผ่าน:

- **Node Selector:** ผูก Label ง่ายๆ (เช่น `disktype: ssd`)
- **Node Affinity / Anti-Affinity:** กฎที่ซับซ้อนขึ้น (เช่น "ห้ามรัน Pod 2 ตัวนี้ใน Node เดียวกัน" หรือ "พยายามรันบน Node นี้ถ้าเป็นไปได้")
- **Taints & Tolerations:**
  - `Taint` อยู่บน Node (เหมือนยาไล่แมลง) ปฏิเสธไม่ให้ Pod ทั่วไปมารัน
  - `Toleration` อยู่บน Pod (เหมือนวัคซีน) อนุญาตให้ Pod นี้ทนต่อ Taint และไปรันบน Node นั้นได้

---

## ⚠️ Best Practices & Gotchas

1. **ห้ามใช้ tag `latest` สำหรับ Image** ใน Production เพราะจะทำให้ Rollback ยาก
2. **กำหนด Resource Requests & Limits เสมอ** เพื่อไม่ให้ Pod ตัวใดตัวหนึ่งแย่ง CPU/Memory ของทั้ง Node
3. **กำหนด Liveness/Readiness Probes**
   - *Liveness:* Pod รันปกติไหม? (ถ้าพัง K8s จะ Restart ทิ้ง)
   - *Readiness:* Pod พร้อมรับ Traffic ไหม? (ถ้ายัง K8s จะยังไม่ส่ง Request มาให้)
4. **Declarative > Imperative:** ให้ใช้ `kubectl apply -f <file>` (Declarative) เสมอ แทนการใช้ `kubectl run` หรือ `kubectl expose` พร่ำเพรื่อ เพื่อให้สามารถตรวจสอบด้วย Git (GitOps) ได้

---

## 7. Helm

Helm คือ Package Manager สำหรับ K8s — เปรียบเสมือน `apt` หรือ `brew`

- **Chart:** แพ็กเกจที่รวม K8s YAML หลายไฟล์ไว้ด้วยกัน
- **Release:** instance ของ Chart ที่ติดตั้งแล้วใน Cluster
- **Values:** ค่า config ที่ปรับเองได้โดยไม่ต้องแก้ Chart

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-redis bitnami/redis -f my-values.yaml
helm list                    # ดู release ทั้งหมด
helm uninstall my-redis      # ถอนการติดตั้งและลบ resource ทั้งหมด
```

> 💡 1 คำสั่ง `helm install` สร้าง K8s objects ได้หลายสิบตัวพร้อมกัน — ดีกว่า apply ทีละไฟล์มาก

---

*ดูตัวอย่างโค้ดและวิธีทำแบบ Hands-on ได้ในโฟลเดอร์ `00-setup` ถึง `08-helm`*

# เพิ่มเติม วิธีเปิดใช้งาน Kubectl Autocomplete (คำสั่ง tab ช่วย)

```bash
sudo apt-get install bash-completion
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
---

## สำหรับ Zsh (เผื่อคุณเปลี่ยนไปใช้ Zsh)

```bash
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
source ~/.zshrc
---

## ทริกเพิ่มเติม (แถมให้เพื่อความสะดวก)
### พิมพ์ kubectl บ่อยๆ อาจจะเมื่อยมือ คนส่วนใหญ่จะนิยมตั้งชื่อย่อ (Alias) ให้เหลือแค่ตัว k ตัวเดียวครับ สามารถตั้งค่าร่วมกับ Autocomplete ได้ง่ายๆ โดยเพิ่มคำสั่งนี้ลงไปใน ~/.bashrc ต่อได้เลย:

```bash
alias k=kubectl
complete -o default -F __start_kubectl k
---
