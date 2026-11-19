# HRIS API

## Tech Stack

- Backend: Golang `1.26.xx` + net/http + go-migrate + swaggo
- Database: PostgreSQL, driver `pgx/v5` + `pgxpool`
- Storage: SeaweedFS

## Backend

### 1. Architecture

```
backend/
  cmd/
    api/            entrypoint server HTTP
  config/
    database.go     koneksi SQL, transaksi, retry deadlock, migrator
    config.go       konfigurasi variabel .env
  internal/
    models/         representasi baris tabel
    feature/        modul per use-case; masing-masing berisi handler, service,
                    repository, DTO, dan routes.go sendiri
      auth/         login lokal, refresh token, logout, aktivasi, reset & ganti password
      oauth/        login Google karyawan (openid email)
      users/        manajemen user, role, permission
      employee/     manajemen data karyawan, jabatan, unit, profil, biodata
      attendance/   manajemen absensi, check-in, check-out, jam kerja
      leave/        manajemen cuti, pengajuan, approval, sisa cuti, kalender cuti
      kpi/          manajemen KPI, indikator, target, penilaian, approval
      payroll/      manajemen gaji, komponen gaji, slip gaji, approval
      dashboard/    beranda, ringkasan data, statistik
      notification/ lonceng pemberitahuan in-app dan gmail beserta trigger-nya
    middleware/     auth JWT, RBAC, log, CORS, rate limit
    pkg/            apperror, response, hash, jwtutil, logger, worker crypto,
                    audit (jejak perubahan, dipakai hampir seluruh modul),
                    paging (parameter paginasi bersama)
  migrations/       .up.sql / .down.sql bernomor
  docs/             Swagger (swaggo), rencana indeks, deploy, backup-restore
```

### 2. Rules

1. Maksimal **200 baris per file** `.go` non-test. File `_test.go` dikecualikan.
2. Untuk setiap perubahan DDL, buat **migrasi** `.up.sql` dan `.down.sql` bernomor di folder `migrations/`.
3. **Tidak boleh stored procedure** dalam bentuk apa pun, termasuk trigger dan function plpgsql. Semua query ditulis as native query di repository. Tidak ada penggunaan ORM.
   - Akses DB lewat `pgx/v5` + `pgxpool` dengan raw SQL. Tidak pakai sqlc atau query builder.
   - Kolom `updated_at` diisi dari query: setiap `UPDATE` wajib menulis `updated_at = now()`.
4. Logika bisnis ada di **service layer**. Handler hanya parsing, validasi bentuk, dan memanggil service. Repository hanya query.
5. Setiap aksi menulis `audit_logs` dalam **transaksi yang sama**.
   - Aksi yang diaudit: create, update, delete, serta login, logout, dan reset/ganti password.
   - Read (GET) tidak diaudit.
   - `record_id` disimpan sebagai string agar muat ID UUID maupun SERIAL.
6. API Response:
   - Single Data

     ```json
     {
       "status": 200,
       "message": "Sukses mendapatkan data ${nama_endpoint}",
       "data": {
         "key": "value"
       }
     }
     ```

   - List Data

     ```json
     {
       "status": 200,
       "message": "Sukses mendapatkan data ${nama_endpoint}",
       "data": [
         {"data1": "..."},
         {"data2": "..."},
         {"data3": "..."}
       ]
     }
     ```

   - Pagination Data. `totalData` = jumlah seluruh baris, bukan jumlah baris di halaman ini.

     ```json
     {
       "status": 200,
       "message": "Sukses mendapatkan data ${nama_endpoint}",
       "data": [
         {"data1": "..."},
         {"data2": "..."},
         {"data3": "..."}
       ],
       "page": 1,
       "perPage": 10,
       "lastPage": 20,
       "totalData": 195
     }
     ```

   - Error. `errors` hanya ada untuk error validasi (key = nama field).

     ```json
     {
       "status": 422,
       "message": "Validasi gagal",
       "errors": {
         "email": "wajib diisi"
       }
     }
     ```

   - Message per aksi:

     | Aksi   | Status | Message                                  |
     |--------|--------|------------------------------------------|
     | GET    | 200    | `Sukses mendapatkan data ${nama_endpoint}` |
     | Create | 201    | `Sukses menambahkan data ${nama_endpoint}` |
     | Update | 200    | `Sukses mengubah data ${nama_endpoint}`    |
     | Delete | 200    | `Sukses menghapus data ${nama_endpoint}`   |

7. Semua kolom waktu pakai `TIMESTAMPTZ`, bukan `TIMESTAMP`.
8. RBAC berbasis permission: tabel `permissions` + `role_permissions`. Middleware cek permission, bukan nama role.
9. Semua waktu di request dan response JSON pakai format **ISO 8601 / RFC 3339 dengan offset**, mis. `2026-11-19T10:00:00+07:00`.
   - Cukup kirim `time.Time` apa adanya; `encoding/json` sudah menghasilkan RFC 3339.
   - Tidak boleh mengubah waktu ke format tampilan (mis. `dd-mm-yyyy`) di backend. Format tampilan adalah tugas frontend.
   - Kolom `DATE` (tanpa jam) dikirim sebagai `yyyy-mm-dd`, mis. `2026-11-19`.

### 3. Test Performance Variable
- Tool: k6
- Jumlah User: 100 User
- RPS: 80 rps
- Target lulus: p95 latency < 300ms, error rate < 1%
