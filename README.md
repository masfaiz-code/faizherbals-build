# Faiz Herbals Build Repo

Repo ini dipakai sebagai runner publik untuk GitHub Actions tanpa membuka source code aplikasi utama.

## Cara kerja

- Workflow di repo ini melakukan checkout ke repo source yang private.
- Build dilakukan di GitHub-hosted runner.
- Hasil build di-upload sebagai Actions artifact dan otomatis dipublish ke Releases repo ini.

## Workflow tersedia

- `.github/workflows/build-android.yml` untuk build APK release satu klik
- `.github/workflows/deploy-pages.yml` untuk deploy halaman download ke GitHub Pages
- Trigger build hanya `workflow_dispatch` agar secret tidak terekspos lewat PR publik

## Halaman download (GitHub Pages)

Landing page di `pages/` menampilkan **APK terbaru** dari GitHub Releases (bukan Actions artifact).

URL (setelah Pages aktif):

```
https://masfaiz-code.github.io/faizherbals-build/
```

### Aktifkan GitHub Pages (sekali saja)

1. Buka repo → **Settings** → **Pages**
2. **Source**: pilih **GitHub Actions**
3. Push ke `main` (atau jalankan workflow `Deploy GitHub Pages` manual)
4. Tunggu job deploy selesai, lalu buka URL di atas

Halaman fetch `GET /repos/masfaiz-code/faizherbals-build/releases/latest` di browser, lalu pakai asset `.apk` dari release. Repo harus **public** agar API + download APK bisa diakses tanpa login.

## Secrets yang wajib

Tambahkan di repo `faizherbals-build`:

- `SOURCE_REPO_SSH_KEY`: private deploy key read-only untuk checkout repo private `masfaiz-code/faizherbals-app`

## Secrets signing Android

Tambahkan agar APK release bisa signed tanpa menyimpan keystore di repo source:

- `ANDROID_KEYSTORE_BASE64`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

## Secrets Aptoide Connect

Untuk auto-publish APK ke Aptoide Connect setelah build sukses:

- `APTOIDE_API_KEY`: generate di [Aptoide Connect Developer Console](https://connect.aptoide.com) -> Settings -> API Keys. Klik `Generate`, copy key (hanya muncul sekali), simpan sebagai secret di repo ini.

### Pre-requisite di Aptoide Connect

Sebelum job `publish-aptoide` bisa jalan, app `faizherbals_toko` harus sudah didaftarkan di Aptoide Connect:

1. Login ke `connect.aptoide.com`
2. Klik `Add App` di top nav
3. Masukkan package name app (lihat `faizherbals_toko/android/app/build.gradle.kts`)
4. Lengkapi form submission untuk versi pertama (manual upload sekali)
5. Setelah app live di Aptoide, versi berikutnya akan auto-published lewat workflow ini

> Aptoide hanya menerima `.apk` (bukan `.aab`). Workflow ini sudah upload APK signed yang sama dengan yang dipublish ke GitHub Release.

## Cara menjalankan build

1. Buka tab `Actions`
2. Pilih workflow `Build Faiz Herbals APK`
3. Klik `Run workflow` dan atur input:
   - `publish_to_aptoide` (default: `true`) - centang untuk auto-publish ke Aptoide
   - `aptoide_release_mode` (default: `IMMEDIATE`) - `IMMEDIATE` = langsung review, `MANUAL` = perlu approve manual di console Aptoide
   - `aptoide_news` (opsional) - catatan rilis untuk locale `id_ID`, misal `Perbaikan bug login`
4. Workflow akan otomatis:
   - ambil source dari `masfaiz-code/faizherbals-app`
   - build branch `main`
   - build project `faizherbals_toko`
   - menghasilkan APK release signed
   - upload ke Actions artifact dan GitHub Releases
   - submit APK ke Aptoide Connect via Uploader API (kalau toggle aktif)

## Catatan keamanan

- Jangan tambahkan trigger `pull_request` ke workflow yang memakai secret.
- Workflow ini sengaja dikunci hanya untuk build `masfaiz-code/faizherbals-app` branch `main`.
- Permission `contents: write` hanya dipakai job release; job build utama tetap read-only.
- Setelah signing via secret berjalan, pindahkan `android/key.properties` dan file `.jks` keluar dari repo source.
- `Supabase anon key` masih bisa ada di app client, tapi secret AI/private API sebaiknya dipindah ke backend/proxy.
