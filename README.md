# Open Market Bot 📈 

Open Market Bot adalah script Cloudflare Worker otomatis yang mengambil data kalender ekonomi real-time dari Investing.com dan mengirimkannya sebagai peringatan (alert) ke channel Discord melalui Webhook. 

Bot ini dirancang khusus untuk memantau rilis data ekonomi berdampak tinggi (High/Medium Impact) pada mata uang mayor (USD, EUR, GBP, JPY, AUD) dan memberikan analisis naratif instan mengenai dampaknya terhadap pasar (terutama Dolar AS dan Emas).

## 🌟 Fitur Utama

Bot ini berjalan dalam dua mode utama secara otomatis:

* **Mode A (Agenda Harian):** * Mengirim ringkasan jadwal rilis ekonomi hari ini setiap pukul 06:00 WIB.
    * Dikelompokkan berdasarkan Sesi Trading (Asia, London, New York).
    * Menampilkan data *smart-row*: Perbandingan ekspektasi (Forecast vs Previous) sebelum rilis.
* **Mode B (Alert Rilis Live):** * Mengecek rilis data terbaru setiap 2 menit.
    * Mengirim kartu notifikasi instan saat nilai *Actual* keluar.
    * **Verdict Adaptif:** Memberikan analisis otomatis apakah rilis tersebut *Bullish/Bearish* untuk mata uang terkait dan Emas (Gold), dengan mempertimbangkan data yang terbalik (seperti *unemployment claims*).

## 🛠️ Persyaratan (Prerequisites)

1.  Akun [Cloudflare](https://dash.cloudflare.com/) (Gratis).
2.  Server Discord dan akses untuk membuat **Webhook URL**.

## 🚀 Panduan Setup & Instalasi

Ikuti langkah-langkah berikut untuk memasang bot ini di Cloudflare Workers:

### 1. Buat Worker Baru
1. Masuk ke dashboard Cloudflare > **Workers & Pages** > **Create application**.
2. Pilih **Create Worker**, beri nama `open-market-bot`, lalu klik **Deploy**.
3. Klik **Edit Code**, salin seluruh kode dari file `worker.js` di repositori ini, tempel ke editor Cloudflare, lalu klik **Save and deploy**.

### 2. Konfigurasi Environment & KV (Wajib)
Bot ini memerlukan penyimpanan memori (KV) untuk mencegah pengiriman spam/alert berulang dan Secret untuk Webhook Discord.

1. Buka menu **Workers & Pages** > **KV** dan buat namespace baru bernama `open_market_kv` (atau nama lain bebas).
2. Masuk ke pengaturan Worker `open-market-bot` Anda > tab **Settings** > **Variables**.
3. **KV Namespace Bindings:**
   * Tambahkan binding baru.
   * Variable name: `KV` *(harus persis huruf besar)*.
   * KV namespace: Pilih namespace yang baru Anda buat.
4. **Environment Variables (Secrets):**
   * Klik **Add variable** > pilih opsi **Encrypt** (untuk keamanan).
   * Variable name: `WEBHOOK`
   * Value: Masukkan URL Webhook Discord Anda.

### 3. Atur Jadwal Cron (Triggers)
Agar bot berjalan secara otomatis (polling), atur trigger cron:
1. Ke tab **Triggers** di pengaturan Worker.
2. Tambahkan **Cron Trigger** baru.
3. Masukkan ekspresi cron: `*/2 * * * *` (berjalan setiap 2 menit).

## ⚙️ Kustomisasi (Opsional)

Anda dapat mengubah beberapa variabel konfigurasi di bagian atas kode sesuai kebutuhan:

* `CURRENCIES`: Tambah/kurangi mata uang yang dipantau (Default: `["USD", "EUR", "GBP", "JPY", "AUD"]`).
* `INCLUDE_MEDIUM`: Ubah ke `false` jika hanya ingin notifikasi untuk event *High Impact*.
* `BAR_MODE`: Ubah ke `"gold"` untuk warna bar Discord berdasarkan sentimen emas, atau biarkan `"investing"` untuk mengikuti warna hijau/merah standar Investing.com.

## 🩺 Diagnosa & Testing

Anda dapat memicu endpoint Worker Anda melalui browser (menggunakan URL bawaan Worker `https://nama-worker.subdomain.workers.dev`) untuk melakukan pengetesan:

* `/?test=webhook` : Menguji apakah koneksi Webhook Discord berfungsi.
* `/?test=agenda` : Memaksa bot mengirimkan jadwal hari ini ke Discord (meskipun bukan jam 06:00).
* `/?test=rilis` : Mengirim 1 kartu rilis terbaru (atau dummy) ke Discord untuk menguji format tampilan Mode B.
* `/?debug=feed` : Mengecek status parser data dari Investing.com (melihat jumlah event terbaca).
* `/?debug=raw` : Melihat output HTML mentah jika parser mengalami masalah (berguna jika struktur web sumber berubah).

> **Catatan Endpoint Test:** Terdapat fitur *cooldown* (~90 detik) untuk endpoint testing agar Discord Anda tidak terkena spam jika halaman di-refresh berulang kali.

## ⚠️ Disclaimer

Not Financial Advice. Bot ini mengumpulkan data dan memberikan respons naratif berdasarkan logika statis. Lakukan riset Anda sendiri (DYOR) sebelum mengambil keputusan finansial atau trading.
