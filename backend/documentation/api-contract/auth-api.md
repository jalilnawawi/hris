# Authentication and Authorization API Documentation

## Ketentuan Umum
- Nama field pakai camelCase. Waktu pakai format ISO 8601 dengan offset (RULES #9).
- Tidak ada self-register. Admin membuat user, lalu user mengaktifkan akun lewat link email (token `activation`).
- **Access token** (JWT) dikirim di body response dan disimpan frontend di memory. Request yang butuh login wajib header `Authorization: Bearer <token>`.
- **Refresh token** hanya dikirim lewat cookie, tidak pernah lewat body:
  ```
  Set-Cookie: refreshToken=<token>; HttpOnly; Secure; SameSite=Strict; Path=/api/auth; Max-Age=604800
  ```
  Disimpan di `user_tokens` (type `refresh`) dalam bentuk hash. Setiap refresh, token lama ditandai `used_at` dan diganti token baru (rotation).
- Login, logout, aktivasi, reset password, dan ganti password ditulis ke `audit_logs` (RULES #5).

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
  "status": 429,
  "message": "Terlalu banyak request, coba lagi nanti"
}
```
```json
{
  "status": 500,
  "message": "Terjadi kesalahan pada server"
}
```

### Object
Session (response login dan refresh):
```json
{
  "token": "jwt_access_token",
  "expiresAt": "2026-11-19T10:15:00+07:00"
}
```

---
## Authentication (tanpa login)

### Login: `POST /api/auth/login`
Request Body:
```json
{
  "username": "string",
  "password": "string"
}
```
Response (+ `Set-Cookie: refreshToken`). `users.last_login_at` diperbarui.
```json
{
  "status": 200,
  "message": "Sukses login",
  "data": {
    "token": "jwt_access_token",
    "expiresAt": "2026-11-19T10:15:00+07:00",
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string"
    }
  }
}
```
Username tidak ada, password salah, atau akun belum diaktivasi memakai pesan yang sama agar tidak bisa ditebak.
```json
{
  "status": 401,
  "message": "Username atau password salah"
}
```
```json
{
  "status": 403,
  "message": "Akun nonaktif"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "username": "wajib diisi",
    "password": "wajib diisi"
  }
}
```

### Login with Google: `POST /api/auth/google`
Hanya login. Email dari Google harus sudah terdaftar di `users.email`.

Request Body:
```json
{
  "idToken": "google_id_token"
}
```
Response sama dengan Login (+ `Set-Cookie: refreshToken`).
```json
{
  "status": 401,
  "message": "Akun Google tidak terdaftar"
}
```
```json
{
  "status": 403,
  "message": "Akun nonaktif"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "idToken": "wajib diisi | tidak valid"
  }
}
```

### Refresh Token: `POST /api/auth/refresh`
Tanpa body. Refresh token dibaca dari cookie.

Response (+ `Set-Cookie: refreshToken` baru):
```json
{
  "status": 200,
  "message": "Sukses memperbarui token",
  "data": {
    "token": "jwt_access_token",
    "expiresAt": "2026-11-19T10:30:00+07:00"
  }
}
```
Cookie tidak ada, token kedaluwarsa, atau token sudah pernah dipakai. Jika token yang sudah dipakai dikirim ulang, semua refresh token milik user tersebut di-revoke.
```json
{
  "status": 401,
  "message": "Sesi berakhir, silakan login kembali"
}
```

### Activate Account: `POST /api/auth/activate`
Token dari link email aktivasi. User menentukan password pertama.

Request Body:
```json
{
  "token": "string",
  "password": "string"
}
```
Response:
```json
{
  "status": 200,
  "message": "Sukses mengaktifkan akun"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "token": "wajib diisi | tidak valid atau kedaluwarsa",
    "password": "wajib diisi | minimal 8 karakter"
  }
}
```

### Forgot Password: `POST /api/auth/forgot-password`
Response selalu sama, baik email terdaftar maupun tidak. Email hanya dikirim jika terdaftar.

Request Body:
```json
{
  "email": "string"
}
```
Response:
```json
{
  "status": 200,
  "message": "Jika email terdaftar, link reset password sudah dikirim"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "email": "wajib diisi | format email tidak valid"
  }
}
```

### Reset Password: `POST /api/auth/reset-password`
Token dari link email reset. Semua refresh token user di-revoke.

Request Body:
```json
{
  "token": "string",
  "newPassword": "string"
}
```
Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah password"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "token": "wajib diisi | tidak valid atau kedaluwarsa",
    "newPassword": "wajib diisi | minimal 8 karakter"
  }
}
```

---
## Authentication (wajib login)

### Logout: `POST /api/auth/logout`
Tanpa body. Refresh token dari cookie di-revoke, lalu cookie dihapus (`Max-Age=0`). Tetap 200 walau cookie sudah tidak ada.

Response:
```json
{
  "status": 200,
  "message": "Sukses logout"
}
```

### Get Current User: `GET /api/auth/me`
Data user yang sedang login beserta role dan permission-nya. `role` bernilai `null` jika user belum punya role.

Response:
```json
{
  "status": 200,
  "message": "Sukses mendapatkan data user",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "employeeId": "uuid | null",
    "lastLoginAt": "2026-11-19T10:00:00+07:00",
    "role": {
      "id": 1,
      "name": "admin"
    },
    "permissions": ["employee.read", "employee.create"]
  }
}
```

### Update Current User: `PUT /api/auth/me`
Hanya `username` yang bisa diubah sendiri. Email dipakai untuk login Google dan reset password, jadi hanya bisa diubah admin.

Request Body:
```json
{
  "username": "string"
}
```
Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data user",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "updatedAt": "2026-11-19T10:00:00+07:00"
  }
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "username": "wajib diisi | maksimal 50 karakter | sudah dipakai"
  }
}
```

### Change Password: `PUT /api/auth/change-password`
Semua refresh token user di-revoke dan cookie dihapus. User harus login ulang.

Request Body:
```json
{
  "currentPassword": "string",
  "newPassword": "string"
}
```
Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah password"
}
```
Password lama salah memakai 422, bukan 401, agar frontend tidak menganggap sesi berakhir.
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "currentPassword": "wajib diisi | password salah",
    "newPassword": "wajib diisi | minimal 8 karakter"
  }
}
```

---
## Authorization

### Set User Role: `PUT /api/auth/users/{id}/role`
Permission: `user.update`

Satu user hanya punya satu role (`users.role_id`). Endpoint ini mengganti role user. Tidak ada endpoint revoke terpisah.

Request Body:
```json
{
  "roleId": 1
}
```
Response:
```json
{
  "status": 200,
  "message": "Sukses mengubah data role user",
  "data": {
    "userId": "uuid",
    "roleId": 1
  }
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
  "status": 404,
  "message": "Data user tidak ditemukan"
}
```
```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "roleId": "wajib diisi | role tidak ditemukan"
  }
}
```
