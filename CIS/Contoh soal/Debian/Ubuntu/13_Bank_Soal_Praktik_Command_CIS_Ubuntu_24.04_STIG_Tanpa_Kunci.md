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

**Jawaban:** ........................................................

---
### Soal 2

**Pertanyaan:** Perintah mana yang benar untuk memastikan sistem yang diuji adalah Ubuntu 24.04 LTS?

A. `cat /etc/os-release`
B. `grep DISTRIB_DESCRIPTION /etc/lsb-release`
C. `lsb_release -a`
D. `pacman -Qi base`

**Jawaban:** ........................................................

---
### Soal 3

**Pertanyaan:** Manakah command yang benar untuk masuk ke root shell sementara pada Ubuntu sebelum hardening?

A. `sudo -i`
B. `sudo -v`
C. `su -` jika root password memang diaktifkan kebijakan lokal
D. `arch-chroot /mnt` setelah Ubuntu sudah boot normal

**Jawaban:** ........................................................

---
### Soal 4

**Pertanyaan:** Perintah mana yang benar untuk membuat direktori backup konfigurasi sebelum hardening?

A. `mkdir -p /root/hardening-backup/$(date +%F)`
B. `BACKUP_DIR="/root/hardening-backup/$(date +%F)"`
C. `rm -rf /etc/ssh /etc/pam.d`
D. `cp -a /etc/ssh "$BACKUP_DIR/ssh" 2>/dev/null || true`

**Jawaban:** ........................................................

---
### Soal 5

**Pertanyaan:** Perintah mana yang benar untuk update awal sistem Ubuntu setelah instalasi?

A. `apt update`
B. `apt -y full-upgrade`
C. `apt -y autoremove --purge`
D. `pacman -Syu`

**Jawaban:** ........................................................

---

## 1. Instalasi, Partisi, FSTAB, LUKS, dan FIPS
### Soal 6

**Pertanyaan:** Baris `/etc/fstab` mana yang benar untuk `/tmp` sesuai prinsip hardening Ubuntu STIG/CIS?

A. `UUID=<uuid-tmp> /tmp ext4 defaults,nodev,nosuid,noexec 0 2`
B. `tmpfs /tmp tmpfs rw,nosuid,nodev,noexec,relatime,size=2G,mode=1777 0 0`
C. `UUID=<uuid-tmp> /tmp ext4 defaults,dev,suid,exec 0 2`
D. `UUID=<uuid-tmp> /tmp ext4 defaults,nodev,nosuid,noexec 0 0`

**Jawaban:** ........................................................

---
### Soal 7

**Pertanyaan:** Baris `/etc/fstab` mana yang benar untuk `/var/log/audit` agar audit log dipisah dan tidak dapat menjalankan binary?

A. `UUID=<uuid-audit> /var/log/audit ext4 defaults,nodev,nosuid,noexec 0 2`
B. `UUID=<uuid-audit> /var/log/audit ext4 defaults,dev,suid,exec 0 2`
C. `tmpfs /var/log/audit tmpfs defaults,nodev,nosuid,noexec 0 0`
D. `UUID=<uuid-audit> /var/log/audit ext4 rw,nodev,nosuid,noexec,relatime 0 2`

**Jawaban:** ........................................................

---
### Soal 8

**Pertanyaan:** Perintah mana yang benar untuk memvalidasi mount option beberapa mount point penting setelah instalasi?

A. `findmnt -no TARGET,OPTIONS /tmp /var /var/tmp /var/log /var/log/audit /home`
B. `mount | grep ' /var/log/audit '`
C. `cat /etc/fstab`
D. `chmod noexec /tmp`

**Jawaban:** ........................................................

---
### Soal 9

**Pertanyaan:** Jika instalasi Ubuntu dilakukan manual dengan LUKS2, command mana yang benar secara praktik untuk membuat dan membuka container LUKS?

A. `cryptsetup luksFormat --type luks2 /dev/nvme0n1p3`
B. `cryptsetup open /dev/nvme0n1p3 cryptroot`
C. `mkfs.ext4 /dev/nvme0n1p3` sebelum `luksFormat`
D. `luksctl create /dev/nvme0n1p3 cryptroot`

**Jawaban:** ........................................................

---
### Soal 10

**Pertanyaan:** Setelah LUKS dibuka sebagai `/dev/mapper/cryptroot`, command LVM mana yang benar untuk membuat layout terpisah?

A. `pvcreate /dev/mapper/cryptroot`
B. `vgcreate vgubuntu /dev/mapper/cryptroot`
C. `lvcreate -L 20G -n lv_root vgubuntu`
D. `vgcreate /dev/mapper/cryptroot vgubuntu`

**Jawaban:** ........................................................

---
### Soal 11

**Pertanyaan:** Command mana yang benar untuk membuat filesystem ext4 pada logical volume Ubuntu manual?

A. `mkfs.ext4 -L ROOT /dev/vgubuntu/lv_root`
B. `mkfs.ext4 -L VAR /dev/vgubuntu/lv_var`
C. `mkfs.ext4 -L AUDIT /dev/vgubuntu/lv_audit`
D. `mkfs.fat -F32 /dev/vgubuntu/lv_root`

**Jawaban:** ........................................................

---
### Soal 12

**Pertanyaan:** Untuk FIPS mode pada Ubuntu STIG, command mana yang benar untuk memvalidasi apakah FIPS aktif?

A. `cat /proc/sys/crypto/fips_enabled`
B. `grep fips /proc/cmdline`
C. `fips-mode-setup --check` sebagai command generik wajib Ubuntu
D. `ufw status verbose`

**Jawaban:** ........................................................

---
### Soal 13

**Pertanyaan:** Pilihan mana yang benar terkait implementasi FIPS pada Ubuntu 24.04 STIG?

A. Tambahkan `fips=1` saat instalasi bila lingkungan compliance memang membutuhkan FIPS
B. Gunakan dokumentasi vendor/Ubuntu Pro untuk sistem yang sudah berjalan
C. Aktifkan FIPS dengan `ufw enable fips`
D. Catat exception jika lab/kampus tidak memiliki kebutuhan FIPS formal

**Jawaban:** ........................................................

---
### Soal 14

**Pertanyaan:** Perintah mana yang benar untuk mengunci root login langsung setelah akun admin sudo tersedia?

A. `sudo passwd -l root`
B. `passwd -l root` saat sudah berada sebagai root
C. `passwd -u root`
D. `usermod -U root`

**Jawaban:** ........................................................

---

## 2. Baseline, Package, Service Minimization, dan Chrony
### Soal 15

**Pertanyaan:** Perintah mana yang benar untuk memasang unattended upgrades dan apt-listchanges pada Ubuntu?

A. `apt install -y unattended-upgrades apt-listchanges`
B. `apt update` sebelum instalasi paket
C. `dnf install unattended-upgrades`
D. `pacman -S unattended-upgrades`

**Jawaban:** ........................................................

---
### Soal 16

**Pertanyaan:** Konfigurasi mana yang benar untuk membersihkan komponen lama setelah unattended upgrades?

A. `Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";`
B. `Unattended-Upgrade::Remove-Unused-Dependencies "true";`
C. `Unattended-Upgrade::Remove-New-Unused-Dependencies "true";`
D. `Unattended-Upgrade::Remove-All-Packages "true";`

**Jawaban:** ........................................................

---
### Soal 17

**Pertanyaan:** Perintah mana yang benar untuk audit konfigurasi `Remove-Unused` pada APT?

A. `grep -R "Remove-Unused" /etc/apt/apt.conf.d/`
B. `apt-config dump | grep Remove-Unused`
C. `cat /etc/apt/apt.conf.d/52-hardening-remove-unused`
D. `pacman -Q | grep Remove-Unused`

**Jawaban:** ........................................................

---
### Soal 18

**Pertanyaan:** Konfigurasi mana yang benar untuk mengurangi weak dependencies APT pada baseline tambahan?

A. `APT::Install-Recommends "0";`
B. `APT::Install-Suggests "0";`
C. `APT::Install-Recommends "1";`
D. `APT::Install-Suggests "1";`

**Jawaban:** ........................................................

---
### Soal 19

**Pertanyaan:** Perintah mana yang benar untuk menghapus layanan lama/tidak aman yang disebut dalam implementasi Ubuntu STIG?

A. `apt purge -y telnetd rsh-server ntp systemd-timesyncd || true`
B. `apt purge -y nis talk talkd rsh-client telnet ldap-utils ftp tnftp xinetd || true`
C. `apt autoremove -y --purge`
D. `apt install -y telnetd rsh-server`

**Jawaban:** ........................................................

---
### Soal 20

**Pertanyaan:** Command mana yang benar untuk audit apakah paket lama seperti telnetd/rsh-server/ntp masih terpasang?

A. `dpkg -l | egrep 'telnetd|rsh-server|ntp|systemd-timesyncd|nis|talkd|xinetd' || true`
B. `apt list --installed | egrep 'telnetd|rsh-server|ntp' || true`
C. `rpm -qa | grep telnetd`
D. `systemctl list-unit-files | grep -E 'telnet|rsh|ntp'`

**Jawaban:** ........................................................

---
### Soal 21

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan Chrony?

A. `apt install -y chrony`
B. `systemctl enable --now chrony.service`
C. `systemctl enable --now systemd-timesyncd.service chrony.service`
D. `chronyc tracking`

**Jawaban:** ........................................................

---
### Soal 22

**Pertanyaan:** Konfigurasi mana yang benar di `/etc/chrony/chrony.conf`?

A. `pool ntp.ubuntu.com iburst maxsources 4`
B. `server 192.168.10.1 iburst` jika itu NTP internal resmi
C. `allow 0.0.0.0/0` pada semua server biasa
D. `server time.example.org iburst`

**Jawaban:** ........................................................

---
### Soal 23

**Pertanyaan:** Command audit mana yang benar untuk memastikan Chrony berjalan dan memakai sumber waktu?

A. `systemctl is-enabled chrony.service`
B. `systemctl is-active chrony.service`
C. `chronyc sources -v`
D. `timedatectl set-ntp false`

**Jawaban:** ........................................................

---
### Soal 24

**Pertanyaan:** Jika soal meminta mengaktifkan `systemd-timesyncd` bersamaan dengan Chrony pada baseline Ubuntu STIG, jawaban yang benar adalah?

A. `systemctl enable --now systemd-timesyncd.service`
B. `systemctl enable --now chrony.service systemd-timesyncd.service`
C. `apt install -y ntp systemd-timesyncd chrony`
D. `timedatectl set-ntp true` sebagai pengganti Chrony tanpa audit

**Jawaban:** ........................................................

---

## 3. AIDE, AppArmor, Kernel, Memory, dan USB
### Soal 25

**Pertanyaan:** Perintah mana yang benar untuk memasang AIDE pada Ubuntu?

A. `apt install -y aide aide-common`
B. `aideinit`
C. `aide -c /etc/aide/aide.conf --check`
D. `pacman -S aide`

**Jawaban:** ........................................................

---
### Soal 26

**Pertanyaan:** Setelah `aideinit`, command mana yang benar untuk menjadikan database baru sebagai database aktif?

A. `cp -p /var/lib/aide/aide.db.new /var/lib/aide/aide.db`
B. `aide -c /etc/aide/aide.conf --check`
C. `rm -rf /var/lib/aide`
D. `mv /etc/aide/aide.conf /tmp`

**Jawaban:** ........................................................

---
### Soal 27

**Pertanyaan:** Entry AIDE mana yang benar untuk memantau integritas audit tool?

A. `/sbin/auditctl p+i+n+u+g+s+b+acl+xattrs+sha512`
B. `/sbin/auditd p+i+n+u+g+s+b+acl+xattrs+sha512`
C. `/sbin/ausearch p+i+n+u+g+s+b+acl+xattrs+sha512`
D. `/var/log/audit/audit.log p+i+n+u+g+s+b+acl+xattrs+sha512` sebagai satu-satunya kontrol audit tools

**Jawaban:** ........................................................

---
### Soal 28

**Pertanyaan:** Perintah mana yang benar agar laporan AIDE tidak silent?

A. `sed -i 's/^SILENTREPORTS=.*/SILENTREPORTS=no/' /etc/default/aide`
B. `grep SILENTREPORTS /etc/default/aide`
C. `echo 'SILENTREPORTS=no' >> /etc/default/aide` bila belum ada
D. `echo 'SILENTREPORTS=yes' > /etc/default/aide`

**Jawaban:** ........................................................

---
### Soal 29

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan AppArmor pada Ubuntu?

A. `apt install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra`
B. `systemctl enable --now apparmor.service`
C. `aa-status`
D. `systemctl enable --now selinux.service`

**Jawaban:** ........................................................

---
### Soal 30

**Pertanyaan:** Perintah mana yang benar untuk menjadikan profil AppArmor enforcing setelah diuji?

A. `aa-enforce /etc/apparmor.d/* 2>/dev/null || true`
B. `aa-status`
C. `aa-complain /etc/apparmor.d/*` sebagai hardening final
D. `systemctl stop apparmor.service`

**Jawaban:** ........................................................

---
### Soal 31

**Pertanyaan:** Sysctl mana yang benar untuk hardening kernel dan network pada Ubuntu STIG?

A. `net.ipv4.tcp_syncookies = 1`
B. `kernel.randomize_va_space = 2`
C. `kernel.kptr_restrict = 2`
D. `kernel.dmesg_restrict = 1`

**Jawaban:** ........................................................

---
### Soal 32

**Pertanyaan:** Sysctl mana yang benar untuk mencegah host biasa menjadi router IPv4/IPv6?

A. `net.ipv4.ip_forward = 0`
B. `net.ipv6.conf.all.forwarding = 0`
C. `net.ipv4.ip_forward = 1`
D. `net.ipv6.conf.all.forwarding = 1`

**Jawaban:** ........................................................

---
### Soal 33

**Pertanyaan:** Sysctl mana yang benar untuk menolak redirect/source route?

A. `net.ipv4.conf.all.accept_redirects = 0`
B. `net.ipv4.conf.default.accept_redirects = 0`
C. `net.ipv4.conf.all.accept_source_route = 0`
D. `net.ipv6.conf.default.accept_source_route = 0`

**Jawaban:** ........................................................

---
### Soal 34

**Pertanyaan:** Command mana yang benar untuk membuat konfigurasi sysctl persisten dan menerapkannya?

A. `cat >/etc/sysctl.d/60-stig-hardening.conf <<'EOF'`
B. `sysctl --system`
C. `sysctl net.ipv4.tcp_syncookies kernel.randomize_va_space` untuk validasi
D. `echo net.ipv4.tcp_syncookies=1 >> /etc/modules`

**Jawaban:** ........................................................

---
### Soal 35

**Pertanyaan:** Konfigurasi mana yang benar untuk membatasi core dump?

A. `* hard core 0` di `/etc/security/limits.d/60-hardening-coredump.conf`
B. `fs.suid_dumpable = 0` di sysctl
C. `systemctl disable --now systemd-coredump.socket 2>/dev/null || true` jika tidak dibutuhkan
D. `fs.suid_dumpable = 1`

**Jawaban:** ........................................................

---
### Soal 36

**Pertanyaan:** Perintah mana yang benar untuk menonaktifkan USB mass storage pada server yang tidak membutuhkan USB storage?

A. `cat >/etc/modprobe.d/usb-storage.conf` berisi `install usb-storage /bin/false` dan `blacklist usb-storage`
B. `modprobe -r usb-storage 2>/dev/null || true`
C. `modprobe -n -v usb-storage` untuk validasi
D. `echo usb-storage >> /etc/modules`

**Jawaban:** ........................................................

---
### Soal 37

**Pertanyaan:** Command mana yang benar untuk audit module USB storage masih loaded atau tidak?

A. `lsmod | grep usb_storage || true`
B. `modprobe -n -v usb-storage`
C. `grep -R 'usb-storage' /etc/modprobe.d/`
D. `systemctl status usb-storage.service`

**Jawaban:** ........................................................

---
### Soal 38

**Pertanyaan:** Pilih command yang benar untuk mengaktifkan semua core dump demi forensik tanpa batas sesuai baseline STIG.

A. `ulimit -c unlimited`
B. `fs.suid_dumpable = 2`
C. `Storage=external` dengan `ProcessSizeMax=infinity`
D. `systemctl enable --now systemd-coredump.socket`

**Jawaban:** ........................................................

---

## 4. Network, UFW, Firewalld Adaptasi, dan SSH
### Soal 39

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan UFW pada Ubuntu?

A. `apt install -y ufw`
B. `systemctl enable --now ufw.service`
C. `ufw --force enable`
D. `ufw status verbose`

**Jawaban:** ........................................................

---
### Soal 40

**Pertanyaan:** Command UFW mana yang benar untuk default policy baseline?

A. `ufw default deny incoming`
B. `ufw default allow outgoing`
C. `ufw default deny routed`
D. `ufw default allow incoming`

**Jawaban:** ........................................................

---
### Soal 41

**Pertanyaan:** Server dikelola via SSH dari subnet admin `192.168.10.0/24`. Command mana yang benar sebelum enable UFW?

A. `ufw allow from 192.168.10.0/24 to any port 22 proto tcp comment 'Allow SSH from admin subnet'`
B. `ufw limit 22/tcp comment 'Rate limit SSH'`
C. `ufw deny 22/tcp`
D. `ufw allow 22/tcp`

**Jawaban:** ........................................................

---
### Soal 42

**Pertanyaan:** Untuk web server resmi, command mana yang benar untuk membuka HTTP/HTTPS di UFW?

A. `ufw allow 80/tcp comment 'HTTP'`
B. `ufw allow 443/tcp comment 'HTTPS'`
C. `ufw allow 1:65535/tcp`
D. `ufw allow all`

**Jawaban:** ........................................................

---
### Soal 43

**Pertanyaan:** Command audit mana yang benar untuk melihat port terbuka dan status firewall?

A. `ufw status numbered`
B. `ufw status verbose`
C. `ss -tulpen`
D. `netstat -tulpen` jika paket tersedia

**Jawaban:** ........................................................

---
### Soal 44

**Pertanyaan:** Jika soal menyebut Ubuntu STIG resmi memakai UFW, opsi `firewall-cmd` berikut menjadi jawaban...

A. Benar sebagai pengganti resmi UFW STIG
B. Salah untuk jawaban resmi UFW
C. Bisa hanya sebagai adaptasi organisasi jika firewalld dipilih dan didokumentasikan
D. Selalu wajib karena UFW tidak ada di Ubuntu

**Jawaban:** ........................................................

---
### Soal 45

**Pertanyaan:** Jika organisasi memilih firewalld sebagai adaptasi lokal, command mana yang benar secara sintaks untuk enable dan set default zone?

A. `apt install -y firewalld`
B. `systemctl enable --now firewalld`
C. `firewall-cmd --set-default-zone=public`
D. `firewall-cmd --reload`

**Jawaban:** ........................................................

---
### Soal 46

**Pertanyaan:** Firewalld adaptasi: command mana yang benar untuk mengizinkan SSH hanya dari IP admin `192.168.10.20/32`?

A. `firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.10.20/32" port protocol="tcp" port="22" accept'`
B. `firewall-cmd --reload`
C. `firewall-cmd --permanent --zone=public --add-service=all`
D. `firewall-cmd --zone=trusted --add-interface=lo --permanent`

**Jawaban:** ........................................................

---
### Soal 47

**Pertanyaan:** Jika pertanyaan berbunyi: 'Perintah mana yang benar untuk membuka semua service di public zone sesuai STIG?'

A. `firewall-cmd --permanent --zone=public --add-service=all`
B. `firewall-cmd --permanent --zone=public --add-port=1-65535/tcp`
C. `ufw allow 1:65535/tcp`
D. `ufw default allow incoming`

**Jawaban:** ........................................................

---
### Soal 48

**Pertanyaan:** Perintah mana yang benar untuk memasang dan mengaktifkan SSH server pada Ubuntu jika dibutuhkan?

A. `apt install -y ssh`
B. `systemctl enable --now ssh.service`
C. `systemctl is-active ssh.service`
D. `ss -tulpen | grep ':22'`

**Jawaban:** ........................................................

---
### Soal 49

**Pertanyaan:** Jika SSH tidak dibutuhkan pada sistem, tindakan mana yang benar?

A. `systemctl disable --now ssh.service`
B. `apt purge -y openssh-server`
C. `ufw deny 22/tcp`
D. `systemctl enable --now ssh.service`

**Jawaban:** ........................................................

---
### Soal 50

**Pertanyaan:** Command mana yang benar untuk permission file SSH server dan host keys?

A. `chown root:root /etc/ssh/sshd_config`
B. `chmod 600 /etc/ssh/sshd_config`
C. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key' -exec chmod 600 {} \;`
D. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chmod 644 {} \;`

**Jawaban:** ........................................................

---
### Soal 51

**Pertanyaan:** Directive SSH mana yang benar untuk mencegah root login, password kosong, dan metode host trust lama?

A. `PermitRootLogin no`
B. `PermitEmptyPasswords no`
C. `HostbasedAuthentication no`
D. `IgnoreRhosts yes`

**Jawaban:** ........................................................

---
### Soal 52

**Pertanyaan:** Directive SSH mana yang benar untuk session timeout dan pembatasan percobaan autentikasi?

A. `ClientAliveInterval 600`
B. `ClientAliveCountMax 1`
C. `LoginGraceTime 60`
D. `MaxAuthTries 4`

**Jawaban:** ........................................................

---
### Soal 53

**Pertanyaan:** Directive SSH mana yang benar untuk membatasi forwarding dan GUI exposure?

A. `X11Forwarding no`
B. `AllowTcpForwarding no`
C. `AllowAgentForwarding no`
D. `PermitTunnel no`

**Jawaban:** ........................................................

---
### Soal 54

**Pertanyaan:** Directive SSH mana yang benar untuk banner dan logging?

A. `Banner /etc/issue.net`
B. `LogLevel VERBOSE`
C. `UsePAM yes`
D. `Banner /etc/shadow`

**Jawaban:** ........................................................

---
### Soal 55

**Pertanyaan:** Ciphers SSH server mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `aes256-gcm@openssh.com`
B. `aes128-gcm@openssh.com`
C. `aes256-ctr`
D. `3des-cbc`

**Jawaban:** ........................................................

---
### Soal 56

**Pertanyaan:** MACs SSH mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `hmac-sha2-512-etm@openssh.com`
B. `hmac-sha2-256-etm@openssh.com`
C. `hmac-sha2-512`
D. `hmac-md5`

**Jawaban:** ........................................................

---
### Soal 57

**Pertanyaan:** KexAlgorithms mana yang sesuai dengan file implementasi Ubuntu STIG?

A. `ecdh-sha2-nistp521`
B. `diffie-hellman-group-exchange-sha256`
C. `diffie-hellman-group16-sha512`
D. `diffie-hellman-group1-sha1`

**Jawaban:** ........................................................

---
### Soal 58

**Pertanyaan:** Command mana yang benar untuk validasi dan restart SSH setelah mengubah konfigurasi?

A. `sshd -t`
B. `systemctl restart ssh.service`
C. `sshd -T | egrep 'permitrootlogin|ciphers|macs|kexalgorithms'`
D. `rm /etc/ssh/sshd_config`

**Jawaban:** ........................................................

---
### Soal 59

**Pertanyaan:** Command mana yang benar untuk konfigurasi SSH client hardening?

A. `mkdir -p /etc/ssh/ssh_config.d`
B. Buat `/etc/ssh/ssh_config.d/60-stig-hardening.conf` berisi `Host *` dan daftar `Ciphers`/`MACs`
C. `ssh -G localhost | egrep 'ciphers|macs' | head`
D. `echo PermitRootLogin yes >> /etc/ssh/ssh_config`

**Jawaban:** ........................................................

---
### Soal 60

**Pertanyaan:** Command mana yang benar untuk membuat banner login non-informatif?

A. `cat >/etc/issue` berisi `Authorized users only. All activity may be monitored and reported.`
B. `cat >/etc/issue.net` berisi notice serupa
C. `chmod 644 /etc/issue /etc/issue.net /etc/motd`
D. `echo $(uname -a) > /etc/issue.net`

**Jawaban:** ........................................................

---
### Soal 61

**Pertanyaan:** Perintah mana yang benar untuk menonaktifkan wireless/Bluetooth pada server yang tidak membutuhkannya?

A. `systemctl disable --now bluetooth.service 2>/dev/null || true`
B. `apt purge -y bluez 2>/dev/null || true`
C. `rfkill list 2>/dev/null || true` untuk audit
D. `systemctl enable --now bluetooth.service`

**Jawaban:** ........................................................

---

## 5. Identity, PAM, Password Policy, sudo, su, MFA/SSSD, dan Session
### Soal 62

**Pertanyaan:** Paket mana yang benar untuk password quality pada Ubuntu?

A. `apt install -y libpam-pwquality cracklib-runtime`
B. `dpkg -l | grep libpam-pwquality`
C. `apt install -y libpam-modules` untuk modul PAM dasar
D. `apt install -y pam_google_authenticator` sebagai kewajiban STIG Ubuntu

**Jawaban:** ........................................................

---
### Soal 63

**Pertanyaan:** Konfigurasi `/etc/security/pwquality.conf` mana yang benar menurut panduan implementasi?

A. `minlen = 15`
B. `dcredit = -1`
C. `ucredit = -1`
D. `ocredit = -1`

**Jawaban:** ........................................................

---
### Soal 64

**Pertanyaan:** Konfigurasi pwquality mana yang benar untuk membatasi pola password lemah?

A. `lcredit = -1`
B. `maxrepeat = 3`
C. `maxclassrepeat = 4`
D. `dictcheck = 1`

**Jawaban:** ........................................................

---
### Soal 65

**Pertanyaan:** Konfigurasi pwquality mana yang benar untuk enforcement?

A. `enforcing = 1`
B. `retry = 3`
C. `minlen = 4`
D. `dictcheck = 0`

**Jawaban:** ........................................................

---
### Soal 66

**Pertanyaan:** Command mana yang benar untuk mengatur password aging di `/etc/login.defs`?

A. `sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   60/' /etc/login.defs`
B. `sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   1/' /etc/login.defs`
C. `sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   7/' /etc/login.defs`
D. `sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   99999/' /etc/login.defs`

**Jawaban:** ........................................................

---
### Soal 67

**Pertanyaan:** Command mana yang benar untuk menerapkan password aging pada user interaktif yang sudah ada?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1}' /etc/passwd | while read -r user; do chage --maxdays 60 --mindays 1 --warndays 7 "$user"; done`
B. `chage -l <nama_user>` untuk validasi
C. `passwd -d <nama_user>`
D. `chage --maxdays 99999 <nama_user>`

**Jawaban:** ........................................................

---
### Soal 68

**Pertanyaan:** Konfigurasi `/etc/security/faillock.conf` mana yang benar untuk account lockout ketat?

A. `deny = 3`
B. `fail_interval = 900`
C. `unlock_time = 0`
D. `even_deny_root`

**Jawaban:** ........................................................

---
### Soal 69

**Pertanyaan:** Command mana yang benar untuk audit/reset faillock user?

A. `faillock --user <nama_user>`
B. `faillock --user <nama_user> --reset`
C. `rm -rf /etc/security/faillock.conf`
D. `passwd -d <nama_user>`

**Jawaban:** ........................................................

---
### Soal 70

**Pertanyaan:** Baris PAM mana yang benar untuk password history di `/etc/pam.d/common-password` jika diterapkan hati-hati?

A. `password requisite pam_pwhistory.so remember=5 use_authtok`
B. `password requisite pam_pwhistory.so remember=0`
C. `password optional pam_permit.so`
D. `password required pam_deny.so` sebagai satu-satunya baris password

**Jawaban:** ........................................................

---
### Soal 71

**Pertanyaan:** Command mana yang benar untuk mencari password kosong dan nullok?

A. `awk -F: '($2==""){print $1}' /etc/shadow`
B. `grep -R "nullok" /etc/pam.d/ || true`
C. `passwd -l <nama_user>` jika ditemukan akun kosong
D. `passwd -d <nama_user>`

**Jawaban:** ........................................................

---
### Soal 72

**Pertanyaan:** Command mana yang benar untuk konfigurasi sudo hardening?

A. `apt install -y sudo`
B. Buat `/etc/sudoers.d/60-stig-hardening` berisi `Defaults use_pty`
C. Tambahkan `Defaults logfile="/var/log/sudo.log"`
D. `visudo -cf /etc/sudoers.d/60-stig-hardening`

**Jawaban:** ........................................................

---
### Soal 73

**Pertanyaan:** Konfigurasi sudo mana yang benar untuk reauthentication dan timeout?

A. `Defaults timestamp_timeout=5`
B. `Defaults passwd_timeout=1`
C. `Defaults timestamp_timeout=-1`
D. `ALL=(ALL) NOPASSWD:ALL` untuk semua admin

**Jawaban:** ........................................................

---
### Soal 74

**Pertanyaan:** Command mana yang benar untuk membatasi `su` melalui group khusus?

A. `groupadd -f sugroup`
B. Tambahkan `auth required pam_wheel.so use_uid group=sugroup` di `/etc/pam.d/su`
C. `usermod -aG sugroup <nama_admin>`
D. `chmod 777 /bin/su`

**Jawaban:** ........................................................

---
### Soal 75

**Pertanyaan:** Command mana yang benar untuk melihat user dalam sudo group dan menilai apakah sesuai kebutuhan?

A. `getent group sudo`
B. `groups <nama_user>`
C. `id <nama_user>`
D. `chmod -R 777 /etc/sudoers.d`

**Jawaban:** ........................................................

---
### Soal 76

**Pertanyaan:** Jika soal meminta mengaktifkan direct root SSH login sesuai Ubuntu STIG, command mana yang benar?

A. `echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config`
B. `passwd -u root`
C. `systemctl restart ssh.service`
D. `usermod -U root`

**Jawaban:** ........................................................

---
### Soal 77

**Pertanyaan:** Area MFA/SSSD/smart card/PIV/PKI pada Ubuntu STIG sebaiknya diterapkan dengan command apa di lab pribadi tanpa infrastruktur PKI?

A. `apt install -y sssd` lalu mengarang domain palsu
B. `authselect enable-smartcard` di Ubuntu
C. `Catat exception / Not Applicable bila tidak ada infrastruktur organisasi`
D. `Hapus PAM agar MFA tidak diperlukan`

**Jawaban:** ........................................................

---
### Soal 78

**Pertanyaan:** Command mana yang benar untuk audit keberadaan SSSD bila organisasi memakai identity provider?

A. `systemctl status sssd.service`
B. `dpkg -l | grep sssd`
C. `grep -R '^services\|^domains' /etc/sssd/sssd.conf 2>/dev/null`
D. `chmod 777 /etc/sssd/sssd.conf`

**Jawaban:** ........................................................

---
### Soal 79

**Pertanyaan:** Command mana yang benar untuk session/login banner GDM jika workstation memakai GNOME?

A. `mkdir -p /etc/dconf/db/gdm.d`
B. Buat `/etc/dconf/db/gdm.d/01-banner-message` dengan `banner-message-enable=true`
C. `dconf update`
D. `ufw allow gdm`

**Jawaban:** ........................................................

---
### Soal 80

**Pertanyaan:** Konfigurasi GDM mana yang benar untuk mencegah user enumeration?

A. `[org/gnome/login-screen]` lalu `disable-user-list=true`
B. `dconf update`
C. `disable-user-list=false`
D. `cat /etc/passwd > /etc/issue`

**Jawaban:** ........................................................

---
### Soal 81

**Pertanyaan:** Konfigurasi dconf mana yang benar untuk lock screen dan automount workstation?

A. `idle-delay=uint32 900`
B. `lock-enabled=true`
C. `lock-delay=uint32 0`
D. `automount=false` dan `autorun-never=true`

**Jawaban:** ........................................................

---
### Soal 82

**Pertanyaan:** Command mana yang benar untuk menonaktifkan Ctrl-Alt-Delete reboot pada sistem kritis?

A. `systemctl mask ctrl-alt-del.target`
B. `systemctl daemon-reload`
C. `systemctl unmask ctrl-alt-del.target`
D. `systemctl start ctrl-alt-del.target`

**Jawaban:** ........................................................

---

## 6. Logging, rsyslog, auditd, Audit Rules, AIDE, dan Permission
### Soal 83

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan rsyslog?

A. `apt install -y rsyslog`
B. `systemctl enable --now rsyslog.service`
C. `systemctl is-active rsyslog.service`
D. `systemctl disable --now rsyslog.service`

**Jawaban:** ........................................................

---
### Soal 84

**Pertanyaan:** Konfigurasi rsyslog mana yang benar untuk mode pembuatan file log?

A. `$FileCreateMode 0640`
B. `systemctl restart rsyslog.service`
C. `grep -R "FileCreateMode" /etc/rsyslog.conf /etc/rsyslog.d/*.conf`
D. `$FileCreateMode 0777`

**Jawaban:** ........................................................

---
### Soal 85

**Pertanyaan:** Konfigurasi forwarding rsyslog mana yang benar jika organisasi punya server log `192.168.10.50:6514`?

A. `*.* @@192.168.10.50:6514`
B. `systemctl restart rsyslog.service`
C. `*.* @192.168.10.50:514` sebagai TLS otomatis
D. `@@` berarti TCP

**Jawaban:** ........................................................

---
### Soal 86

**Pertanyaan:** Command mana yang benar untuk audit permission log yang terlalu permisif?

A. `find /var/log -type f -perm /137 -ls`
B. `find /var/log -type f -exec chmod go-wx {} \;` sebagai perbaikan konservatif
C. `find /var/log -type d -exec chmod go-w {} \;`
D. `chmod -R 777 /var/log`

**Jawaban:** ........................................................

---
### Soal 87

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan auditd?

A. `apt install -y auditd audispd-plugins`
B. `systemctl enable --now auditd.service`
C. `auditctl -s`
D. `systemctl disable --now auditd.service`

**Jawaban:** ........................................................

---
### Soal 88

**Pertanyaan:** Parameter boot mana yang benar agar audit aktif sejak boot awal?

A. `audit=1`
B. `audit_backlog_limit=8192`
C. `audit=0`
D. `noaudit`

**Jawaban:** ........................................................

---
### Soal 89

**Pertanyaan:** Command mana yang benar untuk memasukkan parameter audit ke GRUB dan memperbarui konfigurasi?

A. Edit `/etc/default/grub` agar `GRUB_CMDLINE_LINUX` memuat `audit=1 audit_backlog_limit=8192`
B. `update-grub`
C. `cat /proc/cmdline | grep -E 'audit=1|audit_backlog_limit=8192'` setelah reboot
D. `grub-mkconfig -o /boot/grub/grub.cfg` sebagai command standar Ubuntu wajib

**Jawaban:** ........................................................

---
### Soal 90

**Pertanyaan:** Konfigurasi `/etc/audit/auditd.conf` mana yang benar untuk retention ketat?

A. `max_log_file = 100`
B. `max_log_file_action = keep_logs`
C. `disk_full_action = halt`
D. `disk_error_action = halt`

**Jawaban:** ........................................................

---
### Soal 91

**Pertanyaan:** Konfigurasi auditd mana yang berisiko menghapus bukti audit dan tidak sesuai baseline?

A. `max_log_file_action = ignore`
B. `max_log_file_action = rotate` tanpa retention jelas
C. `disk_full_action = ignore`
D. `max_log_file_action = keep_logs`

**Jawaban:** ........................................................

---
### Soal 92

**Pertanyaan:** Command mana yang benar untuk remote audit offloading jika ada audit server?

A. `sed -i -E 's/^active\s*=.*/active = yes/' /etc/audit/plugins.d/au-remote.conf`
B. `sed -i -E 's/^remote_server\s*=.*/remote_server = 192.168.10.60/' /etc/audit/audisp-remote.conf`
C. `systemctl restart auditd.service`
D. `rm -rf /var/log/audit`

**Jawaban:** ........................................................

---
### Soal 93

**Pertanyaan:** Rule audit mana yang benar untuk perubahan file identitas user/group?

A. `-w /etc/passwd -p wa -k identity`
B. `-w /etc/group -p wa -k identity`
C. `-w /etc/shadow -p wa -k identity`
D. `-w /etc/gshadow -p wa -k identity`

**Jawaban:** ........................................................

---
### Soal 94

**Pertanyaan:** Rule audit mana yang benar untuk `/etc/security/opasswd`?

A. `-w /etc/security/opasswd -p wa -k identity`
B. `-w /etc/security/opasswd -p x -k identity`
C. `-w /etc/security/opasswd -p r -k identity` sebagai satu-satunya rule
D. `chmod 777 /etc/security/opasswd`

**Jawaban:** ........................................................

---
### Soal 95

**Pertanyaan:** Rule audit mana yang benar untuk perubahan sudoers dan sudo log?

A. `-w /etc/sudoers -p wa -k scope`
B. `-w /etc/sudoers.d/ -p wa -k scope`
C. `-w /var/log/sudo.log -p wa -k actions`
D. `-w /etc/sudoers -p x -k scope`

**Jawaban:** ........................................................

---
### Soal 96

**Pertanyaan:** Rule audit mana yang benar untuk perubahan waktu?

A. `-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change`
B. `-a always,exit -F arch=b32 -S adjtimex,settimeofday,clock_settime -k time-change`
C. `-w /etc/localtime -p wa -k time-change`
D. `-a never,exit -S clock_settime -k time-change`

**Jawaban:** ........................................................

---
### Soal 97

**Pertanyaan:** Rule audit mana yang benar untuk perubahan hostname/network identity?

A. `-a always,exit -F arch=b64 -S sethostname,setdomainname -k system-locale`
B. `-a always,exit -F arch=b32 -S sethostname,setdomainname -k system-locale`
C. `-w /etc/hosts -p wa -k system-locale`
D. `-w /etc/netplan/ -p wa -k system-locale`

**Jawaban:** ........................................................

---
### Soal 98

**Pertanyaan:** Rule audit mana yang benar untuk perubahan permission/ownership?

A. `-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -k perm_mod`
B. `-a always,exit -F arch=b64 -S chown,fchown,lchown,fchownat -F auid>=1000 -F auid!=unset -k perm_mod`
C. `-a always,exit -F arch=b64 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod`
D. `-a never,exit -S chmod`

**Jawaban:** ........................................................

---
### Soal 99

**Pertanyaan:** Rule audit mana yang benar untuk file deletion events?

A. `-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=1000 -F auid!=unset -k delete`
B. `-a always,exit -F arch=b32 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=1000 -F auid!=unset -k delete`
C. `-a never,exit -S unlink`
D. `rm -rf /var/log/audit`

**Jawaban:** ........................................................

---
### Soal 100

**Pertanyaan:** Rule audit mana yang benar untuk mount events?

A. `-a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=unset -k mounts`
B. `-a always,exit -F arch=b32 -S mount -F auid>=1000 -F auid!=unset -k mounts`
C. `-a never,exit -S mount`
D. `mount -a` sebagai audit rule

**Jawaban:** ........................................................

---
### Soal 101

**Pertanyaan:** Rule audit mana yang benar untuk login/session record?

A. `-w /var/log/faillog -p wa -k logins`
B. `-w /var/log/lastlog -p wa -k logins`
C. `-w /var/run/utmp -p wa -k session`
D. `-w /var/log/wtmp -p wa -k session`

**Jawaban:** ........................................................

---
### Soal 102

**Pertanyaan:** Rule audit mana yang benar untuk kernel module changes?

A. `-w /sbin/insmod -p x -k modules`
B. `-w /sbin/rmmod -p x -k modules`
C. `-w /sbin/modprobe -p x -k modules`
D. `-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k modules`

**Jawaban:** ........................................................

---
### Soal 103

**Pertanyaan:** Command mana yang benar untuk membuat audit rule privileged commands secara dinamis?

A. `find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null`
B. `printf -- "-a always,exit -F path=%s -F perm=x -F auid>=%s -F auid!=unset -k privileged\n" "$file" "$UID_MIN"`
C. `awk '/^\s*UID_MIN/{print $2}' /etc/login.defs`
D. `chmod -R u+s /usr/bin`

**Jawaban:** ........................................................

---
### Soal 104

**Pertanyaan:** Command mana yang benar untuk load dan validasi audit rules?

A. `augenrules --load`
B. `auditctl -l | head -50`
C. `auditctl -s`
D. `auditctl -D` sebagai final hardening

**Jawaban:** ........................................................

---
### Soal 105

**Pertanyaan:** Command mana yang benar untuk membuat audit immutable setelah rules stabil?

A. `echo '-e 2' >/etc/audit/rules.d/99-finalize.rules`
B. `augenrules --load`
C. `grep -E '^-e 2' /etc/audit/audit.rules`
D. `auditctl -e 0`

**Jawaban:** ........................................................

---
### Soal 106

**Pertanyaan:** Permission mana yang benar untuk file audit dan audit tools?

A. `chown root:root /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true`
B. `chmod 640 /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true`
C. `chmod 750 /sbin/auditctl /sbin/auditd /sbin/ausearch /sbin/aureport /sbin/autrace /sbin/augenrules 2>/dev/null || true`
D. `chmod 777 /etc/audit/auditd.conf`

**Jawaban:** ........................................................

---
### Soal 107

**Pertanyaan:** Permission mana yang benar untuk `/var/log/audit`?

A. `chown root:adm /var/log/audit 2>/dev/null || true`
B. `chmod 750 /var/log/audit 2>/dev/null || true`
C. `find /var/log/audit -type f -exec chown root:adm {} \; -exec chmod 640 {} \; 2>/dev/null || true`
D. `chmod -R 777 /var/log/audit`

**Jawaban:** ........................................................

---
### Soal 108

**Pertanyaan:** Command mana yang benar untuk permission file account penting?

A. `chown root:root /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true`
B. `chmod 644 /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true`
C. `chown root:shadow /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true`
D. `chmod 640 /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true`

**Jawaban:** ........................................................

---
### Soal 109

**Pertanyaan:** Command mana yang benar untuk audit permission file penting?

A. `stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow /etc/shells`
B. `ls -l /etc/passwd /etc/shadow`
C. `chmod 777 /etc/shadow`
D. `chown user:user /etc/passwd`

**Jawaban:** ........................................................

---
### Soal 110

**Pertanyaan:** Command mana yang benar untuk mencari dan memperbaiki file world-writable?

A. `find / -xdev -type f -perm -0002 -print 2>/dev/null`
B. `find / -xdev -type f -perm -0002 -exec chmod o-w {} \; 2>/dev/null`
C. `find / -xdev -type d -perm -0002 ! -perm -1000 -print 2>/dev/null`
D. `find / -xdev -type d -perm -0002 ! -perm -1000 -exec chmod a+t {} \; 2>/dev/null`

**Jawaban:** ........................................................

---
### Soal 111

**Pertanyaan:** Command mana yang benar untuk mencari file tanpa owner/group?

A. `find / -xdev \( -nouser -o -nogroup \) -print 2>/dev/null`
B. `find / -nouser -print 2>/dev/null`
C. `find / -nogroup -print 2>/dev/null`
D. `chown -R root:root /`

**Jawaban:** ........................................................

---
### Soal 112

**Pertanyaan:** Command mana yang benar untuk review SUID/SGID?

A. `find / -xdev \( -perm -4000 -o -perm -2000 \) -type f -print 2>/dev/null | sort`
B. `find / -xdev -type f -perm -4000 -print 2>/dev/null`
C. `find / -xdev -type f -perm -2000 -print 2>/dev/null`
D. `chmod -R u+s /usr/bin`

**Jawaban:** ........................................................

---
### Soal 113

**Pertanyaan:** Command mana yang benar untuk audit shadowed password dan empty password?

A. `awk -F: '($2 != "x") {print $1 ": password field is not x"}' /etc/passwd`
B. `awk -F: '($2 == "") {print $1}' /etc/shadow`
C. `passwd -l <nama_user>` jika ditemukan akun kosong
D. `passwd -d <nama_user>`

**Jawaban:** ........................................................

---
### Soal 114

**Pertanyaan:** Command mana yang benar untuk audit duplicate UID/GID/user/group?

A. `cut -d: -f3 /etc/passwd | sort | uniq -d`
B. `cut -d: -f3 /etc/group | sort | uniq -d`
C. `cut -d: -f1 /etc/passwd | sort | uniq -d`
D. `cut -d: -f1 /etc/group | sort | uniq -d`

**Jawaban:** ........................................................

---
### Soal 115

**Pertanyaan:** Command mana yang benar untuk audit home directory user interaktif?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1,$6}' /etc/passwd`
B. `[ -d "$home" ] || echo "Missing home: $user $home"` dalam loop
C. `[ -d "$home" ] && chmod go-w "$home"` sebagai perbaikan permission umum
D. `chmod -R 777 /home`

**Jawaban:** ........................................................

---
### Soal 116

**Pertanyaan:** Command mana yang benar untuk mencari dotfile berisiko?

A. `find /home -name '.rhosts' -o -name '.netrc' -o -name '.forward' 2>/dev/null`
B. `find /home -name '.netrc' -exec ls -l {} \; 2>/dev/null`
C. `find /home -name '.bashrc' -delete`
D. `rm -rf /home/*`

**Jawaban:** ........................................................

---
### Soal 117

**Pertanyaan:** Pilih command yang benar untuk memperbaiki semua permission secara cepat tanpa audit dan tanpa review.

A. `chmod -R 600 /etc`
B. `chown -R root:root /home`
C. `find / -perm /6000 -delete`
D. `chmod -R 777 /var/log`

**Jawaban:** ........................................................

---
### Soal 118

**Pertanyaan:** Command mana yang benar untuk checklist validasi akhir pasca-reboot?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd`
B. `ufw status verbose`
C. `sshd -t`
D. `aa-status`

**Jawaban:** ........................................................

---
### Soal 119

**Pertanyaan:** Command mana yang benar untuk validasi audit dan AIDE akhir?

A. `auditctl -s`
B. `auditctl -l | head`
C. `aide -c /etc/aide/aide.conf --check`
D. `rm -rf /var/lib/aide`

**Jawaban:** ........................................................

---
### Soal 120

**Pertanyaan:** Command mana yang benar untuk validasi password policy akhir?

A. `grep -v '^#' /etc/security/pwquality.conf | sed '/^$/d'`
B. `grep -E '^PASS_MAX_DAYS|^PASS_MIN_DAYS|^PASS_WARN_AGE' /etc/login.defs`
C. `faillock --user <nama_user>`
D. `passwd -d <nama_user>` sebagai validasi

**Jawaban:** ........................................................

---

## 7. Pengenalan STIG, Severity, Exception, dan Prinsip Audit
### Soal 121

**Pertanyaan:** Dalam konteks Ubuntu STIG, perintah atau bukti mana yang benar untuk menunjukkan sistem memakai rilis vendor-supported?

A. `cat /etc/os-release`
B. `grep DISTRIB_DESCRIPTION /etc/lsb-release`
C. `ubuntu-security-status 2>/dev/null || true`
D. `uname -a` saja cukup untuk membuktikan vendor support

**Jawaban:** ........................................................

---
### Soal 122

**Pertanyaan:** Jika suatu rule STIG seperti FIPS, PIV/PKI, atau remote audit server tidak bisa diterapkan di lab kampus, tindakan mana yang benar?

A. Catat sebagai exception/N/A dengan alasan
B. Jelaskan risiko dan mitigasi pengganti
C. Abaikan tanpa catatan
D. Palsukan output command agar lulus

**Jawaban:** ........................................................

---
### Soal 123

**Pertanyaan:** Data apa saja yang tepat masuk exception register?

A. Rule/Area
B. Alasan
C. Risiko
D. Mitigasi dan tanggal review

**Jawaban:** ........................................................

---
### Soal 124

**Pertanyaan:** Perintah mana yang benar untuk mengaudit layanan aktif yang perlu dibandingkan dengan daftar port/protocol/service yang disetujui?

A. `ss -tulpen`
B. `systemctl --type=service --state=running`
C. `ufw status verbose`
D. `lsof -i -P -n` jika tersedia

**Jawaban:** ........................................................

---
### Soal 125

**Pertanyaan:** Jika sistem adalah server tanpa GUI, command mana yang benar untuk area GDM/GNOME?

A. Tandai rule GUI sebagai N/A bila paket GUI memang tidak ada
B. `dpkg -l | grep -E 'gdm3|ubuntu-desktop|gnome-shell' || true`
C. `systemctl status gdm3.service 2>/dev/null || true`
D. `apt install -y ubuntu-desktop` agar rule GUI bisa diterapkan

**Jawaban:** ........................................................

---
### Soal 126

**Pertanyaan:** Jika setelah hardening SSH admin terkunci dari server remote, apa pencegahan yang benar sebelum remediation?

A. Siapkan console/recovery access
B. Validasi `sshd -t` sebelum restart
C. Buat rule firewall SSH sebelum `ufw enable`
D. Restart SSH tanpa sesi cadangan dan tanpa akses console

**Jawaban:** ........................................................

---
### Soal 127

**Pertanyaan:** Untuk pertanyaan command yang mencampur Debian/Arch dan Ubuntu, opsi mana yang harus dipilih pada Ubuntu?

A. `apt install -y ufw`
B. `systemctl enable --now ufw.service`
C. `pacstrap -K /mnt ufw`
D. `pacman -S ufw`

**Jawaban:** ........................................................

---
### Soal 128

**Pertanyaan:** Pilih command yang benar untuk membuat Ubuntu STIG memakai `pacman` sebagai package manager resmi.

A. `pacman -Syu`
B. `pacman-key --init`
C. `pacstrap -K /mnt base`
D. `makepkg -si`

**Jawaban:** ........................................................

---

## 8. Tambahan Praktik Detail per Modul Ubuntu STIG
### Soal 129

**Pertanyaan:** Perintah mana yang benar untuk memeriksa apakah service penting aktif setelah hardening?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd`
B. `systemctl is-enabled ssh ufw apparmor chrony rsyslog auditd`
C. `systemctl status ssh ufw apparmor chrony rsyslog auditd`
D. `service --status-all` sebagai satu-satunya bukti compliance

**Jawaban:** ........................................................

---
### Soal 130

**Pertanyaan:** Command mana yang benar untuk memastikan hanya port yang disetujui yang listening?

A. `ss -tulpen`
B. `ufw status verbose`
C. `lsof -i -P -n` jika tersedia
D. `nmap localhost` jika tool tersedia dan diizinkan

**Jawaban:** ........................................................

---
### Soal 131

**Pertanyaan:** Jika UFW sudah aktif dan admin ingin melihat rule bernomor untuk review, command mana yang benar?

A. `ufw status numbered`
B. `ufw status verbose`
C. `ufw delete <nomor_rule>` untuk menghapus rule yang tidak disetujui setelah review
D. `ufw allow all`

**Jawaban:** ........................................................

---
### Soal 132

**Pertanyaan:** Command mana yang benar untuk menambahkan rule UFW HTTPS hanya jika host memang web server?

A. `ufw allow 443/tcp comment 'HTTPS'`
B. `ufw allow 80/tcp comment 'HTTP'` bila HTTP memang dibutuhkan
C. `ufw allow 443/udp` sebagai pengganti HTTPS standar TCP
D. `ufw allow 1:65535/tcp`

**Jawaban:** ........................................................

---
### Soal 133

**Pertanyaan:** Command mana yang benar untuk audit konfigurasi SSH effective setelah drop-in diterapkan?

A. `sshd -T | grep permitrootlogin`
B. `sshd -T | grep clientaliveinterval`
C. `sshd -T | grep x11forwarding`
D. `cat /etc/ssh/sshd_config.d/60-stig-hardening.conf`

**Jawaban:** ........................................................

---
### Soal 134

**Pertanyaan:** Jika `sshd -t` gagal setelah mengubah cipher/MAC/KEX, tindakan mana yang benar?

A. Jangan restart SSH dulu
B. Perbaiki file drop-in yang error
C. Gunakan sesi console/cadangan untuk rollback
D. Tetap jalankan `systemctl restart ssh.service` agar cepat tahu error

**Jawaban:** ........................................................

---
### Soal 135

**Pertanyaan:** Command mana yang benar untuk memeriksa SSH client tidak memakai algoritma yang tidak sesuai baseline?

A. `ssh -G localhost | egrep 'ciphers|macs' | head`
B. `grep -R '^\s*Ciphers\|^\s*MACs' /etc/ssh/ssh_config /etc/ssh/ssh_config.d/ 2>/dev/null`
C. `ssh -Q cipher` untuk melihat algoritma tersedia, bukan konfigurasi aktif
D. `echo 'Ciphers 3des-cbc' >> /etc/ssh/ssh_config`

**Jawaban:** ........................................................

---
### Soal 136

**Pertanyaan:** Command mana yang benar untuk menghapus paket Bluetooth setelah memastikan server tidak membutuhkannya?

A. `systemctl disable --now bluetooth.service 2>/dev/null || true`
B. `apt purge -y bluez 2>/dev/null || true`
C. `apt autoremove -y --purge`
D. `rfkill unblock bluetooth`

**Jawaban:** ........................................................

---
### Soal 137

**Pertanyaan:** Perintah mana yang benar untuk memverifikasi AppArmor profile dan status enforcing/complain?

A. `aa-status`
B. `systemctl is-active apparmor.service`
C. `journalctl -u apparmor.service --no-pager | tail`
D. `getenforce` sebagai bukti utama AppArmor

**Jawaban:** ........................................................

---
### Soal 138

**Pertanyaan:** Command mana yang benar untuk validasi kernel hardening yang sudah diterapkan?

A. `sysctl net.ipv4.tcp_syncookies`
B. `sysctl kernel.randomize_va_space`
C. `sysctl kernel.kptr_restrict`
D. `sysctl fs.protected_hardlinks fs.protected_symlinks`

**Jawaban:** ........................................................

---
### Soal 139

**Pertanyaan:** Command mana yang benar untuk konfigurasi rsyslog agar tidak menjadi server penerima log remote jika tidak dibutuhkan?

A. Review `/etc/rsyslog.conf` dan `/etc/rsyslog.d/*.conf` dari modul input TCP/UDP
B. `grep -R 'imudp\|imtcp\|InputTCPServerRun\|UDPServerRun' /etc/rsyslog.conf /etc/rsyslog.d/ 2>/dev/null`
C. Disable konfigurasi listener yang tidak sah lalu restart rsyslog
D. `ufw allow 514/udp` pada semua host

**Jawaban:** ........................................................

---
### Soal 140

**Pertanyaan:** Command mana yang benar untuk melihat journal tanpa membuka akses luas ke user biasa?

A. `journalctl -xe` sebagai root/sudo
B. `journalctl -u ssh.service`
C. `ls -ld /var/log/journal /run/log/journal 2>/dev/null`
D. `chmod -R 777 /var/log/journal`

**Jawaban:** ........................................................

---
### Soal 141

**Pertanyaan:** Rule audit mana yang benar untuk AppArmor/MAC tools jika ingin mencatat perubahan MAC policy?

A. `-w /etc/apparmor/ -p wa -k MAC-policy`
B. `-w /etc/apparmor.d/ -p wa -k MAC-policy`
C. `-a always,exit -F path=/usr/bin/aa-enforce -F perm=x -k MAC-policy` jika path tersedia dan digunakan
D. `auditctl -D` sebagai MAC protection

**Jawaban:** ........................................................

---
### Soal 142

**Pertanyaan:** Command mana yang benar untuk mencatat perubahan netplan dalam audit rules?

A. `-w /etc/netplan/ -p wa -k system-locale`
B. `-w /etc/hosts -p wa -k system-locale`
C. `-w /etc/hostname -p wa -k system-locale`
D. `-w /etc/netplan/ -p x -k execute-netplan` sebagai satu-satunya rule

**Jawaban:** ........................................................

---
### Soal 143

**Pertanyaan:** Command mana yang benar untuk membuat file sudoers drop-in aman setelah ditulis?

A. `chmod 440 /etc/sudoers.d/60-stig-hardening`
B. `chown root:root /etc/sudoers.d/60-stig-hardening`
C. `visudo -cf /etc/sudoers.d/60-stig-hardening`
D. `chmod 777 /etc/sudoers.d/60-stig-hardening`

**Jawaban:** ........................................................

---
### Soal 144

**Pertanyaan:** Command mana yang benar untuk audit user interaktif yang mungkin memiliki shell login valid?

A. `awk -F: '($3>=1000)&&($1!="nobody"){print $1,$7}' /etc/passwd`
B. `getent passwd`
C. `grep -E '/bin/(bash|sh|zsh)$' /etc/passwd`
D. `cat /etc/shadow` untuk semua peserta ujian

**Jawaban:** ........................................................

---
### Soal 145

**Pertanyaan:** Command mana yang benar untuk mengunci akun yang tidak boleh login?

A. `passwd -l <nama_user>`
B. `usermod -L <nama_user>`
C. `usermod -s /usr/sbin/nologin <nama_user>`
D. `passwd -d <nama_user>`

**Jawaban:** ........................................................

---
### Soal 146

**Pertanyaan:** Command mana yang benar untuk memeriksa group `shadow` kosong atau tidak sebelum remediation?

A. `getent group shadow`
B. `grep '^shadow:' /etc/group`
C. `awk -F: '$1=="shadow"{print $4}' /etc/group`
D. `groupdel shadow` sebagai langkah audit

**Jawaban:** ........................................................

---
### Soal 147

**Pertanyaan:** Command mana yang benar untuk audit last successful login display/session records?

A. `lastlog`
B. `last`
C. `lastb` sebagai root/sudo jika tersedia
D. `rm -f /var/log/wtmp /var/log/btmp`

**Jawaban:** ........................................................

---
### Soal 148

**Pertanyaan:** Command mana yang benar untuk melakukan reboot setelah update kernel dan validasi ulang?

A. `reboot`
B. `uname -r` setelah boot ulang
C. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd` setelah boot ulang
D. `poweroff` tanpa rencana recovery pada server remote

**Jawaban:** ........................................................

---
### Soal 149

**Pertanyaan:** Jika semua opsi dalam soal adalah command destruktif seperti menghapus `/etc`, `/var/log`, atau audit rules, jawaban yang benar adalah?

A. Pilih command paling cepat
B. PASS / kosongkan jawaban
C. Pilih command yang memakai `-rf` karena praktis
D. Pilih semua opsi

**Jawaban:** ........................................................

---
### Soal 150

**Pertanyaan:** Command mana yang benar untuk membuat laporan ringkas hasil validasi akhir ke file teks?

A. `systemctl is-active ssh ufw apparmor chrony rsyslog auditd > hardening-validation.txt`
B. `ufw status verbose >> hardening-validation.txt`
C. `auditctl -s >> hardening-validation.txt`
D. `cat /etc/shadow >> hardening-validation.txt`

**Jawaban:** ........................................................

---
