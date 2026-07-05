Write-Up: Forensics PCAP - Cookie Hunter

Deskripsi Soal
Diberikan file the_data_challenge.pcapng. Temukan flag yang tersembunyi di dalamnya.

Tools
- Wireshark
- Command line: tshark, strings

Langkah-langkah

1. Buka file the_data_challenge.pcapng menggunakan Wireshark.

2. Di Wireshark, gunakan filter: http
   Tampak sekitar 8 koneksi TCP berisi request HTTP.

3. Telusuri satu per satu HTTP request yang muncul.

4. Pada request GET /api/flag terdapat header:
   Authorization: Bearer CTF_ITFAIR{buk4n_b3nd3r4_y4ng_k4mu_c4r1_h3h3}
   Ini terlihat seperti flag, tapi sebenarnya palsu (red herring).

5. Pada request POST /api/verify terdapat body JSON:
   {"token":"InN1YiI6ImFkbWluIiwiZmxhZyI6IkNURl9JVEZBSVJ7ZmFrZV9iIg==","admin":true}
   Nilai token berupa base64. Jika didecode menghasilkan:
   "sub":"admin","flag":"CTF_ITFAIR{fake_b"
   Ini juga merupakan red herring, flag tidak lengkap.

6. Lanjutkan ke packet berikutnya. Pada request GET /dashboard terdapat header:
   Cookie: session=abc123; auth=CTF_ITFAIR{w1r3sh4rk_d4t4_4n4lyz3r}
   Flag berada dalam plaintext pada field auth di Cookie header.

   Flag: CTF_ITFAIR{w1r3sh4rk_d4t4_4n4lyz3r}

Cara Alternatif via CLI

Menggunakan tshark:
tshark -r the_data_challenge.pcapng -Y "http" -T fields -e http.cookie

Menggunakan strings:
strings the_data_challenge.pcapng | grep CTF_ITFAIR

Kesimpulan
Flag tersembunyi di header Cookie pada request GET /dashboard. Tidak perlu decode apapun karena flag sudah dalam bentuk plaintext. Tantangan hanya terletak pada menemukan packet yang tepat di antara noise dan red herring.
