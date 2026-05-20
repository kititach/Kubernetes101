# CLAUDE.md — Kubernetes Learning Lab

## Project Overview

โปรเจกต์นี้คือ Hands-on Learning Lab สำหรับเรียน Kubernetes ตั้งแต่ระดับเริ่มต้นจนถึง Advanced
เป้าหมายคือสร้างความเข้าใจผ่านการลงมือทำจริงบน Local Cluster (Minikube)

**สถานะ:** จบ 9/9 Modules แล้ว — กำลังเตรียมสอนคนอื่น

## โครงสร้างไฟล์

```
k8s/
├── CLAUDE.md          # ไฟล์นี้ — context สำหรับ AI
├── K8S-GUIDE.md       # Reference หลักของทุก Concept (อ่านเมื่อต้องการทบทวน)
├── STATUS.md          # ติดตามความคืบหน้า — อ่านก่อนทำงานทุกครั้ง
├── 00-setup/          # ติดตั้ง kubectl + minikube + ทดสอบ cluster
├── 01-basics/         # Pod lifecycle, kubectl commands, Namespace
│   └── pod.yaml
├── 02-workloads/      # Deployment, Rolling Update, Probes, Resources
│   └── nginx-deployment.yaml
├── 03-networking/     # Services (ClusterIP/NodePort), Ingress
│   ├── clusterip-svc.yaml
│   ├── nodeport-svc.yaml
│   └── ingress.yaml
├── 04-storage/        # PVC, Dynamic Provisioning
│   ├── pvc.yaml
│   └── storage-pod.yaml
├── 05-config/         # ConfigMap, Secret (ENV injection)
│   ├── configmap.yaml
│   └── app-pod.yaml
├── 06-scheduling/     # Node Selector, Taints & Tolerations
│   ├── nodeselector-pod.yaml
│   └── toleration-pod.yaml
├── 07-security/       # RBAC, ServiceAccount, Security Context
│   ├── rbac.yaml
│   └── secure-pod.yaml
└── 08-helm/           # Helm CLI, Charts, Values customization
    └── redis-values.yaml
```

**ทุก module มี YAML จริงที่ `kubectl apply -f` ได้ทันที**

## เครื่องมือที่ใช้

| เครื่องมือ | เวอร์ชัน | วัตถุประสงค์ |
|-----------|---------|-------------|
| `kubectl` | v1.36.1 | คุยกับ K8s Cluster |
| `minikube` | v1.38.1 | รัน Local Cluster (Docker driver) |
| `helm` | v3.21.0 | ติดตั้งแอปซับซ้อนผ่าน Charts (~/.local/bin) |
| Kubernetes | v1.35.1 | Cluster version |

## แนวทางการทำ Lab

1. **อ่าน STATUS.md ก่อนทุกครั้ง** — เพื่อรู้ว่าอยู่ขั้นไหน
2. เริ่มจาก `<module>/README.md` เพื่อดู hands-on steps
3. ใช้ `K8S-GUIDE.md` เป็น Reference เมื่อต้องการทบทวน concept
4. **อัปเดต STATUS.md** ทุกครั้งที่เสร็จ lab — ไม่ว่าจะสำเร็จหรือล้มเหลว
5. ไม่จำเป็นต้อง cleanup ระหว่าง module — ทำเมื่อชื่อ resource ชนกันเท่านั้น

## แผนการเรียน (9 Modules / 6 Phase)

| Module | Phase | หัวข้อหลัก | สถานะ |
|--------|-------|-----------|-------|
| 00-setup | Phase 1 | ติดตั้ง minikube, kubectl, ทดสอบ cluster | ✅ |
| 01-basics | Phase 1 | Pod lifecycle, kubectl commands, Namespace | ✅ |
| 02-workloads | Phase 2 | Deployment, Rolling Update, Probes, Resources | ✅ |
| 03-networking | Phase 3 | Services, Ingress | ✅ |
| 04-storage | Phase 4 | PV, PVC, StorageClass | ✅ |
| 05-config | Phase 5 | ConfigMap, Secret | ✅ |
| 06-scheduling | Phase 5 | Node Selector, Taints/Tolerations | ✅ |
| 07-security | Phase 6 | RBAC, ServiceAccount, Security Context | ✅ |
| 08-helm | Phase 6 | Helm Charts, Values | ✅ |

## Gaps — หัวข้อที่อยู่ใน Module README แต่ไม่อยู่ใน K8S-GUIDE

| หัวข้อ | ดูที่ |
|--------|------|
| Namespace | `01-basics/README.md` |
| Resource Requests & Limits | `02-workloads/README.md` |
| Liveness & Readiness Probes | `02-workloads/README.md` |
| Network Policies | `03-networking/README.md` |
| ServiceAccount | `07-security/README.md` |
| Security Context | `07-security/README.md` |
| Helm | `08-helm/README.md` |

## Hands-on ยังไม่ได้ทำ (ทฤษฎีอยู่ใน README แล้ว)

- StatefulSet, DaemonSet, Job/CronJob → `02-workloads/README.md`
- Network Policies → `03-networking/README.md`
- Secret mount เป็น volume → `05-config/README.md`
- Node Affinity / Anti-Affinity → `06-scheduling/README.md`

## Gotchas จากการลงมือทำจริง

สิ่งเหล่านี้ไม่มีใน README เดิม — ค้นพบระหว่าง lab:

- **Rolling Update + rollback:** `kubectl rollout undo` กับ `kubectl apply` เวอร์ชันเก่าให้ผลต่างกัน ควรใช้ apply
- **Ingress:** ต้องรอ ingress-nginx controller pod พร้อมก่อน apply Ingress หรือจะเจอ webhook error
- **Secret:** base64 ≠ encryption — ใครมีสิทธิ์ `kubectl get secret` ถอดได้ทันที ความปลอดภัยมาจาก RBAC
- **Security Context + nginx:alpine:** รัน `runAsUser: 1000` ไม่ได้ เพราะ nginx ต้องการ root bind port 80 และสร้าง cache dir — ใช้ `nginxinc/nginx-unprivileged` แทน
- **Helm latest tag warning:** bitnami chart ใช้ rolling tag บาง image — ควรเตือนผู้เรียนเรื่อง production risk

## Best Practices ที่ต้องจำ

- ห้ามใช้ tag `latest` ใน Production Image
- กำหนด Resource Requests & Limits เสมอ
- กำหนด Liveness/Readiness Probes ทุก Deployment
- ใช้ `kubectl apply -f` (Declarative) แทน `kubectl run` (Imperative)
- เก็บ YAML ทุกตัวใน Git (GitOps)
- ไม่เก็บข้อมูลสำคัญใน Pod โดยตรง — ใช้ PVC หรือ Secret แทน
- ใช้ `kubectl auth can-i` เพื่อ verify RBAC ก่อน deploy จริง
- Image ที่ต้องการ non-root ต้องออกแบบมาสำหรับนั้นโดยตรง (เช่น nginx-unprivileged)
