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

**Tahap 2: HTTP Request and Cookie Analysis**

_Intercepted request di Burp Suite_
> GET /DVWA/vulnerabilities/brute/?username=admin&password=test&Login=Login HTTP/1.1
>
> Host: 192.168.1.10
>
> Referer: http://192.168.1.10/DVWA/vulnerabilities/brute/
>
> Cookie: security=low; PHPSESSID=857da***dfe44

Dari intercepted request yang terdapat di Burp Suite pada percobaan pertama, hal yang perlu diperhatikan dan dicatat adalah **Cookie ID**, sedangkan username dan password boleh saja belum sesuai. Cookie atau Session ID ini berfungsi untuk authentication token saat mengoperasikan Hydra dan string valuenya harus sesuai, sebab tanpanya permintaan ke vulnerable page justru akan diarahkan ke laman login utama DVWA, bukan ke laman login Brute Force.

**Tahap 3: Exploitation via Hydra**

> hydra -l admin \
>
>   -P /usr/share/wordlists/simplified_rockyou.txt \
>
>   192.168.1.10 \
>
>   http-get-form \
>
>   "/DVWA/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie: PHPSESSID=857da***dfe44; security=low:F=Username and/or password incorrect."

**Penjelasan:**
5 baris bash command yang digunakan dengan tool Hydra dimaksudkan untuk mengotomatisasi serangan terhadap login page secara bertubi-tubi sebanyak ditemukkannya pasangan kredensial (username & password) yang sesuai. Parameter yang perlu diperhatikan pada command ini antara lain: 'admin' sebagai username, path file teks (.txt) yang berisikan daftar password yang akan diuji, IPv4 address target, path DVWA brute, HTTP GET, dan Session ID. Parameter pendukung antara lain: keterangan jika pengujian gagal berupa "username and/or password incorrect." Perlu diketahui pula jika value username bukan "admin", maka pengujian gagal, dengan keterangan sebagaimana disebutkan sebelumnya.


**Tahap 4: Verification**

> [80][http-get-form] host: 192.168.1.10   login: admin   password: password

Pengujian berhasil diindikasikan dengan pemberitahuan pasangan kredensial (username & password) dengan teks bertinta warna cerah, dalam kasus ini adalah "login: admin" dan "password: password."

# Recommendation

Mitigasi serangan ini dapat ditempuh dengan beberapa mekanisme berikut:

## A. Code-Level Recommendation (Application / PHP)

1. Implementasi account lockout setelah beberapa percobaan login gagal, misalkan: setelah 5x percobaan.
2. Menerapkan server-side response delay apapun outcome yang muncul.
3. Memigrasi credential submission dari HTTP GET ke HTTP POST, menghapus kredensial dari URL, riwayat browser, dan server access logs.
4. Mengganti string-concatenated SQL queries with parameterized queries, contoh: mysqli prepare() atau bind_param(), as defense-in-depth alongside input escaping, closing related injection surface in the same code path.
5. Memasang CAPTCHA, reCAPTCHA v3 atau hCAPTCHA, setelah sekian kali percobaan gagal.
6. Memasang multi-factor authentication (MFA) terhadap akun administrator dan berbagai privileged accounts lainnya. 

**B. Configuration and Infrastructure-Level Recommendation**

1. Deploy rate-limiting rules at reverse proxy or Web App firewall, contoh: ModSecurity dengan OWASP Core Rule Set, atau cloud WAF, yang ditargetkan khusus kepada authentication endpoint dan independent of any application-level fix.
2. Deploy log-based intrusion detection mechanism, contoh: fail2ban, agar secara otomatis memblokir alamat IPv4 sumber yang terindikasi melakukan pola brute-force pada network layer.
3. Centralize (memusatkan) authentication event logging and mengintegrasikannya dengan SIEM untuk mendeteksi percobaan login yang aneh dalam periode waktu singkat, daripada bergantung pada preventive control.
4. Enforce and periodically audit a strong password policy untuk semua akun, with particular emphasis on administrative credentials.

# References
[CWE-307: Improper Restriction of Excessive Authentication Attempts](https://cwe.mitre.org/data/definitions/307.html)
