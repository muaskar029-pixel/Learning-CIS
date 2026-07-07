# 01 - Pengenalan CIS Ubuntu Linux 24.04 LTS STIG Benchmark

**Sumber utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0, 06-12-2025.  
**Target dokumen:** modul belajar tematik untuk memahami hardening Ubuntu 24.04 LTS berbasis STIG.  
**Catatan penting:** file ini adalah dokumentasi pembelajaran dan panduan konseptual. Untuk audit resmi, tetap gunakan dokumen benchmark asli, hasil scanning, dan kebijakan organisasi.

---


## 1. Gambaran umum dokumen

**CIS Ubuntu Linux 24.04 LTS STIG Benchmark** adalah benchmark konfigurasi keamanan untuk Ubuntu 24.04 LTS yang disusun dalam format rekomendasi CIS, tetapi isinya merepresentasikan aturan dari **Canonical Ubuntu 24.04 LTS Secure Technical Implementation Guide (STIG)**. Dokumen ini tidak disusun seperti benchmark Linux biasa yang terbagi menjadi bab filesystem, service, network, access control, logging, dan maintenance. Di PDF ini, seluruh rekomendasi teknis ditempatkan dalam satu bab besar, yaitu **1 STIG RULES**.

Karena itu, untuk belajar, pendekatan terbaik bukan membaca mentah dari `1.1` sampai `1.188`, melainkan mengelompokkan aturan berdasarkan tema hardening. Pendekatan tematik membuat hubungan antar kontrol lebih mudah terlihat. Contohnya, aturan tentang `auditd`, audit rule, audit log, `rsyslog`, permission log, dan journal tidak muncul sebagai satu bab rapi, tetapi secara konsep semuanya masuk ke tema **logging dan auditing**.

Dokumen ini harus dipahami sebagai **secure baseline**. Artinya, ia memberikan titik awal konfigurasi aman untuk sistem Ubuntu 24.04 LTS, bukan satu-satunya sumber kebenaran keamanan. Hardening yang baik tetap harus disesuaikan dengan fungsi sistem, kebutuhan aplikasi, risiko organisasi, kebutuhan user, audit compliance, dan dampak operasional.

## 2. CIS Benchmark biasa vs CIS STIG Benchmark

CIS Benchmark biasa biasanya disusun dengan struktur yang mengikuti domain teknis sistem operasi, misalnya initial setup, services, network, firewall, access control, logging, auditing, dan maintenance. Pendekatannya bertujuan memberi baseline keamanan yang relatif umum untuk berbagai organisasi.

CIS STIG Benchmark berbeda karena orientasinya lebih dekat dengan kebutuhan kepatuhan STIG. Ciri yang tampak dalam dokumen Ubuntu 24.04 LTS STIG adalah adanya:

- kode aturan berbentuk `UBTU-24-xxxxx`;
- severity `CAT I`, `CAT II`, dan `CAT III`;
- `GROUP ID` dan `RULE ID`;
- rujukan ke CCI dan NIST SP 800-53;
- banyak aturan yang mengarah pada kebutuhan lingkungan pemerintahan/pertahanan seperti consent banner, MFA, smart card, PIV, PKI, FIPS, audit event detail, dan offloading audit log.

Dengan kata lain, CIS Benchmark biasa lebih mudah dipahami sebagai baseline konfigurasi aman umum, sedangkan CIS STIG Benchmark lebih ketat karena membawa bahasa kontrol formal, compliance, dan bukti audit. Untuk pembelajaran, dokumen STIG lebih menuntut karena satu rekomendasi sering bukan hanya soal “aktifkan fitur”, tetapi juga soal bukti audit, jejak perubahan, pemetaan identitas, dan konsekuensi legal/operasional.

## 3. Fungsi STIG dalam hardening sistem

STIG berfungsi sebagai standar konfigurasi teknis yang membantu organisasi memastikan sistem operasi berjalan dalam keadaan terkendali. Hardening dalam STIG tidak hanya membahas pencegahan serangan, tetapi juga:

- memastikan sistem hanya menjalankan komponen yang dibutuhkan;
- memastikan autentikasi kuat dan akses istimewa terkendali;
- memastikan komunikasi administratif dilindungi kriptografi;
- memastikan aktivitas sensitif tercatat dalam audit log;
- memastikan log dan audit rule tidak mudah dimodifikasi;
- memastikan file penting memiliki ownership dan permission yang tepat;
- memastikan baseline konfigurasi dapat diperiksa ulang;
- memastikan sistem dapat dibuktikan kepatuhannya saat audit.

STIG sangat menekankan **accountability**. Dalam sistem yang sudah di-hardening, administrator tidak cukup berkata “sistem aman”. Administrator harus bisa menunjukkan bukti bahwa konfigurasi sudah sesuai, perubahan dicatat, akses istimewa dibatasi, dan log terlindungi.

## 4. Struktur rekomendasi

Setiap rekomendasi dalam benchmark memiliki struktur yang relatif konsisten. Memahami struktur ini lebih penting daripada menghafal semua nomor aturan.

### Description

Bagian **Description** menjelaskan keadaan yang diharapkan. Contohnya, sistem harus memiliki `auditd`, sistem tidak boleh memiliki `telnet`, SSH harus memakai algoritma tertentu, atau permission file log harus dibatasi. Description menjawab pertanyaan: **kontrol apa yang ingin dicapai?**

### Rationale

Bagian **Rationale** menjelaskan alasan keamanan di balik kontrol. Bagian ini sangat penting untuk belajar konseptual karena menjawab pertanyaan: **mengapa konfigurasi ini dibutuhkan?** Misalnya, Telnet dilarang karena komunikasi tidak terenkripsi; audit log harus dilindungi karena penyerang dapat menghapus jejak; dan time synchronization penting karena forensik bergantung pada timestamp yang benar.

### Audit

Bagian **Audit** menjelaskan cara memeriksa apakah sistem sudah sesuai. Dalam audit resmi, bagian ini digunakan untuk menghasilkan status pass/fail. Audit tidak selalu berarti menjalankan satu command sederhana. Beberapa aturan bersifat manual karena membutuhkan penilaian konteks, misalnya validasi kebijakan organisasi, pengecualian operasional, atau bukti konfigurasi yang tidak bisa sepenuhnya dipastikan oleh alat otomatis.

### Remediation

Bagian **Remediation** menjelaskan tindakan untuk membawa sistem menuju kondisi sesuai benchmark. Remediation harus diuji terlebih dahulu di lab atau sistem pilot. Menerapkan hardening STIG langsung ke produksi berisiko memutus akses SSH, merusak autentikasi, mengganggu aplikasi, atau membuat user tidak bisa login.

### Additional Information

Bagian ini biasanya berisi pemetaan ke CCI, NIST SP 800-53, atau informasi tambahan lain. Bagian ini berguna untuk laporan compliance karena menghubungkan konfigurasi teknis dengan kontrol keamanan formal.

### CIS Controls

Bagian CIS Controls menghubungkan rekomendasi dengan Safeguards CIS Controls v7/v8 dan Implementation Group. Ini membantu organisasi melihat bagaimana satu konfigurasi teknis mendukung kontrol keamanan yang lebih luas.

## 5. Assessment Status: Automated dan Manual

### Automated

Status **Automated** berarti kondisi teknis dapat diperiksa secara otomatis sampai menghasilkan status pass/fail. Contohnya, apakah paket `chrony` terpasang, apakah `ufw` aktif, apakah file tertentu memiliki mode yang benar, atau apakah baris konfigurasi SSH tertentu ada.

Automated bukan berarti risikonya kecil. Banyak aturan CAT I dan CAT II bersifat automated. Automated hanya menunjukkan bahwa cara penilaiannya dapat diotomatisasi.

### Manual

Status **Manual** berarti pemeriksaan tidak bisa sepenuhnya ditentukan oleh alat. Pemeriksa mungkin perlu membaca konfigurasi, memvalidasi kebijakan organisasi, melihat bukti operasional, atau memastikan pengecualian memang sah. Contohnya adalah aturan tentang penyimpanan audit log selama periode tertentu, enkripsi data at rest, pemetaan identitas PKI, atau validasi port/protocol/service sesuai kebijakan PPSM.

Poin manual tetap wajib diperhatikan. Tooling assessment kadang hanya memeriksa rekomendasi automated, tetapi dalam audit kepatuhan, aturan manual tetap masuk ruang lingkup.

## 6. Severity: CAT I, CAT II, CAT III

Dokumen ini menggunakan profile berbasis severity:

| Severity | Makna konseptual | Dampak umum |
|---|---|---|
| CAT I | Tingkat tinggi | Risiko sangat besar, sering terkait komunikasi tidak aman, blank password, root access, Ctrl-Alt-Delete, FIPS, atau kontrol kritis lain. |
| CAT II | Tingkat sedang | Risiko penting yang memengaruhi kontrol akses, audit, permission, SSH hardening, AppArmor, PAM, PKI, atau konfigurasi sistem. |
| CAT III | Tingkat rendah | Risiko tetap penting, tetapi biasanya lebih bersifat penguatan tambahan, kebersihan konfigurasi, atau operational assurance. |

CAT III tidak boleh dianggap “tidak penting”. Dalam lingkungan STIG, CAT III tetap bagian dari compliance. Perbedaannya adalah prioritas risiko, bukan boleh/tidaknya diabaikan.

## 7. Cara membaca dokumen ini secara aman

Urutan kerja yang disarankan:

1. Baca overview dan pahami bahwa STIG adalah baseline keras, bukan konfigurasi yang selalu cocok mentah untuk semua sistem.
2. Kelompokkan aturan berdasarkan tema: baseline, service, network, SSH, identity, auditing, permission.
3. Pisahkan sistem server, workstation, VM lab, dan sistem produksi.
4. Catat aturan yang bisa mengganggu akses, terutama SSH, PAM, root login, sudo, MFA, AppArmor, firewall, dan FIPS.
5. Lakukan assessment awal.
6. Terapkan remediation di lab.
7. Uji login, SSH, sudo, aplikasi, service, logging, dan reboot.
8. Baru lanjut ke pilot deployment dan full deployment.
9. Dokumentasikan pengecualian jika ada kontrol yang tidak bisa diterapkan.

## 8. Pembagian modul belajar 5 file

| File | Fokus utama | Tema aturan |
|---|---|---|
| `01_Pengenalan_CIS_Ubuntu_STIG.md` | Pengenalan dokumen | Overview, struktur rekomendasi, Automated/Manual, CAT I/II/III, cara membaca audit-remediation. |
| `02_Baseline_Service_Integrity.md` | Baseline, package, service, integrity | AIDE, package verification, Chrony, service tidak aman, AppArmor, vendor-supported release, memory/kernel protection. |
| `03_Network_SSH_Firewall.md` | Network, firewall, SSH | UFW, remote access, SSH crypto, timeout, banner, wireless, TCP syncookies, port/protocol/service. |
| `04_Identity_Authentication_Access.md` | Identitas, autentikasi, akses | PAM, password policy, MFA, SSSD, smart card, PIV/PKI, session lock, root login, sudo, su. |
| `05_Logging_Auditing_Permission.md` | Logging, auditing, permission | rsyslog, auditd, audit rules, audit log storage, journalctl, `/var/log`, audit tools, ownership dan permission file sistem. |


## 10. Ringkasan cakupan 188 aturan

Dokumen sumber berisi **188 aturan** dari `UBTU-24-90890` sampai `UBTU-24-909000`. Dalam paket modul ini, aturan teknis dibagi ke empat modul tematik utama:

| Modul | Jumlah kode aturan yang dicakup | Fokus |
|---|---:|---|
| 02 - Baseline, Package, Service, dan System Integrity | 25 | AIDE, service minimum, Chrony, APT, AppArmor, kernel/memory protection. |
| 03 - Network, SSH, dan Firewall Hardening | 20 | UFW, SSH, remote access, port/protocol/service, TCP syncookies, wireless. |
| 04 - Identity, Authentication, dan Access Control | 49 | PAM, password, MFA, SSSD, smart card, PIV/PKI, session, root, sudo. |
| 05 - Logging, Auditing, Permission, dan File Protection | 94 + appendix bila ada | rsyslog, auditd, audit rules, audit log storage, ownership, permission, journal. |

Cakupan dibuat tematik. Artinya, beberapa kontrol seperti `sudo` dan `privileged commands` dapat dibahas konseptual di modul 04, tetapi audit rule detailnya dibahas di modul 05.

## 11. Prinsip besar yang harus diingat

Hardening STIG bukan sekadar menyalin command. Ia adalah proses mengubah sistem menjadi lebih terkendali. Setiap perubahan harus bisa dijelaskan dalam tiga lapis:

- **lapis teknis:** konfigurasi apa yang berubah;
- **lapis risiko:** ancaman apa yang dikurangi;
- **lapis operasional:** dampak apa terhadap user, aplikasi, admin, dan audit.

Sistem yang aman bukan hanya sistem yang “lulus checklist”, tetapi sistem yang tetap bisa digunakan, bisa dipantau, bisa diaudit, dan memiliki pengecualian yang terdokumentasi.
