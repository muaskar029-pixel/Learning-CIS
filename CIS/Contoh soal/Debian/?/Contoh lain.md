Berikut contoh **soal pilihan ganda berdasarkan panduan `template.md`** yang kamu kirim. Aku buat mengikuti bagian besar dalam panduan: metadata, profil, device, storage/LUKS/LVM, package, konfigurasi sistem, kernel, systemd, SSH, PAM, firewall, AppArmor, account, dan finishing. 

# Latihan Soal Pilihan Ganda Berdasarkan Panduan Instalasi/Hardening

## A. Metadata dan Profil Sistem

**1. Pada bagian metadata, nilai `Level` pada panduan adalah...**
A. Public
B. Internal
C. Secret
D. Restricted

**Jawaban: C. Secret**

---

**2. Fungsi bagian `profiles` dalam panduan adalah untuk...**
A. Menentukan jenis filesystem yang dipakai
B. Mendokumentasikan identitas, level akses, posisi, divisi, dan ruang lingkup operasional
C. Mengatur password bootloader
D. Mengaktifkan firewall

**Jawaban: B**

---

**3. Pada bagian `access`, panduan membagi akses menjadi dua kelompok utama, yaitu...**
A. Local dan Public
B. Root dan User
C. Operation dan Recovery
D. Server dan Workstation

**Jawaban: C**

---

**4. Informasi seperti `Hostname`, `Local Network`, `Intern Network`, dan `Public Network` digunakan untuk menjelaskan...**
A. Struktur partisi
B. Ruang akses operasional perangkat
C. Konfigurasi package manager
D. Daftar user sudo

**Jawaban: B**

---

## B. Device dan Hardware

**5. Perintah `lspci` pada bagian device digunakan untuk...**
A. Melihat daftar partisi
B. Melihat perangkat PCI seperti VGA, USB controller, ethernet, SATA, dan NVMe controller
C. Mengaktifkan firewall
D. Membuat LUKS container

**Jawaban: B**

---

**6. Berdasarkan tabel device, ethernet controller yang digunakan adalah...**
A. Realtek RTL8111
B. Intel Ethernet Connection I219-LM
C. Broadcom NetXtreme
D. Qualcomm Atheros

**Jawaban: B**

---

**7. Pada bagian storage, nilai rotational number `0` berarti storage tersebut adalah...**
A. HDD
B. Tape drive
C. SSD atau NVMe
D. Optical disk

**Jawaban: C**

---

**8. Perintah `smartctl -i /dev/sda` digunakan untuk...**
A. Membuat filesystem ext4
B. Menampilkan informasi detail perangkat storage
C. Menghapus user
D. Mengatur SSH port

**Jawaban: B**

---

**9. Pada bagian memory, total slot memori fisik yang terdeteksi adalah...**
A. 1
B. 2
C. 3
D. 4

**Jawaban: B**

---

**10. Perintah untuk mengecek dukungan KVM/virtualisasi CPU pada panduan adalah...**
A. `lsblk -f`
B. `lspci -knn`
C. `grep -E --color=auto 'vmx|svm' /proc/cpuinfo`
D. `systemctl status kvm`

**Jawaban: C**

---

## C. Storage, Partisi, LUKS, dan LVM

**11. Perintah `nvme format --lbaf=0 --ms=0 --ses=1 /dev/nvme0n1` pada panduan digunakan pada tahap...**
A. Package installation
B. Persiapan/formating storage NVMe
C. Konfigurasi firewall
D. Membuat user

**Jawaban: B**

---

**12. Skema partisi pada panduan menggunakan label partisi...**
A. MBR
B. GPT
C. BSD disklabel
D. Apple Partition Map

**Jawaban: B**

---

**13. Pada bagian `sfdisk`, partisi pertama diberi tipe...**
A. swap
B. home
C. uefi
D. lvm

**Jawaban: C**

---

**14. Fungsi `cryptsetup luksFormat --type luks2` adalah...**
A. Membuat filesystem ext4
B. Membuat container enkripsi LUKS2
C. Membuat user baru
D. Mengaktifkan AppArmor

**Jawaban: B**

---

**15. Opsi `--sector-size 4096` pada konfigurasi LUKS berkaitan dengan...**
A. Ukuran sektor enkripsi/storage
B. Ukuran RAM
C. Ukuran swap
D. Port SSH

**Jawaban: A**

---

**16. Perintah `cryptsetup open /dev/nvme0n1p2 main` digunakan untuk...**
A. Menutup container LUKS
B. Membuka container LUKS dengan nama mapping `main`
C. Menghapus LVM
D. Membuat partisi EFI

**Jawaban: B**

---

**17. Setelah LUKS dibuka, panduan membuat physical volume dengan perintah...**
A. `vgcreate base /dev/nvme0n1`
B. `pvcreate --dataalignment 2M /dev/mapper/main`
C. `lvcreate -L 15G base -n root`
D. `mkfs.ext4 /dev/base/root`

**Jawaban: B**

---

**18. Volume group yang dibuat dalam panduan bernama...**
A. main
B. blackbird
C. base
D. rootfs

**Jawaban: C**

---

**19. Logical volume `root` pada panduan dibuat dengan ukuran...**
A. 5G
B. 10G
C. 15G
D. 50G

**Jawaban: C**

---

**20. Mount option `nodev,nosuid,noexec` pada `/home` bertujuan untuk...**
A. Mempercepat booting
B. Membatasi device file, SUID/SGID, dan eksekusi file langsung dari partisi tersebut
C. Mengaktifkan IPv6
D. Menghapus log otomatis

**Jawaban: B**

---

## D. Mount Point dan Filesystem Hardening

**21. Mount point `/var/log/audit` pada panduan dipisahkan ke logical volume...**
A. `vtmp`
B. `vlog`
C. `vaud`
D. `vpac`

**Jawaban: C**

---

**22. Pemisahan `/var/log/audit` secara konseptual penting karena...**
A. Agar tampilan desktop lebih ringan
B. Agar audit log memiliki ruang dan kontrol tersendiri
C. Agar bisa menjalankan game
D. Agar user biasa bisa menghapus log

**Jawaban: B**

---

**23. Logical volume `vpac` digunakan untuk mount point...**
A. `/var/cache/pacman`
B. `/var/log`
C. `/tmp`
D. `/home`

**Jawaban: A**

---

**24. Mount option `errors=remount-ro` berarti jika filesystem bermasalah maka...**
A. Sistem langsung reboot
B. Filesystem diubah menjadi read-only
C. Semua file dihapus
D. User langsung logout

**Jawaban: B**

---

**25. Dalam catatan panduan, opsi `errors=panic` berarti...**
A. Sistem mengabaikan error
B. Filesystem otomatis diformat ulang
C. Kernel panic ketika terjadi kerusakan tertentu pada filesystem
D. Firewall langsung aktif

**Jawaban: C**

---

**26. Logical volume `swap` dibuat dengan ukuran...**
A. 1G
B. 2G
C. 8G
D. 16G

**Jawaban: D**

---

**27. Volume `priv` pada panduan dienkripsi lagi menggunakan...**
A. SSH
B. LUKS2
C. AppArmor
D. Firewalld

**Jawaban: B**

---

## E. Package Installation

**28. Paket `linux-hardened` pada panduan menunjukkan bahwa sistem diarahkan memakai...**
A. Kernel default biasa
B. Kernel hardened
C. Kernel Windows
D. Kernel BSD

**Jawaban: B**

---

**29. Paket `firewalld` pada daftar pacstrap digunakan untuk...**
A. Mengelola firewall berbasis zone
B. Mengelola bootloader
C. Mengatur password user
D. Membuat filesystem

**Jawaban: A**

---

**30. Paket `apparmor` digunakan untuk...**
A. Mandatory Access Control berbasis profile
B. Manajemen wallpaper
C. Menjalankan SSH
D. Membuat partisi GPT

**Jawaban: A**

---

**31. Paket `openssh` diperlukan agar sistem dapat...**
A. Menjalankan remote shell/SSH
B. Membuat swap
C. Mengatur DNSOverTLS
D. Menjalankan Flatpak

**Jawaban: A**

---

**32. Paket `intel-ucode` atau `amd-ucode` dipasang sesuai dengan...**
A. Jenis GPU
B. Jenis processor
C. Jenis filesystem
D. Jenis shell

**Jawaban: B**

---

## F. Konfigurasi Sistem Dasar

**33. Perintah `genfstab -U /mnt > /mnt/etc/fstab` digunakan untuk...**
A. Membuat user
B. Membuat file konfigurasi mount berdasarkan UUID
C. Mengaktifkan firewall
D. Mengatur SSH banner

**Jawaban: B**

---

**34. Hostname pada panduan diatur menjadi...**
A. archlinux
B. blackmon
C. debian12
D. localhost

**Jawaban: B**

---

**35. Perintah `chattr +i /etc/hostname` bertujuan untuk...**
A. Membuat file dapat dieksekusi
B. Membuat file immutable agar tidak mudah diubah
C. Menghapus hostname
D. Membuka akses user biasa

**Jawaban: B**

---

**36. File `/etc/environment` dalam panduan berisi variabel seperti...**
A. `EDITOR`, `SYSTEMD_EDITOR`, dan `BLACKBIRD_MODE`
B. `PermitRootLogin`
C. `PasswordAuthentication`
D. `DefaultZone`

**Jawaban: A**

---

**37. Timezone yang digunakan pada panduan adalah...**
A. UTC
B. Asia/Tokyo
C. Asia/Jakarta
D. Europe/London

**Jawaban: C**

---

**38. File `/usr/lib/os-release` dimodifikasi untuk mengubah identitas sistem menjadi...**
A. Ubuntu
B. Debian
C. Blackbird
D. Fedora

**Jawaban: C**

---

## G. Kernel Parameter dan Boot

**39. Parameter `ipv6.disable=1` pada `/etc/cmdline.d/05-nets.conf` berarti...**
A. Mengaktifkan IPv6
B. Menonaktifkan IPv6
C. Mengaktifkan DNSSEC
D. Menonaktifkan SSH

**Jawaban: B**

---

**40. Parameter `lsm=landlock,lockdown,yama,apparmor,bpf` digunakan untuk...**
A. Mengatur urutan dan aktivasi Linux Security Modules
B. Mengatur ukuran partisi
C. Mengganti password root
D. Mengaktifkan Bluetooth

**Jawaban: A**

---

**41. Dalam panduan, initramfs dibuat menggunakan...**
A. dracut
B. mkinitcpio
C. update-initramfs
D. grub-install

**Jawaban: B**

---

**42. Hook `sd-encrypt` pada mkinitcpio berkaitan dengan...**
A. Dekripsi LUKS saat boot berbasis systemd initramfs
B. Menjalankan firewall
C. Menghapus user
D. Mengatur DNS

**Jawaban: A**

---

**43. Bootloader yang digunakan pada bagian finishing adalah...**
A. GRUB legacy
B. systemd-boot melalui `bootctl`
C. LILO
D. rEFInd saja

**Jawaban: B**

---

## H. systemd-networkd dan resolved

**44. File `20-ethernet.network` pada panduan menghubungkan interface ethernet ke...**
A. wlan0
B. bridge
C. docker0
D. lo

**Jawaban: B**

---

**45. File `.netdev` dengan `Kind=bridge` digunakan untuk...**
A. Membuat virtual bridge interface
B. Membuat user baru
C. Mengatur SSH cipher
D. Menghapus firewall

**Jawaban: A**

---

**46. Pada konfigurasi `systemd-resolved`, opsi `DNSOverTLS=true` berarti...**
A. DNS dimatikan
B. DNS query diarahkan menggunakan TLS jika didukung
C. DHCP dimatikan
D. IPv6 wajib aktif

**Jawaban: B**

---

**47. Opsi `LLMNR=no` dan `MulticastDNS=no` bertujuan untuk...**
A. Mengurangi mekanisme resolusi nama lokal yang tidak diperlukan
B. Mengaktifkan Bluetooth
C. Membuka semua port
D. Menjalankan NTP lama

**Jawaban: A**

---

**48. Pada konfigurasi sleep, `AllowSuspend=no` berarti...**
A. Sistem boleh suspend
B. Sistem tidak mengizinkan suspend
C. Sistem wajib hibernate
D. Sistem selalu reboot

**Jawaban: B**

---

**49. Konfigurasi journald pada panduan menggunakan `Storage=persistent`, yang berarti...**
A. Log hanya disimpan di RAM
B. Log tetap disimpan secara persisten di disk
C. Log langsung dihapus
D. Log hanya dikirim ke Bluetooth

**Jawaban: B**

---

## I. Pacman dan Update

**50. Pada konfigurasi pacman, `SigLevel = Required DatabaseOptional TrustedOnly` berkaitan dengan...**
A. Verifikasi tanda tangan paket dan database
B. Pengaturan wallpaper
C. Pengaturan port SSH
D. Penghapusan user

**Jawaban: A**

---

**51. Service `update.service` dalam panduan menjalankan...**
A. `pacman --sync --refresh --sysupgrade --noconfirm`
B. `apt update`
C. `dnf upgrade`
D. `flatpak uninstall`

**Jawaban: A**

---

**52. Timer `update.timer` dikonfigurasi berjalan...**
A. Setiap reboot saja
B. Setiap jam
C. Setiap bulan
D. Tidak pernah

**Jawaban: B**

---

**53. Risiko update otomatis tanpa pengujian pada sistem produksi adalah...**
A. Paket bisa berubah dan menyebabkan aplikasi rusak
B. Sistem menjadi lebih lambat mengetik
C. Hostname hilang
D. Memory otomatis bertambah

**Jawaban: A**

---

## J. SSH Hardening

**54. Port SSH pada panduan diubah menjadi...**
A. 22
B. 2222
C. 10022
D. 443

**Jawaban: C**

---

**55. `AddressFamily inet` pada konfigurasi SSH berarti SSH hanya menggunakan...**
A. IPv6
B. IPv4
C. Bluetooth
D. Unix socket saja

**Jawaban: B**

---

**56. `PermitRootLogin no` bertujuan untuk...**
A. Mengizinkan login root langsung
B. Menolak login root langsung melalui SSH
C. Menghapus akun root
D. Mengubah port SSH

**Jawaban: B**

---

**57. `PasswordAuthentication no` berarti...**
A. SSH hanya menerima password
B. Password login SSH dinonaktifkan
C. Semua user bebas login
D. SSH dimatikan

**Jawaban: B**

---

**58. `PermitEmptyPasswords no` penting karena...**
A. Mencegah akun tanpa password login melalui SSH
B. Mempercepat koneksi SSH
C. Mengaktifkan DNS
D. Mengizinkan root login

**Jawaban: A**

---

**59. `MaxAuthTries 2` membatasi...**
A. Jumlah user dalam sistem
B. Jumlah percobaan autentikasi SSH
C. Jumlah partisi LVM
D. Jumlah service systemd

**Jawaban: B**

---

**60. `AllowTcpForwarding no` bertujuan untuk...**
A. Mencegah penggunaan SSH sebagai tunnel TCP
B. Mengaktifkan HTTP
C. Membuka port 10022
D. Mengaktifkan X11

**Jawaban: A**

---

**61. `X11Forwarding no` bertujuan untuk...**
A. Menonaktifkan forwarding GUI X11 melalui SSH
B. Mengaktifkan desktop Hyprland
C. Menghapus SSH key
D. Mengaktifkan root login

**Jawaban: A**

---

**62. `UseDNS no` pada SSH dapat membantu...**
A. Menghindari lookup DNS saat koneksi SSH
B. Mengaktifkan DNSOverTLS
C. Menjalankan resolved
D. Menghapus DNS server

**Jawaban: A**

---

## K. PAM dan Autentikasi

**63. File `/etc/pam.d/login` digunakan untuk mengatur...**
A. Perilaku autentikasi login lokal
B. Konfigurasi firewall
C. Konfigurasi bridge
D. Konfigurasi storage

**Jawaban: A**

---

**64. Modul `pam_nologin.so` berfungsi untuk...**
A. Mencegah login ketika file nologin aktif
B. Membuat user baru
C. Menghapus password
D. Menjalankan firewall

**Jawaban: A**

---

**65. `pam_gnome_keyring.so` dalam konfigurasi PAM berkaitan dengan...**
A. Integrasi keyring GNOME
B. Firewall zone
C. LUKS header
D. Audit log

**Jawaban: A**

---

**66. `use_authtok` pada PAM biasanya berarti...**
A. Menggunakan token/password autentikasi yang sudah ada dalam proses PAM
B. Menghapus password lama
C. Mengaktifkan SSH root
D. Mengubah hostname

**Jawaban: A**

---

## L. Firewall Firewalld

**67. Firewall yang digunakan pada panduan adalah...**
A. UFW
B. firewalld
C. pfSense
D. Windows Firewall

**Jawaban: B**

---

**68. Konsep zone pada firewalld digunakan untuk...**
A. Mengelompokkan kebijakan trafik berdasarkan area/interface/sumber jaringan
B. Mengatur ukuran RAM
C. Menghapus paket
D. Membuat user

**Jawaban: A**

---

**69. Zone `admin.xml` pada panduan mengizinkan port...**
A. 22
B. 80
C. 10022
D. 3306

**Jawaban: C**

---

**70. Zone `tunnel.xml` mengikat interface...**
A. enp0s31f6
B. wlan0
C. tailscale0
D. lo

**Jawaban: C**

---

**71. Fungsi source address pada firewalld zone adalah...**
A. Membatasi sumber IP yang masuk ke zone tersebut
B. Mengatur username
C. Mengatur ukuran partisi
D. Menghapus DNS

**Jawaban: A**

---

**72. Jika SSH hanya boleh diakses dari IP admin tertentu, pendekatan firewall yang tepat adalah...**
A. Membuka port SSH untuk semua jaringan
B. Membatasi port SSH berdasarkan source address admin
C. Mematikan firewall
D. Mengaktifkan Telnet

**Jawaban: B**

---

## M. AppArmor dan Security Controls

**73. AppArmor termasuk mekanisme...**
A. Mandatory Access Control
B. Package manager
C. Filesystem biasa
D. Bootloader

**Jawaban: A**

---

**74. Perintah `systemctl enable apparmor.service` digunakan untuk...**
A. Mengaktifkan AppArmor saat boot
B. Menghapus AppArmor
C. Mengubah hostname
D. Membuat LVM

**Jawaban: A**

---

**75. Parameter `lockdown` dalam LSM berkaitan dengan...**
A. Pembatasan akses tertentu ke kernel untuk meningkatkan keamanan
B. Mengatur DNS
C. Membuka firewall
D. Mengaktifkan Bluetooth

**Jawaban: A**

---

**76. `systemd-coredump` dikonfigurasi dengan `Storage=none` dan `ProcessSizeMax=0` untuk...**
A. Menonaktifkan penyimpanan core dump
B. Menyimpan core dump lebih besar
C. Mengaktifkan crash report publik
D. Membuka akses user biasa

**Jawaban: A**

---

## N. Account dan User Management

**77. Group `intern`, `assets`, `vadmin`, `podmin`, dan `public` dibuat untuk...**
A. Mengelompokkan akses dan peran user
B. Mengatur bootloader
C. Menghapus disk
D. Mengatur DNS server

**Jawaban: A**

---

**78. Perintah `passwd -l root` digunakan untuk...**
A. Mengganti password root
B. Mengunci akun root
C. Menghapus root
D. Membuka login root SSH

**Jawaban: B**

---

**79. User yang ditambahkan ke group `libvirt,kvm` kemungkinan memiliki kebutuhan untuk...**
A. Virtualisasi
B. Menjalankan FTP
C. Mengatur DNS
D. Membaca log audit saja

**Jawaban: A**

---

**80. File `/etc/sudoers.d/user` berisi aturan...**
A. Hak sudo user tertentu
B. Konfigurasi SSH key
C. Konfigurasi DNS
D. Konfigurasi LUKS

**Jawaban: A**

---

**81. Risiko memberikan banyak user akses sudo tanpa kontrol adalah...**
A. Permukaan privilege escalation meningkat
B. Storage otomatis penuh
C. DNS menjadi lambat
D. Bluetooth mati

**Jawaban: A**

---

**82. `chattr +i /etc/sudoers.d/user` bertujuan untuk...**
A. Membuat file aturan sudo immutable
B. Menghapus hak sudo
C. Membuka file untuk semua user
D. Mengaktifkan SSH

**Jawaban: A**

---

## O. pam_mount dan Private Volume

**83. `pam_mount` pada panduan digunakan untuk...**
A. Mount volume otomatis saat login berbasis PAM
B. Mengaktifkan firewall
C. Menghapus user
D. Menjalankan update pacman

**Jawaban: A**

---

**84. Volume private pada panduan menggunakan tipe filesystem PAM mount...**
A. ext4 langsung
B. crypt
C. vfat
D. nfs

**Jawaban: B**

---

**85. Opsi `<mntoptions require="nosuid,nodev" />` berarti...**
A. Mount wajib memenuhi opsi `nosuid` dan `nodev`
B. Mount harus executable
C. Mount harus tanpa enkripsi
D. Mount hanya untuk root

**Jawaban: A**

---

## P. Finishing dan Boot Image

**86. Pada tahap finishing, file kernel dan microcode dipindahkan ke...**
A. `/boot/kernel`
B. `/etc/kernel`
C. `/var/kernel`
D. `/home/kernel`

**Jawaban: A**

---

**87. File preset `linux-hardened.preset` mengatur output UKI ke...**
A. `/boot/grub/grub.cfg`
B. `/boot/efi/linux/blackbird.efi`
C. `/etc/fstab`
D. `/var/log/audit/audit.log`

**Jawaban: B**

---

**88. UKI adalah singkatan dari...**
A. Unified Kernel Image
B. Universal Key Identity
C. User Kernel Interface
D. Unix Key Installer

**Jawaban: A**

---

**89. Perintah `bootctl --path=/boot install` digunakan untuk...**
A. Memasang systemd-boot
B. Menghapus bootloader
C. Membuat user
D. Mengaktifkan firewall

**Jawaban: A**

---

## Q. Soal Analisis Konseptual

**90. Mengapa `/tmp`, `/var/tmp`, `/var/log`, dan `/var/log/audit` diberi opsi `noexec`?**
A. Karena direktori tersebut tidak seharusnya menjadi tempat menjalankan program
B. Karena agar filesystem lebih cepat
C. Karena agar user bisa menjalankan binary bebas
D. Karena agar LUKS tidak aktif

**Jawaban: A**

---

**91. Mengapa SSH dipindahkan dari port default 22 ke 10022?**
A. Untuk menghapus SSH
B. Untuk mengurangi noise scanning otomatis pada port default, meski bukan pengganti autentikasi kuat
C. Agar password kosong bisa digunakan
D. Agar root login aktif

**Jawaban: B**

---

**92. Mengapa `PasswordAuthentication no` lebih aman jika key SSH sudah disiapkan?**
A. Karena mengurangi risiko brute force password
B. Karena menghapus user
C. Karena membuka Telnet
D. Karena mematikan firewall

**Jawaban: A**

---

**93. Mengapa `chattr +i` tidak boleh digunakan sembarangan pada semua file konfigurasi?**
A. Karena bisa menyulitkan update, perubahan konfigurasi, dan pemeliharaan sistem
B. Karena membuat file lebih mudah dihapus
C. Karena mematikan kernel
D. Karena menghapus AppArmor

**Jawaban: A**

---

**94. Mengapa update otomatis hourly perlu dipertimbangkan risikonya?**
A. Karena perubahan paket tanpa pengujian bisa merusak stabilitas sistem
B. Karena membuat RAM hilang
C. Karena menghapus filesystem
D. Karena mematikan keyboard

**Jawaban: A**

---

**95. Mengapa DNSOverTLS dan DNSSEC dipakai pada konfigurasi resolved?**
A. Untuk meningkatkan privasi dan validasi integritas resolusi DNS
B. Untuk membuka semua port
C. Untuk mengganti SSH
D. Untuk membuat swap

**Jawaban: A**

---

**96. Mengapa Bluetooth di desktop bisa diaktifkan tetapi pada server biasanya dinonaktifkan?**
A. Karena kebutuhan perangkat desktop berbeda dengan server
B. Karena Bluetooth wajib untuk boot
C. Karena server tidak punya kernel
D. Karena Bluetooth mengganti firewall

**Jawaban: A**

---

**97. Mengapa penggunaan `nodev` penting pada partisi user atau sementara?**
A. Untuk mencegah filesystem tersebut memperlakukan device special file sebagai perangkat valid
B. Untuk mempercepat download
C. Untuk mengaktifkan IPv6
D. Untuk menambah kapasitas RAM

**Jawaban: A**

---

**98. Mengapa `nosuid` penting pada `/home`, `/tmp`, dan `/var/tmp`?**
A. Untuk mencegah penyalahgunaan SUID/SGID dari area yang dapat ditulis user
B. Untuk mengaktifkan sudo tanpa password
C. Untuk membuka root login
D. Untuk menghapus audit log

**Jawaban: A**

---

**99. Mengapa audit log sebaiknya berada pada partisi terpisah?**
A. Agar log audit tidak terganggu oleh pertumbuhan log biasa atau data lain
B. Agar user biasa bisa mengedit audit log
C. Agar auditd tidak dipe
