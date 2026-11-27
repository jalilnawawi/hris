# Departments API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Waktu pakai format ISO 8601 dengan offset, mis. `2026-11-19T10:00:00+07:00` (RULES #9).
- `id` department bertipe number (`SERIAL`). `createdBy` / `updatedBy` bertipe string UUID.

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
Department:
```json
{
  "id": 1,
  "name": "string",
  "description": "string | null",
  "createdAt": "string",
  "updatedAt": "string",
  "createdBy": "string | null",
  "updatedBy": "string | null"
}
```

---
## Departments

### Create Department: `POST /api/departments`
Permission: `department.create`

Request Body:
```json
{
  "name": "string",
  "description": "string | null"
}
```
- `name`: wajib, maks 100 karakter, unik.
- `description`: opsional, maks 255 karakter.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data department",
  "data": { "...": "Department" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 100 karakter | sudah dipakai",
    "description": "maksimal 255 karakter"
  }
}
```

### List Departments: `GET /api/departments`
Permission: `department.read`

Tanpa paginasi, jumlah department kecil. Diurutkan berdasarkan `name` naik.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data department",
  "data": [{ "...": "Department" }]
}
```

### Get Department by ID: `GET /api/departments/{id}`
Permission: `department.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data department",
  "data": { "...": "Department" }
}
```
```json
{
  "status": 404,
  "message": "Data department tidak ditemukan"
}
```

### Update Department: `PUT /api/departments/{id}`
Permission: `department.update`

Request Body dan aturan validasi sama dengan Create Department.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data department",
  "data": { "...": "Department" }
}
```
```json
{
  "status": 404,
  "message": "Data department tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 100 karakter | sudah dipakai",
    "description": "maksimal 255 karakter"
  }
}
```

### Delete Department: `DELETE /api/departments/{id}`
Permission: `department.delete`

Department yang masih dipakai di `employees` atau `master_kpi` tidak bisa dihapus.

Response:
```json
{
  "status": 200,
  "message": "Sukses menghapus data department"
}
```
```json
{
  "status": 404,
  "message": "Data department tidak ditemukan"
}
```
```json
{
  "status": 409,
  "message": "Department masih dipakai karyawan atau KPI"
}
```
