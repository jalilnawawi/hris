# Face Recognition API Documentation (FastAPI)

## Ketentuan Umum
- Service internal, hanya dipanggil backend Go (check-in / check-out attendance). Tidak diakses frontend dan tidak diekspos ke internet.
- Base URL diatur di config `.env` backend Go.
- Autentikasi pakai header `X-API-Key: <internal_api_key>`, bukan JWT user. Key disimpan di `.env` kedua service.
- Nama field pakai camelCase. Format response mengikuti RULES #6.
- Ambang batas kecocokan (threshold) diatur di config FastAPI. Go hanya membaca `match`.

### Error Umum
Berlaku untuk semua endpoint, tidak diulang per endpoint.
```json
{
  "status": 401,
  "message": "API key tidak valid"
}
```
```json
{
  "status": 500,
  "message": "Terjadi kesalahan pada server"
}
```

---
## Face Recognition

### Verify Face: `POST /api/face-recognition`

Content-Type: `multipart/form-data`.

Form Fields:
- `photo`: wajib, file foto wajah dari check-in / check-out, `image/jpeg`, `image/png`, atau `image/webp`, maks 2 MB.
- `referencePhotoUrl`: wajib, URL SeaweedFS dari `employees.photo_url`. FastAPI mengunduh foto referensi dari URL ini.

Aturan:
- `photo` dan foto referensi harus berisi tepat satu wajah.
- Wajah tidak cocok bukan error: response tetap 200 dengan `match: false`.

Response:
```json
{
  "status": 200,
  "message": "Sukses verifikasi wajah",
  "data": {
    "match": true,
    "score": 0.87
  }
}
```
- `match`: `true` jika `score` >= threshold.
- `score`: tingkat kemiripan, number 0 sampai 1. Hanya untuk log, tidak dipakai Go untuk keputusan.

```json
{
  "status": 422,
  "message": "Validasi gagal",
  "errors": {
    "photo": "wajib diisi | format harus jpeg, png, atau webp | maksimal 2 MB | wajah tidak terdeteksi | lebih dari satu wajah",
    "referencePhotoUrl": "wajib diisi | tidak bisa diunduh | wajah tidak terdeteksi | lebih dari satu wajah"
  }
}
```

---
## Known Gap: Liveness Detection
Belum ada pengecekan liveness / anti-spoofing. Foto dari foto cetak atau dari layar perangkat lain kemungkinan lolos verifikasi. Akan diselesaikan belakangan.

Rencana saat diselesaikan:
- FastAPI menambah field `isLive` (boolean) di response `data`.
- Go menolak check-in / check-out jika `isLive: false` (422, `photo`: `wajah tidak terdeteksi sebagai wajah asli`).

---
## Mapping di Backend Go
Go memanggil endpoint ini dengan timeout dari config `.env`. Hasilnya dipetakan ke response check-in / check-out:

| Hasil FastAPI | Response Go |
|---------------|-------------|
| 200, `match: true` | Lanjut simpan attendance |
| 200, `match: false` | 422, `photo`: `wajah tidak cocok` |
| 422 pada `photo` | 422, `photo`: `wajah tidak terdeteksi` |
| 422 pada `referencePhotoUrl` | 409, `Foto employee belum diatur atau tidak valid, hubungi admin` |
| 401, 500, timeout, koneksi gagal | 503, `Layanan face recognition tidak tersedia, coba lagi` |
