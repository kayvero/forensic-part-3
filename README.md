# Cheatsheet Forensik CTF — Browser, Audio, Stego, Timeline

Referensi tools & command untuk 4 kategori: **Browser Forensics**, **Audio Forensic**, **Steganografi**, dan **Timeline Analysis**. Format tiap tool: penjelasan, penerapan umum, penerapan spesifik, tabel opsi/flag. Tool yang butuh instalasi manual (GUI/aplikasi) ditandai dengan link download di bagian **Catatan Instalasi**.

---

# 1. Browser Forensics

### `sqlite3` (CLI)
**Penjelasan:** Browser Chromium/Firefox menyimpan history, cookies, download, dan login form dalam database SQLite — `sqlite3` dipakai query langsung dari command line tanpa GUI.

**Umum:**
```bash
sqlite3 History "SELECT url, title, last_visit_time FROM urls;"
```
**Spesifik:**
```bash
sqlite3 Cookies "SELECT host_key, name, encrypted_value FROM cookies WHERE host_key LIKE '%target.com%';"
```
| File Target (Chrome) | Isi |
|---|---|
| `History` | Riwayat kunjungan URL, waktu akses |
| `Cookies` | Cookie tersimpan (value biasanya terenkripsi) |
| `Login Data` | Kredensial tersimpan (terenkripsi via DPAPI/OS keychain) |
| `Web Data` | Autofill form, kartu kredit tersimpan |

---

### `DB Browser for SQLite` *(aplikasi GUI)*
**Penjelasan:** GUI untuk buka & browse file database SQLite (History, Cookies, Web Data) tanpa perlu hafal query — bisa filter, sort, dan export tabel langsung.

**Cara pakai:**
1. Download & install (lihat Catatan Instalasi).
2. `Open Database` → pilih file `History`/`Cookies`/`Web Data` dari profile browser target.
3. Tab **Browse Data** → pilih tabel (misal `urls`, `downloads`, `logins`) → lihat/filter isinya.
4. Tab **Execute SQL** kalau mau query custom (join antar tabel, filter waktu tertentu).

| Fitur | Keterangan |
|---|---|
| Browse Data | Lihat isi tabel tanpa nulis query |
| Execute SQL | Jalankan query custom |
| Export | Simpan hasil ke CSV |

---

### `Hindsight`
**Penjelasan:** Tool khusus parsing forensik browser Chromium (History, Cookies, Cache, Local Storage, Extensions) menjadi satu laporan timeline terstruktur — lebih cepat daripada query manual satu-satu.

**Umum:**
```bash
hindsight.py -i "Default" -o output_report
```
**Spesifik:**
```bash
hindsight.py -i "Default" -o output_report -f xlsx   # laporan dalam format Excel
```
| Opsi | Keterangan |
|---|---|
| `-i` | Path folder profile browser (biasanya folder `Default`) |
| `-o` | Nama file/report output |
| `-f` | Format output (xlsx, json, sqlite) |

---

### `BrowsingHistoryView` *(aplikasi GUI — NirSoft)*
**Penjelasan:** GUI ringan dari NirSoft untuk membaca riwayat browser (Chrome, Firefox, Edge, IE) sekaligus dari satu tampilan tabel, tanpa perlu tahu lokasi/struktur database masing-masing browser.

**Cara pakai:**
1. Download & jalankan (tidak perlu install, portable `.exe`).
2. Tool otomatis load history dari semua browser yang terdeteksi di sistem, atau pilih **Advanced Options** untuk load dari folder profile custom (misal hasil ekstraksi disk image).
3. Gunakan filter tanggal/URL di toolbar untuk mempersempit hasil.

| Fitur | Keterangan |
|---|---|
| Advanced Options | Load history dari path custom (bukan sistem aktif) |
| Filter | Cari berdasarkan URL, judul, tanggal |
| Export | Simpan ke CSV/HTML |

---

### `ESEDatabaseView` *(aplikasi GUI — NirSoft)*
**Penjelasan:** Internet Explorer/Edge (legacy) menyimpan cache & history dalam format ESE database (`WebCacheV01.dat`), bukan SQLite — tool ini dipakai untuk membaca format tersebut.

**Cara pakai:**
1. Download & jalankan.
2. `File > Open Database` → pilih `WebCacheV01.dat`.
3. Browse tabel `Container_1`, `Container_2`, dst — masing-masing berisi record history/cache berbeda.

| Fitur | Keterangan |
|---|---|
| Open Database | Load file `.dat` ESE format |
| Table selector | Pilih container/tabel yang mau dilihat |

---

### `firepwd.py`
**Penjelasan:** Script Python untuk decrypt password tersimpan di Firefox (`key4.db` + `logins.json`) tanpa perlu master password (kecuali di-set manual oleh user).

**Umum:**
```bash
python3 firepwd.py -d /path/to/firefox/profile
```
**Spesifik:**
```bash
python3 firepwd.py -d ./profile_extracted/ > decrypted_creds.txt
```
| Argumen | Keterangan |
|---|---|
| `-d` | Path folder profile Firefox (berisi `key4.db`, `logins.json`) |

---

### `ChromeCacheView` / `MZCacheView` *(aplikasi GUI — NirSoft)*
**Penjelasan:** Membaca isi cache browser (gambar, script, file yang pernah diakses/didownload sementara) — `ChromeCacheView` untuk Chrome, `MZCacheView` untuk Firefox.

**Cara pakai:**
1. Download & jalankan.
2. Tool otomatis scan folder cache default, atau **File > Load from folder** untuk load cache hasil ekstraksi disk image.
3. Klik entry untuk preview isi cache (gambar/file) langsung, atau **Copy Selected Files To...** untuk extract.

| Fitur | Keterangan |
|---|---|
| Load from folder | Load cache dari path custom |
| Preview | Lihat isi file cache langsung di aplikasi |
| Copy Selected Files | Extract file cache ke folder lain |

---

### `Autopsy` (Web Artifacts module)
**Penjelasan:** Autopsy punya ingest module otomatis yang parsing web history, cookies, bookmark, dan download dari disk image tanpa perlu tahu path profile browser manual.

**Cara pakai:**
- Add Data Source → centang ingest module **Recent Activity** → hasil muncul di tab **Web History**, **Web Cookies**, **Web Downloads** pada tree kiri.

| Kategori hasil | Keterangan |
|---|---|
| Web History | URL & waktu kunjungan |
| Web Cookies | Cookie tersimpan per domain |
| Web Downloads | Riwayat file yang didownload |

---

# 2. Audio Forensic

### `Audacity` *(aplikasi GUI)*
**Penjelasan:** Editor audio — visualisasi waveform & spectrogram, tempat paling umum menemukan pesan tersembunyi (SSTV, morse, teks di spectrogram).

**Cara pakai:**
1. Download & install (lihat Catatan Instalasi).
2. `File > Import > Audio` → buka file target.
3. Klik dropdown nama track → pilih **Spectrogram** untuk ganti tampilan dari waveform ke spectrogram (di sinilah teks/gambar tersembunyi biasanya terlihat).
4. Kalau spectrogram kurang jelas: klik kanan track → **Spectrogram Settings** → perbesar **Window Size** (misal 2048/4096) untuk resolusi lebih tajam.

| Fitur | Keterangan |
|---|---|
| Spectrogram view | Frekuensi vs waktu (untuk pesan visual tersembunyi) |
| Effect > Reverse | Cek pesan yang disembunyikan terbalik |
| Change Speed/Pitch | Ungkap pesan yang dipercepat/diperlambat |
| Effect > Invert | Cek fase audio yang dibalik |

---

### `Sonic Visualiser` *(aplikasi GUI)*
**Penjelasan:** Alternatif `Audacity` yang lebih fokus ke analisis spectrogram mendalam (resolusi lebih tinggi, banyak color map), sering dipakai kalau spectrogram Audacity kurang detail.

**Cara pakai:**
1. Download & install.
2. `File > Open` → pilih file audio.
3. `Layer > Add Spectrogram` → pilih pane baru untuk tampilan spectrogram.
4. Klik kanan pane → atur **Colour**, **Scale**, dan **Window Size** untuk memperjelas pola tersembunyi.

| Fitur | Keterangan |
|---|---|
| Add Spectrogram | Tambah layer analisis frekuensi |
| Colour 3D Plot | Mode visual alternatif untuk detail halus |

---

### `sox`
**Penjelasan:** CLI audio processing — convert format, analisis, dan generate spectrogram tanpa GUI, cocok untuk automasi/scripting.

**Umum:**
```bash
sox audio.wav -n spectrogram
```
**Spesifik:**
```bash
sox audio.wav -n spectrogram -o output.png -x 3000 -y 1025
```
| Opsi | Keterangan |
|---|---|
| `-n` | Null output (mode analisis) |
| `-o` | Output file spectrogram |
| `-x` / `-y` | Resolusi lebar/tinggi spectrogram |

---

### `ffmpeg`
**Penjelasan:** Convert format audio, extract audio dari video, dan cek info stream — sering dipakai sebagai langkah persiapan sebelum analisis lebih lanjut.

**Umum:**
```bash
ffmpeg -i video.mp4 audio_extracted.wav
```
**Spesifik:**
```bash
ffmpeg -i audio.mp3 -acodec pcm_s16le -ar 44100 audio.wav   # convert ke WAV uncompressed
```
| Opsi | Keterangan |
|---|---|
| `-i` | Input file |
| `-acodec` | Codec audio output |
| `-ar` | Sample rate output |

---

### `mediainfo`
**Penjelasan:** Cek metadata teknis file audio/video (codec, bitrate, duration, tag) — kadang flag disisipkan di metadata custom.

**Umum:**
```bash
mediainfo audio.wav
```
**Spesifik:**
```bash
mediainfo --Full audio.wav | grep -i comment
```
| Opsi | Keterangan |
|---|---|
| `--Full` | Tampilkan semua field metadata |

---

### `steghide` (audio)
**Penjelasan:** Embed/extract data tersembunyi pada file WAV/AU dengan optional passphrase — sama seperti pada gambar.

**Umum:**
```bash
steghide extract -sf audio.wav
```
**Spesifik:**
```bash
steghide extract -sf audio.wav -p "password"
```
| Opsi | Keterangan |
|---|---|
| `extract` | Mode ekstraksi |
| `-sf` | Stego file sumber |
| `-p` | Passphrase |

---

### `WavSteg` (stegolsb)
**Penjelasan:** Steganografi LSB khusus format WAV — embed/extract data dari least significant bit sample audio, mirip `zsteg` tapi untuk audio.

**Umum:**
```bash
stegolsb wavsteg -r -i audio.wav -o extracted_data --lsb-count 2
```
| Opsi | Keterangan |
|---|---|
| `-r` | Mode extract (reveal) |
| `-i` | File input |
| `-o` | File output hasil ekstraksi |
| `--lsb-count` | Jumlah bit LSB yang dipakai (coba 1-4) |

---

### `DeepSound` *(aplikasi GUI, Windows)*
**Penjelasan:** Tool embed/extract file tersembunyi di dalam audio (WAV/FLAC) dengan enkripsi opsional — sering muncul di CTF ketika file WAV punya "kapasitas tersembunyi" mencurigakan.

**Cara pakai:**
1. Download & install (Windows only, lihat Catatan Instalasi).
2. Buka aplikasi → `Open carrier file` → pilih file audio target.
3. Kalau ada file tersembunyi, akan muncul di list **Secret files** → klik **Extract Secret Files** dan masukkan password kalau diminta.

| Fitur | Keterangan |
|---|---|
| Open carrier file | Load file audio yang dicurigai |
| Secret files list | Tampilkan file tersembunyi yang terdeteksi |
| Extract | Simpan file tersembunyi ke disk |

---

### `multimon-ng` (DTMF & Morse)
**Penjelasan:** Decode sinyal audio seperti DTMF (nada telepon) dan Morse code menjadi teks/angka.

**Umum:**
```bash
multimon-ng -t wav -a DTMF audio.wav
```
**Spesifik:**
```bash
multimon-ng -t wav -a MORSE_CW audio.wav
```
| Opsi | Keterangan |
|---|---|
| `-t` | Tipe input file (wav) |
| `-a` | Modul decoder (DTMF, MORSE_CW, dll) |

---

### SSTV Decoder — `RX-SSTV` / `QSSTV` / `Robot36` *(aplikasi GUI)*
**Penjelasan:** Decode sinyal audio Slow-Scan Television menjadi gambar — file WAV yang jika diputar terdengar seperti nada aneh sering merupakan sinyal SSTV.

**Cara pakai:**
1. Download `RX-SSTV` (Windows) atau `QSSTV` (Linux), atau pakai app `Robot36` (Android/web).
2. Kalau pakai desktop: putar file WAV lewat virtual audio cable yang di-route ke input decoder, atau `File > Open WAV` kalau aplikasi mendukung load langsung dari file.
3. Pilih mode SSTV (coba **Robot 36** dulu — paling umum di CTF; kalau gagal coba **Scottie 1/2** atau **Martin 1**).
4. Tunggu proses decode selesai, gambar akan muncul progresif dari atas ke bawah.

| Mode SSTV umum | Keterangan |
|---|---|
| Robot 36 | Paling umum dipakai di CTF |
| Scottie 1/2 | Alternatif jika Robot36 gagal |
| Martin 1 | Alternatif lain, resolusi lebih tinggi tapi lebih lambat |

---

# 3. Steganografi

### `steghide`
**Penjelasan:** Embed/extract data tersembunyi pada JPEG/BMP/WAV/AU dengan optional passphrase.

**Umum:**
```bash
steghide extract -sf file.jpg
```
**Spesifik:**
```bash
steghide extract -sf file.jpg -p "password"
steghide info file.jpg
```
| Opsi | Keterangan |
|---|---|
| `extract` | Mode ekstraksi |
| `-sf` | Stego file sumber |
| `-p` | Passphrase |
| `info` | Cek info embedded data |

---

### `zsteg`
**Penjelasan:** Deteksi steganografi LSB pada PNG/BMP — coba berbagai kombinasi bit-plane secara otomatis.

**Umum:**
```bash
zsteg image.png
```
**Spesifik:**
```bash
zsteg -a image.png
zsteg -E "b1,r,lsb,xy" image.png
```
| Opsi | Keterangan |
|---|---|
| `-a` | Coba semua kombinasi metode |
| `-E` | Extract payload dari channel tertentu |
| `-v` | Verbose |

---

### `stegsolve` *(aplikasi GUI, Java)*
**Penjelasan:** Analisis visual gambar — geser antar bit-plane, color channel, dan filter untuk temukan pesan tersembunyi secara visual.

**Cara pakai:**
1. Download `stegsolve.jar` (lihat Catatan Instalasi), pastikan Java sudah terinstall.
2. Jalankan: `java -jar stegsolve.jar`.
3. `File > Open` → pilih gambar.
4. Gunakan tombol panah kiri/kanan di bawah untuk berganti Data/Plane (Red 0, Green 0, Blue 0, dst) — perhatikan pola aneh yang muncul di salah satu plane.
5. Cek juga menu **Analyse > File Format** dan **Analyse > Frame Browser** (untuk GIF multi-frame).

| Fitur | Keterangan |
|---|---|
| Plane switching | Lihat tiap bit-plane per channel warna |
| Frame Browser | Untuk gambar dengan banyak frame (GIF) |
| Stereogram Solver | Untuk gambar stereogram |
| Image Combiner | Gabungkan 2 gambar (XOR, dsb) untuk cari perbedaan |

---

### `stegoveritas`
**Penjelasan:** Tool otomatis yang menjalankan berbagai teknik steganografi sekaligus (LSB, metadata, frame extraction) dalam satu perintah.

**Umum:**
```bash
stegoveritas image.png
```
**Spesifik:**
```bash
stegoveritas -steg -meta -extractLSB image.png
```
| Opsi | Keterangan |
|---|---|
| `-steg` | Modul deteksi steganografi |
| `-meta` | Ekstrak metadata |
| `-extractLSB` | Ekstrak bit-plane LSB per channel |

---

### `outguess`
**Penjelasan:** Steganografi JPEG berbasis statistical redundancy — alternatif saat `steghide` gagal membuka file JPEG.

**Umum:**
```bash
outguess -r image.jpg output.txt
```
| Opsi | Keterangan |
|---|---|
| `-r` | Mode retrieve/extract |
| `-k` | Passphrase (jika ada) |

---

### `stegcracker`
**Penjelasan:** Bruteforce passphrase `steghide` menggunakan wordlist.

**Umum:**
```bash
stegcracker image.jpg wordlist.txt
```
| Argumen | Keterangan |
|---|---|
| `image.jpg` | File target |
| `wordlist.txt` | Daftar kata bruteforce |

---

### `openstego` *(aplikasi GUI/CLI)*
**Penjelasan:** Tool embed & extract data pada gambar, mendukung watermarking maupun data hiding biasa.

**Cara pakai (GUI):**
1. Download & install (butuh Java).
2. Pilih tab **Extract Data** → `Input Stego File` pilih gambar target → `Output Folder` untuk hasil ekstraksi → klik **Extract Data**.

**CLI:**
```bash
openstego extract -sf image.png -xf output.txt
```
| Opsi | Keterangan |
|---|---|
| `extract` | Mode ekstraksi |
| `-sf` | Stego file |
| `-xf` | File hasil ekstraksi |
| `-p` | Password (jika dipakai saat embed) |

---

### `StegExpose`
**Penjelasan:** Tool deteksi (bukan ekstraksi) steganografi LSB pada gambar — memberi skor kemungkinan sebuah gambar mengandung data tersembunyi, berguna untuk triase banyak file sekaligus.

**Umum:**
```bash
java -jar StegExpose.jar image.png
```
**Spesifik:**
```bash
java -jar StegExpose.jar ./folder_gambar/ --threshold 0.2   # scan satu folder sekaligus
```
| Opsi | Keterangan |
|---|---|
| `--threshold` | Ambang batas skor deteksi |

---

### `jsteg`
**Penjelasan:** Steganografi khusus format JPEG berbasis DCT coefficient — salah satu tool klasik JPEG stego, kadang dipakai sebagai variasi dari `outguess`/`steghide`.

**Umum:**
```bash
jsteg reveal stego.jpg output.txt
```
| Perintah | Keterangan |
|---|---|
| `reveal` | Mode ekstraksi pesan tersembunyi |
| `hide` | Mode embed (jika butuh reproduksi tantangan) |

---

### `SilentEye` *(aplikasi GUI)*
**Penjelasan:** Tool stego cross-platform dengan GUI sederhana, mendukung gambar & audio, sering dipakai sebagai alternatif visual dari `steghide`/`openstego`.

**Cara pakai:**
1. Download & install (lihat Catatan Instalasi).
2. Buka aplikasi → drag & drop file target (gambar/audio) ke jendela utama.
3. Klik tombol **Decode** → masukkan password jika diminta → hasil ekstraksi akan ditampilkan/disimpan.

| Fitur | Keterangan |
|---|---|
| Decode | Ekstrak pesan tersembunyi |
| Encode | Embed pesan (untuk reproduksi/latihan) |
| Plugin | Mendukung banyak algoritma stego via plugin |

---

### `wbStego` *(aplikasi GUI, Windows)*
**Penjelasan:** Tool stego lama tapi masih relevan di beberapa CTF — mendukung embed/extract di gambar BMP, PDF, dan teks.

**Cara pakai:**
1. Download & install (Windows only).
2. Pilih mode **Extract** → pilih file carrier → masukkan password jika ada → tentukan lokasi file output.

| Fitur | Keterangan |
|---|---|
| Extract | Mode ekstraksi data tersembunyi |
| Format support | BMP, PDF, TXT sebagai carrier |

---

# 4. Timeline Analysis

### `log2timeline` / `plaso`
**Penjelasan:** Ekstrak seluruh event timestamp dari berbagai sumber (filesystem, registry, evtx, browser history) menjadi satu file timeline terpadu.

**Umum:**
```bash
log2timeline.py output.plaso disk.img
```
**Spesifik:**
```bash
log2timeline.py --parsers "win7" output.plaso disk.img
```
| Opsi | Keterangan |
|---|---|
| `--parsers` | Pilih parser spesifik (mempercepat proses) |
| `-z` | Set timezone sumber data |

---

### `psort`
**Penjelasan:** Filter, sortir, dan konversi hasil `.plaso` dari `log2timeline` menjadi format yang mudah dibaca (CSV, l2tcsv).

**Umum:**
```bash
psort.py -o l2tcsv -w timeline.csv output.plaso
```
**Spesifik:**
```bash
psort.py -o l2tcsv -w timeline.csv output.plaso "date > '2026-01-01' AND date < '2026-01-02'"
```
| Opsi | Keterangan |
|---|---|
| `-o` | Format output (l2tcsv, json, dll) |
| `-w` | Path file output |
| filter query | Batasi rentang waktu/kriteria tertentu |

---

### `Timesketch` *(aplikasi web, self-hosted)*
**Penjelasan:** Platform web untuk analisis timeline kolaboratif — import hasil `.plaso`, lalu eksplorasi lewat UI dengan search, filter, dan anotasi event, jauh lebih enak daripada scroll CSV besar.

**Cara pakai:**
1. Deploy via Docker (lihat Catatan Instalasi) — Timesketch butuh server, biasanya dijalankan lewat `docker-compose`.
2. Login ke web UI (default `http://localhost`) → buat **New Sketch**.
3. Upload file `.plaso` atau `.csv` hasil `log2timeline`/`psort` sebagai timeline baru.
4. Gunakan search bar (mendukung syntax mirip Elasticsearch) untuk filter event, dan fitur **Star**/**Label** untuk menandai temuan penting.

| Fitur | Keterangan |
|---|---|
| Sketch | Satu "workspace" investigasi berisi satu/lebih timeline |
| Search | Cari event dengan query (field:value) |
| Label/Star | Tandai event penting untuk laporan |

---

### `mactime` (The Sleuth Kit)
**Penjelasan:** Buat timeline berbasis MACB (Modified, Accessed, Changed, Birth) dari body file hasil `fls`.

**Umum:**
```bash
fls -r -m / -o <offset> disk.img > body.txt
mactime -b body.txt -d > timeline.csv
```
| Opsi | Keterangan |
|---|---|
| `-b` | Path body file |
| `-d` | Output format CSV |
| `-y` | Format tanggal ISO 8601 |

---

### `Autopsy` (Timeline module)
**Penjelasan:** Modul di dalam Autopsy yang otomatis menggabungkan seluruh timestamp artefak (file, web history, registry) jadi tampilan timeline visual interaktif.

**Cara pakai:**
- Buka case di Autopsy → tunggu ingest module selesai → buka tab **Timeline** di toolbar atas.
- Pakai **Bar Chart View** untuk lihat ringkasan aktivitas per periode, lalu zoom ke rentang waktu mencurigakan → **Detail View** untuk lihat event satu-satu.

| Fitur | Keterangan |
|---|---|
| Bar Chart View | Ringkasan aktivitas per periode |
| Detail View | List event lengkap dengan filter tanggal/tipe |
| Filter | Batasi tipe event (file system, web, registry, dll) |

---

### Kombinasi manual (`MFTECmd` + `EvtxECmd` + `csvkit`)
**Penjelasan:** Kalau tidak pakai `plaso`, timeline Windows bisa dibangun manual dari gabungan output beberapa tool Eric Zimmerman, lalu disortir berdasarkan kolom timestamp.

**Spesifik:**
```bash
csvstack mft_output.csv evtx_output.csv | csvsort -c Timestamp > full_timeline.csv
```
| Tool | Keterangan |
|---|---|
| `csvstack` | Gabungkan beberapa file CSV |
| `csvsort` | Sortir CSV berdasarkan kolom tertentu |

---

## Catatan Instalasi (Aplikasi GUI)

| Aplikasi | Kategori | Platform | Sumber Download |
|---|---|---|---|
| DB Browser for SQLite | Browser | Windows/macOS/Linux | sqlitebrowser.org |
| BrowsingHistoryView | Browser | Windows | nirsoft.net |
| ESEDatabaseView | Browser | Windows | nirsoft.net |
| ChromeCacheView / MZCacheView | Browser | Windows | nirsoft.net |
| Audacity | Audio | Windows/macOS/Linux | audacityteam.org |
| Sonic Visualiser | Audio | Windows/macOS/Linux | sonicvisualiser.org |
| DeepSound | Audio | Windows | jpinsoft.net (cari "DeepSound") |
| RX-SSTV / QSSTV | Audio | Windows (RX-SSTV) / Linux (QSSTV) | cari "RX-SSTV" / repo QSSTV (`apt install qsstv`) |
| StegSolve | Stego | Semua (butuh Java) | cari "StegSolve.jar" (Caesum) |
| OpenStego | Stego | Semua (butuh Java) | openstego.com |
| StegExpose | Stego | Semua (butuh Java) | GitHub `b3dk7/StegExpose` |
| SilentEye | Stego | Windows/macOS/Linux | silenteye.v1kings.io |
| wbStego | Stego | Windows | cari "wbStego" (arsip developer lama) |
| Timesketch | Timeline | Server (Docker) | GitHub `google/timesketch` |

**Catatan:** Untuk tool NirSoft & aplikasi lama (DeepSound, wbStego), selalu verifikasi hash/sumber resmi sebelum download karena banyak mirror tidak resmi beredar. Kalau butuh link exact per aplikasi (versi terbaru, atau alternatif untuk OS tertentu), bilang saja nanti saya carikan.

## Catatan Tambahan
- Untuk browser: cek dulu apakah profile masih di lokasi default (`%LOCALAPPDATA%\Google\Chrome\User Data\Default` untuk Chrome, `%APPDATA%\Mozilla\Firefox\Profiles\` untuk Firefox) sebelum jalankan tool.
- Untuk audio: selalu cek spectrogram dulu sebelum coba steganografi payload — banyak challenge CTF audio yang "pesannya" justru visual, bukan data biner tersembunyi.
- Untuk stego: urutan cepat triase — `zsteg -a` (PNG/BMP) atau `steghide info` (JPEG) → kalau nihil, baru masuk `stegsolve` untuk analisis visual manual.
- Untuk timeline: kalau data sumber sedikit (cuma beberapa artefak spesifik), kombinasi manual EZ Tools + `csvsort` lebih cepat daripada setup `plaso`/`Timesketch` penuh.
