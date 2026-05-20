# Module 06: Scheduling

เป้าหมาย: ควบคุมว่า Pod ตัวไหน ควรไปรันที่ Worker Node เครื่องไหน

## 1. Node Selector (ง่ายสุด)
ผูก Pod ให้วิ่งไปหา Node ที่มี Label ตรงกันเท่านั้น
```bash
# แปะ label ให้ node ก่อน
kubectl label nodes <node-name> disktype=ssd
```
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

## 2. Taints และ Tolerations (กีดกัน/อนุญาต)
- **Taint (พ่นยาไล่):** ทำบน Node เพื่อห้าม Pod ทั่วไปมารัน
  `kubectl taint nodes <node-name> key=value:NoSchedule`
- **Toleration (ฉีดวัคซีน):** ใส่ใน Pod เพื่อให้ทนต่อ Taint ได้
```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

## 3. Node Affinity / Anti-Affinity (ซับซ้อน)
ใช้สำหรับความต้องการเชิงลึก เช่น "อยากให้อยู่ Node นี้ แต่ถ้าไม่ได้ก็ไม่เป็นไร" (PreferredDuringScheduling) หรือ "ต้องอยู่ Node นี้เท่านั้น" (RequiredDuringScheduling)
