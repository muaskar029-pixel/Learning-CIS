# Penjelasan Konseptual CIS Debian Linux 12 Benchmark

**Bagian A — Pengantar CIS Benchmark**  
**Sumber utama:** *CIS Debian Linux 12 Benchmark v2.0.0 — 05-28-2026*, terutama bagian **Overview**, **Important Usage Information**, **Key Stakeholders**, **Apply the Correct Version of a Benchmark**, **Exceptions**, **Remediation**, **Target Technology Details**, **Recommendation Definitions**, dan **Profile Definitions**.

> Dokumen ini adalah catatan pembelajaran konseptual. Isinya menjelaskan makna, tujuan, dan cara berpikir di balik CIS Debian Linux 12 Benchmark. Dokumen ini bukan salinan resmi CIS Benchmark dan bukan pengganti dokumen resmi CIS.

---

## Daftar Isi

1. [Overview](#1-overview)
2. [Important Usage Information](#2-important-usage-information)
3. [Key Stakeholders](#3-key-stakeholders)
4. [Apply the Correct Version of a Benchmark](#4-apply-the-correct-version-of-a-benchmark)
5. [Exceptions](#5-exceptions)
6. [Remediation](#6-remediation)
7. [Target Technology Details](#7-target-technology-details)
8. [Recommendation Definitions](#8-recommendation-definitions)
9. [Profile Definitions](#9-profile-definitions)
10. [Kesimpulan Besar Bagian Pengantar](#10-kesimpulan-besar-bagian-pengantar)

---

# 1. Overview

## 1.1 Apa itu CIS Benchmark?

**CIS Benchmark** adalah panduan konfigurasi keamanan yang dibuat oleh Center for Internet Security untuk membantu organisasi mengamankan teknologi tertentu. Dalam konteks ini, teknologi yang dibahas adalah **Debian Linux 12**.

CIS Benchmark tidak berfokus pada teori keamanan secara umum, tetapi pada **technical configuration settings**, yaitu pengaturan teknis yang dapat diterapkan pada sistem agar postur keamanannya lebih baik. Contohnya mencakup pengaturan filesystem, service, jaringan, SSH, privilege escalation, PAM, logging, auditing, firewall, dan permission file sistem.

Dengan kata lain, CIS Benchmark menjawab pertanyaan seperti:

- Konfigurasi apa yang sebaiknya diaktifkan?
- Konfigurasi apa yang sebaiknya dinonaktifkan?
- Service apa yang tidak perlu berjalan?
- File penting apa yang harus dibatasi aksesnya?
- Bagaimana cara menilai apakah sistem sudah sesuai baseline keamanan?
- Apa risiko jika rekomendasi tertentu diterapkan?

CIS Benchmark dapat dipahami sebagai **daftar standar konfigurasi aman** yang disusun secara sistematis, diuji, dan dibangun melalui proses konsensus komunitas ahli.

---

## 1.2 Fungsi CIS Benchmark dalam cybersecurity program

CIS Benchmark berfungsi sebagai salah satu komponen penting dalam program keamanan siber organisasi. Perannya bukan untuk menggantikan seluruh sistem keamanan, melainkan memberikan **baseline konfigurasi aman**.

Dalam program cybersecurity, CIS Benchmark membantu dalam beberapa hal berikut.

### a. Membentuk baseline konfigurasi aman

Baseline adalah titik awal standar. Artinya, CIS membantu organisasi menentukan konfigurasi minimum yang dianggap aman untuk sistem tertentu. Dengan baseline, organisasi tidak lagi mengamankan sistem berdasarkan dugaan, kebiasaan pribadi admin, atau tutorial acak dari internet.

Contoh pemikiran baseline:

- Service yang tidak diperlukan sebaiknya tidak aktif.
- Akses root harus dibatasi.
- File sensitif harus memiliki permission yang ketat.
- Logging dan auditing harus aktif.
- Mekanisme autentikasi harus diperkuat.

### b. Mengurangi attack surface

Banyak rekomendasi CIS berangkat dari prinsip **attack surface reduction**. Attack surface adalah semua titik yang dapat dimanfaatkan penyerang untuk menyerang sistem.

Semakin banyak service, modul kernel, port, akun, atau permission terbuka, semakin besar peluang serangan. CIS mendorong sistem agar hanya menjalankan komponen yang benar-benar diperlukan.

### c. Mendukung audit dan kepatuhan

CIS Benchmark menyediakan struktur audit yang jelas. Setiap rekomendasi biasanya memiliki bagian:

- Description
- Rationale
- Impact
- Audit
- Remediation
- References
- CIS Controls mapping

Struktur ini memudahkan auditor atau tim keamanan untuk menilai apakah sistem sudah sesuai dengan standar tertentu.

### d. Membantu standardisasi antar sistem

Tanpa benchmark, setiap server bisa dikonfigurasi berbeda-beda tergantung admin yang mengelola. Hal ini berbahaya karena keamanan menjadi tidak konsisten.

Dengan CIS Benchmark, organisasi dapat membuat standar yang sama untuk banyak sistem. Misalnya, semua server Debian 12 memiliki pola konfigurasi SSH, firewall, permission, logging, dan auditing yang seragam.

### e. Mendukung hardening skala besar

Pada organisasi besar, hardening tidak bisa dilakukan satu per satu secara manual tanpa standar. CIS Benchmark memberikan acuan yang dapat dikombinasikan dengan tool assessment dan automation agar pengamanan banyak server bisa dilakukan lebih efisien.

---

## 1.3 Bedanya benchmark dengan antivirus, EDR, logging, patching, dan monitoring

CIS Benchmark sering disalahpahami sebagai “alat keamanan”. Padahal CIS Benchmark adalah **panduan konfigurasi**, bukan software proteksi langsung. Untuk memahaminya, perlu dibedakan dengan komponen keamanan lain.

| Komponen | Fungsi Utama | Perbedaannya dengan CIS Benchmark |
|---|---|---|
| CIS Benchmark | Panduan konfigurasi aman | Memberi standar pengaturan sistem agar lebih aman sejak awal |
| Antivirus | Mendeteksi dan memblokir malware | Bekerja pada deteksi malware, bukan menyusun baseline konfigurasi sistem |
| EDR | Mendeteksi, menganalisis, dan merespons aktivitas mencurigakan di endpoint | Fokus pada deteksi dan respons insiden, bukan daftar konfigurasi baseline |
| Logging | Merekam aktivitas sistem dan user | CIS dapat mengatur logging, tetapi logging sendiri hanya mekanisme pencatatan |
| Patching | Memperbarui sistem untuk menutup vulnerability | CIS menekankan pentingnya patching, tetapi tidak menggantikan proses update keamanan |
| Monitoring | Memantau kondisi sistem dan aktivitas keamanan | Monitoring mendeteksi keadaan berjalan, sedangkan CIS mengatur konfigurasi awal dan berkelanjutan |

CIS Benchmark lebih dekat dengan konsep **hardening guide**. Ia membantu menjawab bagaimana sistem sebaiknya dikonfigurasi agar tidak terlalu terbuka, tidak terlalu longgar, dan tidak menjalankan komponen yang tidak diperlukan.

Antivirus dan EDR bersifat reaktif atau detektif: mereka membantu ketika ada malware atau aktivitas mencurigakan. Logging dan monitoring membantu melihat apa yang terjadi. Patching menutup celah dari software yang rentan. CIS Benchmark melengkapi semua itu dengan memastikan sistem sejak awal berada dalam kondisi konfigurasi yang lebih aman.

---

## 1.4 Kenapa CIS disebut baseline, bukan satu-satunya standar keamanan?

CIS Benchmark disebut **baseline** karena ia adalah titik awal, bukan akhir dari keamanan. Ada beberapa alasan penting.

### a. Setiap organisasi memiliki kebutuhan berbeda

Sebuah server web, server database, workstation mahasiswa, laptop developer, server Kubernetes, dan server audit memiliki kebutuhan berbeda. Tidak semua rekomendasi bisa diterapkan secara sama rata.

Contoh:

- Server web memang membutuhkan web server aktif.
- Server container mungkin membutuhkan modul kernel tertentu yang pada profil Level 2 bisa dianggap perlu dinonaktifkan.
- Workstation pengguna mungkin membutuhkan fitur GUI, automount, printer, atau Bluetooth, sementara server produksi tidak.

Karena itu, CIS memberikan rekomendasi umum yang aman, tetapi organisasi tetap harus menyesuaikan dengan fungsi sistem.

### b. CIS tidak menggantikan risk management

Keamanan bukan hanya mengikuti checklist. Organisasi tetap harus melakukan analisis risiko. Sebuah rekomendasi bisa sangat aman secara teknis, tetapi mengganggu operasional jika diterapkan tanpa konteks.

Contoh: menonaktifkan USB storage dapat mengurangi risiko malware dari flashdisk, tetapi bisa mengganggu unit kerja yang memang membutuhkan media eksternal untuk proses bisnis tertentu.

### c. CIS bukan pengganti standar lain

CIS dapat digunakan bersama standar atau framework lain, seperti NIST, ISO 27001, STIG, kebijakan internal organisasi, atau regulasi industri. CIS biasanya lebih teknis dan konfiguratif, sedangkan framework lain bisa lebih luas mencakup tata kelola, manajemen risiko, kebijakan, prosedur, dan kontrol organisasi.

### d. CIS bukan jaminan sistem kebal serangan

Sistem yang mengikuti CIS tetap bisa diserang jika:

- aplikasinya rentan,
- password bocor,
- patch tidak diperbarui,
- admin salah konfigurasi,
- akses jaringan terlalu terbuka,
- logging tidak dimonitor,
- user terkena phishing,
- atau ada celah zero-day.

Maka CIS harus dipahami sebagai **fondasi konfigurasi aman**, bukan jaminan keamanan absolut.

---

# 2. Important Usage Information

## 2.1 Cara menggunakan CIS Benchmark

CIS Benchmark dapat digunakan untuk dua aktivitas besar:

1. **Assessment** — menilai apakah sistem sudah sesuai dengan rekomendasi.
2. **Remediation** — memperbaiki konfigurasi agar sesuai dengan rekomendasi.

Secara konseptual, alurnya adalah:

```text
Pahami profil yang dipilih
        ↓
Baca rekomendasi CIS
        ↓
Lakukan assessment/audit
        ↓
Tentukan status pass/fail atau perlu review
        ↓
Analisis impact dan kebutuhan bisnis
        ↓
Lakukan remediation jika sesuai
        ↓
Dokumentasikan hasil dan exception
        ↓
Uji ulang sistem
```

Dalam praktik yang sehat, CIS Benchmark tidak langsung diterapkan secara membabi buta. Setiap rekomendasi perlu dibaca, dipahami, diuji, dan disesuaikan dengan kebutuhan sistem.

---

## 2.2 Perbedaan assessment manual dan automated

CIS membedakan rekomendasi berdasarkan **Assessment Status**, yaitu apakah pengecekannya dapat diotomatisasi atau membutuhkan validasi manual.

### Automated assessment

Rekomendasi berstatus **Automated** berarti kondisi teknisnya dapat diperiksa secara otomatis dan dapat menghasilkan status seperti pass atau fail.

Contoh karakteristik automated assessment:

- Nilai konfigurasi dapat dibaca dari file tertentu.
- Status service dapat dicek oleh tool.
- Permission file dapat diverifikasi secara programatis.
- Parameter kernel dapat diperiksa dengan perintah sistem.
- Paket tertentu dapat dicek apakah terpasang atau tidak.

Automated assessment cocok untuk skala besar karena ratusan server dapat diperiksa secara konsisten oleh tool.

### Manual assessment

Rekomendasi berstatus **Manual** berarti pengecekannya tidak dapat sepenuhnya disimpulkan oleh tool. Ada bagian yang membutuhkan penilaian manusia.

Manual assessment diperlukan ketika:

- kondisi “aman” bergantung pada kebutuhan organisasi,
- tool tidak bisa menentukan apakah service tertentu memang diperlukan,
- konfigurasi perlu dibandingkan dengan kebijakan internal,
- ada konteks bisnis yang harus dipahami,
- atau risiko harus dinilai oleh manusia.

Contoh konseptual:

Sebuah tool bisa melihat bahwa web server aktif. Namun tool tidak selalu bisa mengetahui apakah server itu memang bertugas sebagai web server. Karena itu, manusia perlu menilai konteksnya.

---

## 2.3 Kenapa poin Manual tetap penting?

Poin Manual tetap penting karena keamanan tidak selalu bisa disederhanakan menjadi hasil otomatis. Ada beberapa alasan.

### a. Tidak semua risiko dapat dibaca oleh tool

Tool bisa membaca konfigurasi, tetapi tidak selalu memahami tujuan sistem. Tool dapat mengetahui bahwa port tertentu terbuka, tetapi tidak selalu mengetahui apakah port itu sah menurut kebutuhan bisnis.

### b. Manual assessment menangkap konteks organisasi

Keamanan harus mempertimbangkan:

- fungsi server,
- kebutuhan user,
- kebijakan internal,
- arsitektur jaringan,
- aplikasi yang berjalan,
- kebutuhan operasional,
- dan tingkat risiko yang diterima organisasi.

Hal-hal tersebut sering kali tidak bisa dinilai sepenuhnya oleh automation.

### c. Poin Manual tetap masuk ruang lingkup audit

Dokumen CIS menekankan bahwa rekomendasi Manual dan Automated sama-sama penting. Bahkan jika sebuah tool hanya memeriksa rekomendasi Automated, organisasi tetap harus menangani rekomendasi Manual karena biasanya tetap relevan dalam audit.

### d. Manual bukan berarti opsional

Kesalahan umum adalah menganggap Manual berarti “boleh dilewati”. Yang benar: Manual berarti “perlu diperiksa dengan campur tangan manusia”.

---

## 2.4 Peran CIS-CAT dan tooling assessment

CIS menyebut beberapa alat yang dapat membantu assessment, terutama:

- **CIS-CAT Pro Assessor**
- **CIS Benchmarks Certified 3rd Party Tooling**

Tool seperti ini berfungsi untuk membuat proses assessment lebih skalabel. Jika organisasi memiliki banyak server, pengecekan manual satu per satu akan sangat lambat dan rentan tidak konsisten.

Dengan tooling assessment, organisasi dapat:

- memeriksa banyak sistem dengan metode yang seragam,
- menghasilkan laporan compliance,
- menemukan konfigurasi yang tidak sesuai,
- memprioritaskan remediation,
- dan melacak perkembangan hardening dari waktu ke waktu.

Namun tool bukan pengganti pemahaman manusia. Tool hanya membantu membaca kondisi teknis. Keputusan akhir tetap harus mempertimbangkan risiko, fungsi sistem, dan kebijakan organisasi.

---

## 2.5 Kesalahan umum dalam menggunakan CIS Benchmark

Beberapa kesalahan yang sering terjadi ketika orang baru belajar CIS:

1. Menganggap CIS sebagai script yang tinggal dijalankan.
2. Menganggap semua rekomendasi harus diterapkan tanpa pengecualian.
3. Mengabaikan bagian Impact.
4. Menggunakan benchmark versi atau distribusi yang salah.
5. Melewati rekomendasi Manual.
6. Tidak mendokumentasikan exception.
7. Langsung menerapkan remediation ke sistem produksi.
8. Tidak melakukan rollback plan.
9. Menganggap skor compliance tinggi berarti sistem sudah pasti aman.

CIS harus dibaca sebagai dokumen standar, bukan sekadar checklist teknis.

---

# 3. Key Stakeholders

## 3.1 Siapa saja pihak yang terlibat dalam penerapan CIS?

Penerapan CIS tidak hanya menjadi pekerjaan satu orang admin. Dokumen CIS menekankan bahwa cybersecurity adalah usaha kolaboratif. Banyak pihak perlu terlibat agar hardening berjalan aman dan tidak merusak operasional.

Stakeholder utama meliputi:

| Stakeholder | Peran Utama |
|---|---|
| System administrator | Mengelola konfigurasi sistem, service, user, permission, paket, dan operasional server |
| Application administrator | Memastikan aplikasi tetap berjalan setelah hardening |
| Security specialist | Menilai risiko keamanan dan menentukan prioritas kontrol |
| Auditor | Memeriksa kepatuhan terhadap benchmark dan kebijakan organisasi |
| Help desk | Menangani dampak ke user dan laporan gangguan setelah perubahan |
| Platform deployment personnel | Menyiapkan image, template, atau deployment sistem yang sudah sesuai baseline |
| User atau pemilik layanan | Memberi konteks kebutuhan bisnis dan dampak operasional |
| Manajemen atau pemilik risiko | Menyetujui risiko, exception, dan prioritas remediation |

---

## 3.2 Kenapa hardening tidak boleh dilakukan sendirian tanpa koordinasi?

Hardening dapat mengubah perilaku sistem. Perubahan yang terlihat kecil bagi admin bisa berdampak besar bagi aplikasi atau user.

Contoh:

- Menonaktifkan service tertentu bisa membuat aplikasi gagal berjalan.
- Mengubah konfigurasi SSH bisa mengunci akses admin.
- Mengetatkan permission file bisa membuat service tidak bisa membaca konfigurasi.
- Mengubah konfigurasi PAM bisa memengaruhi login user.
- Menonaktifkan modul kernel tertentu bisa mengganggu container, storage, atau perangkat tertentu.
- Mengaktifkan aturan firewall terlalu ketat bisa memutus akses aplikasi.

Karena itu, hardening harus dilakukan dengan koordinasi. Tujuannya bukan hanya “sistem lebih aman”, tetapi “sistem lebih aman dan tetap berfungsi”.

---

## 3.3 Hubungan admin, security specialist, auditor, help desk, dan user

Penerapan CIS idealnya membentuk alur kerja kolaboratif.

```text
Security specialist menentukan baseline dan prioritas risiko
        ↓
Admin menguji dan menerapkan konfigurasi teknis
        ↓
Application owner memastikan aplikasi tetap berjalan
        ↓
Help desk menerima dan menangani laporan dampak user
        ↓
Auditor memeriksa bukti compliance dan exception
        ↓
Manajemen menyetujui risiko yang diterima
```

### Admin

Admin adalah pelaksana teknis. Ia memahami sistem, service, file konfigurasi, package manager, user, permission, dan mekanisme boot. Admin bertugas memastikan perubahan benar secara teknis.

### Security specialist

Security specialist memahami ancaman dan risiko. Ia membantu menentukan rekomendasi mana yang prioritas, mana yang harus diterapkan segera, dan mana yang perlu exception.

### Auditor

Auditor tidak selalu melakukan konfigurasi, tetapi memeriksa apakah kontrol sudah diterapkan dan apakah bukti penerapannya memadai. Auditor juga memeriksa apakah exception terdokumentasi dengan baik.

### Help desk

Help desk penting karena hardening bisa berdampak ke user. Misalnya user tidak bisa login, akses aplikasi berubah, atau fitur tertentu dibatasi. Help desk menjadi penghubung antara perubahan teknis dan pengalaman user.

### User atau pemilik layanan

User atau pemilik layanan mengetahui kebutuhan operasional. Mereka bisa menjelaskan apakah suatu service memang diperlukan atau tidak. Tanpa masukan mereka, admin bisa saja menonaktifkan komponen yang ternyata penting.

---

## 3.4 Prinsip koordinasi dalam penerapan CIS

Ada beberapa prinsip penting:

1. **Jangan mengamankan sistem tanpa memahami fungsi sistem.**
2. **Jangan menerapkan rekomendasi hanya karena ingin skor compliance tinggi.**
3. **Selalu baca Impact sebelum remediation.**
4. **Selalu dokumentasikan pengecualian.**
5. **Libatkan pihak yang terdampak.**
6. **Uji perubahan sebelum produksi.**
7. **Pastikan ada rencana rollback.**

---

# 4. Apply the Correct Version of a Benchmark

## 4.1 Kenapa versi benchmark harus sesuai sistem?

CIS Benchmark dibuat dan diuji untuk produk serta versi tertentu. CIS Debian Linux 12 Benchmark ditujukan untuk **Debian 12**, bukan untuk semua Linux.

Setiap distribusi Linux memiliki perbedaan, misalnya:

- lokasi file konfigurasi,
- versi paket,
- default service,
- mekanisme package management,
- modul kernel,
- konfigurasi PAM,
- struktur direktori,
- sistem init,
- tool firewall,
- kebijakan default keamanan,
- dan cara distribusi mengemas software.

Karena itu, benchmark harus sesuai dengan sistem target. Menggunakan benchmark yang salah dapat menghasilkan assessment yang keliru.

---

## 4.2 Risiko memakai benchmark yang salah

Menggunakan benchmark yang salah dapat menimbulkan beberapa risiko.

### a. False positive

Sistem dianggap compliant padahal sebenarnya tidak aman, karena tool atau auditor memeriksa hal yang tidak relevan.

### b. False negative

Sistem dianggap tidak compliant padahal konfigurasi sebenarnya benar untuk distribusi tersebut, tetapi berbeda dari benchmark yang dipakai.

### c. Remediation gagal

Instruksi remediation bisa gagal karena file, paket, service, atau perintah yang dimaksud tidak ada pada sistem target.

### d. Sistem rusak atau tidak stabil

Remediation dari distribusi lain bisa mengubah konfigurasi secara tidak sesuai. Ini dapat menyebabkan service gagal, boot bermasalah, login terganggu, atau aplikasi tidak berjalan.

### e. Audit tidak valid

Jika benchmark tidak sesuai, hasil audit tidak mencerminkan postur keamanan sebenarnya. Ini berbahaya karena organisasi bisa membuat keputusan berdasarkan data compliance yang salah.

---

## 4.3 Contoh: Debian 12 tidak boleh sembarang disamakan dengan Debian 11, Ubuntu, Arch, atau RHEL

Walaupun Debian, Ubuntu, Arch, dan RHEL sama-sama Linux, mereka tidak identik.

### Debian 12 vs Debian 11

Keduanya sama-sama Debian, tetapi versi paket, default konfigurasi, lifecycle dukungan, dan beberapa mekanisme keamanan bisa berbeda. Benchmark Debian 12 dibuat untuk kondisi Debian 12, bukan otomatis berlaku untuk Debian 11.

### Debian 12 vs Ubuntu

Ubuntu berbasis Debian, tetapi memiliki kebijakan paket, konfigurasi default, service, AppArmor, cloud-init, snap, dan pola update yang berbeda. Rekomendasi untuk Ubuntu tidak boleh langsung dianggap sama dengan Debian 12.

### Debian 12 vs Arch Linux

Arch bersifat rolling release, sedangkan Debian menggunakan release stabil. Struktur paket, default konfigurasi, dan filosofi distribusinya berbeda. Benchmark Debian 12 tidak boleh diterapkan begitu saja ke Arch.

### Debian 12 vs RHEL

RHEL memiliki ekosistem berbeda, seperti SELinux, dnf/rpm, dan kebijakan enterprise yang berbeda. Banyak konfigurasi RHEL tidak relevan langsung untuk Debian.

---

## 4.4 Cara berpikir yang benar

Sebelum menerapkan CIS, tanyakan:

1. Sistem ini distribusi apa?
2. Versinya berapa?
3. Arsitekturnya apa?
4. Sistem ini server atau workstation?
5. Apakah benchmark yang dipakai memang untuk sistem ini?
6. Apakah benchmark tersebut versi terbaru yang relevan?
7. Apakah ada perbedaan lokal, cloud, container, atau environment khusus?

Untuk konteks dokumen ini, acuannya adalah **CIS Debian Linux 12 Benchmark**, bukan CIS Ubuntu, bukan CIS RHEL, bukan CIS Arch, dan bukan benchmark distribusi lain.

---

# 5. Exceptions

## 5.1 Apa itu pengecualian dalam CIS?

**Exception** adalah kondisi ketika organisasi memutuskan untuk tidak menerapkan rekomendasi CIS tertentu, atau menerapkannya dengan cara berbeda, karena ada alasan teknis, operasional, bisnis, atau risiko yang dapat diterima.

CIS sendiri menyebut item dalam benchmark sebagai **recommendations**, bukan **requirements**. Artinya, rekomendasi CIS adalah panduan yang kuat, tetapi bukan hukum mutlak yang harus diterapkan tanpa mempertimbangkan konteks.

---

## 5.2 Kenapa rekomendasi CIS bukan hukum mutlak?

Ada beberapa alasan.

### a. Fungsi sistem berbeda-beda

Tidak semua sistem memiliki tujuan yang sama. Rekomendasi yang cocok untuk workstation belum tentu cocok untuk server produksi. Rekomendasi yang cocok untuk server database belum tentu cocok untuk server container.

### b. Keamanan harus seimbang dengan ketersediaan layanan

Tujuan keamanan bukan hanya mengunci sistem, tetapi menjaga **confidentiality**, **integrity**, dan **availability**. Sistem yang sangat terkunci tetapi tidak bisa digunakan juga gagal memenuhi tujuan organisasi.

### c. Beberapa rekomendasi dapat mengganggu fungsi tertentu

CIS biasanya menyertakan bagian Impact untuk menjelaskan konsekuensi operasional. Jika impact terlalu besar terhadap fungsi utama sistem, exception bisa diperlukan.

### d. Organisasi memiliki risk appetite berbeda

Ada organisasi yang sangat ketat karena menangani data sensitif. Ada juga lingkungan pembelajaran atau lab yang membutuhkan fleksibilitas. Tingkat risiko yang diterima bisa berbeda.

---

## 5.3 Contoh pengecualian: web server memang harus aktif jika server dipakai sebagai web server

Salah satu contoh penting dalam dokumen CIS adalah rekomendasi yang dapat menyarankan agar web server tidak dipasang atau tidak digunakan pada sistem tertentu. Secara umum, ini masuk akal karena service yang tidak diperlukan meningkatkan attack surface.

Namun jika sistem tersebut memang bertugas sebagai **web server**, maka web server harus aktif. Dalam kasus ini, rekomendasi “web server tidak digunakan” tidak bisa diterapkan secara literal.

Yang benar bukan memaksakan rekomendasi, tetapi membuat exception yang terdokumentasi.

Contoh dokumentasi exception:

| Komponen | Isi |
|---|---|
| Rekomendasi | Web server services are not in use |
| Status | Exception |
| Alasan | Server ini memang berfungsi sebagai web server aplikasi perpustakaan |
| Risiko | Port HTTP/HTTPS terbuka dan service web menjadi attack surface |
| Mitigasi | Hardening web server, reverse proxy, TLS, firewall, patching, logging, WAF jika tersedia |
| Pemilik risiko | Tim infrastruktur / pemilik aplikasi |
| Tanggal review | Ditinjau ulang setiap 3 atau 6 bulan |

---

## 5.4 Pentingnya dokumentasi risiko dan alasan pengecualian

Exception yang tidak terdokumentasi akan terlihat seperti kelalaian. Dalam audit, perbedaan antara “tidak compliant karena lupa” dan “tidak compliant karena exception yang disetujui” sangat penting.

Dokumentasi exception harus menjawab:

1. Rekomendasi mana yang dikecualikan?
2. Sistem mana yang terkena exception?
3. Mengapa exception diperlukan?
4. Apa risiko keamanan yang muncul?
5. Siapa yang menyetujui risiko tersebut?
6. Mitigasi apa yang diterapkan?
7. Kapan exception akan ditinjau ulang?
8. Apakah ada rencana menghilangkan exception di masa depan?

---

## 5.5 Exception bukan alasan untuk mengabaikan keamanan

Exception tidak berarti rekomendasi diabaikan tanpa kontrol. Jika sebuah rekomendasi tidak bisa diterapkan, organisasi harus memikirkan kontrol pengganti.

Contoh:

- Jika web server harus aktif, maka perketat konfigurasi web server.
- Jika USB storage tidak bisa dinonaktifkan, gunakan kontrol seperti USBGuard atau kebijakan akses perangkat.
- Jika SSH root login perlu dibuka sementara, batasi dengan waktu, IP tertentu, key-based authentication, logging, dan approval.
- Jika container membutuhkan modul overlay, amankan runtime container dan host-nya.

Prinsipnya: **exception harus disertai risk acceptance atau compensating control**.

---

# 6. Remediation

## 6.1 Konsep remediation dalam CIS

**Remediation** adalah proses memperbaiki konfigurasi sistem agar sesuai dengan rekomendasi benchmark. Jika assessment menemukan sistem belum sesuai, remediation adalah langkah untuk membawa sistem ke kondisi yang diharapkan.

Dalam CIS, bagian remediation biasanya menjelaskan langkah sistematis untuk mengubah konfigurasi. Namun dalam pembahasan konseptual ini, yang penting adalah memahami bahwa remediation bukan sekadar menjalankan command, tetapi proses perubahan konfigurasi yang harus dikendalikan.

---

## 6.2 Kenapa remediation harus diuji dulu?

Remediation harus diuji karena perubahan keamanan bisa berdampak pada fungsi sistem. CIS sendiri memperingatkan agar hardening pada sistem yang sudah berjalan dilakukan dengan hati-hati dan diuji secara menyeluruh.

Beberapa alasan pengujian penting:

### a. Menghindari downtime

Perubahan konfigurasi bisa membuat service berhenti, aplikasi gagal start, atau akses user terganggu.

### b. Menghindari lockout admin

Pengaturan SSH, PAM, sudo, atau firewall yang salah bisa membuat admin tidak bisa masuk kembali ke server.

### c. Menghindari konflik aplikasi

Aplikasi tertentu mungkin membutuhkan permission, modul kernel, port, atau service yang dianggap tidak perlu oleh baseline umum.

### d. Memastikan konfigurasi cocok dengan environment

Konfigurasi untuk server lab belum tentu cocok untuk server produksi. Konfigurasi untuk workstation biasa belum tentu cocok untuk developer workstation.

### e. Menyiapkan rollback

Jika remediation menyebabkan gangguan, organisasi harus bisa mengembalikan sistem ke keadaan sebelumnya.

---

## 6.3 Risiko langsung menerapkan hardening ke sistem produksi

Menerapkan hardening langsung ke produksi tanpa pengujian adalah praktik berisiko tinggi.

Risikonya antara lain:

- service penting berhenti,
- aplikasi gagal berjalan,
- user tidak bisa login,
- admin terkunci dari SSH,
- firewall memblokir trafik sah,
- container gagal start,
- permission file membuat aplikasi tidak bisa membaca konfigurasi,
- logging memenuhi disk,
- audit configuration membuat sistem masuk kondisi tidak diinginkan,
- atau performa sistem menurun.

Pada sistem produksi, dampak kecil bisa menjadi besar karena menyangkut layanan nyata, data nyata, dan user nyata.

---

## 6.4 Perbedaan lab testing, pilot deployment, dan full deployment

CIS menyarankan pendekatan bertahap. Secara konseptual, terdapat tiga tahap penting.

### a. Lab testing

Lab testing dilakukan pada sistem uji yang merepresentasikan sistem produksi. Tujuannya untuk melihat apakah remediation dapat diterapkan tanpa merusak fungsi dasar.

Ciri lab testing:

- dilakukan di lingkungan non-produksi,
- aman untuk gagal,
- digunakan untuk eksperimen,
- membantu menemukan masalah sebelum produksi,
- cocok untuk membaca Impact dan menguji perubahan.

Contoh: membuat VM Debian 12 yang mirip server produksi, lalu mencoba konfigurasi CIS di sana.

### b. Pilot deployment

Pilot deployment adalah penerapan awal ke sebagian kecil sistem produksi atau semi-produksi yang dipilih secara hati-hati.

Ciri pilot deployment:

- diterapkan ke jumlah sistem terbatas,
- diawasi secara ketat,
- digunakan untuk melihat dampak nyata,
- menjadi tahap sebelum rollout besar.

Contoh: dari 100 server, terapkan hardening dulu ke 3 server non-kritis atau server dengan risiko operasional rendah.

### c. Full deployment

Full deployment adalah penerapan ke seluruh sistem target setelah lab dan pilot berhasil.

Ciri full deployment:

- dilakukan bertahap,
- tetap dimonitor,
- ada dokumentasi,
- ada rollback plan,
- ada evaluasi setelah implementasi.

Full deployment tidak berarti semua diterapkan sekaligus tanpa kontrol. Deployment yang baik biasanya iteratif: diterapkan ke kelompok sistem, dipantau, lalu dilanjutkan ke kelompok berikutnya.

---

## 6.5 Remediation bukan hanya teknis, tetapi proses manajemen perubahan

Dalam organisasi, remediation harus diperlakukan sebagai bagian dari **change management**.

Artinya, sebelum perubahan dilakukan, perlu ada:

- rencana perubahan,
- analisis dampak,
- approval,
- jadwal maintenance,
- backup,
- rollback plan,
- dokumentasi,
- komunikasi ke pihak terdampak,
- dan verifikasi setelah perubahan.

Hardening yang matang bukan hanya aman secara teknis, tetapi juga terkendali secara operasional.

---

# 7. Target Technology Details

## 7.1 Ruang lingkup CIS Debian Linux 12

CIS Debian Linux 12 Benchmark memberikan panduan preskriptif untuk membangun postur konfigurasi aman pada sistem **Debian 12 Linux distribution**.

Ruang lingkupnya adalah konfigurasi sistem Debian 12, terutama pada aspek:

- initial setup,
- filesystem,
- package management,
- mandatory access control,
- bootloader,
- process hardening,
- warning banners,
- GNOME Display Manager,
- services,
- network,
- firewall,
- SSH,
- sudo,
- PAM,
- user accounts,
- logging,
- auditing,
- integrity checking,
- system maintenance,
- permission file dan direktori penting.

Dokumen ini bukan panduan pemrograman aplikasi, bukan panduan forensik, bukan panduan incident response, dan bukan panduan keamanan cloud secara menyeluruh. Fokusnya adalah **secure configuration posture** untuk Debian 12.

---

## 7.2 Batasan platform x86_64

Benchmark ini ditujukan untuk Debian 12 pada platform **x86_64**. Ini penting karena arsitektur hardware dapat memengaruhi paket, modul, perilaku kernel, dan dukungan sistem.

Jika sistem berjalan pada arsitektur berbeda, seperti ARM atau platform embedded, beberapa rekomendasi mungkin tetap relevan secara konsep, tetapi hasil audit dan remediation belum tentu sama.

---

## 7.3 Debian 12.13 sebagai basis pengujian dokumen

Dokumen CIS menyatakan bahwa panduan ini dikembangkan dan diuji terhadap **Debian 12.13**. Ini berarti rekomendasi, audit, dan remediation disusun berdasarkan kondisi Debian 12.13 saat benchmark dibuat.

Konsekuensinya:

- Jika sistem Debian 12 memiliki versi paket berbeda, hasil bisa sedikit berbeda.
- Jika sistem sudah banyak dimodifikasi, beberapa audit bisa tidak sama.
- Jika sistem menggunakan environment khusus, seperti container host, cloud image, atau minimal install, perlu validasi tambahan.

---

## 7.4 Asumsi penggunaan root dan Bash

Benchmark ini secara umum mengasumsikan operasi dilakukan sebagai **root user** dan dijalankan dengan shell default **Bash** pada distribusi terkait.

Ini penting karena:

- root memiliki akses penuh ke area sistem,
- user non-root mungkin tidak bisa membaca atau mengubah file tertentu,
- penggunaan `sudo` bisa menghasilkan perilaku berbeda tergantung konfigurasi,
- shell selain Bash dapat memengaruhi cara script berjalan,
- environment variable dan PATH dapat memengaruhi hasil command.

Dalam dokumen CIS, prompt `#` digunakan sebagai indikasi bahwa command dijalankan sebagai root. Secara konseptual, ini mengingatkan bahwa banyak remediation menyentuh area sensitif sistem.

---

## 7.5 Kenapa konteks distribusi Linux penting?

Linux bukan satu sistem tunggal yang identik di semua distribusi. Kernel mungkin sama-sama Linux, tetapi distribusi berbeda memiliki kebijakan dan struktur berbeda.

Hal-hal yang bisa berbeda antar distribusi:

- package manager: `apt`, `dnf`, `pacman`, dan lainnya,
- default security framework: AppArmor atau SELinux,
- lokasi file konfigurasi,
- versi OpenSSH,
- struktur PAM,
- service default,
- firewall default,
- package naming,
- lifecycle update,
- cloud image defaults,
- filesystem layout.

Karena itu, memahami CIS Debian Linux 12 harus dimulai dari konteks Debian 12, bukan Linux secara umum saja.

---

## 7.6 Catatan penting tentang status Debian 12 dalam dokumen

Pada bagian Target Technology Details, CIS menyebut bahwa dokumen ini adalah final release untuk benchmark Debian Linux 12 dan mendorong migrasi ke versi teknologi yang lebih baru dan masih didukung. Catatan ini penting secara konseptual karena keamanan tidak hanya bergantung pada konfigurasi, tetapi juga pada **lifecycle dukungan sistem operasi**.

Sistem yang sudah melewati masa dukungan umum berisiko karena:

- patch keamanan bisa terbatas,
- vulnerability baru mungkin tidak segera diperbaiki,
- package repository berubah,
- dan standar keamanan baru mungkin lebih cocok diterapkan pada versi yang lebih baru.

Dengan kata lain, hardening tidak menggantikan kebutuhan untuk menggunakan sistem operasi yang masih didukung.

---

# 8. Recommendation Definitions

Bagian Recommendation Definitions menjelaskan komponen-komponen yang muncul dalam setiap rekomendasi CIS. Memahami struktur ini sangat penting sebelum membaca daftar rekomendasi teknis.

Setiap rekomendasi CIS biasanya bukan hanya berisi judul dan command, tetapi memiliki struktur yang membantu pembaca memahami konteks, alasan, dampak, cara audit, dan cara remediation.

---

## 8.1 Title

**Title** adalah judul singkat yang menjelaskan konfigurasi yang dimaksud.

Contoh pola judul CIS biasanya seperti:

```text
Ensure [komponen] is [kondisi yang diharapkan]
```

Fungsi Title:

- memberi gambaran cepat tentang tujuan rekomendasi,
- memudahkan tracking dalam audit,
- menjadi label dalam laporan compliance,
- membantu admin mencari rekomendasi tertentu.

Judul biasanya ringkas, tetapi tidak cukup untuk memahami seluruh rekomendasi. Pembaca tetap harus membaca Description, Rationale, Impact, Audit, dan Remediation.

---

## 8.2 Assessment Status

**Assessment Status** menunjukkan apakah rekomendasi dapat dinilai secara otomatis atau membutuhkan pemeriksaan manual.

Assessment Status terdiri dari:

- Automated
- Manual

Status ini tidak menunjukkan tingkat kepentingan. Automated bukan berarti lebih penting, dan Manual bukan berarti opsional. Status hanya menunjukkan cara penilaiannya.

---

## 8.3 Automated

**Automated** berarti kontrol teknis dapat diperiksa secara otomatis dan dapat divalidasi ke status pass/fail.

Ciri rekomendasi Automated:

- kondisinya bisa dibaca dari sistem,
- hasilnya relatif objektif,
- tool dapat menguji secara konsisten,
- cocok untuk assessment massal.

Contoh konseptual:

- Apakah paket tertentu terpasang?
- Apakah service tertentu aktif?
- Apakah parameter kernel bernilai tertentu?
- Apakah permission file sesuai?

Automated membantu efisiensi, tetapi tetap perlu dipahami dampaknya sebelum remediation.

---

## 8.4 Manual

**Manual** berarti kontrol tidak dapat sepenuhnya diperiksa secara otomatis. Validasi membutuhkan penilaian manusia.

Ciri rekomendasi Manual:

- hasil aman/tidak aman bergantung konteks,
- perlu dibandingkan dengan kebijakan organisasi,
- membutuhkan review konfigurasi yang tidak sederhana,
- atau tool tidak bisa memahami tujuan sistem.

Contoh konseptual:

- Menentukan apakah service yang listening memang disetujui.
- Menentukan apakah password complexity policy sesuai kebijakan organisasi.
- Meninjau apakah file SUID/SGID tertentu memang sah.

Manual tidak boleh dilewati. Manual berarti “perlu dianalisis”.

---

## 8.5 Profile

**Profile** adalah kumpulan rekomendasi yang disesuaikan dengan jenis sistem dan tingkat keamanan yang diinginkan.

Dalam CIS Debian Linux 12 Benchmark, profil utama adalah:

- Level 1 - Server
- Level 2 - Server
- Level 1 - Workstation
- Level 2 - Workstation

Profile membantu organisasi memilih baseline yang sesuai. Tidak semua sistem harus langsung Level 2. Pemilihan profil harus mempertimbangkan fungsi sistem, kebutuhan keamanan, dan dampak operasional.

---

## 8.6 Description

**Description** berisi penjelasan detail tentang pengaturan yang dibahas. Description menjawab pertanyaan: “Apa yang sedang dikonfigurasi?”

Bagian ini dapat menjelaskan:

- apa fungsi komponen,
- apa nilai yang direkomendasikan,
- bagaimana komponen tersebut bekerja secara umum,
- dan konteks teknis yang perlu diketahui.

Description penting karena judul saja sering terlalu singkat.

---

## 8.7 Rationale Statement

**Rationale Statement** menjelaskan alasan keamanan di balik rekomendasi. Bagian ini menjawab pertanyaan: “Mengapa rekomendasi ini penting?”

Rationale membantu pembaca memahami prinsip keamanan, seperti:

- mengurangi attack surface,
- membatasi privilege,
- mencegah akses tidak sah,
- memperkuat autentikasi,
- menjaga integritas log,
- mencegah penyalahgunaan service,
- atau mendukung auditability.

Rationale penting agar admin tidak hanya menjalankan checklist, tetapi memahami alasan di balik perubahan.

---

## 8.8 Impact Statement

**Impact Statement** menjelaskan konsekuensi keamanan, fungsional, atau operasional yang dapat terjadi jika rekomendasi diterapkan.

Bagian ini sangat penting dan tidak boleh dilewati.

Impact dapat berisi peringatan seperti:

- fitur tertentu akan berhenti bekerja,
- aplikasi tertentu bisa terganggu,
- performa dapat menurun,
- user experience berubah,
- service tertentu tidak bisa digunakan,
- atau konfigurasi memerlukan pengecualian untuk environment tertentu.

Dalam praktik hardening, bagian Impact membantu menentukan apakah rekomendasi langsung diterapkan, diuji lebih lanjut, atau dijadikan exception.

---

## 8.9 Audit Procedure

**Audit Procedure** adalah instruksi sistematis untuk menentukan apakah sistem sudah sesuai dengan rekomendasi.

Audit Procedure menjawab pertanyaan: “Bagaimana cara mengecek status compliance?”

Secara konseptual, audit dapat menghasilkan:

- pass,
- fail,
- not applicable,
- atau perlu review manual.

Audit Procedure penting karena compliance harus berbasis bukti, bukan asumsi.

---

## 8.10 Remediation Procedure

**Remediation Procedure** adalah instruksi sistematis untuk mengubah konfigurasi agar sistem menjadi compliant.

Remediation menjawab pertanyaan: “Bagaimana cara memperbaiki jika belum sesuai?”

Namun remediation harus dibaca bersama:

- Description,
- Rationale,
- Impact,
- kebutuhan sistem,
- dan kebijakan organisasi.

Jangan membaca remediation secara terpisah seolah-olah semua command aman diterapkan di semua kondisi.

---

## 8.11 Default Value

**Default Value** menjelaskan nilai bawaan dari suatu pengaturan, jika diketahui. Jika tidak diketahui, biasanya dinyatakan sebagai tidak dikonfigurasi atau tidak didefinisikan.

Default Value penting karena membantu membandingkan:

- kondisi bawaan sistem,
- kondisi yang direkomendasikan CIS,
- dan kondisi aktual sistem.

Perlu dipahami bahwa default bawaan distribusi belum tentu sama dengan baseline CIS. Distribusi bisa memilih default yang seimbang antara kemudahan penggunaan dan keamanan, sedangkan CIS sering lebih ketat.

---

## 8.12 References

**References** berisi rujukan tambahan yang berkaitan dengan rekomendasi. Rujukan ini bisa berupa dokumentasi teknis, standar keamanan, STIG, NIST, CVE, atau sumber lain.

Fungsi References:

- memberi dasar tambahan,
- membantu audit,
- memudahkan pembelajaran lanjutan,
- menunjukkan keterkaitan dengan standar lain.

References membantu membuktikan bahwa rekomendasi tidak berdiri sendiri tanpa dasar.

---

## 8.13 CIS Controls mapping

**CIS Controls mapping** menunjukkan hubungan antara rekomendasi benchmark dengan CIS Critical Security Controls.

Mapping ini membantu organisasi melihat bahwa rekomendasi teknis pada sistem Debian berhubungan dengan kontrol keamanan yang lebih luas.

Contoh konsep:

- Menonaktifkan service tidak perlu berhubungan dengan kontrol pengurangan service yang tidak diperlukan.
- Mengatur logging berhubungan dengan kontrol audit log.
- Mengatur konfigurasi aman berhubungan dengan proses secure configuration.

Mapping berguna untuk audit, compliance, dan penyusunan program keamanan organisasi.

---

## 8.14 Additional Information

**Additional Information** berisi informasi tambahan yang tidak masuk ke bagian lain, tetapi tetap berguna.

Bagian ini bisa berisi:

- catatan teknis,
- pengecualian khusus,
- peringatan tambahan,
- konteks environment tertentu,
- atau informasi yang membantu interpretasi rekomendasi.

Jika tersedia, bagian ini sebaiknya tetap dibaca karena dapat menjelaskan detail penting yang tidak terlihat pada bagian utama.

---

## 8.15 Cara membaca satu rekomendasi CIS secara benar

Urutan membaca yang disarankan:

```text
1. Baca Title
2. Lihat Profile Applicability
3. Lihat Assessment Status
4. Baca Description
5. Pahami Rationale
6. Baca Impact dengan serius
7. Jalankan atau pahami Audit Procedure
8. Tentukan apakah remediation cocok
9. Dokumentasikan hasil atau exception
10. Uji ulang setelah remediation
```

Kesalahan umum adalah langsung menuju Remediation tanpa membaca Impact dan Profile. Ini berbahaya karena dapat menyebabkan konfigurasi diterapkan pada sistem yang tidak cocok.

---

# 9. Profile Definitions

CIS Debian Linux 12 Benchmark mendefinisikan empat profil konfigurasi utama:

1. Level 1 - Server
2. Level 2 - Server
3. Level 1 - Workstation
4. Level 2 - Workstation

Profil membantu memilih tingkat hardening sesuai jenis sistem dan kebutuhan keamanan.

---

## 9.1 Level 1 - Server

**Level 1 - Server** adalah profil baseline untuk server yang bertujuan memberikan peningkatan keamanan yang praktis dan wajar tanpa mengganggu fungsi teknologi secara berlebihan.

Karakter Level 1:

- praktis,
- prudent atau masuk akal,
- memberi manfaat keamanan yang jelas,
- tidak menghambat fungsi sistem di luar batas yang dapat diterima,
- cocok sebagai baseline awal untuk server.

Level 1 Server cocok untuk:

- server umum,
- server internal,
- server lab,
- server produksi yang membutuhkan baseline aman tetapi tetap stabil,
- organisasi yang baru mulai menerapkan hardening.

Level 1 biasanya menjadi pilihan pertama karena lebih seimbang antara keamanan dan operasional.

---

## 9.2 Level 2 - Server

**Level 2 - Server** adalah profil yang memperluas Level 1 Server. Level 2 bukan profil yang berdiri sendiri. Artinya, Level 2 mencakup Level 1 dan menambahkan rekomendasi yang lebih ketat.

Karakter Level 2:

- ditujukan untuk environment dengan keamanan sangat penting,
- berfungsi sebagai defense in depth,
- dapat mengurangi utilitas atau performa sistem,
- membutuhkan pengujian lebih serius,
- lebih mungkin membutuhkan exception.

Level 2 Server cocok untuk:

- server dengan data sensitif,
- server publik berisiko tinggi,
- server yang masuk ruang lingkup compliance ketat,
- sistem kritikal,
- environment dengan ancaman tinggi.

Namun Level 2 harus diterapkan hati-hati karena beberapa rekomendasi dapat mengganggu fungsi tertentu. Contohnya, pembatasan modul kernel atau service tertentu dapat berdampak pada container, cloud, atau aplikasi khusus.

---

## 9.3 Level 1 - Workstation

**Level 1 - Workstation** adalah baseline untuk perangkat kerja user yang tetap mempertimbangkan kegunaan sehari-hari.

Karakter Level 1 Workstation:

- praktis,
- memberi manfaat keamanan jelas,
- tidak terlalu menghambat penggunaan normal,
- cocok untuk laptop atau desktop user,
- lebih mempertimbangkan interaksi manusia dibanding server.

Workstation berbeda dari server karena workstation biasanya membutuhkan:

- GUI,
- browser,
- aplikasi produktivitas,
- perangkat input/output,
- kemungkinan Wi-Fi atau Bluetooth,
- automount media,
- printer,
- dan kenyamanan user.

Level 1 Workstation cocok untuk organisasi yang ingin meningkatkan keamanan endpoint tanpa membuat perangkat terlalu sulit digunakan.

---

## 9.4 Level 2 - Workstation

**Level 2 - Workstation** memperluas Level 1 Workstation dengan kontrol lebih ketat.

Karakter Level 2 Workstation:

- ditujukan untuk workstation dengan kebutuhan keamanan tinggi,
- dapat menjadi defense in depth,
- mungkin mengurangi kenyamanan user,
- dapat membatasi fitur perangkat,
- membutuhkan sosialisasi dan dukungan help desk.

Level 2 Workstation cocok untuk:

- workstation admin,
- perangkat yang mengakses data sensitif,
- perangkat auditor/security team,
- endpoint di lingkungan risiko tinggi,
- perangkat yang menjadi target serangan tinggi.

Namun Level 2 Workstation dapat menyebabkan keluhan user jika diterapkan tanpa komunikasi dan dukungan. Misalnya pembatasan USB, automount, Bluetooth, atau fitur GUI tertentu.

---

## 9.5 Perbedaan Level 1 dan Level 2

| Aspek | Level 1 | Level 2 |
|---|---|---|
| Tujuan | Baseline aman yang praktis | Hardening lebih ketat untuk risiko tinggi |
| Dampak operasional | Relatif rendah sampai sedang | Bisa sedang sampai tinggi |
| Cocok untuk | Mayoritas sistem | Sistem dengan kebutuhan keamanan tinggi |
| Risiko mengganggu fungsi | Lebih kecil | Lebih besar |
| Kebutuhan testing | Tetap perlu | Sangat perlu |
| Kemungkinan exception | Ada | Lebih besar |
| Filosofi | Aman tetapi tetap usable | Keamanan lebih diprioritaskan, termasuk defense in depth |

Level 1 biasanya menjadi titik awal. Level 2 dipilih jika organisasi memang membutuhkan keamanan lebih tinggi dan siap menerima dampak operasional.

---

## 9.6 Perbedaan Server dan Workstation

| Aspek | Server | Workstation |
|---|---|---|
| Fungsi utama | Menyediakan layanan | Digunakan langsung oleh user |
| Interaksi | Biasanya remote/admin | Langsung oleh pengguna |
| GUI | Sering tidak diperlukan | Sering diperlukan |
| Service | Menjalankan service tertentu | Biasanya menjalankan aplikasi user |
| Risiko utama | Serangan jaringan, service publik, privilege escalation | Phishing, malware, USB, browser, akses user |
| Kebutuhan usability | Stabilitas layanan | Kenyamanan dan produktivitas user |
| Contoh | Web server, database server, file server | Laptop pegawai, PC lab, desktop admin |

Server biasanya harus seminimal mungkin. Workstation perlu tetap mendukung pekerjaan user.

---

## 9.7 Kapan memilih Level 1 atau Level 2?

Pemilihan profil harus berdasarkan risiko dan kebutuhan.

### Pilih Level 1 jika:

- organisasi baru mulai menerapkan CIS,
- sistem harus tetap mudah digunakan,
- risiko sistem tidak ekstrem,
- downtime harus dihindari,
- belum ada proses hardening matang,
- atau sistem adalah baseline umum.

### Pilih Level 2 jika:

- sistem menangani data sangat sensitif,
- sistem berada di lingkungan ancaman tinggi,
- compliance menuntut kontrol ketat,
- organisasi memiliki tim yang mampu menguji dan memelihara konfigurasi,
- dampak usability/performance dapat diterima,
- dan exception dapat dikelola dengan baik.

---

## 9.8 Contoh pemilihan profil

| Kondisi Sistem | Profil yang Lebih Masuk Akal | Alasan |
|---|---|---|
| Server web internal biasa | Level 1 Server | Butuh baseline aman tanpa terlalu mengganggu layanan |
| Server database berisi data sensitif | Level 2 Server setelah testing | Risiko tinggi, perlu kontrol lebih ketat |
| Laptop pegawai umum | Level 1 Workstation | Menjaga keamanan tanpa menghambat produktivitas |
| Laptop admin infrastruktur | Level 2 Workstation dengan pengecualian terkontrol | Akses sensitif, risiko tinggi |
| Server Kubernetes/container | Level 1 Server atau Level 2 dengan exception | Beberapa kontrol Level 2 bisa mengganggu container |
| Mesin lab pembelajaran | Level 1, atau sebagian rekomendasi | Kebutuhan eksperimen lebih fleksibel |

---

# 10. Kesimpulan Besar Bagian Pengantar

Bagian pengantar CIS Debian Linux 12 Benchmark mengajarkan bahwa CIS bukan sekadar daftar command, tetapi kerangka berpikir hardening.

Inti pemahamannya adalah:

1. **CIS Benchmark adalah baseline konfigurasi aman**, bukan pengganti seluruh program keamanan.
2. **CIS harus digunakan bersama patching, endpoint protection, logging, dan monitoring.**
3. **Automated dan Manual sama-sama penting.** Automated mudah diperiksa tool, Manual membutuhkan konteks manusia.
4. **Hardening harus kolaboratif.** Admin, security specialist, auditor, help desk, dan user/pemilik layanan harus terlibat.
5. **Benchmark harus sesuai versi sistem.** Debian 12 tidak boleh disamakan begitu saja dengan Debian 11, Ubuntu, Arch, atau RHEL.
6. **Exception adalah hal wajar**, asalkan alasannya jelas, risikonya diterima, dan dokumentasinya baik.
7. **Remediation harus diuji.** Jangan langsung menerapkan hardening ke sistem produksi.
8. **Target teknologi penting.** CIS Debian Linux 12 Benchmark berlaku untuk Debian 12 pada x86_64, dengan asumsi operasi sebagai root dan Bash.
9. **Setiap rekomendasi memiliki struktur.** Title, Assessment Status, Profile, Description, Rationale, Impact, Audit, Remediation, Default Value, References, CIS Controls mapping, dan Additional Information harus dibaca sebagai satu kesatuan.
10. **Profile menentukan tingkat ketatnya hardening.** Level 1 lebih praktis, Level 2 lebih ketat; Server dan Workstation memiliki kebutuhan berbeda.

Pemahaman konseptual ini penting sebelum masuk ke bagian teknis. Tanpa memahami bagian pengantar, seseorang mudah terjebak menjalankan command hardening secara membabi buta, mengejar skor compliance, atau menerapkan rekomendasi yang tidak cocok dengan fungsi sistem.

CIS yang benar bukan sekadar “membuat sistem lulus checklist”, tetapi membangun konfigurasi aman yang tetap sesuai dengan fungsi, risiko, dan kebutuhan organisasi.

---

## Ringkasan Satu Kalimat

**CIS Debian Linux 12 Benchmark adalah panduan baseline hardening untuk Debian 12 yang harus diterapkan secara terencana, sesuai versi sistem, diuji sebelum produksi, melibatkan stakeholder, serta disesuaikan dengan risiko dan kebutuhan nyata organisasi.**
