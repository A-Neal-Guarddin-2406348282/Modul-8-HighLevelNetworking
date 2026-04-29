# gRPC Tutorial

Project ini isinya implementasi gRPC pakai Rust untuk 3 service:
- Payment
- Transaction
- Chat

Kode utamanya ada di:
- [src/grpc_server.rs](src/grpc_server.rs)
- [src/grpc_client.rs](src/grpc_client.rs)
- [proto/services.proto](proto/services.proto)

---

## Cara jalanin

```bash
cargo run --bin grpc-server
cargo run --bin grpc-client
```

---

## Penjelasan singkat

- `PaymentService` pakai unary RPC
- `TransactionService` pakai server streaming
- `ChatService` pakai bi-directional streaming

---

## Refleksi

### 1. Bedanya unary, server streaming, dan bi-directional streaming

- **Unary**: satu request, satu response. Cocok buat hal simpel seperti bayar sekali.
- **Server streaming**: satu request, banyak response dari server. Cocok buat data yang keluar bertahap, misalnya histori transaksi.
- **Bi-directional streaming**: client dan server sama-sama bisa kirim pesan terus-menerus. Cocok buat chat.

Singkatnya:
- unary = simpel
- server streaming = server yang aktif ngirim banyak data
- bi-directional = dua arah, real-time

---

### 2. Keamanan kalau bikin gRPC di Rust

Yang perlu diperhatikan:
- **Authentication**: pastikan user beneran siapa dia, misalnya pakai token.
- **Authorization**: pastikan user cuma bisa akses data yang memang boleh dia buka.
- **Encryption**: pakai TLS biar data nggak gampang disadap.
- **Validasi input**: jangan langsung percaya data dari client.

Kalau untuk data sensitif seperti pembayaran, sebaiknya tambah:
- logging
- audit trail
- rate limiting

---

### 3. Tantangan bi-directional streaming di Rust gRPC

Di [src/grpc_server.rs](src/grpc_server.rs), bagian [`MyChatService`](src/grpc_server.rs) lumayan sensitif karena harus handle pesan dua arah.

Masalah yang biasanya muncul:
- koneksi putus tiba-tiba
- urutan pesan bisa kacau
- error handling jadi lebih ribet
- banyak client bisa bikin resource cepat habis

Kalau dipakai untuk chat beneran, perlu:
- manajemen koneksi yang rapi
- timeout
- cleanup saat client disconnect
- handling error yang lebih aman

---

### 4. Kelebihan dan kekurangan `tokio_stream::wrappers::ReceiverStream`

**Kelebihan:**
- gampang dipakai buat ubah channel jadi stream
- cocok buat gRPC streaming
- enak dipadukan dengan `tokio`

**Kekurangan:**
- alur debug jadi agak lebih susah
- kalau channel penuh, bisa kena backpressure
- kalau handling salah, stream bisa berhenti diam-diam

Di project ini dipakai di [src/grpc_server.rs](src/grpc_server.rs) dan [src/grpc_client.rs](src/grpc_client.rs).

---

### 5. Biar code lebih reusable dan modular

Kalau mau lebih rapi, logic service jangan ditaruh semua di satu file.

Lebih enak kalau dipisah jadi:
- file untuk proto
- file untuk service payment
- file untuk service transaction
- file untuk service chat
- file shared untuk helper/utility

Contohnya:
- [src/grpc_server.rs](src/grpc_server.rs)
- [src/grpc_client.rs](src/grpc_client.rs)

Kalau project makin besar, sebaiknya tambah `lib.rs` supaya code bisa dipakai ulang.

---

### 6. Kalau payment logic makin kompleks, perlu apa lagi?

Kalau cuma demo, `PaymentService` di [src/grpc_server.rs](src/grpc_server.rs) masih terlalu sederhana.

Biar lebih realistis, perlu:
- database
- validasi nominal
- status pembayaran
- retry kalau gagal
- idempotency supaya transaksi dobel nggak kejadian
- integrasi ke payment gateway

---

### 7. Dampak gRPC ke arsitektur distributed system

gRPC cocok buat komunikasi antar service karena:
- cepat
- schema-nya jelas
- support banyak bahasa
- cocok buat microservices

Tapi ada minusnya juga:
- lebih susah dibaca manual daripada JSON
- browser support nggak sepraktis REST
- debugging butuh tools tambahan

Di project ini, definisi contract-nya ada di [proto/services.proto](proto/services.proto).

---

### 8. HTTP/2 gRPC vs HTTP/1.1 REST vs WebSocket

**REST**
- enak buat API publik
- gampang dites
- tapi kurang cocok buat real-time

**gRPC**
- lebih cepat
- cocok buat internal service
- bagus buat streaming

**WebSocket**
- cocok buat chat atau live update
- koneksi selalu terbuka
- tapi biasanya lebih manual pengelolaannya

Kalau lihat project ini:
- payment → unary gRPC
- transaction → server streaming
- chat → bi-directional streaming

---

### 9. Bedanya request-response REST dengan bi-directional streaming

REST biasanya:
- client kirim request
- server balas sekali
- kalau mau update terus, client harus polling

Sedangkan bi-directional streaming:
- client dan server bisa saling kirim pesan kapan saja
- cocok buat chat real-time

Di [src/grpc_server.rs](src/grpc_server.rs), bagian chat memang dibikin model seperti ini.

---

### 10. Protocol Buffers vs JSON

**Protocol Buffers**
- lebih kecil
- lebih cepat
- ada struktur yang jelas
- cocok buat service internal

**JSON**
- lebih gampang dibaca
- lebih fleksibel
- enak buat debugging manual

Di project ini, schema gRPC ada di [proto/services.proto](proto/services.proto), jadi semua message dan service sudah terdefinisi dari awal.

---

## Catatan kecil

- Payment di project ini masih dummy.
- Transaction service cuma simulasi data transaksi.
- Chat service masih sederhana, belum ada storage atau user session beneran.

---

## Kesimpulan

Project ini nunjukin cara pakai gRPC di Rust untuk 3 pola komunikasi:
