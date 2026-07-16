# Implementasi Hardening MariaDB 10.6 Berbasis CIS Benchmark v1.1.0

**Sumber utama:** CIS MariaDB 10.6 Benchmark v1.1.0 - 03-29-2024  
**Jenis dokumen:** Panduan implementasi teknis  
**Target:** MariaDB 10.6 pada Linux, terutama Debian/Ubuntu-family  
**Mode:** Instalasi, konfigurasi, audit, remediation, dan validasi post-hardening

> Peringatan: Panduan ini harus diuji di lab terlebih dahulu. Jangan langsung menerapkan ke server produksi tanpa backup, snapshot, maintenance window, dan rollback plan. Beberapa konfigurasi seperti TLS wajib, perubahan datadir, encryption at rest, password plugin, dan privilege revocation dapat memutus aplikasi.

---

## Daftar Isi

1. Asumsi dan Standar Lingkungan
2. Tahap Persiapan
3. Instalasi MariaDB 10.6
4. Struktur Direktori dan Partisi Hardening
5. Konfigurasi Service Account dan OS-Level Hardening
6. Konfigurasi File Utama CIS MariaDB
7. TLS Certificate dan Secure Transport
8. Data-at-Rest Encryption dan Encrypted Logs
9. Authentication dan Password Policy
10. User, Privilege, dan Access Control
11. Logging dan Audit Plugin
12. Network dan Connection Limits
13. Backup dan Disaster Recovery
14. Replication Security
15. File Permission Audit dan Remediation
16. Validasi Akhir
17. Exception Register
18. Checklist Implementasi

---

# 1. Asumsi dan Standar Lingkungan

Panduan ini memakai asumsi berikut:

- sistem operasi berbasis Linux;
- contoh command memakai Debian/Ubuntu-family;
- MariaDB yang ditargetkan adalah MariaDB 10.6;
- service berjalan sebagai user `mysql` dan group `mysql`;
- file konfigurasi utama berada di `/etc/mysql/` atau `/etc/mysql/mariadb.conf.d/`;
- service systemd bernama `mariadb.service`;
- administrator bekerja dengan hak root atau sudo;
- server dipakai sebagai database server, bukan workstation umum.

Gunakan variabel berikut sebagai contoh:

```bash
DB_DATA_DIR="/srv/mariadb/data"
DB_LOG_DIR="/srv/mariadb/logs"
DB_BACKUP_DIR="/srv/mariadb/backups"
DB_IMPORT_DIR="/srv/mariadb/import"
DB_SSL_DIR="/etc/mysql/ssl"
DB_ENC_DIR="/etc/mysql/encryption"
CIS_CNF="/etc/mysql/mariadb.conf.d/60-cis-hardening.cnf"
```

Catatan distribusi:

- Pada Debian/Ubuntu, file konfigurasi sering berada di `/etc/mysql/mariadb.conf.d/50-server.cnf`.
- Pada distribusi lain, path dapat berbeda.
- Jika Ubuntu versi baru menyediakan MariaDB selain 10.6, benchmark ini tidak lagi 1:1. Cocokkan versi MariaDB dengan requirement tugas atau gunakan repository yang menyediakan MariaDB 10.6 sesuai kebijakan.

---

# 2. Tahap Persiapan

## 2.1 Update sistem

```bash
sudo apt update
sudo apt -y upgrade
sudo apt -y install ca-certificates curl gnupg lsb-release openssl rsync acl
```

## 2.2 Buat snapshot atau backup sebelum hardening

Untuk VM, buat snapshot dulu. Untuk server fisik, minimal backup konfigurasi:

```bash
sudo mkdir -p /root/pre-cis-backup
sudo cp -a /etc/mysql /root/pre-cis-backup/mysql-etc-$(date +%F) 2>/dev/null || true
sudo systemctl status mariadb --no-pager || true
```

## 2.3 Cek versi MariaDB

```bash
mariadb --version || true
mysql --version || true
```

Pastikan major/minor version sesuai target 10.6 jika audit harus mengikuti CIS MariaDB 10.6 secara ketat.

---

# 3. Instalasi MariaDB 10.6

## 3.1 Instal paket MariaDB

Jika repository OS sudah menyediakan MariaDB 10.6:

```bash
sudo apt update
sudo apt -y install mariadb-server mariadb-client mariadb-backup
```

Jika repository OS menyediakan versi lain, gunakan repository resmi/vendor yang disetujui organisasi untuk MariaDB 10.6. Setelah instalasi:

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb --no-pager
mariadb --version
```

## 3.2 Jalankan secure installation

```bash
sudo mariadb-secure-installation
```

Fokus jawaban:

- hapus anonymous user;
- hapus test database;
- nonaktifkan remote root password login bila tidak diperlukan;
- reload privilege tables.

Catatan: Jika root memakai `unix_socket`, login root lokal dapat dilakukan dengan `sudo mariadb`.

---

# 4. Struktur Direktori dan Partisi Hardening

CIS merekomendasikan database dan log tidak berada pada partisi sistem. Contoh implementasi menggunakan `/srv/mariadb`.

## 4.1 Buat direktori

```bash
sudo mkdir -p /srv/mariadb/{data,logs,backups,import}
sudo mkdir -p /etc/mysql/{ssl,encryption}
sudo chown -R mysql:mysql /srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import
sudo chown -R root:root /srv/mariadb/backups
sudo chmod 750 /srv/mariadb/data
sudo chmod 750 /srv/mariadb/logs
sudo chmod 750 /srv/mariadb/import
sudo chmod 700 /srv/mariadb/backups
```

## 4.2 Pindahkan datadir ke partisi non-system

Pastikan database sudah dibackup. Lalu:

```bash
sudo systemctl stop mariadb
sudo rsync -aHAX --numeric-ids /var/lib/mysql/ /srv/mariadb/data/
sudo chown -R mysql:mysql /srv/mariadb/data
sudo chmod 750 /srv/mariadb/data
```

Tambahkan konfigurasi `datadir` pada file CIS hardening nanti:

```ini
datadir=/srv/mariadb/data
```

## 4.3 Sesuaikan AppArmor jika aktif

Pada Ubuntu, AppArmor dapat memblokir datadir baru. Edit profile MariaDB, misalnya:

```bash
sudo nano /etc/apparmor.d/usr.sbin.mariadbd
```

Tambahkan akses:

```text
/srv/mariadb/data/ r,
/srv/mariadb/data/** rwk,
/srv/mariadb/logs/ r,
/srv/mariadb/logs/** rwk,
/srv/mariadb/import/ r,
/srv/mariadb/import/** rwk,
/etc/mysql/encryption/ r,
/etc/mysql/encryption/** r,
/etc/mysql/ssl/ r,
/etc/mysql/ssl/** r,
```

Reload AppArmor:

```bash
sudo systemctl reload apparmor || sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.mariadbd
```

---

# 5. Konfigurasi Service Account dan OS-Level Hardening

## 5.1 Pastikan MariaDB berjalan sebagai user mysql

```bash
ps -ef | grep -E 'mariadbd|mysqld' | grep -v grep
getent passwd mysql
```

Jika shell login masih aktif:

```bash
sudo usermod -s /usr/sbin/nologin mysql 2>/dev/null || sudo usermod -s /bin/false mysql
```

Validasi:

```bash
getent passwd mysql
```

## 5.2 Nonaktifkan MariaDB command history

Untuk root:

```bash
sudo rm -f /root/.mysql_history /root/.mariadb_history
sudo ln -sf /dev/null /root/.mysql_history
sudo ln -sf /dev/null /root/.mariadb_history
```

Untuk user interaktif:

```bash
for d in /home/*; do
  [ -d "$d" ] || continue
  sudo rm -f "$d/.mysql_history" "$d/.mariadb_history"
  sudo ln -sf /dev/null "$d/.mysql_history"
  sudo ln -sf /dev/null "$d/.mariadb_history"
  sudo chown -h "$(basename "$d")":"$(basename "$d")" "$d/.mysql_history" "$d/.mariadb_history" 2>/dev/null || true
done
```

## 5.3 Pastikan MYSQL_PWD tidak digunakan

Audit proses:

```bash
sudo grep -a MYSQL_PWD /proc/*/environ 2>/dev/null || true
```

Audit profile user:

```bash
sudo grep -R "MYSQL_PWD" /root /home /etc/profile /etc/profile.d 2>/dev/null || true
```

Jika ditemukan, hapus dan ganti dengan mekanisme lebih aman seperti `unix_socket`, X.509 certificate, atau file user-specific dengan permission 0600.

## 5.4 Systemd sandboxing opsional

Buat override systemd:

```bash
sudo systemctl edit mariadb
```

Isi contoh:

```ini
[Service]
User=mysql
Group=mysql
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
ReadWritePaths=/srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import /run/mysqld /var/run/mysqld /etc/mysql/encryption
```

Reload dan restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart mariadb
```

Jika MariaDB gagal start, cek:

```bash
journalctl -u mariadb -xe --no-pager
```

---

# 6. Konfigurasi File Utama CIS MariaDB

Buat file konfigurasi CIS:

```bash
sudo tee /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf >/dev/null <<'CONF'
[mariadb]
# CIS MariaDB 10.6 hardening baseline

# OS/data placement
datadir=/srv/mariadb/data

# Network binding - sesuaikan dengan desain jaringan
# Gunakan 127.0.0.1 untuk local-only atau IP database segment yang disetujui.
bind_address=127.0.0.1

# General hardening
local-infile=0
skip-grant-tables=FALSE
skip-symbolic-links
secure_file_priv=/srv/mariadb/import
old_passwords=0
secure_auth=ON
default_password_lifetime=365

# Strict SQL mode
sql_mode=STRICT_ALL_TABLES,ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

# Logging
log_error=/srv/mariadb/logs/mariadb.err
log_warnings=2

# Secure transport
require_secure_transport=ON
tls_version=TLSv1.2,TLSv1.3
# Sesuaikan cipher dengan policy organisasi dan library TLS yang tersedia.
ssl_cipher=ECDHE-ECDSA-AES128-GCM-SHA256

# Connection limits - sesuaikan dengan workload
max_connections=300
max_user_connections=50

# Audit plugin
plugin_load_add=server_audit
server_audit=FORCE_PLUS_PERMANENT
server_audit_logging=ON
server_audit_events=CONNECT
server_audit_file_path=/srv/mariadb/logs/server_audit.log

# Password validation plugins
plugin_load_add=simple_password_check
simple_password_check=FORCE_PLUS_PERMANENT
simple_password_check_minimal_length=14
plugin_load_add=cracklib_password_check
cracklib_password_check=FORCE_PLUS_PERMANENT
strict_password_validation=ON

# Data at rest encryption - aktifkan setelah file key dibuat
plugin_load_add=file_key_management
file_key_management_filename=/etc/mysql/encryption/keyfile.enc
file_key_management_filekey=FILE:/etc/mysql/encryption/keyfile.key
encrypt_binlog=ON
innodb_encrypt_log=ON
encrypt_tmp_files=ON
innodb_encrypt_temporary_tables=ON
innodb_encrypt_tables=ON
CONF
```

Set permission:

```bash
sudo chown root:root /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf
sudo chmod 640 /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf
```

Catatan penting:

- Jangan restart MariaDB dulu jika TLS key dan encryption key belum dibuat.
- Jika plugin tertentu belum tersedia, MariaDB bisa gagal start. Lakukan testing bertahap.

---

# 7. TLS Certificate dan Secure Transport

## 7.1 Buat CA dan certificate server

```bash
sudo mkdir -p /etc/mysql/ssl
cd /etc/mysql/ssl

sudo openssl genrsa 4096 | sudo tee ca-key.pem >/dev/null
sudo openssl req -new -x509 -nodes -days 3650 \
  -key ca-key.pem \
  -out ca.pem \
  -subj "/CN=MariaDB-CIS-Local-CA"

sudo openssl req -newkey rsa:4096 -days 3650 -nodes \
  -keyout server-key.pem \
  -out server-req.pem \
  -subj "/CN=mariadb-server"

sudo openssl x509 -req -in server-req.pem -days 3650 \
  -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
  -out server-cert.pem
```

## 7.2 Buat certificate client untuk X.509 atau mTLS

```bash
sudo openssl req -newkey rsa:4096 -days 3650 -nodes \
  -keyout client-key.pem \
  -out client-req.pem \
  -subj "/CN=mariadb-client"

sudo openssl x509 -req -in client-req.pem -days 3650 \
  -CA ca.pem -CAkey ca-key.pem -set_serial 02 \
  -out client-cert.pem
```

## 7.3 Permission TLS files

```bash
sudo chown -R mysql:mysql /etc/mysql/ssl
sudo chmod 750 /etc/mysql/ssl
sudo chmod 400 /etc/mysql/ssl/*key.pem
sudo chmod 444 /etc/mysql/ssl/*cert.pem /etc/mysql/ssl/ca.pem
```

## 7.4 Tambahkan TLS path ke konfigurasi

```bash
sudo tee -a /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf >/dev/null <<'CONF'

# TLS material
ssl-ca=/etc/mysql/ssl/ca.pem
ssl-cert=/etc/mysql/ssl/server-cert.pem
ssl-key=/etc/mysql/ssl/server-key.pem
CONF
```

---

# 8. Data-at-Rest Encryption dan Encrypted Logs

## 8.1 Buat key management files

```bash
sudo mkdir -p /etc/mysql/encryption
sudo bash -c '(echo -n "1;" ; openssl rand -hex 32) > /etc/mysql/encryption/keyfile'
sudo openssl rand -hex 128 | sudo tee /etc/mysql/encryption/keyfile.key >/dev/null
sudo openssl enc -aes-256-cbc -md sha1 \
  -pass file:/etc/mysql/encryption/keyfile.key \
  -in /etc/mysql/encryption/keyfile \
  -out /etc/mysql/encryption/keyfile.enc
sudo rm -f /etc/mysql/encryption/keyfile
sudo chown -R mysql:mysql /etc/mysql/encryption
sudo chmod 750 /etc/mysql/encryption
sudo chmod 640 /etc/mysql/encryption/keyfile*
```

## 8.2 Restart bertahap

```bash
sudo systemctl restart mariadb
sudo systemctl status mariadb --no-pager
```

Jika gagal, cek:

```bash
journalctl -u mariadb -xe --no-pager
sudo tail -n 100 /srv/mariadb/logs/mariadb.err
```

## 8.3 Validasi encryption variable

```bash
sudo mariadb -e "SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE variable_name LIKE '%ENCRYPT%';"
```

Untuk tabel yang sudah ada, enkripsi dapat diterapkan per tabel:

```sql
ALTER TABLE nama_database.nama_tabel ENCRYPTED=YES ENCRYPTION_KEY_ID=1;
```

Jalankan bertahap karena operasi ini dapat mengunci tabel.

---

# 9. Authentication dan Password Policy

## 9.1 Instal dan aktifkan plugin password check

Masuk ke MariaDB:

```bash
sudo mariadb
```

Jalankan:

```sql
INSTALL SONAME 'simple_password_check';
INSTALL SONAME 'cracklib_password_check';
SHOW PLUGINS;
SHOW VARIABLES LIKE '%pass%';
```

Jika `cracklib_password_check` gagal karena plugin tidak tersedia, instal paket plugin yang sesuai dengan repository MariaDB/OS, lalu ulangi.

## 9.2 Pastikan old password plugin tidak dipakai

```sql
SHOW VARIABLES WHERE Variable_name = 'old_passwords';
SHOW VARIABLES WHERE Variable_name = 'secure_auth';
```

Harapan:

```text
old_passwords = OFF
secure_auth = ON
```

## 9.3 Root memakai unix_socket

```sql
ALTER USER 'root'@'localhost' IDENTIFIED VIA unix_socket;
```

## 9.4 Password lifetime

```sql
SET GLOBAL default_password_lifetime=365;
```

Agar persisten, konfigurasi sudah ditulis di file `60-cis-hardening.cnf`.

## 9.5 Audit akun dengan authentication lemah

```sql
SELECT User,host
FROM mysql.user
WHERE (plugin IN('mysql_native_password', 'mysql_old_password','')
  AND NOT (User = 'root' AND authentication_string = 'invalid')
  AND NOT (User = 'mysql' and authentication_string = 'invalid'));
```

Untuk user aplikasi, pertimbangkan:

```sql
-- Contoh, sesuaikan user dan host
ALTER USER 'app_user'@'10.10.10.20' IDENTIFIED VIA ed25519 USING PASSWORD('GantiPasswordSangatKuat');
```

Jika plugin `ed25519` belum tersedia:

```sql
INSTALL SONAME 'auth_ed25519';
```

## 9.6 Hapus anonymous account

Audit:

```sql
SELECT user,host FROM mysql.user WHERE user = '';
```

Remediasi contoh:

```sql
DROP USER ''@'localhost';
DROP USER ''@'%';
FLUSH PRIVILEGES;
```

Jika host berbeda, sesuaikan dengan hasil audit.

## 9.7 Hilangkan wildcard hostname

Audit:

```sql
SELECT user, host FROM mysql.user WHERE host = '%';
```

Remediasi konsep:

```sql
-- Buat user baru dengan host spesifik
CREATE USER 'app_user'@'10.10.10.20' IDENTIFIED BY 'GantiPasswordSangatKuat';
GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'app_user'@'10.10.10.20';

-- Setelah aplikasi berhasil memakai user baru, hapus user wildcard
DROP USER 'app_user'@'%';
FLUSH PRIVILEGES;
```

---

# 10. User, Privilege, dan Access Control

## 10.1 Audit full database access

```sql
SELECT * FROM information_schema.user_privileges
WHERE grantee NOT LIKE ('\'mysql.%localhost\'');
```

Review hasilnya. Hanya admin yang boleh punya privilege luas.

## 10.2 Audit privilege sensitif

```sql
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'FILE';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'PROCESS';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'SUPER';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'SHUTDOWN';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'CREATE USER';
SELECT DISTINCT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE IS_GRANTABLE = 'YES';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'REPLICATION SLAVE';
```

## 10.3 Revoke privilege untuk non-admin

Contoh:

```sql
REVOKE FILE ON *.* FROM 'app_user'@'10.10.10.20';
REVOKE PROCESS ON *.* FROM 'app_user'@'10.10.10.20';
REVOKE SUPER ON *.* FROM 'app_user'@'10.10.10.20';
REVOKE SHUTDOWN ON *.* FROM 'app_user'@'10.10.10.20';
REVOKE CREATE USER ON *.* FROM 'app_user'@'10.10.10.20';
REVOKE GRANT OPTION ON *.* FROM 'app_user'@'10.10.10.20';
```

## 10.4 Audit DML/DDL grants

```sql
SELECT User,Host,Db
FROM mysql.db
WHERE Select_priv='Y'
 OR Insert_priv='Y'
 OR Update_priv='Y'
 OR Delete_priv='Y'
 OR Create_priv='Y'
 OR Drop_priv='Y'
 OR Alter_priv='Y';
```

Pastikan setiap hasil memang dibutuhkan. Hindari grant global jika cukup database-level.

## 10.5 Review stored procedure dan function

```sql
SHOW PROCEDURE STATUS;
SHOW FUNCTION STATUS;
SELECT * FROM information_schema.routines;
SELECT * FROM information_schema.parameters;
```

Untuk procedure/function yang memakai `DEFINER` user kuat, review ulang kode dan kebutuhan privilege.

---

# 11. Logging dan Audit Plugin

## 11.1 Validasi error log

```sql
SHOW variables LIKE 'log_error';
SHOW GLOBAL VARIABLES LIKE 'log_warnings';
```

Harapan:

```text
log_error = /srv/mariadb/logs/mariadb.err
log_warnings = 2
```

## 11.2 Validasi audit plugin

```sql
SHOW VARIABLES LIKE '%audit%';
SELECT LOAD_OPTION FROM information_schema.plugins WHERE PLUGIN_NAME='SERVER_AUDIT';
```

Harapan:

```text
server_audit_logging = ON
server_audit_events = CONNECT
LOAD_OPTION = FORCE_PLUS_PERMANENT
```

## 11.3 Permission audit log

```bash
sudo touch /srv/mariadb/logs/server_audit.log
sudo chown mysql:mysql /srv/mariadb/logs/server_audit.log
sudo chmod 660 /srv/mariadb/logs/server_audit.log
```

## 11.4 Permission error log

```bash
sudo touch /srv/mariadb/logs/mariadb.err
sudo chown mysql:mysql /srv/mariadb/logs/mariadb.err
sudo chmod 600 /srv/mariadb/logs/mariadb.err
```

## 11.5 Validasi binary log encryption

```sql
SELECT VARIABLE_NAME, VARIABLE_VALUE, 'BINLOG - At Rest Encryption' as Note
FROM information_schema.global_variables
WHERE variable_name LIKE '%ENCRYPT_LOG%';
```

Harapan: nilai terkait encryption log adalah `ON`.

---

# 12. Network dan Connection Limits

## 12.1 Validasi bind address

```sql
SELECT VARIABLE_NAME, VARIABLE_VALUE
FROM information_schema.global_variables
WHERE VARIABLE_NAME = 'bind_address';
```

Pastikan tidak kosong, bukan `*`, dan bukan `::`.

## 12.2 Validasi secure transport

```sql
SELECT @@require_secure_transport;
SHOW variables WHERE variable_name = 'have_ssl';
SELECT @@tls_version;
```

Harapan:

```text
require_secure_transport = 1
have_ssl = YES
tls_version tidak memuat TLSv1 atau TLSv1.1
```

## 12.3 Wajibkan TLS untuk user remote

Audit:

```sql
SELECT user, host, ssl_type FROM mysql.user
WHERE NOT HOST IN ('::1', '127.0.0.1', 'localhost');
```

Remediasi:

```sql
ALTER USER 'app_user'@'10.10.10.20' REQUIRE SSL;
```

Untuk X.509:

```sql
ALTER USER 'app_user'@'10.10.10.20' REQUIRE X509;
```

## 12.4 Connection limits

Audit:

```sql
SELECT VARIABLE_NAME, VARIABLE_VALUE
FROM information_schema.global_variables
WHERE VARIABLE_NAME LIKE 'max_%connections';

SELECT user, host, max_connections, max_user_connections
FROM mysql.user
WHERE user NOT LIKE 'mysql.%' AND user NOT LIKE 'root';
```

Remediasi global:

```sql
SET GLOBAL max_user_connections=50;
SET GLOBAL max_connections=300;
```

Remediasi per user:

```sql
ALTER USER 'app_user'@'10.10.10.20'
WITH MAX_CONNECTIONS_PER_HOUR 1000
MAX_USER_CONNECTIONS 20;
```

---

# 13. Backup dan Disaster Recovery

## 13.1 Backup policy minimal

Buat direktori backup:

```bash
sudo mkdir -p /srv/mariadb/backups
sudo chown root:root /srv/mariadb/backups
sudo chmod 700 /srv/mariadb/backups
```

## 13.2 Backup terenkripsi dengan mariabackup

Contoh backup stream terenkripsi:

```bash
sudo mariabackup --backup --user=root --stream=xbstream \
  | openssl enc -aes-256-cbc -pbkdf2 -salt -out /srv/mariadb/backups/full-$(date +%F).xb.enc
```

Catatan: Jangan menyimpan password enkripsi di script tanpa proteksi. Gunakan secret manager atau mekanisme aman organisasi.

## 13.3 Backup konfigurasi dan material penting

```bash
sudo tar --xattrs --acls -czf /srv/mariadb/backups/mysql-config-$(date +%F).tar.gz \
  /etc/mysql /etc/apparmor.d/usr.sbin.mariadbd /srv/mariadb/logs 2>/dev/null
sudo chmod 600 /srv/mariadb/backups/mysql-config-$(date +%F).tar.gz
```

Pastikan backup mencakup:

- konfigurasi MariaDB;
- TLS certificate dan key;
- encryption key management file;
- audit configuration;
- script operasional;
- dokumentasi restore.

## 13.4 Test restore

Minimal lakukan restore test berkala di server lab:

```bash
# Contoh proses umum, sesuaikan dengan metode backup
openssl enc -d -aes-256-cbc -pbkdf2 -in full-YYYY-MM-DD.xb.enc -out restore.xbstream
mbstream -x < restore.xbstream
mariabackup --prepare --target-dir=/path/restore
```

Dokumentasikan hasil restore test.

---

# 14. Replication Security

Bagian ini hanya diterapkan jika replication digunakan.

## 14.1 Amankan replication traffic

Opsi yang disetujui:

- private network;
- VPN;
- TLS;
- SSH tunnel.

Untuk TLS replication pada replica:

```sql
STOP REPLICA;
CHANGE MASTER TO MASTER_SSL=1;
START REPLICA;
```

## 14.2 Verifikasi certificate primary

```sql
SHOW REPLICA STATUS\G;
```

Remediasi:

```sql
STOP REPLICA;
CHANGE MASTER TO MASTER_SSL_VERIFY_SERVER_CERT=1;
START REPLICA;
```

## 14.3 Replication user tidak boleh SUPER

Audit:

```sql
SELECT user, host FROM mysql.user WHERE user='repl' AND Super_priv = 'Y';
```

Remediasi:

```sql
REVOKE SUPER ON *.* FROM 'repl'@'replica1.example.com';
```

## 14.4 Approved replication cipher

```sql
SHOW REPLICA STATUS\G;
```

Remediasi:

```sql
STOP REPLICA;
CHANGE MASTER TO MASTER_SSL_CIPHER='ECDHE-ECDSA-AES128-GCM-SHA256';
START REPLICA;
```

## 14.5 Mutual TLS

Pada replica:

```sql
STOP REPLICA;
CHANGE MASTER TO
  MASTER_SSL_CERT='/etc/mysql/ssl/client-cert.pem',
  MASTER_SSL_KEY='/etc/mysql/ssl/client-key.pem';
START REPLICA;
```

Pada primary:

```sql
ALTER USER 'repl'@'replica1.example.com' REQUIRE X509;
```

---

# 15. File Permission Audit dan Remediation

## 15.1 Datadir

Audit:

```sql
SHOW VARIABLES WHERE variable_name = 'datadir';
```

```bash
sudo ls -ld /srv/mariadb/data
```

Remediasi:

```bash
sudo chown mysql:mysql /srv/mariadb/data
sudo chmod 750 /srv/mariadb/data
```

## 15.2 Log files

```bash
sudo chown mysql:mysql /srv/mariadb/logs/*.log /srv/mariadb/logs/*.err 2>/dev/null || true
sudo chmod 660 /srv/mariadb/logs/*.log 2>/dev/null || true
sudo chmod 600 /srv/mariadb/logs/*.err 2>/dev/null || true
```

## 15.3 SSL keys

```bash
sudo chown mysql:mysql /etc/mysql/ssl/*
sudo chmod 400 /etc/mysql/ssl/*key.pem
sudo chmod 444 /etc/mysql/ssl/*cert.pem /etc/mysql/ssl/ca.pem
```

## 15.4 Plugin directory

Audit:

```sql
SHOW VARIABLES WHERE variable_name = 'plugin_dir';
```

Lalu jalankan sesuai nilai `plugin_dir`:

```bash
PLUGIN_DIR="/usr/lib/mysql/plugin"   # sesuaikan hasil audit
sudo chown mysql:mysql "$PLUGIN_DIR"
sudo chmod 550 "$PLUGIN_DIR"
```

## 15.5 File key management

```bash
sudo chown -R mysql:mysql /etc/mysql/encryption
sudo chmod 750 /etc/mysql/encryption
sudo chmod 640 /etc/mysql/encryption/keyfile*
```

---

# 16. Validasi Akhir

Jalankan audit gabungan berikut.

```bash
sudo systemctl status mariadb --no-pager
sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true
sudo ls -ld /srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import /srv/mariadb/backups
sudo ls -l /srv/mariadb/logs
sudo ls -l /etc/mysql/ssl
sudo ls -l /etc/mysql/encryption
```

SQL audit:

```sql
SHOW VARIABLES WHERE Variable_name IN ('old_passwords','secure_auth','have_ssl','require_secure_transport','tls_version','ssl_cipher','bind_address','default_password_lifetime','log_error','log_warnings','secure_file_priv','sql_mode');
SHOW VARIABLES LIKE '%audit%';
SHOW PLUGINS;
SELECT LOAD_OPTION FROM information_schema.plugins WHERE PLUGIN_NAME='SERVER_AUDIT';
SELECT user,host FROM mysql.user WHERE user = '';
SELECT user,host FROM mysql.user WHERE host = '%';
SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE IN ('FILE','PROCESS','SUPER','SHUTDOWN','CREATE USER','REPLICATION SLAVE');
SELECT DISTINCT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE IS_GRANTABLE = 'YES';
SELECT user, host, ssl_type FROM mysql.user WHERE NOT HOST IN ('::1','127.0.0.1','localhost');
```

---

# 17. Exception Register

Gunakan tabel ini untuk mencatat pengecualian.

| No | Rekomendasi CIS | Kondisi Saat Ini | Alasan Exception | Risiko | Mitigasi | Owner | Review Date |
|---|---|---|---|---|---|---|---|
| 1 | Contoh: bind_address tidak local-only | MariaDB listen di IP LAN | Aplikasi remote perlu koneksi | Exposure port 3306 | Firewall + TLS + user host spesifik | DBA | YYYY-MM-DD |
| 2 | Contoh: general_log aktif | General log ON | Debug sementara | Log besar/sensitif | Aktif 1 hari, permission 600 | DBA | YYYY-MM-DD |

---

# 18. Checklist Implementasi

| Area | Audit | Status |
|---|---|---|
| Version | MariaDB 10.6 terpasang | [ ] |
| Service Account | MariaDB berjalan sebagai `mysql` | [ ] |
| Login Shell | User `mysql` tidak punya interactive shell | [ ] |
| Datadir | Datadir berada di non-system partition | [ ] |
| Datadir Permission | Datadir `750 mysql:mysql` | [ ] |
| MYSQL_PWD | Tidak ada `MYSQL_PWD` di process/profile | [ ] |
| History | `.mysql_history` diarahkan ke `/dev/null` | [ ] |
| Config | Password tidak ada di global config | [ ] |
| Backup | Backup policy dan restore test ada | [ ] |
| TLS | `require_secure_transport=ON` | [ ] |
| TLS | `have_ssl=YES` | [ ] |
| TLS | TLSv1/TLSv1.1 tidak digunakan | [ ] |
| Auth | `old_passwords=OFF` dan `secure_auth=ON` | [ ] |
| Auth | Anonymous account tidak ada | [ ] |
| Auth | Wildcard host tidak ada | [ ] |
| Password | Password plugin aktif dan minimal length 14 | [ ] |
| Privilege | `FILE`, `PROCESS`, `SUPER`, `GRANT OPTION` direview | [ ] |
| Secure File | `secure_file_priv` dibatasi | [ ] |
| SQL Mode | `STRICT_ALL_TABLES` aktif | [ ] |
| Audit | Server audit aktif | [ ] |
| Audit | Audit plugin `FORCE_PLUS_PERMANENT` | [ ] |
| Logs | Error log ke file dan permission aman | [ ] |
| Encryption | Data-at-rest encryption aktif bila Level 2 | [ ] |
| Replication | Traffic replication terenkripsi bila digunakan | [ ] |
| Replication | Mutual TLS diterapkan bila diwajibkan | [ ] |
| Exception | Semua pengecualian terdokumentasi | [ ] |

---

## Penutup

Implementasi CIS MariaDB harus dipandang sebagai proses bertahap. Mulai dari backup, validasi versi, pemisahan data, permission, TLS, authentication, privilege review, audit logging, lalu encryption dan replication security. Setiap perubahan yang menyentuh aplikasi harus diuji lebih dulu karena hardening yang benar secara keamanan tetap bisa berdampak pada kompatibilitas aplikasi.
