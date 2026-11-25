# Mostrans Price Tools — Phase 2 URL Fix

## Masalah
Input URL `https://apidev.mostrans.co.id/run-service-inactive` sebelumnya ditolak oleh validator task pane karena hanya mengizinkan URL origin tanpa path. Selain itu versi lama selalu menambahkan prefix `/run-service`, yang berbeda dari public endpoint DEV `/run-service-inactive`.

## Fix
- `web/taskpane.html`: menerima hostname HTTPS dengan path `/run-service-inactive` atau `/run-service` (trailing slash optional); URL hanya hostname akan menggunakan default `/run-service-inactive`.
- Validasi HTTPS, tanpa username/password, query string, atau URL fragment tetap dipertahankan.
- Request API disusun dari base path yang diinput, tanpa double prefix.
- Pengecekan Quick Tunnel untuk Execute memakai hostname URL, bukan `endsWith` atas base URL yang kini mengandung path.
- `manifest.xml`: ID add-in dipertahankan; versi dinaikkan ke 1.0.2.0 dan URL task pane di-cache-bust agar lebih mudah menghindari cache Office.
- Tidak perlu mengubah `app.py` atau `worker.py` jika Nginx sudah meneruskan semua endpoint `/run-service-inactive/...` ke Flask sesuai routing yang terpasang.

## Instalasi
1. Backup lalu replace `web/taskpane.html` di repository GitHub Pages (`mostrans-excel-poc`).
2. Commit/push; uji HTML terbaru di `https://meldanophia.github.io/mostrans-excel-poc/web/taskpane.html?v=phase2-api-base-fix-2`.
3. Reinstall/update manifest bila Office masih cache versi lama. Tidak wajib bila taskpane reload benar.
4. Di field **Flask HTTPS URL**, isi `https://apidev.mostrans.co.id/run-service-inactive`.
5. Tes `GET https://apidev.mostrans.co.id/run-service-inactive/health`, kemudian CORS OPTIONS/POST untuk `/run-service-inactive/api/v1/transporter-prices/preview`.
6. Pertahankan `ENABLE_PRICE_UPDATE=false` selama tes Preview.

## Penting: Pastikan reverse proxy dan Flask sejalan
Keberhasilan `/run-service-inactive/health` saja **belum membuktikan** endpoint `/run-service-inactive/api/v1/transporter-prices/preview` tersedia.
- Bila Flask secara internal menggunakan prefix `/run-service`, Nginx harus me-rewrite `/run-service-inactive/...` ke `/run-service/...`, atau backend harus menerima `/run-service-inactive`.
- Bila Flask secara internal memakai `/run-service-inactive`, Nginx harus meneruskan path apa adanya.
- Periksa di Network browser: `OPTIONS`, lalu `POST` endpoint preview. OPTIONS 204 dengan CORS benar; POST 200 JSON.
