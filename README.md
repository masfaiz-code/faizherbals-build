# Faiz Herbals Build Repo

Repo ini dipakai sebagai runner publik untuk GitHub Actions tanpa membuka source code aplikasi utama.

## Cara kerja

- Workflow di repo ini melakukan checkout ke repo source yang private.
- Build dilakukan di GitHub-hosted runner.
- Hasil build di-upload sebagai Actions artifact dan otomatis dipublish ke Releases repo ini.

## Workflow tersedia

- `.github/workflows/build-android.yml` untuk build APK release satu klik
- Trigger saat ini hanya `workflow_dispatch` agar secret tidak terekspos lewat PR publik

## Secrets yang wajib

Tambahkan di repo `faizherbals-build`:

- `SOURCE_REPO_SSH_KEY`: private deploy key read-only untuk checkout repo private `masfaiz-code/faizherbals-app`

## Secrets signing Android

Tambahkan agar APK release bisa signed tanpa menyimpan keystore di repo source:

- `ANDROID_KEYSTORE_BASE64`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

## Cara menjalankan build

1. Buka tab `Actions`
2. Pilih workflow `Build Faiz Herbals APK`
3. Klik `Run workflow`
4. Workflow akan otomatis:
   - ambil source dari `masfaiz-code/faizherbals-app`
   - build branch `main`
   - build project root `.`
   - menghasilkan APK release signed
   - upload ke Actions artifact dan GitHub Releases

## Catatan keamanan

- Jangan tambahkan trigger `pull_request` ke workflow yang memakai secret.
- Workflow ini sengaja dikunci hanya untuk build `masfaiz-code/faizherbals-app` branch `main`.
- Permission `contents: write` hanya dipakai job release; job build utama tetap read-only.
- Setelah signing via secret berjalan, pindahkan `android/key.properties` dan file `.jks` keluar dari repo source.
- `Supabase anon key` masih bisa ada di app client, tapi secret AI/private API sebaiknya dipindah ke backend/proxy.
