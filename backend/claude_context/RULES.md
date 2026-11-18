# HRIS API

## Tech Stack

- Backend: Golang `1.26.xx` + net/http + go-migrate + swaggo
- Database: PostgreSQL
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
    utils/
      audit/        jejak perubahan, dipakai hampir seluruh modul
      paging/       parameter paginasi bersama
    middleware/     auth JWT, RBAC, log, CORS, rate limit
    pkg/            apperror, response, hash, jwtutil, logger, worker crypto
  migrations/       .up.sql / .down.sql bernomor
  docs/             Swagger (swaggo), rencana indeks, deploy, backup-restore
```

### 2. Rules

1. Maksimal **200 baris per file** `.go` non-test. File `_test.go` dikecualikan.
2. Untuk setiap perubahan DDL, buat **migrasi** `.up.sql` dan `.down.sql` bernomor di folder `migrations/`.
3. **Tidak boleh stored procedure** dalam bentuk apa pun. Semua query ditulis as native query di repository. Tidak ada penggunaan ORM.
4. Logika bisnis ada di **service layer**. Handler hanya parsing, validasi bentuk, dan memanggil service. Repository hanya query.
5. Setiap aksi menulis `audit_logs` dalam **transaksi yang sama**.
6. API Response :
   - Single Data

     ```json
     {
       status: 200
       message: "Sukses mendapatkan data ${nama_endpoint}"
       data: {
         "key": "value"
       }
     }
     ```

   - List Data

   ```json
   {
      status: 200
      message: "Sukses mendapatkan data ${nama_endpoint}"
      data: [
        {data1},
        {data2},
        {data3}
      ]
   }
   ```

   - Pagination Data

   ```json
   {
      status:200
      message: "Sukses mendapatkan data ${nama_endpoint}"
      data: [
        {data1},
        {data2},
        {data3}
      ]
      page: 1
      lastPage: 20,
      totalData: 3
   }
   ```
### 3. Test Performance Variable
- Jumlah User: 100 User
- RPS: 80 rps