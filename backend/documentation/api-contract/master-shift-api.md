# Master Shift API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Waktu pakai format ISO 8601 dengan offset, mis. `2026-11-19T10:00:00+07:00` (RULES #9).
- `startTime` / `endTime` (kolom `TIME`) dikirim sebagai `HH:mm:ss` tanpa offset, di request maupun response, mis. `08:00:00`.
- `id` master shift bertipe number (`SERIAL`).

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
MasterShift:
```json
{
  "id": 1,
  "name": "string",
  "startTime": "HH:mm:ss",
  "endTime": "HH:mm:ss",
  "createdAt": "string",
  "updatedAt": "string"
}
```

---
## Master Shifts

### Create Master Shift: `POST /api/master-shifts`
Permission: `masterShift.create`

Request Body:
```json
{
  "name": "string",
  "startTime": "HH:mm:ss",
  "endTime": "HH:mm:ss"
}
```
- `name`: wajib, maks 50 karakter, unik.
- `startTime`: wajib, format `HH:mm:ss`.
- `endTime`: wajib, format `HH:mm:ss`, tidak boleh sama dengan `startTime`.
- `endTime` < `startTime` berarti shift malam yang selesai di hari berikutnya, mis. `22:00:00` - `06:00:00`.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data master shift",
  "data": { "...": "MasterShift" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 50 karakter | sudah dipakai",
    "startTime": "wajib diisi | format harus HH:mm:ss",
    "endTime": "wajib diisi | format harus HH:mm:ss | tidak boleh sama dengan startTime"
  }
}
```

### List Master Shifts: `GET /api/master-shifts`
Permission: `masterShift.read`

Tanpa paginasi, jumlah master shift kecil. Diurutkan berdasarkan `startTime` naik.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data master shift",
  "data": [{ "...": "MasterShift" }]
}
```

### Get Master Shift by ID: `GET /api/master-shifts/{id}`
Permission: `masterShift.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data master shift",
  "data": { "...": "MasterShift" }
}
```
```json
{
  "status": 404,
  "message": "Data master shift tidak ditemukan"
}
```

### Update Master Shift: `PUT /api/master-shifts/{id}`
Permission: `masterShift.update`

Request Body dan aturan validasi sama dengan Create Master Shift.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data master shift",
  "data": { "...": "MasterShift" }
}
```
```json
{
  "status": 404,
  "message": "Data master shift tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 50 karakter | sudah dipakai",
    "startTime": "wajib diisi | format harus HH:mm:ss",
    "endTime": "wajib diisi | format harus HH:mm:ss | tidak boleh sama dengan startTime"
  }
}
```

### Delete Master Shift: `DELETE /api/master-shifts/{id}`
Permission: `masterShift.delete`

Master shift yang masih dipakai di `attendance` tidak bisa dihapus.

Response:
```json
{
  "status": 200,
  "message": "Sukses menghapus data master shift"
}
```
```json
{
  "status": 404,
  "message": "Data master shift tidak ditemukan"
}
```
```json
{
  "status": 409,
  "message": "Master shift masih dipakai di absensi"
}
```
