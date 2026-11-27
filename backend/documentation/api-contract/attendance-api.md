# Attendance API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Kolom `DATE` dikirim sebagai `yyyy-mm-dd`. Waktu pakai format ISO 8601 dengan offset (RULES #9).
- `id` attendance bertipe string UUID. `employeeId` bertipe string UUID, `masterShiftId` bertipe number (`SERIAL`).
- Request hanya mengirim id relasi (`employeeId`, `masterShiftId`). Response mengirim relasi sebagai object `{ id, fullName }` / `{ id, name }` hasil JOIN.
- Semua aksi create, update, dan delete ditulis ke `audit_logs` dalam transaksi yang sama (RULES #5).

### Status
`status` disimpan di kolom `attendance.status`. Nilainya dihitung service saat create dan update, tidak dikirim di request. Urutan pengecekan:
1. `leave`: `checkIn` null dan ada `leave_requests` berstatus `approved` yang mencakup `date` untuk employee tersebut.
2. `absent`: `checkIn` null.
3. `late`: `checkIn` lebih dari `date` + `startTime` shift (dibandingkan pada offset yang sama dengan `checkIn`).
4. `present`: selain di atas.

### Error Umum
Berlaku untuk semua endpoint, tidak diulang per endpoint.
```json
{
  "status": 401,
  "message": "Token tidak valid atau sudah kedaluwarsa"
}
```
```json
{
  "status": 403,
  "message": "Tidak punya akses"
}
```
```json
{
  "status": 500,
  "message": "Terjadi kesalahan pada server"
}
```

### Object
Attendance:
```json
{
  "id": "uuid",
  "employee": {
    "id": "uuid",
    "fullName": "string"
  },
  "masterShift": {
    "id": 1,
    "name": "string"
  },
  "date": "2026-11-19",
  "checkIn": "2026-11-19T08:00:00+07:00 | null",
  "checkOut": "2026-11-19T17:00:00+07:00 | null",
  "status": "present | late | absent | leave",
  "createdAt": "string",
  "updatedAt": "string"
}
```

---
## Attendance

### Create Attendance: `POST /api/attendances`
Permission: `attendance.create`

Request Body:
```json
{
  "employeeId": "uuid",
  "masterShiftId": 1,
  "date": "2026-11-19",
  "checkIn": "2026-11-19T08:00:00+07:00 | null",
  "checkOut": "2026-11-19T17:00:00+07:00 | null"
}
```
- `employeeId`: wajib, harus ada di tabel `employees`.
- `masterShiftId`: wajib, harus ada di tabel `master_shift`.
- `date`: wajib, format `yyyy-mm-dd`, unik untuk kombinasi `employeeId` + `date`.
- `checkIn`: opsional, format ISO 8601 dengan offset. Tanggal `checkIn` harus sama dengan `date`.
- `checkOut`: opsional, format ISO 8601 dengan offset.
  - Wajib null jika `checkIn` null.
  - Harus lebih dari `checkIn`.
  - Tanggal `checkOut` harus sama dengan `date`. Untuk shift malam (`endTime` < `startTime`), boleh `date` + 1 hari.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data attendance",
  "data": { "...": "Attendance" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "employeeId": "wajib diisi | tidak ditemukan",
    "masterShiftId": "wajib diisi | tidak ditemukan",
    "date": "wajib diisi | format harus yyyy-mm-dd | sudah dipakai",
    "checkIn": "format harus ISO 8601 | tanggal harus sama dengan date",
    "checkOut": "format harus ISO 8601 | wajib kosong jika checkIn kosong | harus lebih dari checkIn | tanggal tidak sesuai shift"
  }
}
```

### List Attendances: `GET /api/attendances`
Permission: `attendance.read`

Query Params:
- `page`: opsional, default `1`.
- `perPage`: opsional, default `10`, maks `100`.
- `employeeId`: opsional.
- `dateFrom`: opsional, format `yyyy-mm-dd`, inklusif.
- `dateTo`: opsional, format `yyyy-mm-dd`, inklusif, tidak boleh kurang dari `dateFrom`. Satu hari: isi `dateFrom` = `dateTo`.
- `masterShiftId`: opsional.
- `status`: opsional, salah satu dari `present`, `late`, `absent`, `leave`.

Diurutkan berdasarkan `date` turun, lalu `employee.fullName` naik.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data attendance",
  "data": [{ "...": "Attendance" }],
  "page": 1,
  "perPage": 10,
  "lastPage": 20,
  "totalData": 195
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "dateFrom": "format harus yyyy-mm-dd",
    "dateTo": "format harus yyyy-mm-dd | tidak boleh kurang dari dateFrom",
    "status": "harus salah satu dari present, late, absent, leave"
  }
}
```

### Get Attendance by ID: `GET /api/attendances/{id}`
Permission: `attendance.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data attendance",
  "data": { "...": "Attendance" }
}
```
```json
{
  "status": 404,
  "message": "Data attendance tidak ditemukan"
}
```

### Update Attendance: `PUT /api/attendances/{id}`
Permission: `attendance.update`

Request Body dan aturan validasi sama dengan Create Attendance.
- PUT mengganti seluruh data: field opsional yang tidak dikirim atau bernilai `null` dikosongkan.
- Unik `employeeId` + `date` dicek terhadap data lain, tidak termasuk dirinya sendiri.
- `status` dihitung ulang.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data attendance",
  "data": { "...": "Attendance" }
}
```
```json
{
  "status": 404,
  "message": "Data attendance tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "employeeId": "wajib diisi | tidak ditemukan",
    "masterShiftId": "wajib diisi | tidak ditemukan",
    "date": "wajib diisi | format harus yyyy-mm-dd | sudah dipakai",
    "checkIn": "format harus ISO 8601 | tanggal harus sama dengan date",
    "checkOut": "format harus ISO 8601 | wajib kosong jika checkIn kosong | harus lebih dari checkIn | tanggal tidak sesuai shift"
  }
}
```

### Delete Attendance: `DELETE /api/attendances/{id}`
Permission: `attendance.delete`

Response:
```json
{
  "status": 200,
  "message": "Sukses menghapus data attendance"
}
```
```json
{
  "status": 404,
  "message": "Data attendance tidak ditemukan"
}
```

---
## Check-in / Check-out
Dipakai karyawan untuk absen sendiri.
- `employeeId` diambil dari `users.employee_id` milik user di JWT, tidak dikirim di request. User tanpa `employee_id` ditolak 403.
- `checkIn` / `checkOut` diisi dari waktu server (`now()`) di zona waktu aplikasi, tidak dikirim di request.
- `date` = tanggal waktu server. Shift malam (`endTime` < `startTime`) yang check-in sebelum `endTime` dicatat ke tanggal sebelumnya.

### My Attendances: `GET /api/attendances/me`
Permission: `attendance.checkInOut`

Riwayat attendance milik employee sendiri. Dipakai frontend untuk menentukan tombol check-in atau check-out (cek attendance hari ini).

Query Params:
- `page`: opsional, default `1`.
- `perPage`: opsional, default `10`, maks `100`.
- `dateFrom`: opsional, format `yyyy-mm-dd`, inklusif.
- `dateTo`: opsional, format `yyyy-mm-dd`, inklusif, tidak boleh kurang dari `dateFrom`.
- `status`: opsional, salah satu dari `present`, `late`, `absent`, `leave`.

Diurutkan berdasarkan `date` turun.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data attendance",
  "data": [{ "...": "Attendance" }],
  "page": 1,
  "perPage": 10,
  "lastPage": 20,
  "totalData": 195
}
```
```json
{
  "status": 403,
  "message": "Akun tidak terhubung ke data employee"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "dateFrom": "format harus yyyy-mm-dd",
    "dateTo": "format harus yyyy-mm-dd | tidak boleh kurang dari dateFrom",
    "status": "harus salah satu dari present, late, absent, leave"
  }
}
```

### Check-in: `POST /api/attendances/check-in`
Permission: `attendance.checkInOut`

Request Body:
```json
{
  "masterShiftId": 1
}
```
- `masterShiftId`: wajib, harus ada di tabel `master_shift`.
- Jika attendance untuk `employeeId` + `date` belum ada, dibuat baru.
- Jika sudah ada dengan `checkIn` null (mis. dibuat admin sebagai `absent`), `checkIn` dan `masterShiftId` diisi.
- `status` dihitung ulang sesuai aturan di bagian Status.

Response:
```json
{
  "status": 200,
  "message": "Sukses check-in",
  "data": { "...": "Attendance" }
}
```
```json
{
  "status": 403,
  "message": "Akun tidak terhubung ke data employee"
}
```
```json
{
  "status": 409,
  "message": "Sudah check-in pada tanggal ini"
}
```
```json
{
  "status": 409,
  "message": "Sedang cuti pada tanggal ini"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "masterShiftId": "wajib diisi | tidak ditemukan"
  }
}
```

### Check-out: `POST /api/attendances/check-out`
Permission: `attendance.checkInOut`

Tanpa request body. Mengisi `checkOut` pada attendance milik employee yang `checkIn` tidak null dan `checkOut` null, dengan `date` hari ini atau kemarin (shift malam). Jika ada lebih dari satu, ambil `date` terbaru. `status` tidak berubah.

Response:
```json
{
  "status": 200,
  "message": "Sukses check-out",
  "data": { "...": "Attendance" }
}
```
```json
{
  "status": 403,
  "message": "Akun tidak terhubung ke data employee"
}
```
```json
{
  "status": 404,
  "message": "Belum check-in"
}
```
