# Double Locked Room

**Kategori:** Forensics — Memory & Disk Analysis
**Tingkat Kesulitan:** Hard

## Deskripsi

Saat penggerebekan, tersangka mencabut paksa kabel power komputernya. Tim forensik berhasil mengamankan _hard disk_ yang berisi partisi rahasia terenkripsi, serta file _memory dump_ terakhir dari komputer tersebut. Sayangnya, partisi tersebut digembok dengan BitLocker. Namun, karena komputer dimatikan secara tiba-tiba saat partisi sedang terbuka, ada kemungkinan kuncinya masih tertinggal di dalam RAM.

Diberikan dua buah berkas dari perangkat tersangka:

1. `secret_drive.img` — Sebuah _disk image_ terenkripsi BitLocker.
2. `system_ram.raw` — Berkas _memory dump_ dari komputer tersebut.

Tugasmu adalah mengekstrak FVEK (_Full Volume Encryption Key_) yang masih menempel di dalam _memory dump_, lalu menggunakan kunci tersebut untuk membuka lapisan pertama. Di dalamnya, kamu akan menemukan petunjuk tentang lapisan enkripsi kedua. Rahasia yang disembunyikan tersangka baru bisa diakses setelah kedua lapisan berhasil ditembus.

## Clue / Hint

"Kunci rumah yang sesungguhnya bukanlah _password_ yang diketik, melainkan _FVEK_ yang disimpan Windows di dalam RAM agar CPU bisa membaca disk secara _real-time_."

## File

- `secret_drive.img` — Disk image BitLocker
- `system_ram.raw` — Memory dump

## Flag

Format: `CTF_ITFAIR{...}`
