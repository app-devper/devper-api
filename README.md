# devper-api

Firebase Hosting gateway ที่ทำหน้าที่ reverse proxy ไปยังบริการ Cloud Run ภูมิภาค `asia-southeast1`

- Firebase project: `devperpos`
- Hosting site: `devper-api`
- URL: https://api.devper.app (custom domain via Cloudflare DNS; https://devper-api.web.app ยังใช้ได้)

## Routes

| Path | Target Cloud Run service |
|------|--------------------------|
| `/api/um/**` | `devper-um` |
| `/api/pos/**` | `pos-dev-api` |
| `/api/pharmacy/**` | `pharmacy-api` |
| `/api/gold/**` | `devper-gold` |
| `/api/alert/**` | `alert-api` |
| `/health` | `devper-um` |

Service เหล่านี้ deploy แยกจาก repo ของตัวเอง — repo นี้ deploy เฉพาะไฟล์ static ใน `public/` และ rewrite rules ใน `firebase.json`

## Deploy

จากไดเรกทอรี `devper-api/`:

```bash
firebase deploy --only hosting:devper-api
```

Preview channel ก่อน promote ขึ้น production:

```bash
firebase hosting:channel:deploy preview --only devper-api
```

## POS

POS และ SM ตั้ง `apiUrl` และ `hostApp` ใน `config/app.json` ไว้ที่ `https://api.devper.app` ทั้งคู่ การ pin `hostApp` คือสิ่งที่กันไม่ให้ system host ที่ UM ส่งกลับมา bypass gateway

`pos-dev-api` deploy จาก repo `pos-api` เอง ผ่าน Cloud Build trigger ที่ยิงเมื่อ push เข้า `main` — **build config ของ trigger นั้นอยู่ใน GCP ภูมิภาค `global` ไม่ได้อยู่ใน repo ไหน**

### Health check

`/health` ที่ตารางข้างบน map ไปที่ `devper-um` ดังนั้นการเช็คสุขภาพของ POS ผ่าน gateway ต้องใช้ path ของตัวเอง:

```bash
curl https://api.devper.app/api/pos/health
```

### CORS

`pos-dev-api` อนุญาต origin ตามที่ตั้งไว้บน service เอง ไม่ใช่ที่ gateway:

```
CORS_ALLOWED_ORIGINS=https://devper.web.app,https://devperpos.web.app,https://devper-pos.web.app
```

เพิ่ม front-end host ใหม่เมื่อไร ต้องอัปเดตค่านี้บน Cloud Run service ด้วย ไม่งั้นเบราว์เซอร์จะบล็อก:

```bash
gcloud run services update pos-dev-api \
  --project=devperpos --region=asia-southeast1 \
  --update-env-vars='^##^CORS_ALLOWED_ORIGINS=https://a,https://b'
```

(`^##^` เปลี่ยน separator เพราะค่ามี comma ถ้าไม่ใส่ gcloud จะตัดเป็นหลาย env var)
