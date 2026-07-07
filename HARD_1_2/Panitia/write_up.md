# Write-Up: Double Locked Room

## Kategori

Forensics — Memory & Disk Analysis

## Tingkat Kesulitan

Hard

## Flag

```
CTF_ITFAIR{81tl0ck3r_v3r4cr1pt_4rc0mb0_f41l3d}
```

## Red Herrings (Fake Flag)

Dua fake flag disisipkan sebagai **plaintext** di dalam memory dump untuk menjebak pemain yang terburu-buru:

```
CTF_ITFAIR{p4ssw0rd_b1tl0ck3r_4d4_d1_s1n1}
CTF_ITFAIR{lsass_4d4l4h_kunc1_d4r1_m1m1n}
```

Kedua fake flag ini bisa ditemukan dengan perintah `strings` sederhana. Pemain yang berhenti di sini akan gagal karena flag sesungguhnya hanya bisa diakses melalui dekripsi dua lapis.

Dua decoy key juga disisipkan untuk menjebak pemain yang mencari `FVEK` secara asal:

```
Key: aabb0011aabb0011aabb0011aabb0011
```

## Langkah 1: Ekstrak FVEK dari Memory Dump

Pemain menganalisis `system_ram.raw` untuk mencari *BitLocker Full Volume Encryption Key* (FVEK).

### Opsi A: Strings + Grep (Cepat)

```bash
strings system_ram.raw | grep FVEK
```

Output akan menampilkan:
```
FVEK: 1122334455667788aabbccddeeff0011
```

### Opsi B: Volatility 3 (Realistis)

```bash
vol -f system_ram.raw windows.bitlocker
```

Plugin `windows.bitlocker` pada Volatility 3 akan memindai memori dan menemukan struktur FVEK. Output menunjukkan *FVEK* (*Full Volume Encryption Key*) yang diperlukan.

### Opsi C: Hex Editor

Cari string `FVEK:` di hex editor (HxD / 010 Editor) pada offset `0x400`.

---

## Langkah 2: Buka Lapisan BitLocker

Dengan FVEK yang didapat (`1122334455667788aabbccddeeff0011`), pemain mendekripsi `secret_drive.img`.

### Opsi A: dislocker (Linux - Realistis)

```bash
sudo mkdir /mnt/bitlocker /mnt/secret_drive
sudo dislocker -r -V secret_drive.img -u 1122334455667788aabbccddeeff0011 -- /mnt/bitlocker
sudo mount -o loop /mnt/bitlocker/dislocker-file /mnt/secret_drive
cd /mnt/secret_drive
ls -la
cat README.txt
```

### Opsi B: Python Script

```bash
python solver.py .
```

Solver akan otomatis mengekstrak FVEK dari `system_ram.raw` dan mendekripsi `secret_drive.img`.

---

Setelah lapisan pertama terbuka, pemain menemukan dua file:
- **README.txt** — *"Hmm, masih ada yang ganjil... sepertinya dia pakai lebih dari satu lapisan."*
- **secret.tc** — File container VeraCrypt terenkripsi

---

## Langkah 3: Ekstrak Password VeraCrypt dari Memory

Pemain kembali ke memory dump untuk mencari password VeraCrypt.

### Opsi A: Strings + Grep

```bash
strings system_ram.raw | grep -E "(VCPW|Password|veracrypt)"
```

Output:
```
VCPW: V3r4Cr1ptP@ss2024!
```

### Opsi B: Volatility + VeraCrypt Plugin

```bash
vol -f system_ram.raw windows.veracrypt
```

Plugin VeraCrypt pada Volatility 3 dapat mendeteksi password VeraCrypt yang masih tertinggal di RAM.

### Opsi C: Hex Editor

Cari string `VCPW:` atau `Password:` di hex editor pada offset `0x800`.

---

## Langkah 4: Buka Lapisan VeraCrypt

Dengan password `V3r4Cr1ptP@ss2024!`, pemain mendekripsi `secret.tc`.

### Opsi A: veraCrypt CLI (Linux)

```bash
sudo veracrypt -t -k "" --pim=0 --protect-hidden=no -m ro secret.tc /mnt/secret_drive
# Masukkan password: V3r4Cr1ptP@ss2024!
cat /mnt/secret_drive/flag.txt
```

### Opsi B: Python

```bash
python solver.py .
```

Solver akan otomatis mencari password VeraCrypt, mendekripsi `secret.tc`, dan menampilkan flag.

### Opsi C: Manual Python

```python
import hashlib

password = "V3r4Cr1ptP@ss2024!"
key = hashlib.sha256(password.encode()).digest()

with open("secret.tc", "rb") as f:
    data = f.read()

flag = bytes(data[i] ^ key[i % len(key)] for i in range(len(data)))
print(flag.rstrip(b"\x00").decode())
```

---

## Hasil Akhir

```
CTF_ITFAIR{81tl0ck3r_v3r4cr1pt_4rc0mb0_f41l3d}
```

---

## Ringkasan Tools

| Tahap | Tool | Fungsi |
|-------|------|--------|
| Ekstrak FVEK | `strings`, `volatility windows.bitlocker` | Cari kunci BitLocker di RAM |
| Decrypt BitLocker | `dislocker`, Python XOR | Buka secret_drive.img |
| Ekstrak VeraCrypt PW | `strings`, `volatility windows.veracrypt` | Cari password VeraCrypt di RAM |
| Decrypt VeraCrypt | `veracrypt`, Python XOR | Buka secret.tc dan baca flag |

## Catatan Organizer

### A. Simulasi (Cepat, untuk develop/testing)

Gunakan `generate_challenge.py` — menghasilkan `secret_drive.img` + `system_ram.raw`.

```bash
python generate_challenge.py   # regenerasi file challenge
python solver.py .              # solve otomatis
```

### B. Real VM (Untuk kompetisi sesungguhnya)

1. Buat VHD 100MB di Windows VM, format NTFS, aktifkan BitLocker
2. Buat container VeraCrypt ~50MB di dalam drive BitLocker, mount, letakkan `flag.txt`
3. Unmount container VeraCrypt (proses VeraCrypt tetap berjalan)
4. Capture memory dump: DumpIt.exe / FTK Imager / VMware `.vmem`
5. Copy file `.vmem` + VHD (rename ke `.img`) sebagai challenge
6. Verifikasi: Volatility dapat mendeteksi FVEK dan password VeraCrypt

### File Challenge

| File | Kegunaan |
|------|----------|
| `secret_drive.img` | Disk image BitLocker (layer 1) |
| `system_ram.raw` | Memory dump berisi FVEK + password VeraCrypt |
| `generate_challenge.py` | Generator cepat (tanpa VM) |
| `solver.py` | Solver otomatis |
| `readme.md` | Deskripsi challenge untuk pemain |
| `Panitia/write_up.md` | Write-up ini |
| `Panitia/struktur.md` | Blueprint challenge |
