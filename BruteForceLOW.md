# Executive Summary
Pengujian penetration testing terhadap laman login pada target  ditemukan bahwa web aplikasi memungkinkan percobaan memasukkan kredensial sebanyak-banyaknya tanpa throttling, temporary lockout, atau deteksi anomali. Dengan menggunakan freely available automated tool, pasangan kredensial administrator yang valid dapat ditemukan hanya dalam kurang dari 2 detik.

Dengan kata lain, meskipun web aplikasi DVWA pada bagian Brute Force ini memiliki mekanisme penguncian pada laman utamanya tetapi tidak ada upaya untuk mencegah atau mengeliminasi serangan oleh penyerang. Jika mekanisme pelindungan seperti ini diterapkan di produksi nyata, penyerang dapat menebak secara sistematis dan mudah atas kredensial yang buruk. Konsekuensi dari serangan ini adalah penyerang dapat memperluas perolehan kontrol penuh atas web aplikasi, baik berupa pengamatan (viewing), pengubahan (modifying), atau penghapusan (deleting) data, menonaktifkan (disabling) kontrol keamanan, atau beralih (pivoting) ke sistem-sistem lain yang saling berkaitan.

Kerentanan ini masuk pada kategori salah satu penyebab utama account takeover dan credential-stuffing di dunia industri keamanan siber, khususnya karena serangan tidak membutuhkan eksploitasi yang kompleks, setidaknya hanya memerlukan tool sederhana. Pada kasus ini, serangan diklasifikasikan pada High-severity, dan rekomendasi pencegahan segera berupa antara lain: rate limiting, account lookout, and multi-factor authentication.

**Catatan**: Pengujian ini dilakukan pada infrastruktur dan jaringan lokal, yaitu DVWA sebagai lab uji yang terisolasi, dan hanya ditujukan sebagai upaya pembelajaran. Selain itu, reporting ini dapat digunakan untuk pembuatan portfolio secara profesional.

# Issue Identified

# Risk Breakdown
| Component              | Value                                          |
|------------------------|------------------------------------------------|
|Vulnerability Name      |Authentication Brute Force                      |
|CWE Reference           |CWE-307                                         |
|Severity Rating         |High                                            |
|Affected Component      |DVWA Brute Force (/DVWA/vulnerabilities/brute/) |
|Affected Parameter      |username, password (transmitted via HTTP GET)   |
|Authentication Required |none (pre-auth attack surface)                  |

# Steps to Reproduce
Pengujian berikut yang terkemas sebagai bagian dari Proof of Concept terbagi menjadi 4 tahap.

**Tahap 1: Reconnaissance and Traffic Interception**
Pengujian ini termasuk pada White Box testing sehingga tidak perlu melakukan reconnaissance lebih jauh. Di samping itu, pengujian ini mengidealkan setidaknya satu sistem operasi, sebagai host atau guest. Namun, untuk mencegah dampak pengujian yang tidak diinginkan maka disarankan menggunakan guest OS di dalam VMware Workstation.

Pengujian yang dilakukan ini menggunakan dua (2) sistem operasi, yaitu Parrot OS sebagai host dan Kali Linux sebagai guest yang terisolasi di dalam VMware Workstation. Kedua OS ini terhubung ke dalam satu jaringan fisik yang sama, yaitu 192.168.1.0/24, yang mana host dengan IPv4 address 192.168.1.10/24 dan guest dengan IPv4 address 192.168.1.12/24 dengan konfigurasi jaringan berupa bridge network adapter. Kemudian,, di dalam guest, telah dipasang dan dikonfigurasi Damn Vulnerable Web Application (DVWA) sebagai environment assessment.

Tool yang digunakan pada host hanyalah Burp Suite Community Edition, sedangkan tools yanng digunakan pada guest antara lain: browser (Firefox), FoxyProxy, Hydra, Wordlist, dan curl.
Berikut informasi tools dan kegunaannya pada pengujian ini.
- Burp Suite = bertindak untuk HTTP traffic interception dan request analysis
- browser Firefox = sebagai media menjalankan DVWA
- FoxyProxy = browser-level proxy routing di dalam guest Kali Linux
- Hydra = brute force engine atau automated credential-stuffing
- Wordlist = candidate password set
- curl = low-level manual request verification and diagnostics

Visualisasi tahap 1 dapat dilihat pada gambar berikut:
![Alt text](https://github.com/troders34/DVWA-Reporting/Images/ fdf)

**Tahap 2: HTTP Request and Cookie Analysis**
**Tahap 3: Exploitation via Hydra**
**Tahap 4: Verification**

# Affected Demographic

# Recommendation

# Timeline

# References
