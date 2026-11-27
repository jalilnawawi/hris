# Work Location API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Waktu pakai format ISO 8601 dengan offset, mis. `2026-11-19T10:00:00+07:00` (RULES #9).
- `id` work location bertipe number (`SERIAL`).
- Work location dipakai untuk GPS check saat check-in / check-out. Employee boleh absen di work location mana saja.
- Semua aksi create, update, dan delete ditulis ke `audit_logs` dalam transaksi yang sama (RULES #5).

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
WorkLocation:
```json
{
  "id": 1,
  "name": "string",
  "latitude": -6.2088,
  "longitude": 106.8456,
  "radiusMeters": 100,
  "createdAt": "string",
  "updatedAt": "string"
}
```

---
## Work Locations

### Create Work Location: `POST /api/work-locations`
Permission: `workLocation.create`

Request Body:
```json
{
  "name": "string",
  "latitude": -6.2088,
  "longitude": 106.8456,
  "radiusMeters": 100
}
```
- `name`: wajib, maks 100 karakter, unik.
- `latitude`: wajib, number antara -90 dan 90.
- `longitude`: wajib, number antara -180 dan 180.
- `radiusMeters`: wajib, integer lebih dari 0.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data work location",
  "data": { "...": "WorkLocation" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 100 karakter | sudah dipakai",
    "latitude": "wajib diisi | harus antara -90 dan 90",
    "longitude": "wajib diisi | harus antara -180 dan 180",
    "radiusMeters": "wajib diisi | harus lebih dari 0"
  }
}
```

### List Work Locations: `GET /api/work-locations`
Permission: `workLocation.read`

Tanpa paginasi, jumlah work location kecil. Diurutkan berdasarkan `name` naik.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data work location",
  "data": [{ "...": "WorkLocation" }]
}
```

### Get Work Location by ID: `GET /api/work-locations/{id}`
Permission: `workLocation.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data work location",
  "data": { "...": "WorkLocation" }
}
```
```json
{
  "status": 404,
  "message": "Data work location tidak ditemukan"
}
```

### Update Work Location: `PUT /api/work-locations/{id}`
Permission: `workLocation.update`

Request Body dan aturan validasi sama dengan Create Work Location.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data work location",
  "data": { "...": "WorkLocation" }
}
```
```json
{
  "status": 404,
  "message": "Data work location tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 100 karakter | sudah dipakai",
    "latitude": "wajib diisi | harus antara -90 dan 90",
    "longitude": "wajib diisi | harus antara -180 dan 180",
    "radiusMeters": "wajib diisi | harus lebih dari 0"
  }
}
```

### Delete Work Location: `DELETE /api/work-locations/{id}`
Permission: `workLocation.delete`

Work location yang masih dipakai di `attendance` tidak bisa dihapus.

Response:
```json
{
  "status": 200,
  "message": "Sukses menghapus data work location"
}
```
```json
{
  "status": 404,
  "message": "Data work location tidak ditemukan"
}
```
```json
{
  "status": 409,
  "message": "Work location masih dipakai di absensi"
}
```
