# STATUS — Kubernetes Learning Lab

> **อ่านไฟล์นี้ก่อนทุกครั้งที่เข้ามาทำงาน**  
> อัปเดตท้ายไฟล์ทุกครั้งที่เรียน/ทำ lab เสร็จ — ไม่ว่าจะสำเร็จหรือพัง

---

## 🟢 สถานะล่าสุด — จบหลักสูตรครบทุก Module

### ความคืบหน้าโดยรวม
```
00-setup          ✅ เสร็จแล้ว
01-basics         ✅ เสร็จแล้ว
02-workloads      ✅ เสร็จแล้ว
03-networking     ✅ เสร็จแล้ว
04-storage        ✅ เสร็จแล้ว
05-config         ✅ เสร็จแล้ว
06-scheduling     ✅ เสร็จแล้ว
07-security       ✅ เสร็จแล้ว
08-helm           ✅ เสร็จแล้ว
```

### Lab Status
| หัวข้อ | Lab | สถานะ | หมายเหตุ |
|--------|-----|--------|---------|
| 00-setup | ติดตั้ง minikube + kubectl | ✅ เสร็จแล้ว | kubectl v1.36.1, minikube v1.38.1, K8s v1.35.1, driver: docker |
| 01-basics | Pod lifecycle, kubectl commands, Namespace | ✅ เสร็จแล้ว | Imperative + Declarative, port-forward, namespace isolation |
| 02-workloads | Deployment, Rolling Update, Rollback, Probes, Resources | ✅ เสร็จแล้ว | Self-healing ทดสอบได้จริง, rollout undo มี warning |
| 03-networking | ClusterIP, NodePort, Ingress | ✅ เสร็จแล้ว | DNS name ภายใน cluster, ingress addon ต้องรอ controller ready |
| 04-storage | PVC, PV, Dynamic Provisioning | ✅ เสร็จแล้ว | data อยู่รอดข้าม Pod restart ทดสอบได้จริง |
| 05-config | ConfigMap, Secret | ✅ เสร็จแล้ว | base64 ≠ encryption — ความปลอดภัยมาจาก RBAC |
| 06-scheduling | Node Selector, Taints & Tolerations | ✅ เสร็จแล้ว | Pod ค้าง Pending จริงเมื่อไม่มี toleration |
| 07-security | RBAC, ServiceAccount, Security Context | ✅ เสร็จแล้ว | nginx:alpine รัน non-root ไม่ได้ ต้องใช้ nginx-unprivileged |
| 08-helm | Helm CLI, bitnami/redis, custom values | ✅ เสร็จแล้ว | 1 คำสั่งสร้าง 9 K8s objects, PING Redis สำเร็จ |

---

## 🗺️ แผนการเรียน (Backlog)

#### Phase 1 — Basics & Setup
- [x] **00-setup:** ติดตั้ง Minikube, Kubectl, และความเข้าใจ Architecture (Control Plane vs Worker Nodes)
- [x] **01-basics:** Pod lifecycle, การใช้คำสั่ง `kubectl` พื้นฐาน (get, describe, logs, exec, port-forward), Namespace

#### Phase 2 — Workloads
- [x] **02-workloads:** Deployment, ReplicaSet, การทำ Rolling Update / Rollback
- [x] **02-workloads:** Resource Requests & Limits
- [x] **02-workloads:** Liveness & Readiness Probes
- [ ] **02-workloads:** StatefulSet (สำหรับ Database) — ทฤษฎีใน README, ยังไม่ได้ทำ hands-on
- [ ] **02-workloads:** DaemonSet (สำหรับ Logging/Monitoring agents) — ทฤษฎีใน README, ยังไม่ได้ทำ hands-on
- [ ] **02-workloads:** Jobs & CronJobs — ทฤษฎีใน README, ยังไม่ได้ทำ hands-on

#### Phase 3 — Networking
- [x] **03-networking:** Services (ClusterIP, NodePort)
- [x] **03-networking:** Ingress Controller (Nginx) & Ingress resources
- [ ] **03-networking:** Network Policies (พื้นฐาน Security) — อยู่ใน README, ยังไม่ได้ทำ hands-on

#### Phase 4 — Storage
- [x] **04-storage:** PersistentVolume (PV) และ PersistentVolumeClaim (PVC)
- [x] **04-storage:** StorageClasses (Dynamic Provisioning)

#### Phase 5 — Configuration & Scheduling
- [x] **05-config:** ConfigMaps และ Secrets (ENV injection)
- [ ] **05-config:** Secret mount เป็น volume — ยังไม่ได้ทำ hands-on
- [x] **06-scheduling:** Node Selector
- [x] **06-scheduling:** Taints และ Tolerations
- [ ] **06-scheduling:** Node Affinity / Anti-Affinity — ทฤษฎีใน README, ยังไม่ได้ทำ hands-on

#### Phase 6 — Advanced & Operations
- [x] **07-security:** RBAC (Role, RoleBinding, ServiceAccount)
- [x] **07-security:** Security Context (Non-root containers)
- [x] **08-helm:** Helm basics, การติดตั้งแอปผ่าน Helm Charts (Redis)

---

## 📋 Key File Locations

| ไฟล์ | หน้าที่ |
|------|--------|
| `CLAUDE.md` | Context สำหรับ AI — โครงสร้างโปรเจกต์, gaps, best practices, gotchas |
| `K8S-GUIDE.md` | คู่มือและ Reference หลักของทุก Concept |
| `STATUS.md` | ไฟล์นี้ — สำหรับอัปเดตความคืบหน้า |
| `00-setup/README.md` | คู่มือการติดตั้งสภาพแวดล้อม |

---

## 📜 ประวัติการแก้ไข

### 2026-05-19 — สร้างโปรเจกต์
- ร่างโครงสร้างหลักสูตรและสร้างโฟลเดอร์สำหรับ Labs ทั้งหมด 9 Modules
- สร้างไฟล์ `STATUS.md` และ `K8S-GUIDE.md`

### 2026-05-19 — เพิ่ม CLAUDE.md
- สร้างไฟล์ `CLAUDE.md` สำหรับ AI context — ครอบคลุมโครงสร้างโปรเจกต์, แผนการเรียน, gaps ระหว่าง K8S-GUIDE กับ module README, และวิธีอัปเดต STATUS.md

### 2026-05-19 — จบ Module 00-setup
- ติดตั้ง kubectl v1.36.1 และ minikube v1.38.1 สำเร็จ
- รัน cluster บน Docker driver, Kubernetes v1.35.1, Node: Ready

### 2026-05-19 — จบ Module 01-basics
- สร้าง Pod แบบ Imperative และ Declarative (pod.yaml)
- ทดสอบ describe, logs, port-forward ครบ
- เพิ่มเนื้อหา Namespace เข้าหลักสูตร
- ไฟล์: 01-basics/pod.yaml

### 2026-05-19 — จบ Module 02-workloads
- สร้าง Deployment 3 replicas, ทดสอบ Self-healing (ลบ Pod แล้วสร้างใหม่อัตโนมัติ)
- ทำ Rolling Update 1.24→1.25 และ Rollback กลับ 1.24 สำเร็จ
- เพิ่ม Resource Requests & Limits และ Liveness/Readiness Probes เข้าหลักสูตร
- ไฟล์: 02-workloads/nginx-deployment.yaml

### 2026-05-19 — จบ Module 03-networking
- ClusterIP: เข้าถึงผ่าน DNS name ภายใน cluster ได้ทันที
- NodePort: เปิด port 30080 เข้าจากภายนอกผ่าน minikube IP
- Ingress: route traffic ด้วย Host header, ใช้ nginx ingress controller
- ไฟล์: 03-networking/clusterip-svc.yaml, nodeport-svc.yaml, ingress.yaml

### 2026-05-19 — จบ Module 04-storage
- PVC `Bound` ทันทีผ่าน Dynamic Provisioning (StorageClass: standard)
- ทดสอบ data persistence: เขียนไฟล์ → ลบ Pod → สร้างใหม่ → ข้อมูลยังอยู่
- ไฟล์: 04-storage/pvc.yaml, storage-pod.yaml

### 2026-05-19 — จบ Module 05-config
- ConfigMap inject เป็น ENV ผ่าน envFrom, Secret inject ผ่าน secretKeyRef
- Secret เก็บเป็น base64 — ไม่ใช่การเข้ารหัส ต้องใช้ RBAC ควบคุมการเข้าถึง
- ไฟล์: 05-config/configmap.yaml, app-pod.yaml

### 2026-05-19 — จบ Module 06-scheduling
- Node Selector: label node `disktype=ssd` → Pod ถูก schedule ถูก node
- Taints & Tolerations: taint node → Pod ไม่มี toleration ค้าง Pending, Pod มี toleration รันได้
- ไฟล์: 06-scheduling/nodeselector-pod.yaml, toleration-pod.yaml

### 2026-05-19 — จบ Module 07-security
- RBAC: ServiceAccount + Role (list pods only) + RoleBinding ทดสอบด้วย `kubectl auth can-i`
- Security Context: runAsUser=1000 — nginx:alpine crash เพราะต้องการ root ต้องใช้ nginx-unprivileged แทน
- ไฟล์: 07-security/rbac.yaml, secure-pod.yaml

### 2026-05-19 — จบ Module 08-helm
- ติดตั้ง Helm v3.21.0 ใน ~/.local/bin (ไม่ต้องใช้ sudo)
- ติดตั้ง Redis ผ่าน bitnami chart ด้วย custom values (standalone, no auth)
- 1 คำสั่ง helm install สร้าง 9 K8s objects ครบ (StatefulSet, Service, ConfigMap ฯลฯ)
- ทดสอบ redis-cli PING จาก Pod ภายใน cluster ได้ PONG
- ไฟล์: 08-helm/redis-values.yaml
