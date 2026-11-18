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