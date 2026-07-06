# Soal 2: The Leaky Ping (ICMP Data Exfiltration) — Puzzle Variant

## Kategori
Forensics — Network Traffic Analysis

## Tingkat Kesulitan
Medium

## Deskripsi
Setelah mengungkap akses tersangka ke server internal pada investigasi sebelumnya, tim keamanan menemukan aktivitas mencurigakan lainnya. Puluhan paket ICMP (Ping) dikirim secara sporadis dari komputer tersangka ke server luar. Tidak seperti ping biasa, paket-paket ini memiliki pola yang tidak wajar.

Diberikan file `leaky_ping.pcap`. Tersangka mengirimkan potongan gambar dan potongan flag secara bersamaan melalui paket ICMP. Rekonstruksi kedua potongan puzzle ini untuk mendapatkan flag!

## Clue / Hint
"Mantan karyawan mengirimkan data keluar gedung, tapi sistem keamanan hanya mendeteksi aktivitas Ping biasa ke server luar. Ada yang berupa teks, ada yang berupa potongan file — semuanya tercampur."

## Flag
```
CTF_ITFAIR{g4mb4r_p0t0ng4n}
```
Flag dapat diperoleh dari dua cara: merangkai potongan teks flag, atau merekonstruksi gambar.

## Write-Up

### Tools yang Dibutuhkan
- **Wireshark / tshark** — untuk ekstraksi data dari pcap
- **Python + Scapy** (opsional) — alternatif ekstraksi
- **PowerShell** (opsional) — untuk solve.ps1

### Analisis Pendahuluan
File pcap berisi 46 paket yang terdiri dari 21 paket ICMP dan 25 paket TCP/HTTP (sebagai red herring / noise tambahan agar terlihat seperti lalu lintas normal).

**21 paket ICMP** terbagi menjadi:

| Tipe | Jumlah | Ciri |
|------|--------|------|
| **Image chunk** | 5 paket | Binary, payload > 100 bytes |
| **Flag piece** | 6 paket | Text pendek (printable ASCII) |
| **Noise** | 10 paket | Random bytes (non-printable) |

**25 paket TCP/HTTP** — percakapan web normal, tidak mengandung flag.

### Langkah 1: Filter ICMP (abaikan TCP)
```
icmp
```
Dengan filter ICMP, paket TCP/HTTP otomatis terfilter dan hanya tersisa 21 paket ICMP.

Paket image chunk dan flag piece sudah diacak (shuffle). Gunakan **sequence number (icmp.seq)** untuk mengurutkannya kembali.

### Langkah 2: Ekstrak Semua Payload
```bash
tshark -r leaky_ping.pcap -Y "icmp" -T fields -e data.data -e icmp.seq
```

Flag pieces yang terlihat langsung dari output tshark:
```
seq=10  4354465f49  → "CTF_I"
seq=20  5446414952  → "TFAIR"
seq=30  7b67346d62 → "{g4mb"
seq=40  34725f7030 → "4r_p0"
seq=50  74306e6734 → "t0ng4"
seq=60  6e7d       → "n}"
```

### Langkah 3: Filter & Sortir

**Pendekatan 1 — Scapy (Python):**
```python
from scapy.all import *

pkt = rdpcap("leaky_ping.pcap")

img = []
flag = []

for p in pkt:
    if ICMP in p and p[ICMP].type == 8 and Raw in p:
        load = bytes(p[Raw].load)
        seq = p[ICMP].seq
        if len(load) > 100:
            img.append((seq, load))
        elif all(32 <= b < 127 for b in load):
            flag.append((seq, load.decode()))

img.sort(key=lambda x: x[0])
with open("reconstructed.png", "wb") as f:
    f.write(b"".join(c[1] for c in img))

flag.sort(key=lambda x: x[0])
print("Flag:", "".join(p[1] for p in flag))
```

**Pendekatan 2 — solve.ps1 (PowerShell, tanpa Python):**
```powershell
# Jalankan: powershell -ExecutionPolicy Bypass -File solve.ps1
$tshark = "C:\Program Files\Wireshark\tshark.exe"
$pcap = "leaky_ping.pcap"

$dump = & $tshark -r $pcap -Y "icmp" -T fields -e data.data -e icmp.seq
$chunks = $dump | ForEach-Object {
    $parts = $_ -split "`t"
    if ($parts[0].Length -gt 200) {
        [PSCustomObject]@{ Hex = $parts[0]; Seq = [int]$parts[1] }
    }
} | Sort-Object Seq

$hex = ($chunks | ForEach-Object { $_.Hex }) -join ""
$bytes = for ($i = 0; $i -lt $hex.Length; $i += 2) {
    [Convert]::ToByte($hex.Substring($i, 2), 16)
}
[IO.File]::WriteAllBytes("reconstructed.png", $bytes)
Write-Host "Flag tersimpan di reconstructed.png"
```

### Langkah 4: Buka Gambar & Baca Flag
Buka `reconstructed.png` — flag akan terlihat di dalam gambar.

## Hasil
- **Flag dari teks**: Rangkai 6 potongan flag berdasarkan urutan seq
- **Flag dari gambar**: Buka reconstructed.png
