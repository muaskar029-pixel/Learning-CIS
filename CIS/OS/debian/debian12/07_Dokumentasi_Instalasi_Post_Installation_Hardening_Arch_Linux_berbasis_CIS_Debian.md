# Dokumentasi Instalasi hingga Post-Installation Hardening Arch Linux Berbasis Prinsip CIS Debian Linux 12 Benchmark

**Nama dokumen:** Dokumentasi Instalasi dan Post-Installation Hardening Arch Linux  
**Basis hardening:** Adaptasi prinsip CIS Debian Linux 12 Benchmark v2.0.0 ke Arch Linux  
**Target sistem:** Arch Linux UEFI, GPT, server/workstation minimal  
**Status:** Panduan implementasi teknis  
**Catatan penting:** Ini bukan benchmark resmi CIS untuk Arch Linux. Panduan ini adalah adaptasi konseptual dan teknis dari rekomendasi CIS Debian Linux 12 ke ekosistem Arch Linux.

---

## 0. Prinsip Dasar

CIS Debian Linux 12 Benchmark disusun untuk Debian 12, bukan Arch Linux. Karena itu, beberapa hal seperti `apt`, struktur paket Debian, `pam-auth-update`, dan nama service Debian tidak dapat disalin mentah-mentah ke Arch. Yang dipakai dalam dokumen ini adalah **prinsip hardening-nya**, lalu diterjemahkan ke mekanisme Arch Linux.

Prinsip yang diadaptasi:

1. Gunakan sistem minimal.
2. Pisahkan filesystem penting.
3. Batasi filesystem sementara dengan `nodev`, `nosuid`, dan `noexec`.
4. Kurangi kernel module, service, dan software yang tidak diperlukan.
5. Aktifkan Mandatory Access Control dengan AppArmor.
6. Lindungi bootloader.
7. Keraskan parameter kernel dan jaringan.
8. Amankan SSH, privilege escalation, PAM, user, dan group.
9. Aktifkan logging, auditd, firewall, dan integrity checking.
10. Dokumentasikan exception jika ada rekomendasi yang tidak bisa diterapkan.

---

## 1. Ruang Lingkup dan Asumsi

Panduan ini menggunakan skenario berikut:

- Mode boot: UEFI.
- Skema partisi: GPT.
- Bootloader: GRUB.
- Root filesystem: LUKS2 + LVM.
- Kernel utama: `linux-hardened`.
- Kernel cadangan: `linux-lts`.
- Initramfs: `mkinitcpio`.
- Network manager: `NetworkManager`.
- Firewall lokal: UFW.
- Time synchronization: chrony.
- Mandatory Access Control: AppArmor.
- Auditing: auditd.
- Integrity checking: AIDE.
- Shell perintah: Bash.

Panduan ini mengasumsikan perintah dijalankan sebagai `root` pada environment instalasi Arch Linux.

> **Peringatan:** Bagian partisi dan format disk akan menghapus data. Pastikan nama disk benar sebelum menjalankan perintah.

---

## 2. Mapping Singkat CIS Debian ke Arch Linux

| Area CIS Debian | Adaptasi di Arch Linux |
|---|---|
| Package management APT | `pacman`, `pacman-key`, `/etc/pacman.conf`, mirror resmi |
| AppArmor | Paket `apparmor`, kernel parameter `lsm=...apparmor...`, service `apparmor.service` |
| Bootloader password | GRUB password dengan `grub-mkpasswd-pbkdf2` |
| Filesystem partitioning | GPT + LUKS2 + LVM + mount option `nodev,nosuid,noexec` |
| UFW | Paket `ufw`, default deny incoming, allow outgoing |
| SSH hardening | `/etc/ssh/sshd_config.d/99-cis-hardening.conf` |
| PAM | Edit manual file PAM Arch seperti `/etc/pam.d/system-auth` dan konfigurasi `/etc/security/*` |
| journald/rsyslog | `systemd-journald`, `rsyslog`, dan `logrotate` |
| auditd | Paket `audit`, service `auditd.service`, rule di `/etc/audit/rules.d/` |
| AIDE | Paket `aide`, database integrity checking |

---

## 3. Pre-Installation Planning

Sebelum instalasi, lakukan perencanaan berikut.

### 3.1 Tentukan Profil Sistem

Tentukan apakah sistem ini akan menjadi:

- **Server Level 1 style:** aman, praktis, tetap usable.
- **Server Level 2 style:** lebih ketat, cocok untuk sistem kritikal, tetapi bisa mengurangi kenyamanan.
- **Workstation Level 1 style:** aman untuk desktop harian.
- **Workstation Level 2 style:** lebih ketat, cocok untuk lingkungan sensitif.

Untuk dokumentasi ini, baseline yang digunakan adalah **Level 1 Server style** dengan beberapa tambahan defense-in-depth.

### 3.2 Tentukan Exception

Contoh exception yang perlu didokumentasikan:

| Rekomendasi | Exception | Alasan |
|---|---|---|
| Disable web server | Tidak diterapkan | Mesin memang digunakan sebagai web server |
| Disable `overlay` module | Tidak diterapkan | Docker/container membutuhkan overlay filesystem |
| Disable `usb-storage` | Tidak diterapkan | Admin perlu backup offline via USB |
| Disable IPv6 | Tidak diterapkan | Infrastruktur memakai IPv6 |
| Disable GUI | Tidak diterapkan | Mesin dipakai sebagai workstation |

Format dokumentasi exception:

```text
Tanggal:
Rekomendasi:
Status exception:
Alasan bisnis/teknis:
Risiko:
Mitigasi pengganti:
Penanggung jawab:
Tanggal review ulang:
```

---

## 4. Boot ke Arch ISO

### 4.1 Verifikasi Mode Boot

```bash
ls /sys/firmware/efi/efivars
```

Jika direktori tersedia, sistem sedang boot dalam mode UEFI.

### 4.2 Sinkronisasi Waktu Live ISO

```bash
timedatectl set-ntp true
timedatectl status
```

### 4.3 Cek Disk

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

Contoh panduan ini memakai disk:

```text
/dev/nvme0n1
```

Ganti sesuai disk milikmu.

---

## 5. Skema Partisi Hardening

### 5.1 Rancangan Partisi

| Partisi | Ukuran Contoh | Fungsi | Keterangan |
|---|---:|---|---|
| `/dev/nvme0n1p1` | 512 MiB | EFI System Partition | FAT32 |
| `/dev/nvme0n1p2` | 1 GiB | `/boot` | ext4, tidak dienkripsi |
| `/dev/nvme0n1p3` | Sisa disk | LUKS2 container | Berisi LVM |

Di dalam LUKS dibuat LVM:

| Logical Volume | Mount point | Ukuran Contoh | Mount option |
|---|---|---:|---|
| `lv_root` | `/` | 40G | defaults |
| `lv_var` | `/var` | 20G | nodev,nosuid |
| `lv_var_tmp` | `/var/tmp` | 10G | nodev,nosuid,noexec |
| `lv_log` | `/var/log` | 10G | nodev,nosuid,noexec |
| `lv_audit` | `/var/log/audit` | 5G | nodev,nosuid,noexec |
| `lv_home` | `/home` | sisa | nodev,nosuid |

`/tmp` dan `/dev/shm` akan memakai `tmpfs` dengan `nodev,nosuid,noexec`.

### 5.2 Buat Partisi

```bash
sgdisk --zap-all /dev/nvme0n1
sgdisk -n 1:0:+512M -t 1:ef00 -c 1:"EFI System" /dev/nvme0n1
sgdisk -n 2:0:+1G    -t 2:8300 -c 2:"Linux boot" /dev/nvme0n1
sgdisk -n 3:0:0      -t 3:8309 -c 3:"Linux LUKS" /dev/nvme0n1
partprobe /dev/nvme0n1
lsblk
```

### 5.3 Format EFI dan `/boot`

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 -L BOOT /dev/nvme0n1p2
```

### 5.4 Buat LUKS2

```bash
cryptsetup luksFormat --type luks2 /dev/nvme0n1p3
cryptsetup open /dev/nvme0n1p3 cryptlvm
```

### 5.5 Buat LVM

```bash
pvcreate /dev/mapper/cryptlvm
vgcreate vgarch /dev/mapper/cryptlvm

lvcreate -L 40G -n lv_root vgarch
lvcreate -L 20G -n lv_var vgarch
lvcreate -L 10G -n lv_var_tmp vgarch
lvcreate -L 10G -n lv_log vgarch
lvcreate -L 5G  -n lv_audit vgarch
lvcreate -l 100%FREE -n lv_home vgarch
```

### 5.6 Format Logical Volume

```bash
mkfs.ext4 -L ROOT /dev/vgarch/lv_root
mkfs.ext4 -L VAR /dev/vgarch/lv_var
mkfs.ext4 -L VAR_TMP /dev/vgarch/lv_var_tmp
mkfs.ext4 -L LOG /dev/vgarch/lv_log
mkfs.ext4 -L AUDIT /dev/vgarch/lv_audit
mkfs.ext4 -L HOME /dev/vgarch/lv_home
```

### 5.7 Mount Filesystem

```bash
mount /dev/vgarch/lv_root /mnt

mkdir -p /mnt/{boot,home,var,tmp}
mkdir -p /mnt/boot/efi
mkdir -p /mnt/var/tmp
mkdir -p /mnt/var/log/audit

mount /dev/nvme0n1p2 /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot/efi
mount /dev/vgarch/lv_home /mnt/home
mount /dev/vgarch/lv_var /mnt/var
mkdir -p /mnt/var/tmp /mnt/var/log/audit
mount /dev/vgarch/lv_var_tmp /mnt/var/tmp
mount /dev/vgarch/lv_log /mnt/var/log
mkdir -p /mnt/var/log/audit
mount /dev/vgarch/lv_audit /mnt/var/log/audit
```

---

## 6. Instalasi Base System

### 6.1 Install Paket Dasar

Untuk CPU AMD:

```bash
pacstrap -K /mnt \
  base linux-hardened linux-hardened-headers linux-lts linux-lts-headers linux-firmware amd-ucode \
  grub efibootmgr cryptsetup lvm2 \
  sudo vim nano man-db man-pages texinfo bash-completion \
  networkmanager openssh ufw nftables \
  chrony rsyslog logrotate audit apparmor aide \
  pacman-contrib reflector
```

Untuk CPU Intel, ganti `amd-ucode` menjadi `intel-ucode`.

### 6.2 Generate Fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Edit `/mnt/etc/fstab`:

```bash
vim /mnt/etc/fstab
```

Pastikan mount option sesuai baseline:

```fstab
# /var
UUID=<uuid-var> /var ext4 rw,nodev,nosuid,relatime 0 2

# /var/tmp
UUID=<uuid-var-tmp> /var/tmp ext4 rw,nodev,nosuid,noexec,relatime 0 2

# /var/log
UUID=<uuid-log> /var/log ext4 rw,nodev,nosuid,noexec,relatime 0 2

# /var/log/audit
UUID=<uuid-audit> /var/log/audit ext4 rw,nodev,nosuid,noexec,relatime 0 2

# /home
UUID=<uuid-home> /home ext4 rw,nodev,nosuid,relatime 0 2

# /tmp sebagai tmpfs
tmpfs /tmp tmpfs rw,nosuid,nodev,noexec,relatime,size=2G,mode=1777 0 0

# /dev/shm sebagai tmpfs aman
tmpfs /dev/shm tmpfs rw,nosuid,nodev,noexec,relatime,mode=1777 0 0
```

Validasi:

```bash
cat /mnt/etc/fstab
```

---

## 7. Konfigurasi Sistem Dasar

Masuk ke sistem:

```bash
arch-chroot /mnt
```

### 7.1 Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

### 7.2 Locale

```bash
vim /etc/locale.gen
```

Aktifkan:

```text
en_US.UTF-8 UTF-8
id_ID.UTF-8 UTF-8
```

Generate:

```bash
locale-gen
```

Buat `/etc/locale.conf`:

```bash
cat > /etc/locale.conf <<'EOF_LOCALE'
LANG=en_US.UTF-8
EOF_LOCALE
```

### 7.3 Hostname dan Hosts

```bash
echo "arch-cis" > /etc/hostname

cat > /etc/hosts <<'EOF_HOSTS'
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-cis.localdomain arch-cis
EOF_HOSTS
```

### 7.4 Password Root

```bash
passwd
```

---

## 8. Initramfs, LUKS, dan Bootloader

### 8.1 Konfigurasi mkinitcpio

Edit:

```bash
vim /etc/mkinitcpio.conf
```

Pastikan HOOKS memuat `encrypt` dan `lvm2`:

```text
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```

Generate initramfs:

```bash
mkinitcpio -P
```

### 8.2 Konfigurasi GRUB untuk LUKS

Ambil UUID partisi LUKS:

```bash
blkid /dev/nvme0n1p3
```

Edit:

```bash
vim /etc/default/grub
```

Contoh:

```text
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-LUKS-P3>:cryptlvm root=/dev/vgarch/lv_root audit=1 audit_backlog_limit=8192 lsm=landlock,lockdown,yama,integrity,apparmor,bpf"
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3"
GRUB_DISABLE_RECOVERY=true
```

### 8.3 Install GRUB UEFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ARCH-CIS --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### 8.4 Password Bootloader GRUB

Buat hash password:

```bash
grub-mkpasswd-pbkdf2
```

Tambahkan superuser GRUB:

```bash
cat > /etc/grub.d/40_custom <<'EOF_GRUB'
#!/bin/sh
exec tail -n +3 $0
set superusers="admin"
password_pbkdf2 admin <HASIL_HASH_GRUB_PBKDF2>
EOF_GRUB
chmod 700 /etc/grub.d/40_custom
```

Generate ulang GRUB:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Amankan konfigurasi GRUB:

```bash
chown root:root /boot/grub/grub.cfg
chmod 600 /boot/grub/grub.cfg
```

---

## 9. User Admin dan Sudo

### 9.1 Buat User Admin

```bash
useradd -m -G wheel -s /bin/bash kautsar
passwd kautsar
```

### 9.2 Konfigurasi Sudo

```bash
EDITOR=vim visudo
```

Aktifkan:

```text
%wheel ALL=(ALL:ALL) ALL
```

Tambahkan konfigurasi sudo hardening:

```bash
cat > /etc/sudoers.d/00-cis-hardening <<'EOF_SUDO'
Defaults use_pty
Defaults logfile="/var/log/sudo.log"
Defaults timestamp_timeout=5
Defaults passwd_timeout=1
EOF_SUDO
chmod 440 /etc/sudoers.d/00-cis-hardening
visudo -cf /etc/sudoers.d/00-cis-hardening
```

### 9.3 Batasi `su`

Edit `/etc/pam.d/su` dan aktifkan pembatasan wheel:

```text
auth required pam_wheel.so use_uid group=wheel
```

Maknanya: hanya user dalam group `wheel` yang boleh menggunakan `su`.

---

## 10. Enable Service Dasar

```bash
systemctl enable NetworkManager.service
systemctl enable sshd.service
systemctl enable chronyd.service
systemctl enable ufw.service
systemctl enable auditd.service
systemctl enable apparmor.service
systemctl enable rsyslog.service
```

Keluar dan reboot:

```bash
exit
umount -R /mnt
cryptsetup close cryptlvm
reboot
```

---

## 11. Post-Installation Hardening

Setelah boot ke sistem baru, login sebagai user admin dan gunakan `sudo`.

```bash
sudo -v
```

---

## 12. Package Management Hardening

### 12.1 Update Sistem

```bash
sudo pacman -Syu
```

### 12.2 Verifikasi Keyring Pacman

```bash
sudo pacman-key --init
sudo pacman-key --populate archlinux
sudo pacman-key --refresh-keys
```

### 12.3 Batasi Permission Konfigurasi Pacman

```bash
sudo chown root:root /etc/pacman.conf
sudo chmod 644 /etc/pacman.conf
sudo chown -R root:root /etc/pacman.d
sudo find /etc/pacman.d -type d -exec chmod 755 {} \;
sudo find /etc/pacman.d -type f -exec chmod 644 {} \;
```

### 12.4 Bersihkan Paket Tidak Diperlukan

Cek orphan package:

```bash
pacman -Qtdq
```

Jika benar-benar tidak dibutuhkan:

```bash
sudo pacman -Rns $(pacman -Qtdq)
```

Jika tidak ada orphan package, perintah kedua tidak perlu dijalankan.

---

## 13. Filesystem Kernel Module Hardening

### 13.1 Nonaktifkan Filesystem Tidak Digunakan

Buat file:

```bash
sudo vim /etc/modprobe.d/cis-uncommon-filesystems.conf
```

Isi:

```text
install cramfs /bin/false
blacklist cramfs

install freevxfs /bin/false
blacklist freevxfs

install hfs /bin/false
blacklist hfs

install hfsplus /bin/false
blacklist hfsplus

install jffs2 /bin/false
blacklist jffs2

install udf /bin/false
blacklist udf
```

### 13.2 Overlay dan SquashFS

Tambahkan hanya jika tidak memakai container, live image, snap, atau kebutuhan khusus:

```text
install overlay /bin/false
blacklist overlay

install squashfs /bin/false
blacklist squashfs
```

> Jika sistem memakai Docker/Podman, biasanya `overlay` diperlukan. Jika begitu, dokumentasikan sebagai exception.

### 13.3 FireWire dan USB Storage

Jika tidak dibutuhkan:

```bash
sudo vim /etc/modprobe.d/cis-removable-media.conf
```

Isi:

```text
install firewire-core /bin/false
blacklist firewire-core

install usb-storage /bin/false
blacklist usb-storage
```

> Jika backup via USB masih dibutuhkan, jangan nonaktifkan `usb-storage`; buat exception tertulis.

### 13.4 Rebuild Initramfs

```bash
sudo mkinitcpio -P
```

### 13.5 Audit Module

```bash
lsmod | egrep 'cramfs|freevxfs|hfs|hfsplus|jffs2|udf|firewire_core|usb_storage|overlay|squashfs'
```

Tidak ada output berarti module tersebut tidak sedang aktif.

---

## 14. Mount Option Hardening

Validasi mount option:

```bash
findmnt -no TARGET,OPTIONS /tmp
findmnt -no TARGET,OPTIONS /dev/shm
findmnt -no TARGET,OPTIONS /home
findmnt -no TARGET,OPTIONS /var
findmnt -no TARGET,OPTIONS /var/tmp
findmnt -no TARGET,OPTIONS /var/log
findmnt -no TARGET,OPTIONS /var/log/audit
```

Expected baseline:

| Mount point | Option minimal |
|---|---|
| `/tmp` | `nodev,nosuid,noexec` |
| `/dev/shm` | `nodev,nosuid,noexec` |
| `/home` | `nodev,nosuid` |
| `/var` | `nodev,nosuid` |
| `/var/tmp` | `nodev,nosuid,noexec` |
| `/var/log` | `nodev,nosuid,noexec` |
| `/var/log/audit` | `nodev,nosuid,noexec` |

Reload fstab jika ada perubahan:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

---

## 15. AppArmor Hardening

### 15.1 Pastikan AppArmor Aktif

```bash
sudo systemctl enable --now apparmor.service
aa-enabled
sudo aa-status
```

### 15.2 Kernel Parameter AppArmor

Pastikan `/etc/default/grub` memiliki:

```text
lsm=landlock,lockdown,yama,integrity,apparmor,bpf
```

Generate ulang GRUB jika mengubah parameter:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

### 15.3 Enforcing Profile

Cek status:

```bash
sudo aa-status
```

Set profile ke enforce jika sudah siap:

```bash
sudo aa-enforce /etc/apparmor.d/*
```

Jika ada aplikasi bermasalah, jangan langsung mematikan AppArmor secara global. Gunakan mode complain sementara untuk profile tertentu:

```bash
sudo aa-complain /etc/apparmor.d/<nama-profile>
```

---

## 16. Kernel dan Process Hardening

Buat file sysctl:

```bash
sudo vim /etc/sysctl.d/60-cis-hardening.conf
```

Isi:

```text
# Filesystem protection
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.suid_dumpable = 0

# Kernel information restriction
kernel.yama.ptrace_scope = 1
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.randomize_va_space = 2

# IPv4 hardening
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.tcp_syncookies = 1

# IPv6 hardening
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0
```

Load konfigurasi:

```bash
sudo sysctl --system
```

### 16.1 Disable Core Dump

Buat file limits:

```bash
sudo vim /etc/security/limits.d/10-cis-core-dump.conf
```

Isi:

```text
* hard core 0
```

Konfigurasi systemd-coredump:

```bash
sudo mkdir -p /etc/systemd/coredump.conf.d
sudo vim /etc/systemd/coredump.conf.d/10-cis-hardening.conf
```

Isi:

```ini
[Coredump]
Storage=none
ProcessSizeMax=0
```

Reload:

```bash
sudo systemctl daemon-reload
```

---

## 17. Network Device dan Service Minimization

### 17.1 Cek Interface

```bash
ip link
rfkill list
```

Untuk server, matikan Wi-Fi dan Bluetooth jika tidak digunakan:

```bash
sudo rfkill block wifi
sudo rfkill block bluetooth
sudo systemctl disable --now bluetooth.service 2>/dev/null || true
```

### 17.2 Nonaktifkan Service Tidak Dibutuhkan

Cek service aktif:

```bash
systemctl --type=service --state=running
ss -tulpen
```

Disable service umum yang tidak dibutuhkan:

```bash
for svc in avahi-daemon cups bluetooth rpcbind nfs-server smb snmpd vsftpd named dnsmasq xinetd; do
  sudo systemctl disable --now "$svc.service" 2>/dev/null || true
done
```

> Jangan disable service yang memang menjadi fungsi utama server. Jika server adalah web server, maka service web merupakan exception yang sah.

---

## 18. Firewall UFW

### 18.1 Default Policy

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw default deny routed
```

### 18.2 Allow SSH Terbatas

Jika IP admin diketahui, lebih aman batasi SSH:

```bash
sudo ufw allow from 192.168.10.20 to any port 22 proto tcp
```

Jika belum tahu IP admin:

```bash
sudo ufw allow 22/tcp
```

### 18.3 Enable Firewall

```bash
sudo ufw enable
sudo ufw status verbose
```

---

## 19. SSH Server Hardening

### 19.1 Permission File SSH

```bash
sudo chown root:root /etc/ssh/sshd_config
sudo chmod 600 /etc/ssh/sshd_config
sudo chown -R root:root /etc/ssh/sshd_config.d
sudo chmod 755 /etc/ssh/sshd_config.d
```

Private host key:

```bash
sudo chown root:root /etc/ssh/ssh_host_*_key
sudo chmod 600 /etc/ssh/ssh_host_*_key
```

Public host key:

```bash
sudo chown root:root /etc/ssh/ssh_host_*_key.pub
sudo chmod 644 /etc/ssh/ssh_host_*_key.pub
```

### 19.2 Buat Banner SSH

```bash
sudo tee /etc/issue.net >/dev/null <<'EOF_BANNER'
Authorized access only. All activity may be monitored and reported.
EOF_BANNER
sudo chown root:root /etc/issue.net
sudo chmod 644 /etc/issue.net
```

### 19.3 Konfigurasi SSH Hardening

```bash
sudo vim /etc/ssh/sshd_config.d/99-cis-hardening.conf
```

Isi baseline:

```text
Banner /etc/issue.net
PermitRootLogin no
PermitEmptyPasswords no
PermitUserEnvironment no
HostbasedAuthentication no
IgnoreRhosts yes
GSSAPIAuthentication no
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
GatewayPorts no
LoginGraceTime 60
MaxAuthTries 4
MaxStartups 10:30:60
MaxSessions 2
ClientAliveInterval 300
ClientAliveCountMax 0
LogLevel VERBOSE
UsePAM yes
```

Jika sudah memakai SSH key dan tidak perlu password:

```text
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
```

Validasi konfigurasi:

```bash
sudo sshd -t
sudo systemctl reload sshd
```

> Jangan menutup sesi SSH lama sebelum mencoba login baru. Ini mencegah lockout.

---

## 20. PAM dan Password Policy

> **Peringatan:** Kesalahan konfigurasi PAM bisa membuat semua login gagal. Uji dari TTY atau sesi root cadangan.

### 20.1 Install Komponen Kualitas Password

```bash
sudo pacman -S --needed pam libpwquality cracklib
```

### 20.2 Password Quality

Edit:

```bash
sudo vim /etc/security/pwquality.conf
```

Contoh baseline:

```text
minlen = 14
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
maxrepeat = 3
maxsequence = 3
dictcheck = 1
enforce_for_root
```

### 20.3 Failed Login Lockout

Edit:

```bash
sudo vim /etc/security/faillock.conf
```

Isi:

```text
deny = 5
unlock_time = 900
even_deny_root
root_unlock_time = 60
```

### 20.4 Password History

Edit:

```bash
sudo vim /etc/security/pwhistory.conf
```

Isi:

```text
remember = 24
enforce_for_root
```

### 20.5 Login Definitions

Edit `/etc/login.defs`:

```text
PASS_MAX_DAYS   365
PASS_MIN_DAYS   1
PASS_WARN_AGE   7
UMASK           027
```

Terapkan ke user yang sudah ada:

```bash
sudo chage --maxdays 365 --mindays 1 --warndays 7 kautsar
```

---

## 21. User, Group, dan Root Account Hardening

### 21.1 UID 0

Pastikan hanya root yang memiliki UID 0:

```bash
awk -F: '($3 == 0) { print }' /etc/passwd
```

Output seharusnya hanya:

```text
root:x:0:0:root:/root:/bin/bash
```

### 21.2 Root PATH Integrity

```bash
sudo env | grep '^PATH='
```

Pastikan tidak ada:

- Direktori kosong.
- `.` dalam PATH.
- Direktori world-writable.

### 21.3 Root Umask

Tambahkan ke `/root/.bashrc` atau `/etc/profile.d/cis-umask.sh`:

```bash
sudo tee /etc/profile.d/cis-umask.sh >/dev/null <<'EOF_UMASK'
umask 027
EOF_UMASK
sudo chmod 644 /etc/profile.d/cis-umask.sh
```

### 21.4 Lock System Accounts Non-Login

Cek akun system:

```bash
awk -F: '($3 < 1000 && $1 != "root") { print $1,$7 }' /etc/passwd
```

Akun service sebaiknya memakai shell non-login seperti `/usr/bin/nologin`.

---

## 22. Warning Banner

### 22.1 `/etc/issue`

```bash
sudo tee /etc/issue >/dev/null <<'EOF_ISSUE'
Authorized access only. All activity may be monitored and reported.
EOF_ISSUE
sudo chown root:root /etc/issue
sudo chmod 644 /etc/issue
```

### 22.2 `/etc/motd`

```bash
sudo tee /etc/motd >/dev/null <<'EOF_MOTD'
Authorized users only. Use of this system constitutes consent to monitoring.
EOF_MOTD
sudo chown root:root /etc/motd
sudo chmod 644 /etc/motd
```

Banner tidak boleh menampilkan informasi sensitif seperti:

- Nama OS terlalu detail.
- Versi kernel.
- Nama organisasi internal yang tidak perlu.
- Informasi network internal.

---

## 23. Time Synchronization

Gunakan satu daemon waktu. Panduan ini memakai chrony.

```bash
sudo systemctl disable --now systemd-timesyncd.service 2>/dev/null || true
sudo systemctl enable --now chronyd.service
chronyc tracking
chronyc sources -v
```

---

## 24. Logging Hardening

### 24.1 journald Persistent Logging

```bash
sudo mkdir -p /var/log/journal
sudo chown root:systemd-journal /var/log/journal
sudo chmod 2755 /var/log/journal
```

Buat konfigurasi:

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
sudo vim /etc/systemd/journald.conf.d/10-cis-hardening.conf
```

Isi:

```ini
[Journal]
Storage=persistent
Compress=yes
ForwardToSyslog=yes
SystemMaxUse=1G
RuntimeMaxUse=200M
```

Restart:

```bash
sudo systemctl restart systemd-journald
```

### 24.2 rsyslog

```bash
sudo systemctl enable --now rsyslog.service
```

Pastikan permission log aman:

```bash
sudo find /var/log -type f -exec chmod g-wx,o-rwx {} \;
sudo find /var/log -type d -exec chmod g-w,o-rwx {} \;
```

### 24.3 logrotate

Pastikan logrotate tersedia:

```bash
pacman -Q logrotate
```

Cek konfigurasi:

```bash
ls -l /etc/logrotate.conf /etc/logrotate.d/
```

---

## 25. Auditd Hardening

### 25.1 Enable auditd

```bash
sudo systemctl enable --now auditd.service
sudo auditctl -s
```

Pastikan kernel parameter GRUB memuat:

```text
audit=1 audit_backlog_limit=8192
```

### 25.2 Data Retention Audit

Edit:

```bash
sudo vim /etc/audit/auditd.conf
```

Contoh baseline:

```text
max_log_file = 32
max_log_file_action = keep_logs
space_left_action = email
action_mail_acct = root
admin_space_left_action = halt
disk_full_action = halt
disk_error_action = halt
```

> `halt` sangat ketat. Untuk sistem lab boleh diganti `single` atau `syslog`, tetapi dokumentasikan exception.

### 25.3 Audit Rules Baseline

Buat file:

```bash
sudo vim /etc/audit/rules.d/50-cis-baseline.rules
```

Isi:

```text
# Delete existing rules
-D

# Buffer
-b 8192

# Failure mode: 1=printk, 2=panic. Gunakan 1 untuk baseline aman.
-f 1

# Identity files
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity

# Sudoers
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope

# Login/logout/session
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
-w /var/run/utmp -p wa -k session
-w /var/log/wtmp -p wa -k session
-w /var/log/btmp -p wa -k session

# Time changes
-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change
-a always,exit -F arch=b32 -S adjtimex,settimeofday,clock_settime -k time-change
-w /etc/localtime -p wa -k time-change

# Hostname/network config
-a always,exit -F arch=b64 -S sethostname,setdomainname -k system-locale
-a always,exit -F arch=b32 -S sethostname,setdomainname -k system-locale
-w /etc/hosts -p wa -k system-locale
-w /etc/hostname -p wa -k system-locale
-w /etc/NetworkManager/ -p wa -k network-config

# MAC policy
-w /etc/apparmor/ -p wa -k MAC-policy
-w /etc/apparmor.d/ -p wa -k MAC-policy

# Mounts
-a always,exit -F arch=b64 -S mount -k mounts
-a always,exit -F arch=b32 -S mount -k mounts

# File deletion by users
-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat,rmdir -F auid>=1000 -F auid!=unset -k delete
-a always,exit -F arch=b32 -S unlink,unlinkat,rename,renameat,rmdir -F auid>=1000 -F auid!=unset -k delete

# Permission changes
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat,chown,fchown,fchownat,lchown,setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod
-a always,exit -F arch=b32 -S chmod,fchmod,fchmodat,chown,fchown,fchownat,lchown,setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod

# Kernel modules
-w /usr/bin/kmod -p x -k kernel_modules
-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k kernel_modules

# Immutable rules: aktifkan setelah yakin semua rule benar
# -e 2
```

Load rules:

```bash
sudo augenrules --load
sudo auditctl -s
```

Cek rule:

```bash
sudo auditctl -l
```

---

## 26. AIDE Integrity Checking

### 26.1 Initialize AIDE

```bash
sudo aide --init
```

Database awal biasanya dibuat di lokasi seperti:

```text
/var/lib/aide/aide.db.new
```

Aktifkan sebagai database baseline:

```bash
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

### 26.2 Jalankan Check

```bash
sudo aide --check
```

### 26.3 Jadwalkan Integrity Check

Buat systemd service:

```bash
sudo vim /etc/systemd/system/aide-check.service
```

Isi:

```ini
[Unit]
Description=AIDE integrity check

[Service]
Type=oneshot
ExecStart=/usr/bin/aide --check
```

Buat timer:

```bash
sudo vim /etc/systemd/system/aide-check.timer
```

Isi:

```ini
[Unit]
Description=Run AIDE integrity check daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aide-check.timer
```

---

## 27. File dan Directory Permission Hardening

### 27.1 Permission File Sistem Penting

```bash
sudo chown root:root /etc/passwd /etc/passwd- /etc/group /etc/group- /etc/shells
sudo chmod 644 /etc/passwd /etc/passwd- /etc/group /etc/group- /etc/shells

sudo chown root:root /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow-
sudo chmod 000 /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow-
```

Jika `/etc/security/opasswd` ada:

```bash
sudo chown root:root /etc/security/opasswd 2>/dev/null || true
sudo chmod 600 /etc/security/opasswd 2>/dev/null || true
```

### 27.2 World Writable Files

Cari:

```bash
sudo find / -xdev -type f -perm -0002 -print
sudo find / -xdev -type d -perm -0002 ! -perm -1000 -print
```

Direktori world-writable harus memiliki sticky bit jika memang dibutuhkan, contohnya `/tmp`.

### 27.3 File Tanpa Owner/Group

```bash
sudo find / -xdev \( -nouser -o -nogroup \) -print
```

### 27.4 Review SUID/SGID

```bash
sudo find / -xdev -perm /6000 -type f -print
```

Review satu per satu. Jangan menghapus SUID/SGID secara massal tanpa analisis karena beberapa binary memang membutuhkannya.

---

## 28. Local User dan Group Checks

### 28.1 Shadowed Password

```bash
awk -F: '($2 != "x") { print $1 " does not use shadowed password" }' /etc/passwd
```

### 28.2 Empty Password Field

```bash
sudo awk -F: '($2 == "") { print $1 " has empty password" }' /etc/shadow
```

### 28.3 Group di `/etc/passwd` Harus Ada di `/etc/group`

```bash
for gid in $(cut -s -d: -f4 /etc/passwd | sort -u); do
  grep -q -P "^.*?:[^:]*:$gid:" /etc/group || echo "GID $gid missing in /etc/group"
done
```

### 28.4 Duplicate UID/GID/User/Group

```bash
cut -f3 -d: /etc/passwd | sort | uniq -d
cut -f3 -d: /etc/group | sort | uniq -d
cut -f1 -d: /etc/passwd | sort | uniq -d
cut -f1 -d: /etc/group | sort | uniq -d
```

### 28.5 Home Directory User Interaktif

```bash
awk -F: '($3>=1000 && $7 !~ /(nologin|false)$/) {print $1,$6}' /etc/passwd
```

Pastikan setiap home directory:

- Ada.
- Dimiliki user terkait.
- Tidak writable oleh group/other tanpa alasan.

Contoh perbaikan:

```bash
sudo chown -R kautsar:kautsar /home/kautsar
sudo chmod 750 /home/kautsar
```

### 28.6 Dot Files

```bash
sudo find /home -name ".*" -type f -perm /022 -ls
```

Hilangkan write permission untuk group/other jika tidak perlu.

---

## 29. GNOME atau GUI Hardening Jika Digunakan

Jika sistem server tidak memakai GUI, lewati bagian ini dan pastikan display manager tidak aktif.

```bash
systemctl status gdm.service 2>/dev/null || true
```

Jika menggunakan GNOME/GDM:

- Aktifkan login banner.
- Disable user list.
- Aktifkan screen lock.
- Disable automount dan autorun.
- Pastikan XDMCP tidak aktif.
- Batasi Xwayland jika tidak diperlukan.

Contoh disable automount untuk GNOME:

```bash
gsettings set org.gnome.desktop.media-handling automount false
gsettings set org.gnome.desktop.media-handling automount-open false
gsettings set org.gnome.desktop.media-handling autorun-never true
```

Untuk konfigurasi skala sistem, gunakan dconf profile dan lock database.

---

## 30. Validasi Akhir

### 30.1 Validasi Boot dan Mount

```bash
lsblk -f
findmnt
findmnt -no TARGET,OPTIONS /tmp /dev/shm /home /var /var/tmp /var/log /var/log/audit
```

### 30.2 Validasi Kernel Parameter

```bash
cat /proc/cmdline
sysctl fs.protected_hardlinks fs.protected_symlinks kernel.yama.ptrace_scope kernel.dmesg_restrict kernel.kptr_restrict kernel.randomize_va_space
```

### 30.3 Validasi Service

```bash
systemctl --type=service --state=running
ss -tulpen
```

### 30.4 Validasi Firewall

```bash
sudo ufw status verbose
```

### 30.5 Validasi AppArmor

```bash
aa-enabled
sudo aa-status
```

### 30.6 Validasi Auditd

```bash
sudo auditctl -s
sudo auditctl -l
```

### 30.7 Validasi SSH

```bash
sudo sshd -t
sudo sshd -T | egrep 'permitrootlogin|permitemptypasswords|permituserenvironment|x11forwarding|maxauthtries|maxsessions|clientaliveinterval|clientalivecountmax|loglevel|usepam|banner'
```

### 30.8 Validasi Permission Penting

```bash
stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow /etc/shells
```

---

## 31. Checklist Ringkas

| Area | Status |
|---|---|
| UEFI mode diverifikasi | ☐ |
| Disk dipartisi GPT | ☐ |
| Root memakai LUKS2 | ☐ |
| LVM dibuat | ☐ |
| `/tmp` tmpfs `nodev,nosuid,noexec` | ☐ |
| `/dev/shm` `nodev,nosuid,noexec` | ☐ |
| `/var`, `/var/tmp`, `/var/log`, `/var/log/audit`, `/home` terpisah | ☐ |
| Kernel hardened terpasang | ☐ |
| Kernel LTS cadangan terpasang | ☐ |
| GRUB password aktif | ☐ |
| AppArmor aktif | ☐ |
| UFW aktif | ☐ |
| SSH hardened | ☐ |
| sudo `use_pty` dan log aktif | ☐ |
| PAM password policy diterapkan | ☐ |
| journald persistent | ☐ |
| rsyslog aktif | ☐ |
| auditd aktif | ☐ |
| AIDE baseline dibuat | ☐ |
| Service tidak perlu dimatikan | ☐ |
| File permission penting divalidasi | ☐ |
| User/group checks selesai | ☐ |
| Exception terdokumentasi | ☐ |

---

## 32. Catatan Exception yang Umum pada Arch Linux

### 32.1 Overlay Tidak Dinonaktifkan

Jika memakai Docker/Podman, `overlay` dibutuhkan. Jangan blacklist module ini. Dokumentasikan sebagai exception.

### 32.2 USB Storage Tidak Dinonaktifkan

Jika admin butuh backup offline atau transfer data via USB, `usb-storage` boleh tetap aktif, tetapi akses fisik dan automount harus dikendalikan.

### 32.3 GUI Tetap Digunakan

Jika sistem adalah workstation, GUI boleh dipakai. Terapkan hardening GDM, screen lock, disable automount, dan disable user list.

### 32.4 IPv6 Tetap Aktif

Jika jaringan memakai IPv6, jangan disable IPv6. Yang penting adalah mematikan forwarding, redirect, source routing, dan router advertisement yang tidak diperlukan.

---

## 33. Rekomendasi Operasional Lanjutan

1. Jalankan update berkala:

```bash
sudo pacman -Syu
```

2. Review listening service:

```bash
ss -tulpen
```

3. Review failed login:

```bash
sudo faillock
```

4. Review audit log:

```bash
sudo ausearch -k identity
sudo ausearch -k scope
sudo ausearch -k kernel_modules
```

5. Review integrity:

```bash
sudo aide --check
```

6. Review SUID/SGID setelah update besar:

```bash
sudo find / -xdev -perm /6000 -type f -print
```

7. Backup konfigurasi penting:

```bash
sudo tar -czf /root/arch-cis-config-backup-$(date +%F).tar.gz \
  /etc/fstab /etc/default/grub /etc/mkinitcpio.conf /etc/sysctl.d \
  /etc/modprobe.d /etc/ssh /etc/sudoers.d /etc/audit /etc/apparmor.d
```

---

## 34. Kesimpulan

Instalasi Arch Linux yang mengikuti prinsip CIS Debian tidak cukup hanya dengan menginstal paket keamanan. Hardening harus dimulai sejak desain awal: partisi, mount option, bootloader, kernel, package management, service minimization, firewall, SSH, PAM, logging, audit, dan integrity checking.

Karena Arch Linux bukan Debian, seluruh rekomendasi perlu dipahami sebagai prinsip, bukan salinan literal. Perbedaan distribusi harus selalu diperhatikan. Setiap pengecualian harus didokumentasikan, diuji, dan ditinjau ulang secara berkala.

---

## 35. Referensi Utama

- CIS Debian Linux 12 Benchmark v2.0.0, 05-28-2026.
- ArchWiki, Installation guide.
- ArchWiki, General recommendations.
- ArchWiki, Security.
- ArchWiki, AppArmor.
- ArchWiki, GRUB.
- ArchWiki, dm-crypt.
- ArchWiki, LVM.
- ArchWiki, OpenSSH.
- ArchWiki, Audit framework.
- ArchWiki, UFW.
- ArchWiki, AIDE.

