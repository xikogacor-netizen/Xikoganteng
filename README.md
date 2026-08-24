Xikofly.tools

Starter project: landing page + admin dashboard (login, sidebar menu, tab bar)
buat platform multi-tools. Dibangun pakai Next.js 14 (App Router) + Tailwind CSS,
siap deploy ke Vercel.

## Struktur

```
app/
  page.tsx              -> landing page
  login/page.tsx         -> halaman login admin (demo)
  admin/layout.tsx       -> shell dashboard (sidebar + header)
  admin/page.tsx         -> dashboard overview (stat cards + tab)
  admin/tools/page.tsx   -> manajemen tools (tab: semua/aktif/nonaktif)
  admin/users/page.tsx   -> manajemen user (tab: active/pending/banned)
  admin/settings/page.tsx-> pengaturan situs
components/
  Sidebar.tsx            -> menu samping admin
  Tabbar.tsx             -> komponen tab reusable
  Logo.tsx               -> logo Xikofly
```

## Menjalankan di lokal

```bash
npm install
npm run dev
```

Buka http://localhost:3000

## Deploy ke Vercel

1. Push folder ini ke repo GitHub baru.
2. Buka vercel.com -> New Project -> import repo tadi.
3. Framework preset otomatis kedetek "Next.js", langsung klik Deploy.
4. Kalau mau custom domain `xikofly.tools`, tambahin di Project Settings -> Domains.

## PENTING: Login saat ini masih DEMO

Halaman `/login` sekarang cuma nyimpen cookie dummy, BELUM ada validasi
password sungguhan ke database. Sebelum production, wajib disambungkan ke
auth beneran. Rekomendasi:

### Opsi 1 - NextAuth.js (Auth.js) + Supabase
- `npm install next-auth @auth/supabase-adapter`
- Bikin `app/api/auth/[...nextauth]/route.ts`
- Simpan user & role (admin/user) di tabel Supabase

### Opsi 2 - Clerk (paling cepat setup)
- `npm install @clerk/nextjs`
- Tinggal wrap `<ClerkProvider>` di `app/layout.tsx`, ada built-in role/organization

Setelah auth beneran jalan, ganti logic di `app/login/page.tsx` dan tambahin
middleware (`middleware.ts`) buat proteksi semua route `/admin/*` biar cuma
admin yang bisa akses.

## Menyambungkan tool "Video Clipper"

Proses potong video (download + ffmpeg) **tidak cocok jalan langsung di
Vercel serverless function** karena ada limit durasi eksekusi. Arsitektur
yang disaranin:

```
Next.js (Vercel)  ->  submit job
      |
      v
Queue (Upstash QStash, gratis tier)
      |
      v
Worker terpisah (Railway / Render / Fly.io) -> yt-dlp + ffmpeg
      |
      v
Storage hasil clip (Cloudflare R2 / Supabase Storage)
```

Alur singkat:
1. User submit link video dari halaman tools -> hit API route Next.js.
2. API route simpan job ke database (status: "pending") & push ke queue.
3. Worker di Railway ambil job, download pakai `yt-dlp`, potong pakai
   `ffmpeg`, upload hasil ke storage.
4. Worker update status job jadi "done" + link hasil.
5. Frontend polling/realtime (Supabase Realtime) buat nampilin progress.

## Catatan legal

Cek dulu Terms of Service platform sumber (YouTube, TikTok, dst) sebelum
fitur clipper dipakai secara komersial — beberapa platform membatasi
pengunduhan konten tanpa izin.

## Tools lain yang gampang ditambahin duluan (jalan langsung di Vercel, ringan)

- Image Compressor -> pakai library `sharp` di API route
- QR Generator -> pakai `qrcode` npm package
- Text Summarizer -> panggil Anthropic API (Claude) dari API route
