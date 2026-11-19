## HRIS Web Frontend

## Tech Stack
- Bun + Vite + React TS

### 1. Architecture

```
frontend/src/
  api/              fetch ke endpoint backend, satu file per fitur
  components/
    ui/             komponen re-usable lintas fitur: button, dialog, chart, input
    <fitur>/        komponen khusus satu fitur
  context/          state bersama (mis. sesi/auth)
  hooks/
    <fitur>/        hook pemanggil api/ & context/ per fitur
  pages/
    <fitur>/        halaman per fitur; hanya merangkai
  styles/           CSS global + token Tailwind
  types/            DTO untuk response JSON dari API
  utils/            fungsi re-usable (mis. konversi tanggal WIB)
```

### 2. Rules

1. Pisahkan render Page dengan render Components.
2. Komponen re-usable (button, dll) dikumpulkan di `components/ui/`.
3. Fungsi re-usable (mis. date-convert) dikumpulkan di `utils/`.
4. File Page hanya memanggil **components**, **utils**, dan **hooks**. Page tidak memanggil
   `api/` langsung; pengambilan data lewat hooks.
5. Maksimal **200 baris per file** `.ts`/`.tsx` tulisan tangan. File test dan file hasil generate dikecualikan.
6. Tidak ada data bisnis maupun token di localStorage/sessionStorage.
7. Gunakan skill `/frontend-design` untuk menyempurnakan desain UI.
8. Waktu dari API selalu berformat ISO 8601 dengan offset (mis. `2026-11-19T10:00:00+07:00`). Kolom tanggal tanpa jam berformat `yyyy-mm-dd`.
   - Simpan dan olah waktu sebagai nilai ISO / `Date`. Konversi ke format tampilan hanya saat render.
   - Format tampilan: `dd-mm-yyyy HH:mm:ss` (jam 24) di zona `Asia/Jakarta`. Tanggal tanpa jam: `dd-mm-yyyy`.
   - Konversi lewat satu fungsi di `utils/` (pakai `Intl.DateTimeFormat`, tanpa library tambahan). Jangan format tanggal langsung di komponen.
   - Kirim waktu ke API juga dalam format ISO 8601, bukan format tampilan.