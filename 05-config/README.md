# Module 05: Configuration

เป้าหมาย: การส่งค่า Config และ Password (Secrets) เข้าไปใน Pod โดยไม่ต้องแก้โค้ดหรือ Hardcode ใน Image

## ConfigMap (สำหรับค่าทั่วไป)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
```

## Secret (สำหรับความลับ)
```bash
# สร้าง Secret จาก command line (ง่ายสุด)
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=P@ssw0rd123
```

## การนำไปใช้ใน Pod

เราสามารถ Inject แบบเป็น Environment Variable หรือ Mount เป็นไฟล์ก็ได้

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    envFrom:
    - configMapRef:
        name: app-config
```
