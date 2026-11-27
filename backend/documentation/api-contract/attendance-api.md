# Attendance API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Kolom `DATE` dikirim sebagai `yyyy-mm-dd`. Waktu pakai format ISO 8601 dengan offset (RULES #9).
- `id` attendance bertipe string UUID. `employeeId` bertipe string UUID, `masterShiftId` bertipe number (`SERIAL`).
- Request hanya mengirim id relasi (`employeeId`, `masterShiftId`). Response mengirim relasi sebagai object `{ id, fullName }` / `{ id, name }` hasil JOIN.
- `workLocation` = lokasi kerja saat check-in, diisi otomatis oleh check-in.
- `workLocation`, `checkInLocation`, `checkOutLocation`, `checkInPhoto`, dan `checkOutPhoto` bernilai `null` jika attendance dibuat atau diubah admin (Create / Update Attendance tidak menerima koordinat dan foto).
- `checkInPhoto` / `checkOutPhoto` berisi path endpoint foto (lihat Get Attendance Photo), bukan URL SeaweedFS. Bernilai `null` jika foto tidak ada atau `date` lebih dari 3 bulan lalu (retensi).
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
  "workLocation": {
    "id": 1,
    "name": "string"
  },
  "date": "2026-11-19",
  "checkIn": "2026-11-19T08:00:00+07:00 | null",
  "checkOut": "2026-11-19T17:00:00+07:00 | null",
  "checkInLocation": { "latitude": -6.2088, "longitude": 106.8456 },
  "checkOutLocation": { "latitude": -6.2088, "longitude": 106.8456 },
  "checkInPhoto": "/api/attendances/{id}/photos/check-in | null",
  "checkOutPhoto": "/api/attendances/{id}/photos/check-out | null",
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

### Get Attendance Photo: `GET /api/attendances/{id}/photos/{type}`
Permission: `attendance.read`

- `type`: `check-in` atau `check-out`.
- Backend mengambil file dari SeaweedFS lalu meneruskannya ke client. SeaweedFS tidak diakses langsung oleh frontend.

Response: binary file dengan `Content-Type: image/jpeg`.
```json
{
  "status": 404,
  "message": "Foto attendance tidak ditemukan"
}
```
Berlaku jika attendance tidak ada, `type` tidak valid, foto tidak pernah diunggah, atau foto sudah terhapus oleh retensi 3 bulan.

---
## Check-in / Check-out
Dipakai karyawan untuk absen sendiri.
- `employeeId` diambil dari `users.employee_id` milik user di JWT, tidak dikirim di request. User tanpa `employee_id` ditolak 403.
- `checkIn` / `checkOut` diisi dari waktu server (`now()`) di zona waktu aplikasi, tidak dikirim di request.
- `date` = tanggal waktu server. Shift malam (`endTime` < `startTime`) yang check-in sebelum `endTime` dicatat ke tanggal sebelumnya.

### Face Recognition + GPS
Check-in dan check-out wajib lolos keduanya. Urutan pengecekan di service:
1. Validasi bentuk request (422).
2. Cek state attendance (sudah check-in, sedang cuti, belum check-in) dan `employees.photo_url` wajib ada sebagai foto referensi (409 / 404).
3. GPS check di Go: jarak koordinat ke tiap `work_location` dihitung dengan rumus haversine. Lolos jika minimal satu jarak <= `radiusMeters` (422). Jika lebih dari satu lolos, pilih yang terdekat. GPS dicek sebelum face recognition supaya request di luar radius tidak memanggil FastAPI.
4. Face recognition ke FastAPI, lihat `face-recognition-api.md`. Wajah tidak terdeteksi atau tidak cocok ditolak 422. FastAPI timeout atau error ditolak 503.
5. Simpan attendance beserta koordinat (`check_in_lat` / `check_in_lng` atau `check_out_lat` / `check_out_lng`). Check-in juga menyimpan `work_location_id` hasil langkah 3. Check-out tidak mengubah `work_location_id`.

Penyimpanan foto (setelah langkah 4 lolos, sebelum transaksi DB langkah 5):
- Foto di-resize maks 640 px sisi terpanjang dan di-encode JPEG kualitas 75 (`image/jpeg` + `golang.org/x/image/draw`).
- Upload ke SeaweedFS dengan `ttl=3M`, jadi foto terhapus otomatis setelah 3 bulan. Tidak perlu cron job.
- Yang disimpan di `check_in_photo` / `check_out_photo` adalah `fid` SeaweedFS, bukan URL penuh.
- Jika transaksi DB gagal setelah upload, file yatim ikut terhapus oleh TTL.
- Data biometrik (UU PDP No. 27/2022): persetujuan karyawan dicatat saat onboarding.

Deteksi fake GPS (`Location.isMock()` di Android, `CLLocation.sourceInformation.isSimulatedBySoftware` di iOS) dilakukan di aplikasi mobile sebelum request dikirim.

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

Content-Type: `multipart/form-data`.

Form Fields:
- `masterShiftId`: wajib, harus ada di tabel `master_shift`.
- `photo`: wajib, file `image/jpeg`, `image/png`, atau `image/webp`, maks 2 MB.
- `latitude`: wajib, number antara -90 dan 90.
- `longitude`: wajib, number antara -180 dan 180.
- `accuracy`: wajib, akurasi GPS dari device dalam meter, maks 100.

Aturan:
- Jika attendance untuk `employeeId` + `date` belum ada, dibuat baru.
- Jika sudah ada dengan `checkIn` null (mis. dibuat admin sebagai `absent`), `checkIn`, koordinat, dan `masterShiftId` diisi.
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
  "status": 409,
  "message": "Foto employee belum diatur atau tidak valid, hubungi admin"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "masterShiftId": "wajib diisi | tidak ditemukan",
    "photo": "wajib diisi | format harus jpeg, png, atau webp | maksimal 2 MB | wajah tidak terdeteksi | wajah tidak cocok",
    "latitude": "wajib diisi | harus antara -90 dan 90 | di luar radius lokasi kerja",
    "longitude": "wajib diisi | harus antara -180 dan 180",
    "accuracy": "wajib diisi | maksimal 100 meter"
  }
}
```
```json
{
  "status": 503,
  "message": "Layanan face recognition tidak tersedia, coba lagi"
}
```

### Check-out: `POST /api/attendances/check-out`
Permission: `attendance.checkInOut`

Content-Type: `multipart/form-data`.

Form Fields: `photo`, `latitude`, `longitude`, `accuracy`. Aturan validasi sama dengan Check-in.

Mengisi `checkOut` dan koordinat check-out pada attendance milik employee yang `checkIn` tidak null dan `checkOut` null, dengan `date` hari ini atau kemarin (shift malam). Jika ada lebih dari satu, ambil `date` terbaru. `status` tidak berubah.

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
```json
{
  "status": 409,
  "message": "Foto employee belum diatur atau tidak valid, hubungi admin"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "photo": "wajib diisi | format harus jpeg, png, atau webp | maksimal 2 MB | wajah tidak terdeteksi | wajah tidak cocok",
    "latitude": "wajib diisi | harus antara -90 dan 90 | di luar radius lokasi kerja",
    "longitude": "wajib diisi | harus antara -180 dan 180",
    "accuracy": "wajib diisi | maksimal 100 meter"
  }
}
```
```json
{
  "status": 503,
  "message": "Layanan face recognition tidak tersedia, coba lagi"
}
```
