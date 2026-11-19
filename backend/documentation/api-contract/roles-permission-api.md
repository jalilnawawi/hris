# Roles and Permissions API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Waktu pakai format ISO 8601 dengan offset, mis. `2026-11-19T10:00:00+07:00`.
- `id` role dan permission bertipe number (`SERIAL`). `createdBy` / `updatedBy` bertipe string UUID.
- Permission tidak bisa dibuat, diubah, atau dihapus lewat API. Permission di-seed lewat migration karena kodenya dipakai langsung oleh middleware.

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
Role:
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
Permission:
```json
{
  "id": 1,
  "code": "employee.read",
  "description": "string | null",
  "createdAt": "string"
}
```

---
## Roles

### Create Role: `POST /api/roles`
Permission: `role.create`

Request Body:
```json
{
  "name": "string",
  "description": "string"
}
```
- `name`: wajib, maks 50 karakter, unik.
- `description`: opsional, maks 255 karakter.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data role",
  "data": { "...": "Role" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 50 karakter | sudah dipakai",
    "description": "maksimal 255 karakter"
  }
}
```

### List Roles: `GET /api/roles`
Permission: `role.read`

Tanpa paginasi, jumlah role kecil.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data role",
  "data": [{ "...": "Role" }]
}
```

### Get Role by ID: `GET /api/roles/{id}`
Permission: `role.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data role",
  "data": { "...": "Role" }
}
```
```json
{
  "status": 404,
  "message": "Data role tidak ditemukan"
}
```

### Update Role: `PUT /api/roles/{id}`
Permission: `role.update`

Request Body dan aturan validasi sama dengan Create Role.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data role",
  "data": { "...": "Role" }
}
```
```json
{
  "status": 404,
  "message": "Data role tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "name": "wajib diisi | maksimal 50 karakter | sudah dipakai",
    "description": "maksimal 255 karakter"
  }
}
```

### Delete Role: `DELETE /api/roles/{id}`
Permission: `role.delete`

Relasi di `role_permissions` ikut terhapus (`ON DELETE CASCADE`). Role yang masih dipakai user tidak bisa dihapus.

Response:
```json
{
  "status": 200,
  "message": "Sukses menghapus data role"
}
```
```json
{
  "status": 404,
  "message": "Data role tidak ditemukan"
}
```
```json
{
  "status": 409,
  "message": "Role masih dipakai user"
}
```

---
## Permissions (read-only)

### List Permissions: `GET /api/permissions`
Permission: `permission.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data permission",
  "data": [{ "...": "Permission" }]
}
```

### Get Permission by ID: `GET /api/permissions/{id}`
Permission: `permission.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data permission",
  "data": { "...": "Permission" }
}
```
```json
{
  "status": 404,
  "message": "Data permission tidak ditemukan"
}
```

---
## Role Permissions

### Get Permissions of a Role: `GET /api/roles/{roleId}/permissions`
Permission: `role.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data permission role",
  "data": [{ "...": "Permission" }]
}
```
```json
{
  "status": 404,
  "message": "Data role tidak ditemukan"
}
```

### Set Permissions of a Role: `PUT /api/roles/{roleId}/permissions`
Permission: `role.update`

Mengganti seluruh permission milik role dengan isi `permissionIds`, dalam 1 transaksi. Array kosong berarti role tidak punya permission sama sekali.

Request Body:
```json
{
  "permissionIds": [1, 2, 3]
}
```
- `permissionIds`: wajib (boleh array kosong), semua id harus ada di tabel `permissions`.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data permission role",
  "data": [{ "...": "Permission" }]
}
```
```json
{
  "status": 404,
  "message": "Data role tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "permissionIds": "wajib diisi | id tidak ditemukan: 7, 9"
  }
}
```
