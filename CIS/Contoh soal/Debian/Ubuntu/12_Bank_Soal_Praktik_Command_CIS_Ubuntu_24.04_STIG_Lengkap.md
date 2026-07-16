# Bank Soal Praktik Command CIS Ubuntu Linux 24.04 LTS STIG — Pilihan Ganda Jamak

**Fokus:** latihan praktik command, audit, remediation, konfigurasi file, dan pilihan jawaban jamak berdasarkan modul CIS Ubuntu Linux 24.04 LTS STIG Benchmark yang sudah dibuat.

**Jumlah soal:** 150 soal.

**Aturan latihan:**
- Pilih semua jawaban yang benar.
- Jawaban bisa lebih dari satu.
- Jika tidak ada opsi benar, jawab **PASS / kosongkan jawaban**.
- Jangan menjalankan command destruktif pada sistem asli. Gunakan VM/lab.
- Beberapa opsi sengaja memakai command distro lain atau command yang kelihatan benar tetapi tidak sesuai Ubuntu/STIG.

**Sumber modul yang digunakan:**
- `01_Pengenalan_CIS_Ubuntu_STIG.md`
- `02_Baseline_Service_Integrity.md`
- `03_Network_SSH_Firewall.md`
- `04_Identity_Authentication_Access.md`
- `05_Logging_Auditing_Permission.md`
- `06_Implementasi_Instalasi_Post_Installation_Hardening_Ubuntu_24.04_STIG.md`

---

## Daftar Cakupan

1. Aturan menjawab dan orientasi Ubuntu STIG
2. Instalasi, partisi, FSTAB, LUKS, dan FIPS
3. Baseline, package, service minimization, dan Chrony
4. AIDE, AppArmor, kernel, memory, dan USB
5. Network, UFW, firewalld adaptasi, dan SSH
6. Identity, PAM, password policy, sudo, su, MFA/SSSD, dan session
7. Logging, rsyslog, auditd, audit rules, AIDE, permission, user/group checks
8. Pengenalan STIG, exception register, dan validasi akhir

---

## 0. Aturan Menjawab dan Orientasi Ubuntu STIG
### Soal 1

**Pertanyaan:** Dalam latihan pilihan ganda jamak ini, bagaimana cara menjawab jika semua opsi command salah atau tidak sesuai Ubuntu STIG?

A. Pilih opsi yang paling mirip benar
B. Pilih semua opsi agar tidak kosong
C. PASS / kosongkan jawaban
D. Pilih A sebagai default

**Jawaban benar:** C

**Pembahasan:** Jika tidak ada opsi yang benar, jawab PASS/kosong. Ini penting karena tidak semua command yang terlihat mirip benar sesuai target OS dan modul.

---
### Soal 2

**Pertanyaan:** Perintah mana yang benar untuk memastikan sistem yang diuji adalah Ubuntu 24.04 LTS?

A. `cat /etc/os-release`
B. `grep DISTRIB_DESCRIPTION /etc/lsb-release`
C. `lsb_release -a`
D. `pacman -Qi base`

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu memakai `/etc/os-release`, `/etc/lsb-release`, dan `lsb_release`; `pacman` adalah Arch.

---
### Soal 3

**Pertanyaan:** Manakah command yang benar untuk masuk ke root shell sementara pada Ubuntu sebelum hardening?

A. `sudo -i`
B. `sudo -v`
C. `su -` jika root password memang diaktifkan kebijakan lokal
D. `arch-chroot /mnt` setelah Ubuntu sudah boot normal

**Jawaban benar:** A, B, C

**Pembahasan:** `sudo -i` dan `sudo -v` relevan; `su -` hanya bila kebijakan mengizinkan. `arch-chroot` bukan command Ubuntu post-install normal.

---
### Soal 4

**Pertanyaan:** Perintah mana yang benar untuk membuat direktori backup konfigurasi sebelum hardening?

A. `mkdir -p /root/hardening-backup/$(date +%F)`
B. `BACKUP_DIR="/root/hardening-backup/$(date +%F)"`
C. `rm -rf /etc/ssh /etc/pam.d`
D. `cp -a /etc/ssh "$BACKUP_DIR/ssh" 2>/dev/null || true`

**Jawaban benar:** A, B, D

**Pembahasan:** Backup perlu dibuat sebelum mengubah SSH/PAM/audit. Menghapus konfigurasi inti jelas salah.

---
### Soal 5

**Pertanyaan:** Perintah mana yang benar untuk update awal sistem Ubuntu setelah instalasi?

A. `apt update`
B. `apt -y full-upgrade`
C. `apt -y autoremove --purge`
D. `pacman -Syu`

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu memakai APT; `pacman` bukan Ubuntu.

---

## 1. Instalasi, Partisi, FSTAB, LUKS, dan FIPS
### Soal 6

**Pertanyaan:** Baris `/etc/fstab` mana yang benar untuk `/tmp` sesuai prinsip hardening Ubuntu STIG/CIS?

A. `UUID=<uuid-tmp> /tmp ext4 defaults,nodev,nosuid,noexec 0 2`
B. `tmpfs /tmp tmpfs rw,nosuid,nodev,noexec,relatime,size=2G,mode=1777 0 0`
C. `UUID=<uuid-tmp> /tmp ext4 defaults,dev,suid,exec 0 2`
D. `UUID=<uuid-tmp> /tmp ext4 defaults,nodev,nosuid,noexec 0 0`

**Jawaban benar:** A, B, D

**Pembahasan:** Intinya `/tmp` harus dibatasi `nodev,nosuid,noexec`. Opsi D masih valid secara mount option, meski fsck pass bisa disesuaikan.

---
### Soal 7

**Pertanyaan:** Baris `/etc/fstab` mana yang benar untuk `/var/log/audit` agar audit log dipisah dan tidak dapat menjalankan binary?

A. `UUID=<uuid-audit> /var/log/audit ext4 defaults,nodev,nosuid,noexec 0 2`
B. `UUID=<uuid-audit> /var/log/audit ext4 defaults,dev,suid,exec 0 2`
C. `tmpfs /var/log/audit tmpfs defaults,nodev,nosuid,noexec 0 0`
D. `UUID=<uuid-audit> /var/log/audit ext4 rw,nodev,nosuid,noexec,relatime 0 2`

**Jawaban benar:** A, D

**Pembahasan:** Audit log harus persisten. Tmpfs membuat log hilang saat reboot sehingga bukan pilihan baseline audit yang baik.

---
### Soal 8

**Pertanyaan:** Perintah mana yang benar untuk memvalidasi mount option beberapa mount point penting setelah instalasi?

A. `findmnt -no TARGET,OPTIONS /tmp /var /var/tmp /var/log /var/log/audit /home`
B. `mount | grep ' /var/log/audit '`
C. `cat /etc/fstab`
D. `chmod noexec /tmp`

**Jawaban benar:** A, B, C

**Pembahasan:** `findmnt`, `mount`, dan membaca fstab benar untuk audit. `chmod noexec` bukan cara mengatur mount option.

---
### Soal 9

**Pertanyaan:** Jika instalasi Ubuntu dilakukan manual dengan LUKS2, command mana yang benar secara praktik untuk membuat dan membuka container LUKS?

A. `cryptsetup luksFormat --type luks2 /dev/nvme0n1p3`
B. `cryptsetup open /dev/nvme0n1p3 cryptroot`
C. `mkfs.ext4 /dev/nvme0n1p3` sebelum `luksFormat`
D. `luksctl create /dev/nvme0n1p3 cryptroot`

**Jawaban benar:** A, B

**Pembahasan:** `cryptsetup luksFormat` membuat container dan `cryptsetup open` membuka mapper. Memformat partisi sebelum LUKS akan tertimpa.

---
### Soal 10

**Pertanyaan:** Setelah LUKS dibuka sebagai `/dev/mapper/cryptroot`, command LVM mana yang benar untuk membuat layout terpisah?

A. `pvcreate /dev/mapper/cryptroot`
B. `vgcreate vgubuntu /dev/mapper/cryptroot`
C. `lvcreate -L 20G -n lv_root vgubuntu`
D. `vgcreate /dev/mapper/cryptroot vgubuntu`

**Jawaban benar:** A, B, C

**Pembahasan:** Urutan LVM benar adalah PV, VG, LV. Opsi D salah urutan argumen.

---
### Soal 11

**Pertanyaan:** Command mana yang benar untuk membuat filesystem ext4 pada logical volume Ubuntu manual?

A. `mkfs.ext4 -L ROOT /dev/vgubuntu/lv_root`
B. `mkfs.ext4 -L VAR /dev/vgubuntu/lv_var`
C. `mkfs.ext4 -L AUDIT /dev/vgubuntu/lv_audit`
D. `mkfs.fat -F32 /dev/vgubuntu/lv_root`

**Jawaban benar:** A, B, C

**Pembahasan:** Root/var/audit bisa ext4. FAT32 bukan untuk root filesystem Linux.

---
### Soal 12

**Pertanyaan:** Untuk FIPS mode pada Ubuntu STIG, command mana yang benar untuk memvalidasi apakah FIPS aktif?

A. `cat /proc/sys/crypto/fips_enabled`
B. `grep fips /proc/cmdline`
C. `fips-mode-setup --check` sebagai command generik wajib Ubuntu
D. `ufw status verbose`

**Jawaban benar:** A, B

**Pembahasan:** Pada panduan implementasi, validasi FIPS memakai `/proc/sys/crypto/fips_enabled`; boot parameter dapat dicek via `/proc/cmdline`.

---
### Soal 13

**Pertanyaan:** Pilihan mana yang benar terkait implementasi FIPS pada Ubuntu 24.04 STIG?

A. Tambahkan `fips=1` saat instalasi bila lingkungan compliance memang membutuhkan FIPS
B. Gunakan dokumentasi vendor/Ubuntu Pro untuk sistem yang sudah berjalan
C. Aktifkan FIPS dengan `ufw enable fips`
D. Catat exception jika lab/kampus tidak memiliki kebutuhan FIPS formal

**Jawaban benar:** A, B, D

**Pembahasan:** FIPS adalah area compliance dan vendor-specific. UFW tidak mengaktifkan FIPS.

---
### Soal 14

**Pertanyaan:** Perintah mana yang benar untuk mengunci root login langsung setelah akun admin sudo tersedia?

A. `sudo passwd -l root`
B. `passwd -l root` saat sudah berada sebagai root
C. `passwd -u root`
D. `usermod -U root`

**Jawaban benar:** A, B

**Pembahasan:** Mengunci root sesuai prinsip direct root login prevention. `passwd -u`/`usermod -U` membuka kunci.

---

## 2. Baseline, Package, Service Minimization, dan Chrony
### Soal 15

**Pertanyaan:** Perintah mana yang benar untuk memasang unattended upgrades dan apt-listchanges pada Ubuntu?

A. `apt install -y unattended-upgrades apt-listchanges`
B. `apt update` sebelum instalasi paket
C. `dnf install unattended-upgrades`
D. `pacman -S unattended-upgrades`

**Jawaban benar:** A, B

**Pembahasan:** Ubuntu memakai APT.

---
### Soal 16

**Pertanyaan:** Konfigurasi mana yang benar untuk membersihkan komponen lama setelah unattended upgrades?

A. `Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";`
B. `Unattended-Upgrade::Remove-Unused-Dependencies "true";`
C. `Unattended-Upgrade::Remove-New-Unused-Dependencies "true";`
D. `Unattended-Upgrade::Remove-All-Packages "true";`

**Jawaban benar:** A, B, C

**Pembahasan:** Tiga opsi pertama ada pada panduan implementasi. Opsi D berbahaya dan bukan konfigurasi valid baseline.

---
### Soal 17

**Pertanyaan:** Perintah mana yang benar untuk audit konfigurasi `Remove-Unused` pada APT?

A. `grep -R "Remove-Unused" /etc/apt/apt.conf.d/`
B. `apt-config dump | grep Remove-Unused`
C. `cat /etc/apt/apt.conf.d/52-hardening-remove-unused`
D. `pacman -Q | grep Remove-Unused`

**Jawaban benar:** A, B, C

**Pembahasan:** APT dikonfigurasi melalui `/etc/apt/apt.conf.d`.

---
### Soal 18

**Pertanyaan:** Konfigurasi mana yang benar untuk mengurangi weak dependencies APT pada baseline tambahan?

A. `APT::Install-Recommends "0";`
B. `APT::Install-Suggests "0";`
C. `APT::Install-Recommends "1";`
D. `APT::Install-Suggests "1";`

**Jawaban benar:** A, B

**Pembahasan:** Nilai 0 mencegah pemasangan Recommends/Suggests otomatis.

---
### Soal 19

**Pertanyaan:** Perintah mana yang benar untuk menghapus layanan lama/tidak aman yang disebut dalam implementasi Ubuntu STIG?

A. `apt purge -y telnetd rsh-server ntp systemd-timesyncd || true`
B. `apt purge -y nis talk talkd rsh-client telnet ldap-utils ftp tnftp xinetd || true`
C. `apt autoremove -y --purge`
D. `apt install -y telnetd rsh-server`

**Jawaban benar:** A, B, C

**Pembahasan:** Telnet/RSH/NTP lama/systemd-timesyncd tidak dipakai dalam baseline yang memilih Chrony.

---
### Soal 20

**Pertanyaan:** Command mana yang benar untuk audit apakah paket lama seperti telnetd/rsh-server/ntp masih terpasang?

A. `dpkg -l | egrep 'telnetd|rsh-server|ntp|systemd-timesyncd|nis|talkd|xinetd' || true`
B. `apt list --installed | egrep 'telnetd|rsh-server|ntp' || true`
C. `rpm -qa | grep telnetd`
D. `systemctl list-unit-files | grep -E 'telnet|rsh|ntp'`

**Jawaban benar:** A, B, D

**Pembahasan:** Ubuntu memakai dpkg/apt/systemctl. `rpm` bukan tool utama Ubuntu.

---
### Soal 21

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan Chrony?

A. `apt install -y chrony`
B. `systemctl enable --now chrony.service`
C. `systemctl enable --now systemd-timesyncd.service chrony.service`
D. `chronyc tracking`

**Jawaban benar:** A, B, D

**Pembahasan:** Chrony dipilih sebagai time sync utama; jangan menjalankan dua daemon waktu bersamaan.

---
### Soal 22

**Pertanyaan:** Konfigurasi mana yang benar di `/etc/chrony/chrony.conf`?

A. `pool ntp.ubuntu.com iburst maxsources 4`
B. `server 192.168.10.1 iburst` jika itu NTP internal resmi
C. `allow 0.0.0.0/0` pada semua server biasa
D. `server time.example.org iburst`

**Jawaban benar:** A, B, D

**Pembahasan:** `allow 0.0.0.0/0` menjadikan host melayani client luas; itu bukan baseline umum.

---
### Soal 23

**Pertanyaan:** Command audit mana yang benar untuk memastikan Chrony berjalan dan memakai sumber waktu?

A. `systemctl is-enabled chrony.service`
B. `systemctl is-active chrony.service`
C. `chronyc sources -v`
D. `timedatectl set-ntp false`

**Jawaban benar:** A, B, C

**Pembahasan:** `timedatectl set-ntp false` justru mematikan NTP di systemd.

---
### Soal 24

**Pertanyaan:** Jika soal meminta mengaktifkan `systemd-timesyncd` bersamaan dengan Chrony pada baseline Ubuntu STIG, jawaban yang benar adalah?

A. `systemctl enable --now systemd-timesyncd.service`
B. `systemctl enable --now chrony.service systemd-timesyncd.service`
C. `apt install -y ntp systemd-timesyncd chrony`
D. `timedatectl set-ntp true` sebagai pengganti Chrony tanpa audit

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: baseline implementasi memilih Chrony dan menghindari konflik time daemon.

---

## 3. AIDE, AppArmor, Kernel, Memory, dan USB
### Soal 25

**Pertanyaan:** Perintah mana yang benar untuk memasang AIDE pada Ubuntu?

A. `apt install -y aide aide-common`
B. `aideinit`
C. `aide -c /etc/aide/aide.conf --check`
D. `pacman -S aide`

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu memakai APT; AIDE perlu init database dan check.

---
### Soal 26

**Pertanyaan:** Setelah `aideinit`, command mana yang benar untuk menjadikan database baru sebagai database aktif?

A. `cp -p /var/lib/aide/aide.db.new /var/lib/aide/aide.db`
B. `aide -c /etc/aide/aide.conf --check`
C. `rm -rf /var/lib/aide`
D. `mv /etc/aide/aide.conf /tmp`

**Jawaban benar:** A, B

**Pembahasan:** Copy database baru lalu check. Menghapus database/konfigurasi salah.

---
### Soal 27

**Pertanyaan:** Entry AIDE mana yang benar untuk memantau integritas audit tool?

A. `/sbin/auditctl p+i+n+u+g+s+b+acl+xattrs+sha512`
B. `/sbin/auditd p+i+n+u+g+s+b+acl+xattrs+sha512`
C. `/sbin/ausearch p+i+n+u+g+s+b+acl+xattrs+sha512`
D. `/var/log/audit/audit.log p+i+n+u+g+s+b+acl+xattrs+sha512` sebagai satu-satunya kontrol audit tools

**Jawaban benar:** A, B, C

**Pembahasan:** AIDE memantau tool audit seperti auditctl/auditd/ausearch. Audit log punya kontrol permission/retention sendiri.

---
### Soal 28

**Pertanyaan:** Perintah mana yang benar agar laporan AIDE tidak silent?

A. `sed -i 's/^SILENTREPORTS=.*/SILENTREPORTS=no/' /etc/default/aide`
B. `grep SILENTREPORTS /etc/default/aide`
C. `echo 'SILENTREPORTS=no' >> /etc/default/aide` bila belum ada
D. `echo 'SILENTREPORTS=yes' > /etc/default/aide`

**Jawaban benar:** A, B, C

**Pembahasan:** `SILENTREPORTS=no` memastikan notifikasi/laporan tidak dibungkam.

---
### Soal 29

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan AppArmor pada Ubuntu?

A. `apt install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra`
B. `systemctl enable --now apparmor.service`
C. `aa-status`
D. `systemctl enable --now selinux.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu baseline memakai AppArmor, bukan service SELinux.

---
### Soal 30

**Pertanyaan:** Perintah mana yang benar untuk menjadikan profil AppArmor enforcing setelah diuji?

A. `aa-enforce /etc/apparmor.d/* 2>/dev/null || true`
B. `aa-status`
C. `aa-complain /etc/apparmor.d/*` sebagai hardening final
D. `systemctl stop apparmor.service`

**Jawaban benar:** A, B

**Pembahasan:** `aa-enforce` memperketat, `aa-status` memvalidasi. Complain/stop melemahkan.

---
### Soal 31

**Pertanyaan:** Sysctl mana yang benar untuk hardening kernel dan network pada Ubuntu STIG?

A. `net.ipv4.tcp_syncookies = 1`
B. `kernel.randomize_va_space = 2`
C. `kernel.kptr_restrict = 2`
D. `kernel.dmesg_restrict = 1`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua nilai ini muncul dalam baseline implementasi hardening.

---
### Soal 32

**Pertanyaan:** Sysctl mana yang benar untuk mencegah host biasa menjadi router IPv4/IPv6?

A. `net.ipv4.ip_forward = 0`
B. `net.ipv6.conf.all.forwarding = 0`
C. `net.ipv4.ip_forward = 1`
D. `net.ipv6.conf.all.forwarding = 1`

**Jawaban benar:** A, B

**Pembahasan:** Nilai 0 menonaktifkan forwarding pada host non-router.

---
### Soal 33

**Pertanyaan:** Sysctl mana yang benar untuk menolak redirect/source route?

A. `net.ipv4.conf.all.accept_redirects = 0`
B. `net.ipv4.conf.default.accept_redirects = 0`
C. `net.ipv4.conf.all.accept_source_route = 0`
D. `net.ipv6.conf.default.accept_source_route = 0`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua opsi mengurangi manipulasi routing/redirect.

---
### Soal 34

**Pertanyaan:** Command mana yang benar untuk membuat konfigurasi sysctl persisten dan menerapkannya?

A. `cat >/etc/sysctl.d/60-stig-hardening.conf <<'EOF'`
B. `sysctl --system`
C. `sysctl net.ipv4.tcp_syncookies kernel.randomize_va_space` untuk validasi
D. `echo net.ipv4.tcp_syncookies=1 >> /etc/modules`

**Jawaban benar:** A, B, C

**Pembahasan:** Konfigurasi sysctl disimpan di `/etc/sysctl.d`, bukan `/etc/modules`.

---
### Soal 35

**Pertanyaan:** Konfigurasi mana yang benar untuk membatasi core dump?

A. `* hard core 0` di `/etc/security/limits.d/60-hardening-coredump.conf`
B. `fs.suid_dumpable = 0` di sysctl
C. `systemctl disable --now systemd-coredump.socket 2>/dev/null || true` jika tidak dibutuhkan
D. `fs.suid_dumpable = 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Core dump bisa memuat data sensitif; disable/batasi sesuai baseline.

---
### Soal 36

**Pertanyaan:** Perintah mana yang benar untuk menonaktifkan USB mass storage pada server yang tidak membutuhkan USB storage?

A. `cat >/etc/modprobe.d/usb-storage.conf` berisi `install usb-storage /bin/false` dan `blacklist usb-storage`
B. `modprobe -r usb-storage 2>/dev/null || true`
C. `modprobe -n -v usb-storage` untuk validasi
D. `echo usb-storage >> /etc/modules`

**Jawaban benar:** A, B, C

**Pembahasan:** Opsi D justru memuat module.

---
### Soal 37

**Pertanyaan:** Command mana yang benar untuk audit module USB storage masih loaded atau tidak?

A. `lsmod | grep usb_storage || true`
B. `modprobe -n -v usb-storage`
C. `grep -R 'usb-storage' /etc/modprobe.d/`
D. `systemctl status usb-storage.service`

**Jawaban benar:** A, B, C

**Pembahasan:** usb-storage adalah kernel module, bukan service systemd normal.

---
### Soal 38

**Pertanyaan:** Pilih command yang benar untuk mengaktifkan semua core dump demi forensik tanpa batas sesuai baseline STIG.

A. `ulimit -c unlimited`
B. `fs.suid_dumpable = 2`
C. `Storage=external` dengan `ProcessSizeMax=infinity`
D. `systemctl enable --now systemd-coredump.socket`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: baseline mengarah ke pembatasan core dump, bukan mengaktifkan tanpa batas.

---

## 4. Network, UFW, Firewalld Adaptasi, dan SSH
### Soal 39

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan UFW pada Ubuntu?

A. `apt install -y ufw`
B. `systemctl enable --now ufw.service`
C. `ufw --force enable`
D. `ufw status verbose`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai alur implementasi UFW.

---
### Soal 40

**Pertanyaan:** Command UFW mana yang benar untuk default policy baseline?

A. `ufw default deny incoming`
B. `ufw default allow outgoing`
C. `ufw default deny routed`
D. `ufw default allow incoming`

**Jawaban benar:** A, B, C

**Pembahasan:** Default incoming dan routed dibatasi; outgoing allow pada baseline umum.

---
### Soal 41

**Pertanyaan:** Server dikelola via SSH dari subnet admin `192.168.10.0/24`. Command mana yang benar sebelum enable UFW?

A. `ufw allow from 192.168.10.0/24 to any port 22 proto tcp comment 'Allow SSH from admin subnet'`
B. `ufw limit 22/tcp comment 'Rate limit SSH'`
C. `ufw deny 22/tcp`
D. `ufw allow 22/tcp`

**Jawaban benar:** A, B, D

**Pembahasan:** A paling sesuai allowlist; B rate-limit; D benar secara sintaks tapi lebih luas dari subnet admin. C dapat lockout.

---
### Soal 42

**Pertanyaan:** Untuk web server resmi, command mana yang benar untuk membuka HTTP/HTTPS di UFW?

A. `ufw allow 80/tcp comment 'HTTP'`
B. `ufw allow 443/tcp comment 'HTTPS'`
C. `ufw allow 1:65535/tcp`
D. `ufw allow all`

**Jawaban benar:** A, B

**Pembahasan:** Buka port yang dibutuhkan saja.

---
### Soal 43

**Pertanyaan:** Command audit mana yang benar untuk melihat port terbuka dan status firewall?

A. `ufw status numbered`
B. `ufw status verbose`
C. `ss -tulpen`
D. `netstat -tulpen` jika paket tersedia

**Jawaban benar:** A, B, C, D

**Pembahasan:** `ss` adalah modern default; `netstat` valid jika `net-tools` ada.

---
### Soal 44

**Pertanyaan:** Jika soal menyebut Ubuntu STIG resmi memakai UFW, opsi `firewall-cmd` berikut menjadi jawaban...

A. Benar sebagai pengganti resmi UFW STIG
B. Salah untuk jawaban resmi UFW
C. Bisa hanya sebagai adaptasi organisasi jika firewalld dipilih dan didokumentasikan
D. Selalu wajib karena UFW tidak ada di Ubuntu

**Jawaban benar:** B, C

**Pembahasan:** Modul Ubuntu STIG dan implementasi memakai UFW. Firewalld bisa valid secara sintaks tetapi bukan baseline modul ini.

---
### Soal 45

**Pertanyaan:** Jika organisasi memilih firewalld sebagai adaptasi lokal, command mana yang benar secara sintaks untuk enable dan set default zone?

A. `apt install -y firewalld`
B. `systemctl enable --now firewalld`
C. `firewall-cmd --set-default-zone=public`
D. `firewall-cmd --reload`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Valid untuk firewalld sebagai adaptasi lokal, bukan UFW resmi modul.

---
### Soal 46

**Pertanyaan:** Firewalld adaptasi: command mana yang benar untuk mengizinkan SSH hanya dari IP admin `192.168.10.20/32`?

A. `firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.10.20/32" port protocol="tcp" port="22" accept'`
B. `firewall-cmd --reload`
C. `firewall-cmd --permanent --zone=public --add-service=all`
D. `firewall-cmd --zone=trusted --add-interface=lo --permanent`

**Jawaban benar:** A, B, D

**Pembahasan:** A membatasi SSH, B menerapkan, D valid untuk loopback/trusted jika dirancang. C terlalu luas.

---
### Soal 47

**Pertanyaan:** Jika pertanyaan berbunyi: 'Perintah mana yang benar untuk membuka semua service di public zone sesuai STIG?'

A. `firewall-cmd --permanent --zone=public --add-service=all`
B. `firewall-cmd --permanent --zone=public --add-port=1-65535/tcp`
C. `ufw allow 1:65535/tcp`
D. `ufw default allow incoming`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi bertentangan dengan prinsip allowlist.

---
### Soal 48

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan SSH server pada Ubuntu jika dibutuhkan?

A. `apt install -y ssh`
B. `systemctl enable --now ssh.service`
C. `systemctl is-active ssh.service`
D. `ss -tulpen | grep ':22'`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Ubuntu menggunakan service `ssh.service`.

---
### Soal 49

**Pertanyaan:** Jika SSH tidak dibutuhkan pada sistem, tindakan mana yang benar?

A. `systemctl disable --now ssh.service`
B. `apt purge -y openssh-server`
C. `ufw deny 22/tcp`
D. `systemctl enable --now ssh.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Jika tidak diperlukan, service SSH dinonaktifkan/dihapus dan port ditutup.

---
### Soal 50

**Pertanyaan:** Command mana yang benar untuk permission file SSH server dan host keys?

A. `chown root:root /etc/ssh/sshd_config`
B. `chmod 600 /etc/ssh/sshd_config`
C. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key' -exec chmod 600 {} \;`
D. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chmod 644 {} \;`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Private key ketat; public key boleh 644; config root-owned.

---
### Soal 51

**Pertanyaan:** Directive SSH mana yang benar untuk mencegah root login, password kosong, dan metode host trust lama?

A. `PermitRootLogin no`
B. `PermitEmptyPasswords no`
C. `HostbasedAuthentication no`
D. `IgnoreRhosts yes`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua ada pada drop-in SSH hardening.

---
### Soal 52

**Pertanyaan:** Directive SSH mana yang benar untuk session timeout dan pembatasan percobaan autentikasi?

A. `ClientAliveInterval 600`
B. `ClientAliveCountMax 1`
C. `LoginGraceTime 60`
D. `MaxAuthTries 4`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai file implementasi.

---
### Soal 53

**Pertanyaan:** Directive SSH mana yang benar untuk membatasi forwarding dan GUI exposure?

A. `X11Forwarding no`
B. `AllowTcpForwarding no`
C. `AllowAgentForwarding no`
D. `PermitTunnel no`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua membatasi penyalahgunaan SSH sebagai tunnel/forwarding.

---
### Soal 54

**Pertanyaan:** Directive SSH mana yang benar untuk banner dan logging?

A. `Banner /etc/issue.net`
B. `LogLevel VERBOSE`
C. `UsePAM yes`
D. `Banner /etc/shadow`

**Jawaban benar:** A, B, C

**Pembahasan:** Banner harus menunjuk file notice, bukan file sensitif.

---
### Soal 55

**Pertanyaan:** Ciphers SSH server mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `aes256-gcm@openssh.com`
B. `aes128-gcm@openssh.com`
C. `aes256-ctr`
D. `3des-cbc`

**Jawaban benar:** A, B, C

**Pembahasan:** `3des-cbc` tidak masuk daftar hardening.

---
### Soal 56

**Pertanyaan:** MACs SSH mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `hmac-sha2-512-etm@openssh.com`
B. `hmac-sha2-256-etm@openssh.com`
C. `hmac-sha2-512`
D. `hmac-md5`

**Jawaban benar:** A, B, C

**Pembahasan:** `hmac-md5` tidak sesuai baseline.

---
### Soal 57

**Pertanyaan:** KexAlgorithms mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `ecdh-sha2-nistp521`
B. `diffie-hellman-group-exchange-sha256`
C. `diffie-hellman-group16-sha512`
D. `diffie-hellman-group1-sha1`

**Jawaban benar:** A, B, C

**Pembahasan:** `group1-sha1` lemah dan tidak sesuai.

---
### Soal 58

**Pertanyaan:** Command mana yang benar untuk validasi dan restart SSH setelah mengubah konfigurasi?

A. `sshd -t`
B. `systemctl restart ssh.service`
C. `sshd -T | egrep 'permitrootlogin|ciphers|macs|kexalgorithms'`
D. `rm /etc/ssh/sshd_config`

**Jawaban benar:** A, B, C

**Pembahasan:** Validasi syntax sebelum restart; jangan hapus config.

---
### Soal 59

**Pertanyaan:** Command mana yang benar untuk konfigurasi SSH client hardening?

A. `mkdir -p /etc/ssh/ssh_config.d`
B. Buat `/etc/ssh/ssh_config.d/60-stig-hardening.conf` berisi `Host *` dan daftar `Ciphers`/`MACs`
C. `ssh -G localhost | egrep 'ciphers|macs' | head`
D. `echo PermitRootLogin yes >> /etc/ssh/ssh_config`

**Jawaban benar:** A, B, C

**Pembahasan:** SSH client hardening berbeda dari server; PermitRootLogin adalah server directive.

---
### Soal 60

**Pertanyaan:** Command mana yang benar untuk membuat banner login non-informatif?

A. `cat >/etc/issue` berisi `Authorized users only. All activity may be monitored and reported.`
B. `cat >/etc/issue.net` berisi notice serupa
C. `chmod 644 /etc/issue /etc/issue.net /etc/motd`
D. `echo $(uname -a) > /etc/issue.net`

**Jawaban benar:** A, B, C

**Pembahasan:** Banner tidak boleh membocorkan kernel/versi internal.

---
### Soal 61

**Pertanyaan:** Perintah mana yang benar untuk menonaktifkan wireless/Bluetooth pada server yang tidak membutuhkannya?

A. `systemctl disable --now bluetooth.service 2>/dev/null || true`
B. `apt purge -y bluez 2>/dev/null || true`
C. `rfkill list 2>/dev/null || true` untuk audit
D. `systemctl enable --now bluetooth.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Bluetooth/wireless memperluas attack surface jika tidak dibutuhkan.

---

## 5. Identity, PAM, Password Policy, sudo, su, MFA/SSSD, dan Session
### Soal 62

**Pertanyaan:** Paket mana yang benar untuk password quality pada Ubuntu?

A. `apt install -y libpam-pwquality cracklib-runtime`
B. `dpkg -l | grep libpam-pwquality`
C. `apt install -y libpam-modules` untuk modul PAM dasar
D. `apt install -y pam_google_authenticator` sebagai kewajiban STIG Ubuntu

**Jawaban benar:** A, B, C

**Pembahasan:** Google Authenticator bukan kewajiban umum modul STIG; STIG lebih banyak membahas MFA/SSSD/smart card/PKI sesuai organisasi.

---
### Soal 63

**Pertanyaan:** Konfigurasi `/etc/security/pwquality.conf` mana yang benar menurut panduan implementasi?

A. `minlen = 15`
B. `dcredit = -1`
C. `ucredit = -1`
D. `ocredit = -1`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua bagian dari password complexity baseline.

---
### Soal 64

**Pertanyaan:** Konfigurasi pwquality mana yang benar untuk membatasi pola password lemah?

A. `lcredit = -1`
B. `maxrepeat = 3`
C. `maxclassrepeat = 4`
D. `dictcheck = 1`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua muncul dalam baseline implementasi.

---
### Soal 65

**Pertanyaan:** Konfigurasi pwquality mana yang benar untuk enforcement?

A. `enforcing = 1`
B. `retry = 3`
C. `minlen = 4`
D. `dictcheck = 0`

**Jawaban benar:** A, B

**Pembahasan:** Enforcing dan retry benar; minlen pendek/dictcheck mati tidak sesuai.

---
### Soal 66

**Pertanyaan:** Command mana yang benar untuk mengatur password aging di `/etc/login.defs`?

A. `sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   60/' /etc/login.defs`
B. `sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   1/' /etc/login.defs`
C. `sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   7/' /etc/login.defs`
D. `sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   99999/' /etc/login.defs`

**Jawaban benar:** A, B, C

**Pembahasan:** Panduan memakai max 60, min 1, warn 7.

---
### Soal 67

**Pertanyaan:** Command mana yang benar untuk menerapkan password aging pada user interaktif yang sudah ada?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1}' /etc/passwd | while read -r user; do chage --maxdays 60 --mindays 1 --warndays 7 "$user"; done`
B. `chage -l <nama_user>` untuk validasi
C. `passwd -d <nama_user>`
D. `chage --maxdays 99999 <nama_user>`

**Jawaban benar:** A, B

**Pembahasan:** `passwd -d` menghapus password; maxdays 99999 bertentangan dengan baseline.

---
### Soal 68

**Pertanyaan:** Konfigurasi `/etc/security/faillock.conf` mana yang benar untuk account lockout ketat?

A. `deny = 3`
B. `fail_interval = 900`
C. `unlock_time = 0`
D. `even_deny_root`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai contoh baseline ketat. Di lab bisa disesuaikan dengan exception.

---
### Soal 69

**Pertanyaan:** Command mana yang benar untuk audit/reset faillock user?

A. `faillock --user <nama_user>`
B. `faillock --user <nama_user> --reset`
C. `rm -rf /etc/security/faillock.conf`
D. `passwd -d <nama_user>`

**Jawaban benar:** A, B

**Pembahasan:** faillock punya command audit dan reset. Menghapus config/menghapus password salah.

---
### Soal 70

**Pertanyaan:** Baris PAM mana yang benar untuk password history di `/etc/pam.d/common-password` jika diterapkan hati-hati?

A. `password requisite pam_pwhistory.so remember=5 use_authtok`
B. `password requisite pam_pwhistory.so remember=0`
C. `password optional pam_permit.so`
D. `password required pam_deny.so` sebagai satu-satunya baris password

**Jawaban benar:** A

**Pembahasan:** `remember=5 use_authtok` sesuai contoh. Opsi lain salah/berbahaya.

---
### Soal 71

**Pertanyaan:** Command mana yang benar untuk mencari password kosong dan nullok?

A. `awk -F: '($2==""){print $1}' /etc/shadow`
B. `grep -R "nullok" /etc/pam.d/ || true`
C. `passwd -l <nama_user>` jika ditemukan akun kosong
D. `passwd -d <nama_user>`

**Jawaban benar:** A, B, C

**Pembahasan:** `passwd -d` justru membuat password kosong.

---
### Soal 72

**Pertanyaan:** Command mana yang benar untuk konfigurasi sudo hardening?

A. `apt install -y sudo`
B. Buat `/etc/sudoers.d/60-stig-hardening` berisi `Defaults use_pty`
C. Tambahkan `Defaults logfile="/var/log/sudo.log"`
D. `visudo -cf /etc/sudoers.d/60-stig-hardening`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Sudo harus memakai pty, log file, dan divalidasi dengan visudo.

---
### Soal 73

**Pertanyaan:** Konfigurasi sudo mana yang benar untuk reauthentication dan timeout?

A. `Defaults timestamp_timeout=5`
B. `Defaults passwd_timeout=1`
C. `Defaults timestamp_timeout=-1`
D. `ALL=(ALL) NOPASSWD:ALL` untuk semua admin

**Jawaban benar:** A, B

**Pembahasan:** Nilai negatif atau NOPASSWD global melemahkan reauthentication.

---
### Soal 74

**Pertanyaan:** Command mana yang benar untuk membatasi `su` melalui group khusus?

A. `groupadd -f sugroup`
B. Tambahkan `auth required pam_wheel.so use_uid group=sugroup` di `/etc/pam.d/su`
C. `usermod -aG sugroup <nama_admin>`
D. `chmod 777 /bin/su`

**Jawaban benar:** A, B, C

**Pembahasan:** Pembatasan `su` dilakukan via PAM/group; chmod 777 berbahaya.

---
### Soal 75

**Pertanyaan:** Command mana yang benar untuk melihat user dalam sudo group dan menilai apakah sesuai kebutuhan?

A. `getent group sudo`
B. `groups <nama_user>`
C. `id <nama_user>`
D. `chmod -R 777 /etc/sudoers.d`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit membership dulu; jangan membuat sudoers world-writable.

---
### Soal 76

**Pertanyaan:** Jika soal meminta mengaktifkan direct root SSH login sesuai Ubuntu STIG, command mana yang benar?

A. `echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config`
B. `passwd -u root`
C. `systemctl restart ssh.service`
D. `usermod -U root`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: STIG/baseline mencegah direct root login.

---
### Soal 77

**Pertanyaan:** Area MFA/SSSD/smart card/PIV/PKI pada Ubuntu STIG sebaiknya diterapkan dengan command apa di lab pribadi tanpa infrastruktur PKI?

A. `apt install -y sssd` lalu mengarang domain palsu
B. `authselect enable-smartcard` di Ubuntu
C. `Catat exception / Not Applicable bila tidak ada infrastruktur organisasi`
D. `Hapus PAM agar MFA tidak diperlukan`

**Jawaban benar:** C

**Pembahasan:** MFA/PKI sangat kontekstual. Tanpa infrastruktur organisasi, buat exception; jangan mengarang konfigurasi.

---
### Soal 78

**Pertanyaan:** Command mana yang benar untuk audit keberadaan SSSD bila organisasi memakai identity provider?

A. `systemctl status sssd.service`
B. `dpkg -l | grep sssd`
C. `grep -R '^services\|^domains' /etc/sssd/sssd.conf 2>/dev/null`
D. `chmod 777 /etc/sssd/sssd.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** SSSD config sensitif; jangan dibuat world-writable.

---
### Soal 79

**Pertanyaan:** Command mana yang benar untuk session/login banner GDM jika workstation memakai GNOME?

A. `mkdir -p /etc/dconf/db/gdm.d`
B. Buat `/etc/dconf/db/gdm.d/01-banner-message` dengan `banner-message-enable=true`
C. `dconf update`
D. `ufw allow gdm`

**Jawaban benar:** A, B, C

**Pembahasan:** GDM banner memakai dconf, bukan firewall.

---
### Soal 80

**Pertanyaan:** Konfigurasi GDM mana yang benar untuk mencegah user enumeration?

A. `[org/gnome/login-screen]` lalu `disable-user-list=true`
B. `dconf update`
C. `disable-user-list=false`
D. `cat /etc/passwd > /etc/issue`

**Jawaban benar:** A, B

**Pembahasan:** Disable user list mencegah daftar user tampil di login GUI.

---
### Soal 81

**Pertanyaan:** Konfigurasi dconf mana yang benar untuk lock screen dan automount workstation?

A. `idle-delay=uint32 900`
B. `lock-enabled=true`
C. `lock-delay=uint32 0`
D. `automount=false` dan `autorun-never=true`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai contoh hardening workstation GNOME.

---
### Soal 82

**Pertanyaan:** Command mana yang benar untuk menonaktifkan Ctrl-Alt-Delete reboot pada sistem kritis?

A. `systemctl mask ctrl-alt-del.target`
B. `systemctl daemon-reload`
C. `systemctl unmask ctrl-alt-del.target`
D. `systemctl start ctrl-alt-del.target`

**Jawaban benar:** A, B

**Pembahasan:** Mask mencegah aktivasi target. Unmask/start tidak sesuai hardening.

---

## 6. Logging, rsyslog, auditd, Audit Rules, AIDE, dan Permission
### Soal 83

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan rsyslog?

A. `apt install -y rsyslog`
B. `systemctl enable --now rsyslog.service`
C. `systemctl is-active rsyslog.service`
D. `systemctl disable --now rsyslog.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Rsyslog harus tersedia dan aktif.

---
### Soal 84

**Pertanyaan:** Konfigurasi rsyslog mana yang benar untuk mode pembuatan file log?

A. `$FileCreateMode 0640`
B. `systemctl restart rsyslog.service`
C. `grep -R "FileCreateMode" /etc/rsyslog.conf /etc/rsyslog.d/*.conf`
D. `$FileCreateMode 0777`

**Jawaban benar:** A, B, C

**Pembahasan:** 0640 lebih aman; 0777 terlalu terbuka.

---
### Soal 85

**Pertanyaan:** Konfigurasi forwarding rsyslog mana yang benar jika organisasi punya server log `192.168.10.50:6514`?

A. `*.* @@192.168.10.50:6514`
B. `systemctl restart rsyslog.service`
C. `*.* @192.168.10.50:514` sebagai TLS otomatis
D. `@@` berarti TCP

**Jawaban benar:** A, B, D

**Pembahasan:** `@@` adalah TCP; TLS perlu konfigurasi gtls/CA tambahan. `@` UDP bukan TLS otomatis.

---
### Soal 86

**Pertanyaan:** Command mana yang benar untuk audit permission log yang terlalu permisif?

A. `find /var/log -type f -perm /137 -ls`
B. `find /var/log -type f -exec chmod go-wx {} \;` sebagai perbaikan konservatif
C. `find /var/log -type d -exec chmod go-w {} \;`
D. `chmod -R 777 /var/log`

**Jawaban benar:** A, B, C

**Pembahasan:** Log harus tidak writable/executable oleh group/others tanpa kebutuhan.

---
### Soal 87

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan auditd?

A. `apt install -y auditd audispd-plugins`
B. `systemctl enable --now auditd.service`
C. `auditctl -s`
D. `systemctl disable --now auditd.service`

**Jawaban benar:** A, B, C

**Pembahasan:** auditd harus installed, enabled, active.

---
### Soal 88

**Pertanyaan:** Parameter boot mana yang benar agar audit aktif sejak boot awal?

A. `audit=1`
B. `audit_backlog_limit=8192`
C. `audit=0`
D. `noaudit`

**Jawaban benar:** A, B

**Pembahasan:** audit=1 dan backlog limit sesuai implementasi.

---
### Soal 89

**Pertanyaan:** Command mana yang benar untuk memasukkan parameter audit ke GRUB dan memperbarui konfigurasi?

A. Edit `/etc/default/grub` agar `GRUB_CMDLINE_LINUX` memuat `audit=1 audit_backlog_limit=8192`
B. `update-grub`
C. `cat /proc/cmdline | grep -E 'audit=1|audit_backlog_limit=8192'` setelah reboot
D. `grub-mkconfig -o /boot/grub/grub.cfg` sebagai command standar Ubuntu wajib

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu memakai `update-grub`; `grub-mkconfig` mungkin ada tetapi bukan command standar yang dipakai panduan.

---
### Soal 90

**Pertanyaan:** Konfigurasi `/etc/audit/auditd.conf` mana yang benar untuk retention ketat?

A. `max_log_file = 100`
B. `max_log_file_action = keep_logs`
C. `disk_full_action = halt`
D. `disk_error_action = halt`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai contoh ketat. Di lab bisa disesuaikan dengan exception.

---
### Soal 91

**Pertanyaan:** Konfigurasi auditd mana yang berisiko menghapus bukti audit dan tidak sesuai baseline?

A. `max_log_file_action = ignore`
B. `max_log_file_action = rotate` tanpa retention jelas
C. `disk_full_action = ignore`
D. `max_log_file_action = keep_logs`

**Jawaban benar:** A, B, C

**Pembahasan:** keep_logs adalah arah baseline yang menjaga audit log.

---
### Soal 92

**Pertanyaan:** Command mana yang benar untuk remote audit offloading jika ada audit server?

A. `sed -i -E 's/^active\s*=.*/active = yes/' /etc/audit/plugins.d/au-remote.conf`
B. `sed -i -E 's/^remote_server\s*=.*/remote_server = 192.168.10.60/' /etc/audit/audisp-remote.conf`
C. `systemctl restart auditd.service`
D. `rm -rf /var/log/audit`

**Jawaban benar:** A, B, C

**Pembahasan:** au-remote butuh active dan remote_server; jangan hapus log.

---
### Soal 93

**Pertanyaan:** Rule audit mana yang benar untuk perubahan file identitas user/group?

A. `-w /etc/passwd -p wa -k identity`
B. `-w /etc/group -p wa -k identity`
C. `-w /etc/shadow -p wa -k identity`
D. `-w /etc/gshadow -p wa -k identity`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua file identitas sensitif diaudit.

---
### Soal 94

**Pertanyaan:** Rule audit mana yang benar untuk `/etc/security/opasswd`?

A. `-w /etc/security/opasswd -p wa -k identity`
B. `-w /etc/security/opasswd -p x -k identity`
C. `-w /etc/security/opasswd -p r -k identity` sebagai satu-satunya rule
D. `chmod 777 /etc/security/opasswd`

**Jawaban benar:** A

**Pembahasan:** Perubahan write/attribute pada opasswd harus tercatat.

---
### Soal 95

**Pertanyaan:** Rule audit mana yang benar untuk perubahan sudoers dan sudo log?

A. `-w /etc/sudoers -p wa -k scope`
B. `-w /etc/sudoers.d/ -p wa -k scope`
C. `-w /var/log/sudo.log -p wa -k actions`
D. `-w /etc/sudoers -p x -k scope`

**Jawaban benar:** A, B, C

**Pembahasan:** Sudoers dan sudo log perlu watch write/attribute.

---
### Soal 96

**Pertanyaan:** Rule audit mana yang benar untuk perubahan waktu?

A. `-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change`
B. `-a always,exit -F arch=b32 -S adjtimex,settimeofday,clock_settime -k time-change`
C. `-w /etc/localtime -p wa -k time-change`
D. `-a never,exit -S clock_settime -k time-change`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit mencakup syscall 64/32-bit dan file localtime.

---
### Soal 97

**Pertanyaan:** Rule audit mana yang benar untuk perubahan hostname/network identity?

A. `-a always,exit -F arch=b64 -S sethostname,setdomainname -k system-locale`
B. `-a always,exit -F arch=b32 -S sethostname,setdomainname -k system-locale`
C. `-w /etc/hosts -p wa -k system-locale`
D. `-w /etc/netplan/ -p wa -k system-locale`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua relevan untuk perubahan identitas jaringan.

---
### Soal 98

**Pertanyaan:** Rule audit mana yang benar untuk perubahan permission/ownership?

A. `-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -k perm_mod`
B. `-a always,exit -F arch=b64 -S chown,fchown,lchown,fchownat -F auid>=1000 -F auid!=unset -k perm_mod`
C. `-a always,exit -F arch=b64 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod`
D. `-a never,exit -S chmod`

**Jawaban benar:** A, B, C

**Pembahasan:** chmod/chown/xattr harus tercatat untuk user interaktif.

---
### Soal 99

**Pertanyaan:** Rule audit mana yang benar untuk file deletion events?

A. `-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=1000 -F auid!=unset -k delete`
B. `-a always,exit -F arch=b32 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=1000 -F auid!=unset -k delete`
C. `-a never,exit -S unlink`
D. `rm -rf /var/log/audit`

**Jawaban benar:** A, B

**Pembahasan:** Deletion/rename user actions perlu diaudit.

---
### Soal 100

**Pertanyaan:** Rule audit mana yang benar untuk mount events?

A. `-a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=unset -k mounts`
B. `-a always,exit -F arch=b32 -S mount -F auid>=1000 -F auid!=unset -k mounts`
C. `-a never,exit -S mount`
D. `mount -a` sebagai audit rule

**Jawaban benar:** A, B

**Pembahasan:** Mount syscall dicatat untuk user interaktif.

---
### Soal 101

**Pertanyaan:** Rule audit mana yang benar untuk login/session record?

A. `-w /var/log/faillog -p wa -k logins`
B. `-w /var/log/lastlog -p wa -k logins`
C. `-w /var/run/utmp -p wa -k session`
D. `-w /var/log/wtmp -p wa -k session`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Login dan session records harus diaudit.

---
### Soal 102

**Pertanyaan:** Rule audit mana yang benar untuk kernel module changes?

A. `-w /sbin/insmod -p x -k modules`
B. `-w /sbin/rmmod -p x -k modules`
C. `-w /sbin/modprobe -p x -k modules`
D. `-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k modules`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Eksekusi tools dan syscall module loading/unloading perlu dicatat.

---
### Soal 103

**Pertanyaan:** Command mana yang benar untuk membuat audit rule privileged commands secara dinamis?

A. `find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null`
B. `printf -- "-a always,exit -F path=%s -F perm=x -F auid>=%s -F auid!=unset -k privileged\n" "$file" "$UID_MIN"`
C. `awk '/^\s*UID_MIN/{print $2}' /etc/login.defs`
D. `chmod -R u+s /usr/bin`

**Jawaban benar:** A, B, C

**Pembahasan:** Privileged commands diaudit berdasarkan SUID/SGID baseline; jangan menambah SUID massal.

---
### Soal 104

**Pertanyaan:** Command mana yang benar untuk load dan validasi audit rules?

A. `augenrules --load`
B. `auditctl -l | head -50`
C. `auditctl -s`
D. `auditctl -D` sebagai final hardening

**Jawaban benar:** A, B, C

**Pembahasan:** `auditctl -D` menghapus rule dan bukan final hardening.

---
### Soal 105

**Pertanyaan:** Command mana yang benar untuk membuat audit immutable setelah rules stabil?

A. `echo '-e 2' >/etc/audit/rules.d/99-finalize.rules`
B. `augenrules --load`
C. `grep -E '^-e 2' /etc/audit/audit.rules`
D. `auditctl -e 0`

**Jawaban benar:** A, B, C

**Pembahasan:** `-e 2` mengunci rule sampai reboot; `-e 0` melemahkan.

---
### Soal 106

**Pertanyaan:** Permission mana yang benar untuk file audit dan audit tools?

A. `chown root:root /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true`
B. `chmod 640 /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true`
C. `chmod 750 /sbin/auditctl /sbin/auditd /sbin/ausearch /sbin/aureport /sbin/autrace /sbin/augenrules 2>/dev/null || true`
D. `chmod 777 /etc/audit/auditd.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit config/tools harus root-owned dan tidak writable oleh user biasa.

---
### Soal 107

**Pertanyaan:** Permission mana yang benar untuk `/var/log/audit`?

A. `chown root:adm /var/log/audit 2>/dev/null || true`
B. `chmod 750 /var/log/audit 2>/dev/null || true`
C. `find /var/log/audit -type f -exec chown root:adm {} \; -exec chmod 640 {} \; 2>/dev/null || true`
D. `chmod -R 777 /var/log/audit`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit log harus terlindungi; jangan world-writable.

---
### Soal 108

**Pertanyaan:** Command mana yang benar untuk permission file account penting?

A. `chown root:root /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true`
B. `chmod 644 /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true`
C. `chown root:shadow /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true`
D. `chmod 640 /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true`

**Jawaban benar:** A, B, C, D

**Pembahasan:** File passwd/group readable; shadow/gshadow lebih terbatas.

---
### Soal 109

**Pertanyaan:** Command mana yang benar untuk audit permission file penting?

A. `stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow /etc/shells`
B. `ls -l /etc/passwd /etc/shadow`
C. `chmod 777 /etc/shadow`
D. `chown user:user /etc/passwd`

**Jawaban benar:** A, B

**Pembahasan:** Stat/ls adalah audit; chmod/chown longgar salah.

---
### Soal 110

**Pertanyaan:** Command mana yang benar untuk mencari dan memperbaiki file world-writable?

A. `find / -xdev -type f -perm -0002 -print 2>/dev/null`
B. `find / -xdev -type f -perm -0002 -exec chmod o-w {} \; 2>/dev/null`
C. `find / -xdev -type d -perm -0002 ! -perm -1000 -print 2>/dev/null`
D. `find / -xdev -type d -perm -0002 ! -perm -1000 -exec chmod a+t {} \; 2>/dev/null`

**Jawaban benar:** A, B, C, D

**Pembahasan:** File world-writable perlu review/perbaikan; directory world-writable perlu sticky bit bila sah.

---
### Soal 111

**Pertanyaan:** Command mana yang benar untuk mencari file tanpa owner/group?

A. `find / -xdev \( -nouser -o -nogroup \) -print 2>/dev/null`
B. `find / -nouser -print 2>/dev/null`
C. `find / -nogroup -print 2>/dev/null`
D. `chown -R root:root /`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit boleh; jangan chown seluruh filesystem otomatis.

---
### Soal 112

**Pertanyaan:** Command mana yang benar untuk review SUID/SGID?

A. `find / -xdev \( -perm -4000 -o -perm -2000 \) -type f -print 2>/dev/null | sort`
B. `find / -xdev -type f -perm -4000 -print 2>/dev/null`
C. `find / -xdev -type f -perm -2000 -print 2>/dev/null`
D. `chmod -R u+s /usr/bin`

**Jawaban benar:** A, B, C

**Pembahasan:** Review SUID/SGID, jangan menambah SUID massal.

---
### Soal 113

**Pertanyaan:** Command mana yang benar untuk audit shadowed password dan empty password?

A. `awk -F: '($2 != "x") {print $1 ": password field is not x"}' /etc/passwd`
B. `awk -F: '($2 == "") {print $1}' /etc/shadow`
C. `passwd -l <nama_user>` jika ditemukan akun kosong
D. `passwd -d <nama_user>`

**Jawaban benar:** A, B, C

**Pembahasan:** `passwd -d` menghapus password dan berbahaya.

---
### Soal 114

**Pertanyaan:** Command mana yang benar untuk audit duplicate UID/GID/user/group?

A. `cut -d: -f3 /etc/passwd | sort | uniq -d`
B. `cut -d: -f3 /etc/group | sort | uniq -d`
C. `cut -d: -f1 /etc/passwd | sort | uniq -d`
D. `cut -d: -f1 /etc/group | sort | uniq -d`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua adalah audit konsistensi account/group.

---
### Soal 115

**Pertanyaan:** Command mana yang benar untuk audit home directory user interaktif?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1,$6}' /etc/passwd`
B. `[ -d "$home" ] || echo "Missing home: $user $home"` dalam loop
C. `[ -d "$home" ] && chmod go-w "$home"` sebagai perbaikan permission umum
D. `chmod -R 777 /home`

**Jawaban benar:** A, B, C

**Pembahasan:** Home directory tidak boleh writable oleh group/others tanpa kebutuhan.

---
### Soal 116

**Pertanyaan:** Command mana yang benar untuk mencari dotfile berisiko?

A. `find /home -name '.rhosts' -o -name '.netrc' -o -name '.forward' 2>/dev/null`
B. `find /home -name '.netrc' -exec ls -l {} \; 2>/dev/null`
C. `find /home -name '.bashrc' -delete`
D. `rm -rf /home/*`

**Jawaban benar:** A, B

**Pembahasan:** Dotfile seperti .rhosts/.netrc/.forward perlu review. Jangan hapus .bashrc/home massal.

---
### Soal 117

**Pertanyaan:** Pilih command yang benar untuk memperbaiki semua permission secara cepat tanpa audit dan tanpa review.

A. `chmod -R 600 /etc`
B. `chown -R root:root /home`
C. `find / -perm /6000 -delete`
D. `chmod -R 777 /var/log`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi destruktif/berbahaya. STIG membutuhkan audit dan remediation terkontrol.

---
### Soal 118

**Pertanyaan:** Command mana yang benar untuk checklist validasi akhir pasca-reboot?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd`
B. `ufw status verbose`
C. `sshd -t`
D. `aa-status`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua bagian dari validasi akhir.

---
### Soal 119

**Pertanyaan:** Command mana yang benar untuk validasi audit dan AIDE akhir?

A. `auditctl -s`
B. `auditctl -l | head`
C. `aide -c /etc/aide/aide.conf --check`
D. `rm -rf /var/lib/aide`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit dan AIDE perlu dicek; jangan hapus database AIDE.

---
### Soal 120

**Pertanyaan:** Command mana yang benar untuk validasi password policy akhir?

A. `grep -v '^#' /etc/security/pwquality.conf | sed '/^$/d'`
B. `grep -E '^PASS_MAX_DAYS|^PASS_MIN_DAYS|^PASS_WARN_AGE' /etc/login.defs`
C. `faillock --user <nama_user>`
D. `passwd -d <nama_user>` sebagai validasi

**Jawaban benar:** A, B, C

**Pembahasan:** `passwd -d` bukan validasi dan berbahaya.

---

## 7. Pengenalan STIG, Severity, Exception, dan Prinsip Audit
### Soal 121

**Pertanyaan:** Dalam konteks Ubuntu STIG, perintah atau bukti mana yang benar untuk menunjukkan sistem memakai rilis vendor-supported?

A. `cat /etc/os-release`
B. `grep DISTRIB_DESCRIPTION /etc/lsb-release`
C. `ubuntu-security-status 2>/dev/null || true`
D. `uname -a` saja cukup untuk membuktikan vendor support

**Jawaban benar:** A, B, C

**Pembahasan:** Vendor support perlu dibuktikan dari rilis dan status dukungan, bukan hanya kernel string.

---
### Soal 122

**Pertanyaan:** Jika suatu rule STIG seperti FIPS, PIV/PKI, atau remote audit server tidak bisa diterapkan di lab kampus, tindakan mana yang benar?

A. Catat sebagai exception/N/A dengan alasan
B. Jelaskan risiko dan mitigasi pengganti
C. Abaikan tanpa catatan
D. Palsukan output command agar lulus

**Jawaban benar:** A, B

**Pembahasan:** STIG memperbolehkan exception yang terdokumentasi; tidak boleh diabaikan/palsu.

---
### Soal 123

**Pertanyaan:** Data apa saja yang tepat masuk exception register?

A. Rule/Area
B. Alasan
C. Risiko
D. Mitigasi dan tanggal review

**Jawaban benar:** A, B, C, D

**Pembahasan:** Exception harus dapat diaudit dan direview.

---
### Soal 124

**Pertanyaan:** Perintah mana yang benar untuk mengaudit layanan aktif yang perlu dibandingkan dengan daftar port/protocol/service yang disetujui?

A. `ss -tulpen`
B. `systemctl --type=service --state=running`
C. `ufw status verbose`
D. `lsof -i -P -n` jika tersedia

**Jawaban benar:** A, B, C, D

**Pembahasan:** Audit network exposure perlu melihat port, service, dan firewall.

---
### Soal 125

**Pertanyaan:** Jika sistem adalah server tanpa GUI, command mana yang benar untuk area GDM/GNOME?

A. Tandai rule GUI sebagai N/A bila paket GUI memang tidak ada
B. `dpkg -l | grep -E 'gdm3|ubuntu-desktop|gnome-shell' || true`
C. `systemctl status gdm3.service 2>/dev/null || true`
D. `apt install -y ubuntu-desktop` agar rule GUI bisa diterapkan

**Jawaban benar:** A, B, C

**Pembahasan:** Server headless tidak perlu memasang GUI hanya agar bisa menerapkan rule GUI.

---
### Soal 126

**Pertanyaan:** Jika setelah hardening SSH admin terkunci dari server remote, apa pencegahan yang benar sebelum remediation?

A. Siapkan console/recovery access
B. Validasi `sshd -t` sebelum restart
C. Buat rule firewall SSH sebelum `ufw enable`
D. Restart SSH tanpa sesi cadangan dan tanpa akses console

**Jawaban benar:** A, B, C

**Pembahasan:** SSH/PAM/UFW adalah bagian paling berisiko menyebabkan lockout.

---
### Soal 127

**Pertanyaan:** Untuk pertanyaan command yang mencampur Debian/Arch dan Ubuntu, opsi mana yang harus dipilih pada Ubuntu?

A. `apt install -y ufw`
B. `systemctl enable --now ufw.service`
C. `pacstrap -K /mnt ufw`
D. `pacman -S ufw`

**Jawaban benar:** A, B

**Pembahasan:** Ubuntu memakai APT/systemd service Ubuntu, bukan pacstrap/pacman.

---
### Soal 128

**Pertanyaan:** Pilih command yang benar untuk membuat Ubuntu STIG memakai `pacman` sebagai package manager resmi.

A. `pacman -Syu`
B. `pacman-key --init`
C. `pacstrap -K /mnt base`
D. `makepkg -si`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: Ubuntu memakai APT/dpkg, bukan pacman.

---

## 8. Tambahan Praktik Detail per Modul Ubuntu STIG
### Soal 129

**Pertanyaan:** Perintah mana yang benar untuk memeriksa apakah service penting aktif setelah hardening?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd`
B. `systemctl is-enabled ssh ufw apparmor chrony rsyslog auditd`
C. `systemctl status ssh ufw apparmor chrony rsyslog auditd`
D. `service --status-all` sebagai satu-satunya bukti compliance

**Jawaban benar:** A, B, C

**Pembahasan:** systemctl is-active/is-enabled/status memberikan bukti service. `service --status-all` kurang spesifik sebagai satu-satunya bukti.

---
### Soal 130

**Pertanyaan:** Command mana yang benar untuk memastikan hanya port yang disetujui yang listening?

A. `ss -tulpen`
B. `ufw status verbose`
C. `lsof -i -P -n` jika tersedia
D. `nmap localhost` jika tool tersedia dan diizinkan

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua dapat membantu inventaris port; nmap perlu izin organisasi/lab.

---
### Soal 131

**Pertanyaan:** Jika UFW sudah aktif dan admin ingin melihat rule bernomor untuk review, command mana yang benar?

A. `ufw status numbered`
B. `ufw status verbose`
C. `ufw delete <nomor_rule>` untuk menghapus rule yang tidak disetujui setelah review
D. `ufw allow all`

**Jawaban benar:** A, B, C

**Pembahasan:** Review rule bernomor memudahkan penghapusan; allow all salah.

---
### Soal 132

**Pertanyaan:** Command mana yang benar untuk menambahkan rule UFW HTTPS hanya jika host memang web server?

A. `ufw allow 443/tcp comment 'HTTPS'`
B. `ufw allow 80/tcp comment 'HTTP'` bila HTTP memang dibutuhkan
C. `ufw allow 443/udp` sebagai pengganti HTTPS standar TCP
D. `ufw allow 1:65535/tcp`

**Jawaban benar:** A, B

**Pembahasan:** HTTPS standar memakai TCP 443; buka hanya yang dibutuhkan.

---
### Soal 133

**Pertanyaan:** Command mana yang benar untuk audit konfigurasi SSH effective setelah drop-in diterapkan?

A. `sshd -T | grep permitrootlogin`
B. `sshd -T | grep clientaliveinterval`
C. `sshd -T | grep x11forwarding`
D. `cat /etc/ssh/sshd_config.d/60-stig-hardening.conf`

**Jawaban benar:** A, B, C, D

**Pembahasan:** sshd -T menunjukkan effective config; membaca drop-in membantu review sumber konfigurasi.

---
### Soal 134

**Pertanyaan:** Jika `sshd -t` gagal setelah mengubah cipher/MAC/KEX, tindakan mana yang benar?

A. Jangan restart SSH dulu
B. Perbaiki file drop-in yang error
C. Gunakan sesi console/cadangan untuk rollback
D. Tetap jalankan `systemctl restart ssh.service` agar cepat tahu error

**Jawaban benar:** A, B, C

**Pembahasan:** Restart SSH saat syntax error dapat memutus akses.

---
### Soal 135

**Pertanyaan:** Command mana yang benar untuk memeriksa SSH client tidak memakai algoritma yang tidak sesuai baseline?

A. `ssh -G localhost | egrep 'ciphers|macs' | head`
B. `grep -R '^\s*Ciphers\|^\s*MACs' /etc/ssh/ssh_config /etc/ssh/ssh_config.d/ 2>/dev/null`
C. `ssh -Q cipher` untuk melihat algoritma tersedia, bukan konfigurasi aktif
D. `echo 'Ciphers 3des-cbc' >> /etc/ssh/ssh_config`

**Jawaban benar:** A, B, C

**Pembahasan:** `ssh -Q` berguna untuk daftar dukungan, tetapi konfigurasi aktif dilihat dengan `ssh -G`.

---
### Soal 136

**Pertanyaan:** Command mana yang benar untuk menghapus paket Bluetooth setelah memastikan server tidak membutuhkannya?

A. `systemctl disable --now bluetooth.service 2>/dev/null || true`
B. `apt purge -y bluez 2>/dev/null || true`
C. `apt autoremove -y --purge`
D. `rfkill unblock bluetooth`

**Jawaban benar:** A, B, C

**Pembahasan:** Unblock bluetooth berlawanan dengan tujuan disable.

---
### Soal 137

**Pertanyaan:** Perintah mana yang benar untuk memverifikasi AppArmor profile dan status enforcing/complain?

A. `aa-status`
B. `systemctl is-active apparmor.service`
C. `journalctl -u apparmor.service --no-pager | tail`
D. `getenforce` sebagai bukti utama AppArmor

**Jawaban benar:** A, B, C

**Pembahasan:** `getenforce` untuk SELinux, bukan AppArmor.

---
### Soal 138

**Pertanyaan:** Command mana yang benar untuk validasi kernel hardening yang sudah diterapkan?

A. `sysctl net.ipv4.tcp_syncookies`
B. `sysctl kernel.randomize_va_space`
C. `sysctl kernel.kptr_restrict`
D. `sysctl fs.protected_hardlinks fs.protected_symlinks`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua memvalidasi sysctl hardening.

---
### Soal 139

**Pertanyaan:** Command mana yang benar untuk konfigurasi rsyslog agar tidak menjadi server penerima log remote jika tidak dibutuhkan?

A. Review `/etc/rsyslog.conf` dan `/etc/rsyslog.d/*.conf` dari modul input TCP/UDP
B. `grep -R 'imudp\|imtcp\|InputTCPServerRun\|UDPServerRun' /etc/rsyslog.conf /etc/rsyslog.d/ 2>/dev/null`
C. Disable konfigurasi listener yang tidak sah lalu restart rsyslog
D. `ufw allow 514/udp` pada semua host

**Jawaban benar:** A, B, C

**Pembahasan:** Host biasa boleh mengirim log, tetapi tidak harus menerima log dari jaringan.

---
### Soal 140

**Pertanyaan:** Command mana yang benar untuk melihat journal tanpa membuka akses luas ke user biasa?

A. `journalctl -xe` sebagai root/sudo
B. `journalctl -u ssh.service`
C. `ls -ld /var/log/journal /run/log/journal 2>/dev/null`
D. `chmod -R 777 /var/log/journal`

**Jawaban benar:** A, B, C

**Pembahasan:** Journal bisa sensitif; permission jangan dilonggarkan.

---
### Soal 141

**Pertanyaan:** Rule audit mana yang benar untuk AppArmor/MAC tools jika ingin mencatat perubahan MAC policy?

A. `-w /etc/apparmor/ -p wa -k MAC-policy`
B. `-w /etc/apparmor.d/ -p wa -k MAC-policy`
C. `-a always,exit -F path=/usr/bin/aa-enforce -F perm=x -k MAC-policy` jika path tersedia dan digunakan
D. `auditctl -D` sebagai MAC protection

**Jawaban benar:** A, B, C

**Pembahasan:** Audit policy dan tool MAC relevan; `auditctl -D` menghapus rules.

---
### Soal 142

**Pertanyaan:** Command mana yang benar untuk mencatat perubahan netplan dalam audit rules?

A. `-w /etc/netplan/ -p wa -k system-locale`
B. `-w /etc/hosts -p wa -k system-locale`
C. `-w /etc/hostname -p wa -k system-locale`
D. `-w /etc/netplan/ -p x -k execute-netplan` sebagai satu-satunya rule

**Jawaban benar:** A, B, C

**Pembahasan:** Perubahan file konfigurasi jaringan perlu `wa`, bukan hanya execute.

---
### Soal 143

**Pertanyaan:** Command mana yang benar untuk membuat file sudoers drop-in aman setelah ditulis?

A. `chmod 440 /etc/sudoers.d/60-stig-hardening`
B. `chown root:root /etc/sudoers.d/60-stig-hardening`
C. `visudo -cf /etc/sudoers.d/60-stig-hardening`
D. `chmod 777 /etc/sudoers.d/60-stig-hardening`

**Jawaban benar:** A, B, C

**Pembahasan:** Sudoers harus root-owned, mode 440, dan tervalidasi.

---
### Soal 144

**Pertanyaan:** Command mana yang benar untuk audit user interaktif yang mungkin memiliki shell login valid?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1,$7}' /etc/passwd`
B. `getent passwd`
C. `grep -E '/bin/(bash|sh|zsh)$' /etc/passwd`
D. `cat /etc/shadow` untuk semua peserta ujian

**Jawaban benar:** A, B, C

**Pembahasan:** Audit user/shell bisa dari passwd/getent; shadow sensitif dan tidak perlu dibuka luas.

---
### Soal 145

**Pertanyaan:** Command mana yang benar untuk mengunci akun yang tidak boleh login?

A. `passwd -l <nama_user>`
B. `usermod -L <nama_user>`
C. `usermod -s /usr/sbin/nologin <nama_user>`
D. `passwd -d <nama_user>`

**Jawaban benar:** A, B, C

**Pembahasan:** `passwd -d` menghapus password dan berisiko.

---
### Soal 146

**Pertanyaan:** Command mana yang benar untuk memeriksa group `shadow` kosong atau tidak sebelum remediation?

A. `getent group shadow`
B. `grep '^shadow:' /etc/group`
C. `awk -F: '$1=="shadow"{print $4}' /etc/group`
D. `groupdel shadow` sebagai langkah audit

**Jawaban benar:** A, B, C

**Pembahasan:** Audit dulu; jangan menghapus group sistem.

---
### Soal 147

**Pertanyaan:** Command mana yang benar untuk audit last successful login display/session records?

A. `lastlog`
B. `last`
C. `lastb` sebagai root/sudo jika tersedia
D. `rm -f /var/log/wtmp /var/log/btmp`

**Jawaban benar:** A, B, C

**Pembahasan:** wtmp/btmp/lastlog adalah record login; jangan dihapus.

---
### Soal 148

**Pertanyaan:** Command mana yang benar untuk melakukan reboot setelah update kernel dan validasi ulang?

A. `reboot`
B. `uname -r` setelah boot ulang
C. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd` setelah boot ulang
D. `poweroff` tanpa rencana recovery pada server remote

**Jawaban benar:** A, B, C

**Pembahasan:** Reboot harus terencana dan divalidasi pasca-boot.

---
### Soal 149

**Pertanyaan:** Jika semua opsi dalam soal adalah command destruktif seperti menghapus `/etc`, `/var/log`, atau audit rules, jawaban yang benar adalah?

A. Pilih command paling cepat
B. PASS / kosongkan jawaban
C. Pilih command yang memakai `-rf` karena praktis
D. Pilih semua opsi

**Jawaban benar:** B

**Pembahasan:** Dalam ujian pilihan ganda jamak, PASS digunakan saat semua opsi salah.

---
### Soal 150

**Pertanyaan:** Command mana yang benar untuk membuat laporan ringkas hasil validasi akhir ke file teks?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd > hardening-validation.txt`
B. `ufw status verbose >> hardening-validation.txt`
C. `auditctl -s >> hardening-validation.txt`
D. `cat /etc/shadow >> hardening-validation.txt`

**Jawaban benar:** A, B, C

**Pembahasan:** Jangan memasukkan isi `/etc/shadow` ke laporan latihan karena sensitif.

---

## Kunci Jawaban Ringkas

| No. | Jawaban |
|---:|---|
| 1 | C |
| 2 | A, B, C |
| 3 | A, B, C |
| 4 | A, B, D |
| 5 | A, B, C |
| 6 | A, B, D |
| 7 | A, D |
| 8 | A, B, C |
| 9 | A, B |
| 10 | A, B, C |
| 11 | A, B, C |
| 12 | A, B |
| 13 | A, B, D |
| 14 | A, B |
| 15 | A, B |
| 16 | A, B, C |
| 17 | A, B, C |
| 18 | A, B |
| 19 | A, B, C |
| 20 | A, B, D |
| 21 | A, B, D |
| 22 | A, B, D |
| 23 | A, B, C |
| 24 | PASS / kosong |
| 25 | A, B, C |
| 26 | A, B |
| 27 | A, B, C |
| 28 | A, B, C |
| 29 | A, B, C |
| 30 | A, B |
| 31 | A, B, C, D |
| 32 | A, B |
| 33 | A, B, C, D |
| 34 | A, B, C |
| 35 | A, B, C |
| 36 | A, B, C |
| 37 | A, B, C |
| 38 | PASS / kosong |
| 39 | A, B, C, D |
| 40 | A, B, C |
| 41 | A, B, D |
| 42 | A, B |
| 43 | A, B, C, D |
| 44 | B, C |
| 45 | A, B, C, D |
| 46 | A, B, D |
| 47 | PASS / kosong |
| 48 | A, B, C, D |
| 49 | A, B, C |
| 50 | A, B, C, D |
| 51 | A, B, C, D |
| 52 | A, B, C, D |
| 53 | A, B, C, D |
| 54 | A, B, C |
| 55 | A, B, C |
| 56 | A, B, C |
| 57 | A, B, C |
| 58 | A, B, C |
| 59 | A, B, C |
| 60 | A, B, C |
| 61 | A, B, C |
| 62 | A, B, C |
| 63 | A, B, C, D |
| 64 | A, B, C, D |
| 65 | A, B |
| 66 | A, B, C |
| 67 | A, B |
| 68 | A, B, C, D |
| 69 | A, B |
| 70 | A |
| 71 | A, B, C |
| 72 | A, B, C, D |
| 73 | A, B |
| 74 | A, B, C |
| 75 | A, B, C |
| 76 | PASS / kosong |
| 77 | C |
| 78 | A, B, C |
| 79 | A, B, C |
| 80 | A, B |
| 81 | A, B, C, D |
| 82 | A, B |
| 83 | A, B, C |
| 84 | A, B, C |
| 85 | A, B, D |
| 86 | A, B, C |
| 87 | A, B, C |
| 88 | A, B |
| 89 | A, B, C |
| 90 | A, B, C, D |
| 91 | A, B, C |
| 92 | A, B, C |
| 93 | A, B, C, D |
| 94 | A |
| 95 | A, B, C |
| 96 | A, B, C |
| 97 | A, B, C, D |
| 98 | A, B, C |
| 99 | A, B |
| 100 | A, B |
| 101 | A, B, C, D |
| 102 | A, B, C, D |
| 103 | A, B, C |
| 104 | A, B, C |
| 105 | A, B, C |
| 106 | A, B, C |
| 107 | A, B, C |
| 108 | A, B, C, D |
| 109 | A, B |
| 110 | A, B, C, D |
| 111 | A, B, C |
| 112 | A, B, C |
| 113 | A, B, C |
| 114 | A, B, C, D |
| 115 | A, B, C |
| 116 | A, B |
| 117 | PASS / kosong |
| 118 | A, B, C, D |
| 119 | A, B, C |
| 120 | A, B, C |
| 121 | A, B, C |
| 122 | A, B |
| 123 | A, B, C, D |
| 124 | A, B, C, D |
| 125 | A, B, C |
| 126 | A, B, C |
| 127 | A, B |
| 128 | PASS / kosong |
| 129 | A, B, C |
| 130 | A, B, C, D |
| 131 | A, B, C |
| 132 | A, B |
| 133 | A, B, C, D |
| 134 | A, B, C |
| 135 | A, B, C |
| 136 | A, B, C |
| 137 | A, B, C |
| 138 | A, B, C, D |
| 139 | A, B, C |
| 140 | A, B, C |
| 141 | A, B, C |
| 142 | A, B, C |
| 143 | A, B, C |
| 144 | A, B, C |
| 145 | A, B, C |
| 146 | A, B, C |
| 147 | A, B, C |
| 148 | A, B, C |
| 149 | B |
| 150 | A, B, C |
