# Employee API Documentation

## Ketentuan Umum
- Semua endpoint wajib header `Authorization: Bearer <jwt_token>`.
- Tiap endpoint mencantumkan permission yang dibutuhkan. Middleware cek permission, bukan nama role (RULES #8).
- Nama field pakai camelCase. Kolom `DATE` dikirim sebagai `yyyy-mm-dd`. Waktu pakai format ISO 8601 dengan offset (RULES #9).
- `id` employee bertipe string UUID. `departmentId` bertipe number (`SERIAL`).
- Request hanya mengirim id relasi (`departmentId`, `reportingManagerId`). Response mengirim relasi sebagai object `{ id, name }` / `{ id, fullName }` hasil JOIN.
- **Tidak ada endpoint delete.** Employee yang keluar cukup diubah statusnya ke `resigned` / `terminated` lewat `PATCH /api/employees/{id}/status`. Data tetap disimpan karena dirujuk attendance, leave, KPI, dan payroll.
- `employmentStatus` dan `photoUrl` tidak bisa diubah lewat Create / Update. Status hanya lewat `PATCH /status`, foto hanya lewat `PATCH /photo`.
- Semua aksi create dan update ditulis ke `audit_logs` dalam transaksi yang sama (RULES #5).

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
Employee:
```json
{
  "id": "uuid",
  "employeeNumber": "string",
  "fullName": "string",
  "gender": "male | female | null",
  "phoneNumber": "string | null",
  "address": "string | null",
  "position": "string | null",
  "department": {
    "id": 1,
    "name": "string"
  },
  "reportingManager": {
    "id": "uuid",
    "fullName": "string"
  },
  "dateOfBirth": "2000-01-31",
  "dateOfJoining": "2026-11-19",
  "maritalStatus": "single | married | divorced | widowed | null",
  "photoUrl": "string | null",
  "parentName": "string | null",
  "parentPhoneNumber": "string | null",
  "major": "string | null",
  "employmentStatus": "active | resigned | terminated",
  "resignDate": "2026-11-19 | null",
  "createdAt": "2026-11-19T10:00:00+07:00",
  "updatedAt": "2026-11-19T10:00:00+07:00"
}
```
- `department` dan `reportingManager` bernilai `null` jika belum diisi.
- `dateOfBirth` dan `dateOfJoining` bernilai `null` jika belum diisi.

---
## Employees

### Create Employee: `POST /api/employees`
Permission: `employee.create`

Employee baru selalu berstatus `active` dengan `resignDate` null.

Request Body:
```json
{
  "employeeNumber": "string",
  "fullName": "string",
  "gender": "male",
  "phoneNumber": "string",
  "address": "string",
  "position": "string",
  "departmentId": 1,
  "reportingManagerId": "uuid",
  "dateOfBirth": "2000-01-31",
  "dateOfJoining": "2026-11-19",
  "maritalStatus": "single",
  "parentName": "string",
  "parentPhoneNumber": "string",
  "major": "string"
}
```
- `employeeNumber`: wajib, maks 20 karakter, unik.
- `fullName`: wajib, maks 100 karakter.
- `gender`: opsional, salah satu dari `male`, `female`.
- `phoneNumber`: opsional, maks 15 karakter, hanya angka dan awalan `+`.
- `address`: opsional, maks 255 karakter.
- `position`: opsional, maks 50 karakter.
- `departmentId`: opsional, harus ada di tabel `departments`.
- `reportingManagerId`: opsional, harus employee berstatus `active`, tidak boleh diri sendiri.
- `dateOfBirth`: opsional, format `yyyy-mm-dd`, tidak boleh di masa depan.
- `dateOfJoining`: opsional, format `yyyy-mm-dd`.
- `maritalStatus`: opsional, salah satu dari `single`, `married`, `divorced`, `widowed`.
- `parentName`: opsional, maks 100 karakter.
- `parentPhoneNumber`: opsional, maks 15 karakter, hanya angka dan awalan `+`.
- `major`: opsional, maks 100 karakter.

Response:
```json
{
  "status": 201,
  "message": "Sukses menambahkan data employee",
  "data": { "...": "Employee" }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "employeeNumber": "wajib diisi | maksimal 20 karakter | sudah dipakai",
    "fullName": "wajib diisi | maksimal 100 karakter",
    "gender": "nilai tidak valid",
    "departmentId": "departemen tidak ditemukan",
    "reportingManagerId": "employee tidak ditemukan | employee tidak aktif | tidak boleh diri sendiri",
    "dateOfBirth": "format tanggal tidak valid | tidak boleh di masa depan",
    "maritalStatus": "nilai tidak valid"
  }
}
```

### List Employees: `GET /api/employees`
Permission: `employee.read`

Query Params:
- `page`: opsional, default `1`.
- `perPage`: opsional, default `10`, maks `100`.
- `q`: opsional, cari di `fullName` atau `employeeNumber` (case-insensitive).
- `departmentId`: opsional.
- `employmentStatus`: opsional, salah satu dari `active`, `resigned`, `terminated`.

Diurutkan berdasarkan `fullName` naik.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data employee",
  "data": [{ "...": "Employee" }],
  "page": 1,
  "perPage": 10,
  "lastPage": 20,
  "totalData": 195
}
```

### Get Employee by ID: `GET /api/employees/{id}`
Permission: `employee.read`

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data employee",
  "data": { "...": "Employee" }
}
```
```json
{
  "status": 404,
  "message": "Data employee tidak ditemukan"
}
```

### Update Employee: `PUT /api/employees/{id}`
Permission: `employee.update`

Request Body dan aturan validasi sama dengan Create Employee. PUT mengganti seluruh data: field opsional yang tidak dikirim atau bernilai `null` dikosongkan.

Tambahan validasi `reportingManagerId`: tidak boleh membentuk siklus atasan. Contoh: A atasan B, maka B tidak boleh dijadikan atasan A, termasuk lewat rantai bertingkat (A → B → C → A). Dicek dengan menelusuri rantai atasan dari `reportingManagerId` ke atas; jika rantai sampai ke `{id}`, request ditolak.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data employee",
  "data": { "...": "Employee" }
}
```
```json
{
  "status": 404,
  "message": "Data employee tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "employeeNumber": "wajib diisi | maksimal 20 karakter | sudah dipakai",
    "reportingManagerId": "employee tidak ditemukan | employee tidak aktif | tidak boleh diri sendiri | membentuk siklus atasan"
  }
}
```

### Update Employee Status: `PATCH /api/employees/{id}/status`
Permission: `employee.update`

Jika status berubah ke `resigned` atau `terminated` dan employee punya akun user, dalam transaksi yang sama:
- `users.is_active` diubah menjadi `false`.
- Semua refresh token milik user tersebut di-revoke.

Jika status kembali ke `active` (rehire), akun user **tetap nonaktif**. Admin mengaktifkan akun secara terpisah.

Request Body:
```json
{
  "employmentStatus": "resigned",
  "resignDate": "2026-11-19"
}
```
- `employmentStatus`: wajib, salah satu dari `active`, `resigned`, `terminated`.
- `resignDate`: wajib jika status `resigned` atau `terminated`, tidak boleh sebelum `dateOfJoining`. Harus `null` jika status `active`.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data status employee",
  "data": { "...": "Employee" }
}
```
```json
{
  "status": 404,
  "message": "Data employee tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "employmentStatus": "wajib diisi | nilai tidak valid",
    "resignDate": "wajib diisi | format tanggal tidak valid | tidak boleh sebelum tanggal bergabung | harus kosong jika status active"
  }
}
```

### Update Employee Photo: `PATCH /api/employees/{id}/photo`
Permission: `employee.update`

Content-Type: `multipart/form-data`. Backend upload file ke SeaweedFS, lalu menyimpan URL dari response upload ke `photo_url`.

Form Data:
- `photo`: wajib, file `image/jpeg`, `image/png`, atau `image/webp`, maks 2 MB.

Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data foto employee",
  "data": { "...": "Employee" }
}
```
```json
{
  "status": 404,
  "message": "Data employee tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "photo": "wajib diisi | format harus jpeg, png, atau webp | maksimal 2 MB"
  }
}
```
