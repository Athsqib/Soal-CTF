# Write-Up: Track My Steps

## Deskripsi Soal

Diberikan dua buah file forensik:

- `History` — basis data SQLite riwayat browser Chrome milik suspect
- `capture.pcap` — file packet capture jaringan

**Clue:** "Tersangka sempat mencari cara mengakses server rahasia dan meninggalkan jejak penting di riwayat pencariannya."

**File:**

- [History](../History)
- [capture.pcap](../capture.pcap)

## Langkah Penyelesaian

### 1. Analisis File History

Buka file `History` menggunakan DB Browser for SQLite atau tool SQLite lainnya.

### 2. Eksplorasi Tabel urls

Cari tabel yang berisi riwayat URL. Di Chrome, tabel tersebut bernama `urls`.

Query:

```sql
SELECT id, url, title, visit_count FROM urls;
```

### 3. Temukan Entri Mencurigakan

Dari hasil query, terlihat 12 entri riwayat browsing. Beberapa entri yang mencurigakan:

| id  | url                                                                   | title                                               |
| --- | --------------------------------------------------------------------- | --------------------------------------------------- |
| 5   | `https://www.google.com/search?q=internal+server+access+guide`        | internal server access guide - Google Search        |
| 6   | `https://www.google.com/search?q=how+to+access+internal+admin+portal` | how to access internal admin portal - Google Search |
| 8   | `http://internal-admin-server.local/login?token=...`                  | Internal Admin Server - Login                       |

Entri nomor 8 menunjukkan tersangka mengakses server internal dengan token.

### 4. Ekstrak Token dari History

Token pada URL entri ke-8:

```
http://internal-admin-server.local/login?token=aHR0cHM6Ly93d3cudHJhbnNmZXJub3cubmV0L2RsLzIwMjYwNzA1VG1lWWhoZ2o=
```

Ambil nilai parameter `token`:

```
aHR0cHM6Ly93d3cudHJhbnNmZXJub3cubmV0L2RsLzIwMjYwNzA1VG1lWWhoZ2o=
```

### 5. Decode Base64

String tersebut adalah base64. Dekode menggunakan command line atau tool online:

```bash
echo "aHR0cHM6Ly93d3cudHJhbnNmZXJub3cubmV0L2RsLzIwMjYwNzA1VG1lWWhoZ2o=" | base64 -d
```

Hasil:

```
https://www.transfernow.net/dl/20260705TmeYhhgj
```

### 6. Download file dari link

### 7. Analisis File PCAP

Flag juga dapat ditemukan dengan menganalisis file `capture.pcap` menggunakan Wireshark atau tshark:

1. Buka `capture.pcap` di Wireshark
2. Filter dengan `http` untuk melihat paket HTTP
3. Temukan paket nomor 3 — HTTP GET request ke:
   ```
   GET /login?token=Q1RGX0lURkFJUntjM2tfaDFzdDByMV9tdV90NGt1dF9rM3Q0dTRufQ==
   ```
4. Ekstrak nilai token dan decode base64 untuk mendapatkan flag

Atau dengan tshark:

```bash
tshark -r capture.pcap -Y "http.request" -T fields -e http.request.full_uri
```

## Flag

```
CTF_ITFAIR{c3k_h1st0r1_mu_t4kut_k3t4u4n}
```
