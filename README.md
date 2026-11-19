# Marbel AI

Marbel AI adalah aplikasi chat AI berbasis web (dan Android) dengan proxy multi-model gratis, pembuatan gambar, serta mode ilustrasi video. Antarmuka menggunakan Bahasa Indonesia dan dirancang ringan untuk dijalankan di server sendiri, layanan hosting gratis, GitHub Pages, atau sebagai APK Android.

## Fitur

- **Chat AI multi-model gratis** — memilih model otomatis (paling cepat merespons) atau model tertentu.
- **Failover upstream** — proxy mencoba beberapa provider berurutan jika salah satu gagal/rate-limit.
- **Mode langsung (tanpa backend)** — di GitHub Pages dan aplikasi Android (Capacitor), klien dapat memanggil provider yang mendukung CORS langsung dari browser/WebView.
- **Buat gambar** — generate gambar dari prompt teks (via API a0.dev, tanpa API key).
- **Buat ilustrasi video** — menghasilkan gambar dari prompt lalu menampilkannya dengan animasi Ken Burns (bukan render video MP4 dari provider gratis).
- **Riwayat percakapan** — beberapa thread percakapan dalam satu sesi.
- **Tema terang/gelap** — toggle manual, mengikuti preferensi sistem saat pertama dibuka.
- **Tampilan responsif** — sidebar dapat disembunyikan, nyaman di desktop maupun ponsel.
- **Salin & ulangi jawaban** — aksi cepat pada pesan asisten.

## Model & Provider

Model yang dikonfigurasi di UI/proxy:

| ID Model (UI) | Keterangan |
| --- | --- |
| `qwen3.8-27b` | Default; dipetakan ke model yang tersedia di upstream aktif |
| `gpt-oss-20b` | Alternatif |
| `qwen3-8b` | Alternatif |

Upstream default (OpenAI-compatible, dari daftar no-cost-ai, tanpa API key bawaan):

1. `https://hermes.ai.unturf.com` (uncloseai)
2. `https://qwen.ai.unturf.com` (uncloseai)
3. `https://text.pollinations.ai` (Pollinations, endpoint `/openai`)
4. `https://api.free.ai` (cadangan)

> Catatan: provider gratis dapat berubah, lambat, atau terkena rate-limit. Server melakukan pemetaan model per-upstream dan failover agar permintaan tetap bisa diproses bila memungkinkan.

## Struktur Proyek

```text
.
├── index.html            # UI utama
├── app.js                # Logika frontend (chat, media, thread, failover langsung)
├── styles.css            # Gaya/tampilan
├── server.js             # Server statis + proxy /api/chat dan /api/models
├── package.json
├── capacitor.config.json # Konfigurasi aplikasi Android (Capacitor)
├── android/              # Proyek Android native
├── build-apk.sh          # Build APK debug/release
├── build-icon.py         # Generator ikon launcher
├── start-servers.sh      # Jalankan server + watchdog (Linux)
├── Dockerfile
├── railway.json
├── render.yaml           # Render Blueprint
├── MarbelAI.apk          # APK yang sudah dibangun (bila tersedia)
└── LICENSE
```

## Menjalankan Secara Lokal

### Prasyarat

- Node.js **>= 18**

### Jalankan

```bash
npm install
npm start
# atau
node server.js
```

Buka: `http://localhost:12000`

Server akan menyajikan UI statis dan endpoint proxy:

- `POST /api/chat` — proxy chat completions (OpenAI-compatible) ke upstream
- `GET /api/models` — daftar model yang diizinkan

> Dependensi di `package.json` terutama untuk Capacitor/Android. Untuk menjalankan web + proxy saja, `node server.js` tidak memerlukan paket npm tambahan.

## Variabel Lingkungan

| Variabel | Default | Fungsi |
| --- | --- | --- |
| `PORT` | `12000` | Port server |
| `UPSTREAM` | daftar 4 upstream default | Daftar endpoint upstream, dipisah koma (dicoba berurutan) |
| `UPSTREAM_PREFIX` | kosong | Prefix path upstream (mis. `/zen`) |
| `DEFAULT_MODEL` | `qwen3.8-27b` | Model default jika klien tidak mengirim model |
| `MODELS_LIST` | `qwen3.8-27b,gpt-oss-20b,qwen3-8b` | Model yang ditampilkan/diizinkan lewat `/api/models` |
| `API_KEY` | kosong | Token Bearer opsional untuk upstream yang membutuhkannya |
| `ALLOW_ORIGIN` | `*` | Nilai header CORS `Access-Control-Allow-Origin` |
| `USE_SSE` | `1` | Izinkan mode streaming jika diminta klien |
| `FORCE_NO_STREAM` | `1` | Paksa `stream:false` secara default untuk keandalan |
| `MODELS_TTL` | `300` | Cache `/api/models` (detik) |
| `DEBUG_LOG` | `0` | Log request dasar (`1` untuk aktif) |

## Deployment

### Render (Blueprint)

Repositori ini menyertakan [`render.yaml`](render.yaml). Buat **New Blueprint** di Render dan hubungkan repo ini; service web Node akan dibuat dengan env default untuk upstream/model gratis.

### Railway

[`railway.json`](railway.json) mengatur build Nixpacks dan start command `node server.js`. Set variabel lingkungan sesuai kebutuhan (mis. `PORT`, `UPSTREAM`, `MODELS_LIST`).

### Docker

```bash
docker build -t marbel-ai .
docker run --rm -p 10000:10000 marbel-ai
```

`Dockerfile` mengekspos port `10000` dan mengatur env default untuk upstream/model.

## Build APK Android (Capacitor)

Aplikasi Android membungkus UI web dengan Capacitor. Pada mode native, aplikasi dapat memanggil provider langsung; bila perlu dapat memakai proxy cadangan yang dikonfigurasi di `app.js`.

### Prasyarat

- JDK 17+
- Android SDK (platform/build-tools sesuai proyek Android)
- Node.js 18+
- `ANDROID_HOME` (atau `ANDROID_SDK_ROOT`) ter-set
- Python 3 (untuk `build-icon.py`)

### Perintah

```bash
# Linux/macOS (bash)
./build-apk.sh           # debug
./build-apk.sh release   # release
```

Output umum:

- Debug: `android/app/build/outputs/apk/debug/app-debug.apk`
- Release: `android/app/build/outputs/apk/release/app-release.apk`

Build release akan memakai `android/marbel-release.keystore` jika tersedia; jika tidak, dapat jatuh ke debug signing (tergantung konfigurasi proyek Android).

## Mode GitHub Pages / Frontend Statis

Jika UI dibuka dari host statis (mis. `*.github.io`) atau sebagai aplikasi Capacitor tanpa override, `app.js` mencoba mode langsung ke provider yang mendukung CORS. Untuk memaksa UI memakai backend/proxy tertentu:

```text
https://<host-frontend>/?frontend=https://<host-backend-anda>
```

## Keterbatasan

- **Provider gratis**: ketersediaan, kecepatan, dan kuota bergantung pihak ketiga.
- **Video**: mode "Video" saat ini menghasilkan **ilustrasi beranimasi** (gambar + efek Ken Burns), bukan file video dari layanan text-to-video gratis.
- **Konteks/riwayat**: riwayat disimpan di memori browser sesi berjalan (bukan database persisten).
- **Keamanan API**: jangan mengekspos API key sensitif di frontend publik; gunakan `API_KEY`/proxy server-side bila diperlukan.

## Lisensi

Proyek ini dilisensikan di bawah [MIT License](LICENSE).
