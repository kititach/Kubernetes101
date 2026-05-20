# Module 07: Security

เป้าหมาย: การจำกัดสิทธิ์ในระดับ K8s (RBAC) และระดับ Container (Security Context)

## RBAC (Role-Based Access Control)
ควบคุมว่าใคร (User / ServiceAccount) ทำอะไร (Verb) กับ Object ไหน (Resource) ได้บ้าง

### 1. ServiceAccount (บัญชีสำหรับ Pod)
```bash
kubectl create serviceaccount my-app-sa
```

### 2. Role (ประกาศสิทธิ์)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" = core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### 3. RoleBinding (ผูก Role เข้ากับ Account)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-app-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Security Context
กำหนดให้ Pod รันเป็น User ธรรมดา (ไม่ใช่ root) และห้ามแก้ไขระบบไฟล์หลัก

```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: alpine
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```
