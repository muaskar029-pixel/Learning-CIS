# CIS MariaDB 10.6 Benchmark v1.1.0 - Dokumentasi Konseptual

**Sumber utama:** CIS MariaDB 10.6 Benchmark v1.1.0 - 03-29-2024  
**Jenis dokumen:** Modul belajar konseptual  
**Fokus:** Memahami maksud, risiko, dan logika hardening MariaDB 10.6 berdasarkan CIS Benchmark  
**Catatan:** Dokumen ini menjelaskan konsep. Untuk langkah command dan konfigurasi teknis, gunakan file implementasi terpisah.

---

## Daftar Isi

1. Pengenalan CIS MariaDB 10.6 Benchmark
2. Cara Membaca Rekomendasi CIS MariaDB
3. Profile Definitions
4. Bab 1 - Operating System Level Configuration
5. Bab 2 - Installation and Planning
6. Bab 3 - File Permissions
7. Bab 4 - General
8. Bab 5 - MariaDB Permissions
9. Bab 6 - Auditing and Logging
10. Bab 7 - Authentication
11. Bab 8 - Network
12. Bab 9 - Replication
13. Ringkasan Pola Berpikir Hardening MariaDB
14. Checklist Konseptual

---

# 1. Pengenalan CIS MariaDB 10.6 Benchmark

## 1.1 Apa itu CIS MariaDB 10.6 Benchmark?

CIS MariaDB 10.6 Benchmark adalah dokumen panduan konfigurasi keamanan untuk membangun postur aman pada MariaDB 10.6. Fokusnya bukan membuat database menjadi kebal sepenuhnya, tetapi memberikan **baseline konfigurasi teknis** yang dapat dijadikan standar awal sebelum sistem digunakan di lingkungan produksi.

CIS Benchmark berisi rekomendasi preskriptif, yaitu rekomendasi yang langsung mengarah pada kondisi konfigurasi tertentu. Contohnya:

- database sebaiknya tidak berada di partisi sistem;
- service MariaDB harus berjalan dengan akun khusus berprinsip least privilege;
- password tidak boleh disimpan di environment variable atau command line;
- file database, log, key, certificate, dan plugin harus memiliki permission ketat;
- koneksi jaringan harus diamankan dengan TLS;
- privilege database harus dibatasi berdasarkan kebutuhan;
- audit logging harus aktif untuk melacak aktivitas penting.

## 1.2 Posisi CIS dalam program keamanan

CIS tidak menggantikan seluruh program cybersecurity. Ia menjadi salah satu komponen dalam program keamanan yang lebih luas. Database tetap perlu didukung oleh:

- patch management;
- vulnerability management;
- backup dan disaster recovery;
- monitoring;
- audit log review;
- network segmentation;
- incident response;
- akses administratif yang terkendali;
- hardening sistem operasi tempat database berjalan.

Dengan kata lain, hardening MariaDB bukan hanya soal mengubah isi `mariadb.cnf`. Keamanan MariaDB bergantung pada kombinasi antara konfigurasi database, konfigurasi sistem operasi, manajemen user, keamanan jaringan, backup, logging, dan kontrol operasional.

## 1.3 Ruang lingkup dokumen

Benchmark ini membahas MariaDB 10.6. Di bagian overview, dokumen CIS menjelaskan bahwa panduan ini diuji pada MariaDB 10.6 yang berjalan di Ubuntu Linux, tetapi secara umum dapat diterapkan pada distribusi Linux lain dengan penyesuaian path, nama service, dan paket.

Artinya, rekomendasinya dapat dibagi menjadi dua kelompok besar:

1. **Kontrol yang terkait langsung dengan MariaDB RDBMS**  
   Contoh: `require_secure_transport`, password plugin, user privilege, audit plugin, `sql_mode`, `ssl_type`.

2. **Kontrol yang terkait dengan sistem operasi Linux**  
   Contoh: permission file, permission datadir, akun service `mysql`, shell login, partisi non-system, systemd sandbox, dan lokasi log.

Perbedaan ini penting karena audit yang dilakukan dari koneksi remote MariaDB tidak selalu bisa memeriksa file permission di Linux. Karena itu CIS menyediakan profile yang membedakan antara `MariaDB RDBMS` dan `MariaDB RDBMS on Linux`.

## 1.4 Siapa yang memakai dokumen ini?

Benchmark ini ditujukan untuk beberapa peran:

- **System administrator**, yang mengelola OS, service, filesystem, dan permission.
- **Database administrator**, yang mengatur user, privilege, authentication, replication, dan database configuration.
- **Security specialist**, yang menilai risiko konfigurasi dan menyusun baseline keamanan.
- **Auditor**, yang memeriksa apakah sistem sesuai dengan kontrol keamanan.
- **Help desk atau deployment personnel**, yang membantu implementasi dan pemeliharaan sistem.

MariaDB sering menjadi komponen inti aplikasi. Karena itu hardening tidak boleh dilakukan sembarangan oleh satu pihak saja. Perubahan seperti `require_secure_transport=ON`, pembatasan cipher, perubahan privilege, atau pemindahan datadir dapat mengganggu aplikasi apabila tidak diuji dan dikoordinasikan.

---

# 2. Cara Membaca Rekomendasi CIS MariaDB

Setiap rekomendasi dalam CIS MariaDB biasanya memiliki struktur berikut.

## 2.1 Title

Title adalah judul ringkas rekomendasi. Contohnya:

- `1.2 Use Dedicated Least Privileged Account for MariaDB Daemon/Service`
- `4.9 Enable data-at-rest encryption in MariaDB`
- `8.1 Ensure require_secure_transport is Set to ON and have_ssl is Set to YES`

Judul menunjukkan kondisi konfigurasi yang diharapkan.

## 2.2 Assessment Status

Assessment Status menjelaskan apakah rekomendasi dapat diaudit secara otomatis atau memerlukan penilaian manual.

### Automated

Automated berarti kondisi teknis dapat diperiksa secara pass/fail menggunakan query SQL, command OS, atau pemeriksaan file konfigurasi. Contohnya:

- mengecek `SHOW VARIABLES LIKE 'have_ssl';`
- mengecek permission `datadir`;
- mengecek apakah anonymous account ada;
- mengecek apakah `server_audit` aktif.

Automated bukan berarti implementasi selalu otomatis. Artinya **audit atau assessment** dapat dibuat otomatis dengan hasil jelas.

### Manual

Manual berarti penilaian membutuhkan konteks organisasi atau keputusan manusia. Contohnya:

- apakah backup policy sudah cukup;
- apakah disaster recovery plan sesuai RTO dan RPO;
- apakah privilege user memang dibutuhkan;
- apakah replication traffic aman menggunakan private network, VPN, TLS, atau SSH tunnel;
- apakah penggunaan `unix_socket` authentication sesuai kebutuhan organisasi.

Poin Manual tetap penting karena beberapa risiko tidak bisa dinilai hanya dari output command. Sistem bisa saja tampak benar secara teknis, tetapi salah secara kebijakan.

## 2.3 Profile

Profile menunjukkan cakupan keamanan yang ditargetkan. Profile bukan sekadar label, tetapi menentukan jenis sistem dan tingkat keketatan kontrol.

## 2.4 Description

Description menjelaskan pengaturan yang dibahas. Bagian ini menjawab pertanyaan: “Apa yang sedang dikonfigurasi?”

## 2.5 Rationale

Rationale menjelaskan alasan keamanan di balik rekomendasi. Bagian ini penting untuk belajar konseptual karena memperlihatkan hubungan antara konfigurasi dan risiko.

Contoh:

- password di command line berisiko terlihat di process list atau shell history;
- wildcard hostname memperluas lokasi asal koneksi database;
- `FILE` privilege dapat dipakai membaca atau menulis file di host server;
- audit plugin yang bisa di-unload memungkinkan aktivitas database tidak tercatat.

## 2.6 Impact

Impact menjelaskan dampak operasional jika rekomendasi diterapkan. Ini sangat penting dalam hardening produksi.

Contoh:

- mengaktifkan TLS dapat membuat client lama gagal konek;
- memindahkan datadir perlu downtime;
- mengubah permission log dapat mengganggu monitoring tool;
- menerapkan `STRICT_ALL_TABLES` dapat membuat aplikasi yang sebelumnya menerima data tidak valid menjadi gagal insert/update.

## 2.7 Audit Procedure

Audit Procedure adalah langkah memeriksa kondisi sistem. Pada MariaDB, audit sering menggunakan:

- SQL query terhadap `information_schema`;
- query terhadap `mysql.user` atau `mysql.global_priv`;
- command Linux seperti `ls -l`, `find`, `grep`, `ps`, `df`;
- pemeriksaan file konfigurasi.

## 2.8 Remediation Procedure

Remediation Procedure adalah langkah memperbaiki konfigurasi agar sesuai rekomendasi. Remediation tidak boleh langsung diterapkan ke produksi tanpa backup, testing, dan rollback plan.

## 2.9 Default Value

Default Value menjelaskan nilai bawaan bila diketahui. Nilai default tidak selalu aman. Contohnya:

- `require_secure_transport` default-nya OFF;
- `default_password_lifetime` default-nya 0, artinya password tidak kedaluwarsa;
- `general_log` default-nya OFF;
- `slow_query_log` default-nya OFF;
- `secure_file_priv` bisa tidak diset.

## 2.10 References, CIS Controls, dan Additional Information

References menunjuk dokumentasi pendukung. CIS Controls menunjukkan hubungan rekomendasi dengan kontrol keamanan umum, misalnya akses kontrol, enkripsi data, logging, dan backup. Additional Information berisi caveat atau konteks tambahan yang tidak masuk ke bagian lain.

---

# 3. Profile Definitions

CIS MariaDB 10.6 memiliki empat profile utama.

## 3.1 Level 1 - MariaDB RDBMS on Linux

Profile ini berlaku untuk MariaDB 10.6 yang berjalan di Linux dan mencakup pemeriksaan yang melibatkan sistem operasi. Targetnya:

- praktis;
- memberikan manfaat keamanan jelas;
- tidak mengganggu fungsi secara berlebihan.

Contohnya:

- permission `datadir`;
- service berjalan sebagai user `mysql`;
- shell login user `mysql` dinonaktifkan;
- database berada di partisi non-system;
- file log dan key memiliki permission ketat.

## 3.2 Level 2 - MariaDB RDBMS on Linux

Profile ini memperluas Level 1. Cocok untuk lingkungan dengan kebutuhan keamanan lebih tinggi. Biasanya:

- memberikan defense in depth;
- dapat berdampak pada performa atau kompatibilitas;
- membutuhkan pengujian lebih serius.

Contoh:

- sandboxing MariaDB;
- enkripsi binary dan relay log;
- pembatasan TLS versi dan cipher;
- data-at-rest encryption;
- lock account yang tidak digunakan.

## 3.3 Level 1 - MariaDB RDBMS

Profile ini berlaku untuk MariaDB sebagai RDBMS dan berisi check yang dapat dinilai dari koneksi MariaDB, tanpa memeriksa filesystem Linux. Cocok untuk audit jarak jauh.

Contoh:

- user privilege;
- password policy;
- anonymous account;
- wildcard hostname;
- TLS requirement;
- authentication plugin.

## 3.4 Level 2 - MariaDB RDBMS

Profile ini memperluas Level 1 RDBMS untuk lingkungan yang lebih ketat, tetapi tetap berfokus pada aspek yang dapat diaudit melalui MariaDB.

## 3.5 Cara memilih profile

Untuk pembelajaran dan baseline awal, gunakan Level 1. Untuk server produksi yang menyimpan data sensitif, Level 2 layak dipertimbangkan tetapi harus diuji.

Prinsipnya:

- **Level 1** = aman, wajar, relatif minim gangguan.
- **Level 2** = lebih ketat, cocok untuk data sensitif, bisa berdampak pada operasional.
- **RDBMS on Linux** = butuh akses OS.
- **RDBMS** = bisa dinilai melalui koneksi database.

---

# 4. Bab 1 - Operating System Level Configuration

Bab ini membahas hubungan antara MariaDB dan sistem operasi Linux tempat MariaDB berjalan.

## 4.1 Database pada partisi non-system

MariaDB menyimpan data utama pada `datadir`. Jika database berada di partisi sistem seperti `/`, `/var`, atau `/usr`, pertumbuhan database dapat menghabiskan ruang disk sistem operasi. Akibatnya sistem bisa mengalami denial of service karena OS tidak memiliki ruang untuk log, temporary file, package operation, atau proses kritis.

Memisahkan database ke partisi non-system membantu containment. Bila database tumbuh tidak terkendali, dampaknya lebih terisolasi ke partisi data, bukan seluruh OS.

## 4.2 Dedicated least privileged account

MariaDB sebaiknya berjalan dengan akun khusus, biasanya `mysql`, bukan `root`. Akun ini hanya digunakan untuk service MariaDB dan tidak memiliki hak administratif sistem.

Konsepnya adalah **least privilege**. Jika ada vulnerability pada MariaDB lalu attacker memperoleh eksekusi sebagai user service, dampaknya dibatasi oleh permission user tersebut. Bila MariaDB berjalan sebagai root, kompromi service dapat menjadi kompromi penuh sistem.

## 4.3 Command history MariaDB

Client MariaDB dapat menyimpan history perintah di file seperti `.mysql_history`. Masalahnya, administrator kadang tanpa sadar mengetik query yang berisi password, token, key, atau data sensitif. Jika file history terbaca, informasi tersebut bocor.

Menonaktifkan atau mengarahkan history ke `/dev/null` mengurangi risiko kebocoran data dari shell history database.

## 4.4 MYSQL_PWD environment variable

`MYSQL_PWD` memungkinkan password database disimpan sebagai environment variable. Ini berbahaya karena environment variable dapat terlihat melalui proses, debug, script, atau profil user.

Password sebaiknya tidak disimpan dalam bentuk cleartext di environment variable. Gunakan metode autentikasi yang lebih aman seperti socket authentication, certificate-based authentication, file konfigurasi user-specific dengan permission ketat, atau mekanisme secret management.

## 4.5 Interactive login user MariaDB

User service seperti `mysql` tidak perlu login interaktif. Jika shell login aktif, attacker yang memperoleh credential user service dapat login ke OS dan menjalankan perintah langsung.

Shell user service sebaiknya diarahkan ke `/bin/false` atau `/usr/sbin/nologin`.

## 4.6 Sandbox environment

Sandbox membatasi ruang gerak proses MariaDB. CIS menyebut opsi seperti `chroot`, systemd isolation, atau container. Tujuannya adalah membatasi akses MariaDB terhadap filesystem dan resource host.

Sandbox bukan pengganti patching atau permission, tetapi lapisan tambahan. Bila MariaDB dieksploitasi, sandbox membantu mencegah proses mengakses bagian host yang tidak diperlukan.

---

# 5. Bab 2 - Installation and Planning

Bab ini berisi perencanaan sebelum MariaDB dipakai sebagai service produksi.

## 5.1 Backup and Disaster Recovery

Backup adalah kontrol availability. Tanpa backup, insiden seperti corruption, ransomware, human error, atau hardware failure dapat menyebabkan kehilangan data permanen.

CIS membagi backup dan DR menjadi beberapa konsep:

### Backup policy

Backup policy mendefinisikan apa yang dibackup, kapan, oleh siapa, di mana disimpan, berapa lama retensinya, dan bagaimana diuji.

### Verify backups are good

Backup yang tidak pernah diuji bukan backup yang dapat dipercaya. Verifikasi backup memastikan file backup benar-benar bisa dipulihkan.

### Secure backup credentials

Akun backup sering membutuhkan privilege tinggi. Jika credential backup bocor, attacker dapat memperoleh akses luas ke database.

### Secure backup files

Backup berisi seluruh isi database. Karena itu backup harus diperlakukan seperti data produksi: permission ketat, enkripsi, dan akses terbatas.

### Point-in-time recovery

Binary log memungkinkan pemulihan sampai titik waktu tertentu setelah full backup terakhir. Ini penting untuk mengurangi kehilangan data.

### Disaster recovery plan

DR plan mencakup strategi pemulihan saat bencana besar, termasuk RTO, RPO, offsite backup, replication, dan kapasitas recovery site.

### Backup configuration and related files

Database tidak hanya terdiri dari data. Konfigurasi, certificate, key, audit log, plugin configuration, UDF, dan file custom juga harus masuk perencanaan backup agar restore lengkap.

## 5.2 Dedicated machine

Database server idealnya tidak dicampur dengan service lain yang tidak perlu. Semakin banyak service di host, semakin besar attack surface. Server database yang khusus lebih mudah diamankan, dipantau, dan disegmentasi.

## 5.3 Password tidak di command line

Menulis password di command line berisiko karena password dapat muncul di:

- shell history;
- process list;
- log job scheduler;
- script deployment;
- terminal recording.

Gunakan prompt password, file konfigurasi user-specific dengan permission 0600, certificate authentication, atau secret manager.

## 5.4 Tidak menggunakan username bersama

Satu akun database tidak seharusnya dipakai banyak aplikasi atau banyak orang. Reuse username membuat audit tidak jelas dan memperluas dampak kompromi.

Akun database harus dapat dikaitkan dengan satu aplikasi, satu sistem, atau satu identitas administratif.

## 5.5 Cryptographic material unik

Certificate, private key, dan material kriptografi harus unik untuk setiap instance. Menggunakan certificate default atau key yang dipakai banyak server meningkatkan risiko impersonation dan kompromi lintas sistem.

## 5.6 Password lifetime

CIS merekomendasikan `password_lifetime` tidak lebih dari 365 hari. Tujuannya membatasi durasi credential yang mungkin sudah bocor. Namun kebijakan ini harus selaras dengan mekanisme secret rotation dan jenis akun. Akun service perlu rotasi terkontrol agar tidak merusak aplikasi.

## 5.7 Lock account tidak aktif

Akun yang tidak digunakan sebaiknya dilock. Akun dormant adalah target menarik karena jarang dipantau dan mungkin masih memiliki privilege lama.

## 5.8 Socket peer-credential authentication

`unix_socket` authentication memungkinkan user OS lokal login ke MariaDB tanpa password, berdasarkan identitas user OS. Ini dapat berguna untuk root lokal atau DBA di server khusus, tetapi berbahaya jika akses OS tidak ketat.

Gunakan hanya bila:

- server memang dedicated;
- akses OS hanya untuk admin tepercaya;
- terminal tidak dibiarkan terbuka;
- remote login tidak bergantung pada socket authentication.

## 5.9 Bind address

MariaDB sebaiknya hanya mendengarkan pada alamat IP yang disetujui. Jika bind address kosong, `*`, atau `::`, service dapat menerima koneksi dari semua interface.

Pembatasan bind address adalah bentuk attack surface reduction.

## 5.10 TLS versions dan cipher

TLS versi lama seperti TLS 1.0 atau 1.1 tidak layak untuk baseline modern. MariaDB sebaiknya hanya menerima TLS yang disetujui, misalnya TLS 1.2 dan TLS 1.3, serta cipher yang kuat.

Namun pembatasan TLS harus diuji karena client lama bisa gagal koneksi.

## 5.11 Client certificate X.509

Client certificate menambah bukti identitas selain password. Pada lingkungan sensitif, user remote dapat diwajibkan menggunakan X.509 agar koneksi tidak hanya bergantung pada password.

---

# 6. Bab 3 - File Permissions

File permission adalah fondasi keamanan MariaDB di level OS. Database dapat memiliki konfigurasi aman secara SQL, tetapi tetap bocor jika file datadir, log, certificate, atau key dapat dibaca user biasa.

## 6.1 Permission datadir

`datadir` berisi database. Permission longgar dapat memungkinkan user OS membaca table file, file privilege, atau file internal MariaDB. CIS merekomendasikan akses terbatas untuk user dan group MariaDB.

## 6.2 Permission log files

MariaDB memiliki beberapa log:

- binary log;
- error log;
- slow query log;
- relay log;
- general log;
- audit log.

Log dapat berisi query, metadata, nama user, host, error, bahkan data sensitif. Karena itu permission harus dibatasi.

## 6.3 SSL key files

Private key TLS harus sangat ketat. Jika private key bocor, attacker dapat melakukan impersonation server atau serangan man-in-the-middle.

## 6.4 Plugin directory

Plugin directory berisi plugin atau UDF. Jika attacker dapat menulis ke plugin directory, ia dapat menempatkan code berbahaya yang dijalankan oleh MariaDB.

## 6.5 Audit log path

Audit log harus bisa ditulis oleh MariaDB, tetapi tidak boleh mudah dimodifikasi oleh user biasa. Integritas audit log penting untuk investigasi.

## 6.6 File key management plugin

Data-at-rest encryption membutuhkan file key management. File key dan filekey harus dilindungi karena menjadi akses ke data terenkripsi.

---

# 7. Bab 4 - General

Bab General membahas konfigurasi umum yang mengurangi attack surface dan meningkatkan keamanan runtime.

## 7.1 Security patches

MariaDB harus dipatch. Hardening konfigurasi tidak cukup jika server memiliki vulnerability yang sudah diketahui publik.

## 7.2 Hapus example/test database

Database contoh atau test pada server produksi menambah permukaan serangan dan dapat menjadi target enumerasi.

## 7.3 allow-suspicious-udfs

User Defined Function dapat memperluas kemampuan MariaDB dengan shared library. Jika `allow-suspicious-udfs` aktif, risiko pemuatan fungsi berbahaya meningkat.

## 7.4 local_infile

`local_infile` berkaitan dengan kemampuan membaca file dari client untuk dimasukkan ke database. Pada skenario tertentu, fitur ini bisa dieksploitasi melalui SQL injection untuk membaca file sensitif dari client atau server.

## 7.5 skip-grant-tables

`skip-grant-tables` membuat MariaDB berjalan tanpa sistem privilege. Ini hanya boleh digunakan dalam kondisi recovery sangat khusus, bukan produksi. Jika aktif, user dapat mengakses database tanpa kontrol privilege normal.

## 7.6 symbolic links

Symbolic link pada file database dapat mengarahkan operasi MariaDB ke lokasi lain. Ini berisiko terutama jika service memiliki permission luas.

## 7.7 secure_file_priv

`secure_file_priv` membatasi direktori yang boleh digunakan untuk operasi import/export file seperti `LOAD DATA INFILE` atau `SELECT ... INTO OUTFILE`. Jika tidak dibatasi, attacker dengan privilege tertentu dapat membaca atau menulis file di lokasi yang tidak seharusnya.

## 7.8 STRICT_ALL_TABLES

`STRICT_ALL_TABLES` membuat MariaDB menolak data invalid daripada diam-diam memotong atau menyesuaikan data. Ini meningkatkan integritas data dan mencegah aplikasi bergantung pada perilaku longgar.

## 7.9 Data-at-rest encryption

Enkripsi di level MariaDB melindungi data, binary log, temporary file, dan tablespace dari akses tidak sah pada storage atau backup. Ini menjadi lapisan tambahan di luar full disk encryption.

---

# 8. Bab 5 - MariaDB Permissions

Bab ini berfokus pada privilege database. Prinsip utamanya adalah **least privilege**.

## 8.1 Full database access hanya untuk admin

Akses penuh ke database internal seperti `mysql.*` memungkinkan user melihat atau mengubah privilege, password hash, dan konfigurasi internal. Akses ini hanya boleh untuk admin.

## 8.2 FILE privilege

`FILE` memungkinkan user membaca dan menulis file di host server dalam batas permission MariaDB. Ini sangat sensitif karena dapat digunakan untuk membaca file lokal atau membuat file berbahaya.

## 8.3 PROCESS privilege

`PROCESS` memungkinkan user melihat statement yang sedang berjalan milik user lain. Query dapat mengandung informasi sensitif.

## 8.4 SUPER privilege

`SUPER` sangat kuat. Ia berkaitan dengan banyak fungsi administratif seperti global variable, logging control, kill process, binary log, dan lainnya. MariaDB mendorong migrasi ke dynamic privileges yang lebih spesifik.

## 8.5 SHUTDOWN privilege

Privilege ini memungkinkan user mematikan server MariaDB. Bila diberikan ke user tidak tepat, availability dapat terganggu.

## 8.6 CREATE USER privilege

Privilege ini mengizinkan user membuat, menghapus, atau mengubah user lain. Ini harus sangat dibatasi.

## 8.7 GRANT OPTION

`GRANT OPTION` memungkinkan user memberikan privilege ke user lain. Ini bisa menjadi jalur privilege escalation.

## 8.8 REPLICATION SLAVE privilege

Privilege replication memungkinkan user mengambil binary log dari source. Binary log bisa berisi perubahan data sensitif. Berikan hanya untuk akun replication yang sah.

## 8.9 DML/DDL grants

Privilege seperti `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, dan `ALTER` harus dibatasi ke database tertentu dan user tertentu. Hindari grant global yang tidak perlu.

## 8.10 Stored procedures dan DEFINER/INVOKER

Stored procedure dan function dapat berjalan dengan konteks privilege tertentu. Jika `DEFINER` adalah user kuat, procedure bisa menjadi jalur eskalasi hak akses. Kode, definer, dan invoker harus direview.

---

# 9. Bab 6 - Auditing and Logging

Logging dan auditing membantu deteksi, investigasi, dan akuntabilitas.

## 9.1 Error log

`log_error` harus mengarah ke file, bukan hanya stderr. Logging ke console bersifat ephemeral dan sulit dikontrol permission-nya.

## 9.2 Log pada partisi non-system

Log MariaDB dapat tumbuh besar. Jika log berada di partisi sistem, log dapat memenuhi disk OS dan menyebabkan denial of service.

## 9.3 log_warnings

`log_warnings=2` membantu mencatat error dan warning, termasuk komunikasi error dan aborted connection yang dapat berguna mendeteksi perilaku mencurigakan.

## 9.4 Audit logging

Audit logging mencatat siapa melakukan apa dan kapan. Minimal event `CONNECT` perlu dicatat. Query dan table events dapat diaktifkan sesuai kebutuhan karena dapat menghasilkan log besar.

## 9.5 Audit plugin tidak boleh di-unload

Jika audit plugin dapat di-unload, attacker atau admin jahat dapat mematikan audit sementara lalu melakukan aksi tanpa tercatat. `FORCE_PLUS_PERMANENT` membuat plugin lebih sulit dimatikan.

## 9.6 Binary dan relay log terenkripsi

Binary log dan relay log dapat memuat data sensitif. Enkripsi log melindungi data at rest dan mendukung keamanan replication serta point-in-time recovery.

---

# 10. Bab 7 - Authentication

Authentication menentukan bagaimana user membuktikan identitas ke MariaDB.

## 10.1 mysql_old_password

`mysql_old_password` menggunakan mekanisme lama yang lemah dan rentan. CIS meminta penggunaannya dinonaktifkan, serta koneksi dengan plugin lama diblokir.

## 10.2 Password tidak disimpan di global configuration

File konfigurasi global biasanya dapat dibaca banyak user untuk mengambil default setting. Jika password disimpan di sana, credential dapat bocor.

## 10.3 Strong authentication

CIS menilai `mysql_native_password` dan `mysql_old_password` lemah dibanding alternatif yang lebih kuat. Untuk root lokal, `unix_socket` dapat dipilih. Untuk user lain, opsi seperti `ed25519`, `gssapi`, `pam`, atau `unix_socket` dapat dipertimbangkan sesuai kebutuhan.

## 10.4 Password complexity

CIS meminta password minimal 14 karakter dan dicek terhadap dictionary password lemah atau umum. MariaDB dapat memakai plugin seperti `simple_password_check` dan `cracklib_password_check`.

## 10.5 Wildcard hostname

User dengan host `%` dapat konek dari lokasi mana pun yang diizinkan jaringan. Ini memperluas attack surface. Lebih aman menggunakan host atau IP spesifik.

## 10.6 Anonymous account

Anonymous account adalah user dengan nama kosong. Karena tidak merepresentasikan identitas jelas, akun ini harus dihapus atau diberi nama jelas.

---

# 11. Bab 8 - Network

Bab ini membahas keamanan komunikasi jaringan MariaDB.

## 11.1 require_secure_transport dan have_ssl

`require_secure_transport=ON` memastikan koneksi tidak aman ditolak. `have_ssl=YES` memastikan TLS tersedia. Ini melindungi credential dan data dari sniffing dan man-in-the-middle.

## 11.2 ssl_type untuk remote users

User remote harus diwajibkan memakai SSL/TLS. Nilai `ANY`, `X509`, atau `SPECIFIED` menunjukkan adanya requirement TLS. `X509` dan `SPECIFIED` lebih kuat karena melibatkan certificate.

## 11.3 Maximum connection limits

Connection limit membantu mengurangi risiko denial of service. Limit dapat diterapkan global (`max_connections`, `max_user_connections`) atau per user.

---

# 12. Bab 9 - Replication

Replication memindahkan data dari primary ke replica. Karena replication membawa data sensitif, jalur replication harus diamankan.

## 12.1 Replication traffic secured

Replication traffic harus memiliki confidentiality, integrity, dan mutual authentication. Bisa menggunakan private network, VPN, TLS, atau SSH tunnel.

## 12.2 Server certificate verification

Replica harus memverifikasi certificate primary agar tidak tersambung ke server palsu. Ini mencegah man-in-the-middle dan impersonation.

## 12.3 Replication user tanpa SUPER

Replication user tidak boleh memiliki `SUPER`. Berikan privilege spesifik yang diperlukan, bukan privilege administratif luas.

## 12.4 Approved replication ciphers

Jika replication menggunakan TLS, cipher harus diset ke algoritma yang disetujui. Cipher lemah dapat melemahkan perlindungan data in transit.

## 12.5 Mutual TLS

Mutual TLS membuat primary dan replica saling mengautentikasi. Primary memverifikasi certificate replica, dan replica memverifikasi certificate primary. Ini mencegah replica tidak sah mengambil data.

---

# 13. Ringkasan Pola Berpikir Hardening MariaDB

CIS MariaDB dapat dipahami melalui beberapa pola besar:

## 13.1 Pisahkan data dari sistem

Database dan log tidak boleh memenuhi filesystem OS. Gunakan partisi atau volume terpisah.

## 13.2 Jalankan service dengan privilege minimum

Service harus berjalan sebagai user khusus, tanpa shell login, tanpa hak admin.

## 13.3 Jangan simpan credential sembarangan

Hindari password di command line, environment variable, global config, history file, dan backup tidak terenkripsi.

## 13.4 Batasi privilege database

Setiap user hanya diberi privilege sesuai kebutuhan. Hindari `SUPER`, `FILE`, `PROCESS`, `GRANT OPTION`, dan privilege global untuk non-admin.

## 13.5 Lindungi file di OS

Data, log, SSL key, plugin, dan file key management harus memiliki permission ketat.

## 13.6 Enkripsi data in transit dan at rest

TLS melindungi koneksi jaringan. Data-at-rest encryption melindungi storage, backup, binary log, dan temporary file.

## 13.7 Aktifkan audit

Tanpa audit, insiden sulit diselidiki. Audit plugin harus aktif dan tidak mudah dimatikan.

## 13.8 Uji sebelum produksi

Hardening dapat memutus aplikasi. Selalu lakukan:

1. backup;
2. test environment;
3. pilot deployment;
4. monitoring;
5. rollback plan.

---

# 14. Checklist Konseptual

| Area | Pertanyaan Konseptual |
|---|---|
| OS | Apakah MariaDB berjalan sebagai user khusus, bukan root? |
| OS | Apakah datadir dan log tidak berada di partisi sistem? |
| OS | Apakah user `mysql` tidak bisa login interaktif? |
| Credential | Apakah password tidak tersimpan di command line, environment variable, atau global config? |
| Backup | Apakah backup policy, restore test, dan DR plan sudah ada? |
| File | Apakah datadir, log, SSL key, plugin, dan key file terlindungi permission-nya? |
| General | Apakah test database, symbolic links, `local_infile`, dan `skip-grant-tables` dikendalikan? |
| Privilege | Apakah privilege non-admin sudah dibatasi? |
| Authentication | Apakah anonymous account dan wildcard host tidak ada? |
| Authentication | Apakah password policy dan authentication plugin kuat digunakan? |
| Network | Apakah TLS diwajibkan untuk koneksi remote? |
| Logging | Apakah error log dan audit log aktif serta aman? |
| Replication | Apakah replication traffic terenkripsi dan menggunakan mutual authentication? |
| Operations | Apakah semua exception terdokumentasi? |

---

## Penutup

CIS MariaDB 10.6 Benchmark mengajarkan bahwa keamanan database bukan hanya konfigurasi SQL. Keamanan MariaDB adalah gabungan dari arsitektur, sistem operasi, permission file, user privilege, authentication, network encryption, logging, backup, dan recovery. Tujuan akhirnya adalah membuat database lebih tahan terhadap kesalahan konfigurasi, penyalahgunaan privilege, kebocoran credential, serangan jaringan, dan kehilangan data.
