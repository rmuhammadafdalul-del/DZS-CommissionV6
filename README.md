# DZS Commission Bot V7.7 — Multi-Ticket / ORDER + TESTIMONI

Discord commission bot berbasis **Node.js 20 + discord.js 14 + SQLite** yang siap dijalankan di **Railway, Render Background Worker, atau Pterodactyl**.

## Yang dilanjutkan dari V7.5 / V7.6

- Multi-ticket: satu customer boleh mempunyai banyak order aktif.
- Order ID otomatis `DZS-0001`, `DZS-0002`, dst.
- Ticket private otomatis dibuat di category yang ditentukan.
- Skin: 64×64, 128×128, 256×256, 512×512.
- Claim worker → otomatis menjadi Progress.
- Status: Open → Progress → Waiting → Completed.
- Completed → customer rating 1–5 + review.
- Ticket/order dipisahkan ke **Category ORDER**.
- Feedback dipisahkan ke **Channel TESTIMONI** dan tidak pernah diposting ke channel ticket.
- `/setup-feedback` otomatis memasang panel feedback di channel TESTIMONI.
- Ticket dihapus 15 detik setelah feedback.
- Completed tanpa feedback dibersihkan otomatis setelah 24 jam.
- Claim dibuat atomic agar dua worker tidak dapat mengambil ticket yang sama secara bersamaan.
- Database path dapat dikonfigurasi melalui `DATA_DIR`.
- Graceful shutdown untuk `SIGTERM`/`SIGINT`, cocok untuk restart/deploy platform.

## Struktur

```text
.
├── src/index.js
├── data/                 # SQLite runtime
├── Dockerfile
├── render.yaml
├── railway.toml
├── .env.example
├── .dockerignore
└── package.json
```

## Environment Variables

```env
DISCORD_TOKEN=PASTE_BOT_TOKEN_HERE
GUILD_ID=PASTE_SERVER_ID_HERE
ORDER_CATEGORY_ID=PASTE_ORDER_CATEGORY_ID_HERE
TESTIMONI_CHANNEL_ID=PASTE_TESTIMONI_CHANNEL_ID_HERE
LOG_CHANNEL_ID=PASTE_LOG_CHANNEL_ID_HERE
STAFF_ROLE_ID=PASTE_STAFF_ROLE_ID_HERE
DATA_DIR=./data
```

`LOG_CHANNEL_ID` bersifat opsional untuk fitur inti; jika tidak valid, bot tetap dapat membuat ticket.

## Discord Permissions

Bot minimal membutuhkan:

- View Channels
- Send Messages
- Embed Links
- Read Message History
- Attach Files
- Manage Channels
- Manage Messages

Bot tidak membutuhkan Privileged Gateway Intents.

## Slash Commands

Staff:
- `/setup-commission skin`
- `/setup-commission render`
- `/setup-commission logo`
- `/setup-commission animasi`
- `/setup-feedback`
- `/claim-ticket`
- `/order-status`
- `/close-ticket`

Customer:
- `/myfeedback`

Umum/staff:
- `/stats`

## Railway

1. Upload/push project ke repository Git.
2. Buat service dari repository.
3. Railway akan memakai `Dockerfile`.
4. Isi Variables sesuai `.env.example`.
5. Tambahkan **Volume** ke service dengan mount path:

```text
/app/data
```

Aplikasi sudah menggunakan `/app/data` bila `DATA_DIR=/app/data`.

> Penting: SQLite adalah database lokal. Tanpa persistent volume, database dapat hilang ketika instance/deployment diganti.

`railway.toml` sudah disediakan untuk konfigurasi build/deploy dasar.

## Render

Gunakan `render.yaml` sebagai Blueprint, atau buat **Background Worker** manual menggunakan Dockerfile.

Variables yang perlu diisi:
- `DISCORD_TOKEN`
- `GUILD_ID`
- `TICKET_CATEGORY_ID`
- `FEEDBACK_CHANNEL_ID`
- `LOG_CHANNEL_ID`
- `STAFF_ROLE_ID`

`DATA_DIR` sudah diarahkan ke `/app/data`, dan `render.yaml` menyediakan persistent disk di `/app/data`.

Jangan membuatnya sebagai Web Service hanya untuk bot Discord; gunakan Background Worker.

## Pterodactyl

Gunakan image Node.js 20+.

Startup command:

```bash
npm install --omit=dev && npm start
```

Atau, setelah dependency sudah terpasang:

```bash
npm start
```

Variables:
```text
DISCORD_TOKEN=...
GUILD_ID=...
TICKET_CATEGORY_ID=...
FEEDBACK_CHANNEL_ID=...
LOG_CHANNEL_ID=...
STAFF_ROLE_ID=...
DATA_DIR=./data
```

Pastikan folder `data/` berada di filesystem server yang persistent. Jangan menjalankan dua instance bot menggunakan database SQLite yang sama.

## Instalasi lokal

```bash
npm install
npm start
```

Untuk development, salin `.env.example` menjadi `.env`, lalu isi semua ID Discord.

## Setup Discord

1. Invite bot ke server dengan permission yang diperlukan.
2. Pastikan `STAFF_ROLE_ID` adalah role staff commission.
3. Pastikan `TICKET_CATEGORY_ID` adalah Category Channel.
4. Buat channel feedback dan log, lalu masukkan ID-nya.
5. Jalankan:
   - `/setup-commission skin`
   - `/setup-commission render`
   - `/setup-commission logo`
   - `/setup-commission animasi`
6. Jalankan `/setup-feedback` bila ingin menampilkan informasi feedback.

## Catatan database

SQLite cocok untuk satu instance bot. Jangan menjalankan beberapa replica/instance yang menulis `commission.sqlite` yang sama.

Backup file:

```text
data/commission.sqlite
```

Untuk Railway/Render, backup dilakukan dari persistent volume/disk. Untuk skala multi-instance, migrasikan database ke PostgreSQL/MySQL sebelum menambah replica.

## Troubleshooting

### Bot tidak login
- Cek `DISCORD_TOKEN`.
- Pastikan token adalah Bot Token, bukan client secret.

### Command tidak muncul
- Isi `GUILD_ID` dengan Server ID untuk registrasi command langsung ke server.
- Pastikan bot sudah diundang dengan scope `bot` dan `applications.commands`.

### Ticket gagal dibuat
- Cek `ORDER_CATEGORY_ID`.
- Pastikan ID tersebut adalah Category Channel ORDER.
- Pastikan bot memiliki `Manage Channels`.
- Pastikan role staff dapat melihat category/ticket.

### Database reset setelah redeploy
- Railway: attach Volume di `/app/data`.
- Render: attach Persistent Disk di `/app/data`.
- Pterodactyl: pastikan `./data` persistent.

## Versi

`7.7.0`

## Animasi — 3 Kasta
`/setup-commission animasi` sekarang menampilkan 3 pilihan produk:
- **Basic Style — 2k/Sec** — max 30 detik, custom map/model, Basic Lighting.
- **Medium Style — 7k/Sec** — durasi 1 menit, max 4 karakter, Medium Lighting.
- **Hight Style — 10k/Sec** — support skin 256x/512x, Full Lighting.

Fighting Style Animation hanya berlaku pada scene bertarung dan hanya untuk kasta **Medium** dan **Hight**.
