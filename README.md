# cart-api

API keranjang belanja sederhana (Express) untuk demo Continuous Integration dengan Jenkins.
Special Topic in Software Engineering, Week 8, Kelompok 4.

## Endpoint

| Method | Path          | Keterangan                                   |
|--------|---------------|----------------------------------------------|
| GET    | `/health`     | `{ "status": "ok" }`                         |
| POST   | `/cart/total` | Menghitung subtotal, diskon voucher, total   |

Voucher: `HEMAT10` (10%), `DISKON25` (25%), `MAHASISWA` (15%). Harga dalam Rupiah bilangan bulat.

```bash
curl -X POST http://38.9.47.13:3000/cart/total \
  -H "Content-Type: application/json" \
  -d '{"items":[{"nama":"Buku","harga":50000,"qty":2}],"voucher":"HEMAT10"}'
# {"subtotal":100000,"diskon":10000,"total":90000}
```

## Menjalankan secara lokal

```bash
npm ci
npm test        # unit test
npm start       # http://localhost:3000
```

## Pipeline Jenkins

`Jenkinsfile` menjalankan tiga stage di VPS:

1. **Test**: `npm ci` lalu `npm run test:ci` di container `node:22-alpine`; hasil test dilaporkan ke Jenkins (`reports/junit.xml`).
2. **Build Image**: `docker compose build`.
3. **Deploy**: `docker compose up -d --wait`; aplikasi aktif di `http://38.9.47.13:3000`.

Jika test gagal, stage berikutnya tidak dijalankan sehingga versi lama tetap berjalan.
Jenkins memeriksa repo ini setiap ~2 menit (`pollSCM`), jadi setiap push otomatis memicu pipeline.
