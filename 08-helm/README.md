# Module 08: Helm

เป้าหมาย: ใช้ Helm เพื่อรวมแพ็กเกจ (Chart) และติดตั้งแอปพลิเคชันที่ซับซ้อนได้อย่างง่ายดาย

Helm เปรียบเสมือน `apt` หรือ `yum` สำหรับ Kubernetes ที่รวมเอา YAML หลายๆ ตัวเข้าไว้ด้วยกันเป็นชุดโครงสร้างเดียว (Chart)

---

## ⚙️ การตั้งค่าระบบก่อนใช้งาน
ตัวโปรแกรม `helm` ได้ถูกดาวน์โหลดและติดตั้งแบบ Local ไว้เรียบร้อยแล้วที่ `/home/demo/.local/bin/helm`
คุณสามารถรันคำสั่งด้านล่างเพื่อเพิ่ม PATH ให้เรียกคำสั่งสั้นๆ ได้ หรือใช้พาร์ธเต็มได้โดยตรง:
```bash
export PATH=$PATH:/home/demo/.local/bin
```

---

## 🛠️ ขั้นตอนการทำ Lab

### 1. เพิ่ม Repository ศูนย์กลาง (คล้ายการเพิ่ม PPA)
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 2. ค้นหา Package (เช่น redis)
```bash
helm search repo redis
```

### 3. ตรวจสอบการกำหนดค่าคอนฟิก (Values)
เราจะแก้ไขค่าบางส่วนโดยไม่ใช้ค่า Default โดยเขียนลงใน [redis-values.yaml](redis-values.yaml):
- ปิดการใช้รหัสผ่าน (`auth.enabled: false`)
- ปรับโครงสร้างแบบ Standalone (`architecture: standalone`)
- กำหนด Resource Requests & Limits ให้ประหยัดการกินแรม/ซีพียู

### 4. ติดตั้ง Package ด้วย custom values
```bash
helm install my-redis bitnami/redis -f redis-values.yaml
```

### 5. ดูรายการที่ติดตั้งแล้วในระบบ
```bash
helm list
kubectl get all -l app.kubernetes.io/name=redis
```
*จะพบว่าคำสั่งเดียวสร้าง Service, StatefulSet, ConfigMap, Secret ฯลฯ ครบชุด*

### 6. ทดสอบใช้งาน Redis (PING Test)
รัน Pod ชั่วคราวเพื่อส่งคำสั่ง PING ไปตรวจสอบการตอบกลับ:
```bash
kubectl run redis-client --rm --tty -i --restart='Never' \
  --image docker.io/bitnami/redis:7.4.2-debian-12-r4 \
  --env REDISCLI_AUTH="" \
  -- redis-cli -h my-redis-master ping
```
*(คำตอบที่ได้ควรเป็น `PONG`)*

### 7. ถอนการติดตั้ง
```bash
helm uninstall my-redis
```

---

## ⚠️ ข้อควรระวังในระดับ Production (Gotchas)
> [!WARNING]
> อิมเมจบางตัวใน Bitnami Helm Charts บน Repository สาธารณะ อาจใช้อิมเมจเวอร์ชันล่าสุดที่อัปเดตอัตโนมัติ (Rolling Tags) ใน Production เสมอควรระบุเวอร์ชันอิมเมจแบบคงที่ (`tag: "x.y.z"`) ในไฟล์ `values.yaml` เพื่อความเสถียรของระบบ
