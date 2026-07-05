# devper-api

Firebase Hosting gateway ที่ทำหน้าที่ reverse proxy ไปยังบริการ Cloud Run ภูมิภาค `asia-southeast1`

- Firebase project: `devperpos`
- Hosting site: `devper-api`
- URL: https://api.devper.app (custom domain via Cloudflare DNS; https://devper-api.web.app ยังใช้ได้)

## Routes

| Path | Target Cloud Run service |
|------|--------------------------|
| `/api/um/**` | `devper-um` |
| `/api/pharmacy/**` | `pharmacy-api` |
| `/api/gold/**` | `devper-gold` |
| `/health` | `devper-um` |

Service `devper-um` และ `pharmacy-api` deploy แยกจาก repo ของตัวเอง — repo นี้ deploy เฉพาะไฟล์ static ใน `public/` และ rewrite rules ใน `firebase.json`

## Deploy

จากไดเรกทอรี `devper-api/`:

```bash
firebase deploy --only hosting:devper-api
```

Preview channel ก่อน promote ขึ้น production:

```bash
firebase hosting:channel:deploy preview --only devper-api
```
