# Web Form Request Surat Paklaring PT SSI & JST

Pengganti Google Form dengan tampilan kustom. Data masuk ke Google Sheets, file upload masuk ke Google Drive, melalui Google Apps Script (GAS).

## Isi folder
- `index.html` : web form (satu file, tanpa build).
- `Code.gs` : backend Google Apps Script.

## Alur & logika kondisional
1. **Data diri**: nama, NPP, NIK (16 digit), email, tempat/tanggal lahir.
2. **Pekerjaan**: jabatan terakhir (dropdown, ada opsi "Yang lain"), lokasi, dan riwayat kerja (dropdown).
   - `SSI` : hanya tanggal masuk & keluar SSI.
   - `SSI & JST` : tanggal masuk & keluar SSI dan JST.
   - Pilihan riwayat hanya `SSI` dan `SSI & JST` (opsi JST saja sudah dihapus).
3. **Serah terima aset & resign** (satu halaman):
   - Aset `Sudah` maka kolom upload berkas aset muncul (wajib). `Belum` maka kolom tidak muncul dan tidak dikirim.
   - Persetujuan resign, upload nota intern/surat resign (PDF, maks 10 MB), dan kebutuhan paklaring (dropdown, ada opsi "Yang lain").
4. **Exit interview 1/2** : 5 pertanyaan skala 1-5 (wajib).
5. **Exit interview 2/2** : 5 pertanyaan skala 1-5 dan kritik/saran (semua wajib).

Input pada bagian yang tersembunyi otomatis dinonaktifkan, sehingga tidak divalidasi dan tidak ikut terkirim.

## Cara deploy
### 1. Siapkan Sheet dan folder
1. Buat Google Spreadsheet baru (mis. "Paklaring 2026").
2. Folder upload: tidak perlu dibuat manual. Semua berkas (nota resign dan serah terima aset) otomatis masuk ke **satu folder** bernama `Upload Paklaring SSI & JST` di Drive anda (nama bisa diubah di `CONFIG.FOLDER_NAME`). Jika ingin memakai folder yang sudah ada, isi `CONFIG.FOLDER_ID` dengan ID dari URL (`drive.google.com/drive/folders/<ID_FOLDER>`).

### 2. Pasang GAS
1. Di Spreadsheet buka **Extensions > Apps Script**.
2. Hapus isi default, tempel seluruh isi `Code.gs`.
3. `FOLDER_ID` dan `SHEET_ID` boleh dikosongkan. Disarankan menjalankan fungsi `setupFolder` sekali dari menu **Run** agar folder terbuat dan izin Drive disetujui. URL folder tampil di Execution log.
4. Klik **Deploy > New deployment > Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Klik **Deploy**, setujui izin Drive dan Sheets (klik *Advanced > Go to project* bila ada peringatan), lalu salin **Web app URL** (berakhiran `/exec`).

### 3. Hubungkan web form
Buka `index.html`, ganti baris:

```js
const GAS_URL='PASTE_URL_WEB_APP_GAS_DI_SINI';
```

dengan URL Web App tadi. Sebelum diisi, form berjalan dalam mode demo (hanya log di console).

### 4. Hosting
Unggah `index.html` ke hosting statis apa pun (GitHub Pages, Netlify, Cloudflare Pages, atau server internal). Tidak butuh server tambahan.

## Catatan penting
- Setiap kali `Code.gs` diubah, buat **New version** pada Manage deployments agar perubahan aktif. URL tetap sama.
- Form mengirim `Content-Type: text/plain` agar lolos CORS tanpa preflight. Jangan diubah ke `application/json`.
- File dikirim sebagai base64 (ukuran naik sekitar 33%). Batas 10 MB per file diperiksa di browser dan server (`CONFIG.MAX_MB`).
- Upload serah terima aset dibuat **wajib** bila memilih "Sudah". Untuk membuatnya opsional, hapus atribut `required` pada input `serah_terima_aset` di `index.html` dan syarat terkait di `Code.gs`.
- Nilai yang diawali `= + - @` diberi tanda `'` agar tidak dieksekusi sebagai formula di Sheet.
- Karena Web App berakses "Anyone", siapa pun yang tahu URL dapat mengirim data. Untuk pembatasan lebih ketat, tambahkan token rahasia sederhana pada payload dan cek di `doPost`.
- Folder Drive berisi dokumen sensitif (NIK, nota). Bagikan hanya kepada tim HCC.

## Tampilan
Warna mengikuti tema Google Form asli (coklat-emas `#91692a`, latar krem). Untuk mengganti warna, ubah variabel `--brass`, `--brassd`, dan `--bg` di bagian `:root` pada `index.html`. Foto header tertanam di tag `<img class="hero">`.

## Catatan perubahan kolom
Kolom "No. referensi" sudah dihapus dari Sheet. Jika Sheet `Respon` sudah terlanjur dibuat dengan header lama, hapus sheet tersebut (atau kolom B) agar header dibuat ulang otomatis saat respon berikutnya masuk.

## Tata letak (versi terbaru)
Tampilan mengikuti artifact referensi: banner foto di atas, kartu pratinjau yang terisi otomatis saat form diisi (di HP kartu berada di bawah form), stepper 5 langkah, progres "X dari Y wajib terisi", kolom floating label berikon dengan centang, pesan error per kolom, upload dengan pratinjau dan tombol Hapus, serta tombol Kosongkan formulir. Isi form dan logika kondisional tidak berubah. Warna memakai tema coklat-emas lama; ubah variabel `--green`, `--green-d`, `--bg`, `--field` di `:root` untuk mengganti.
