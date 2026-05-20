# Module 08: Helm

เป้าหมาย: ใช้ Helm เพื่อรวมแพ็กเกจ (Chart) และติดตั้งแอปพลิเคชันที่ซับซ้อนได้อย่างง่ายดาย

Helm เปรียบเสมือน `apt` หรือ `yum` สำหรับ K8s

## การใช้งานเบื้องต้น

```bash
# 1. ติดตั้ง Helm CLI
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 2. เพิ่ม Repository ศูนย์กลาง (คล้าย PPA)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 3. ค้นหา Package
helm search repo redis

# 4. ติดตั้ง Package
helm install my-redis bitnami/redis

# 5. ดูรายการที่ติดตั้งแล้ว
helm list

# 6. ถอนการติดตั้ง
helm uninstall my-redis
```

## การปรับแต่งค่าคอนฟิก (Values)
เรามักจะไม่ใช้ค่า Default แต่จะสร้างไฟล์ `values.yaml` ของตัวเอง
```yaml
# my-values.yaml
auth:
  enabled: false
architecture: standalone
```
```bash
helm install my-redis bitnami/redis -f my-values.yaml
```
