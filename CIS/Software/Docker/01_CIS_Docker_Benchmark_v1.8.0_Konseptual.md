# CIS Docker Benchmark v1.8.0 — Dokumentasi Konseptual

> **Sumber utama:** CIS Docker Benchmark v1.8.0 — 07-24-2025  
> **Fokus:** pemahaman konsep hardening Docker, bukan sekadar menyalin command.  
> **Target pembaca:** admin sistem, admin aplikasi, security specialist, auditor, help desk, dan personel deployment platform yang mengelola Docker Engine pada platform Linux.

---

## 0. Gambaran Umum

CIS Docker Benchmark adalah panduan konfigurasi keamanan untuk membangun postur Docker yang lebih aman. Fokus utamanya bukan mengganti antivirus, EDR, vulnerability scanning, patch management, atau monitoring, tetapi menjadi **baseline konfigurasi teknis** agar Docker Engine, host, image, container runtime, dan Swarm tidak berjalan dengan konfigurasi yang terlalu permisif.

Dalam konteks Docker, risiko utamanya berbeda dari server biasa. Docker bukan hanya service biasa, tetapi **container runtime** yang dapat membuat proses, filesystem, network namespace, mount, volume, dan akses kernel tertentu. Salah konfigurasi Docker dapat membuat container memiliki pengaruh langsung terhadap host. Karena itu, hardening Docker harus dilihat sebagai kombinasi antara:

1. **Hardening host Linux**
2. **Hardening Docker daemon**
3. **Hardening file konfigurasi Docker**
4. **Hardening image dan Dockerfile**
5. **Hardening runtime container**
6. **Operasi keamanan Docker**
7. **Hardening Docker Swarm jika digunakan**

---

## 1. Cara Membaca CIS Docker Benchmark

### 1.1 Recommendation

Setiap butir rekomendasi CIS bukan sekadar daftar perintah, tetapi satu unit kontrol keamanan. Biasanya berisi:

- **Title**: judul kontrol.
- **Assessment Status**: apakah audit bisa otomatis atau perlu manual.
- **Profile**: Level 1, Level 2, atau Docker Swarm.
- **Description**: penjelasan konfigurasi.
- **Rationale**: alasan keamanan.
- **Impact**: dampak operasional.
- **Audit**: cara mengecek kondisi.
- **Remediation**: cara memperbaiki.
- **Default Value**: nilai bawaan.
- **References**: rujukan teknis.
- **CIS Controls mapping**: pemetaan ke CIS Controls.

### 1.2 Automated dan Manual

**Automated** berarti kondisi teknisnya dapat dicek dengan jelas menggunakan script atau tooling. Contohnya permission file `docker.socket`, ownership `/etc/docker`, atau keberadaan audit rule tertentu.

**Manual** berarti kontrol perlu penilaian manusia. Contohnya:

- Apakah user dalam group `docker` memang trusted.
- Apakah Swarm memang dibutuhkan.
- Apakah image berasal dari sumber tepercaya.
- Apakah port yang dibuka memang disetujui.
- Apakah penggunaan rootless mode sesuai kebutuhan aplikasi.

Manual bukan berarti kurang penting. Dalam Docker, banyak risiko terbesar justru bersifat arsitektural dan operasional, sehingga tidak selalu bisa dinilai hanya dengan pass/fail otomatis.

---

## 2. Profile Definitions

### 2.1 Level 1 — Docker — Linux

Level 1 adalah baseline praktis. Tujuannya:

- memberikan manfaat keamanan yang jelas;
- tidak terlalu mengganggu fungsi Docker;
- cocok sebagai titik awal untuk server Docker umum.

Contoh area Level 1:

- host Docker harus di-hardening;
- Docker harus up to date;
- hanya user trusted yang boleh mengontrol Docker daemon;
- Docker socket harus dibatasi;
- container sebaiknya tidak berjalan privileged;
- container sebaiknya tidak berjalan sebagai root;
- image harus berasal dari sumber tepercaya.

### 2.2 Level 2 — Docker — Linux

Level 2 lebih ketat dan cocok untuk lingkungan dengan kebutuhan keamanan tinggi. Karakteristiknya:

- lebih defensif;
- bisa mengurangi kemudahan operasional;
- bisa berdampak pada kompatibilitas aplikasi;
- membutuhkan pengujian lebih serius.

Contoh area Level 2:

- user namespace remapping;
- custom seccomp profile;
- authorization plugin;
- native logging terpusat;
- pembatasan lebih ketat pada runtime container;
- audit tambahan untuk file Docker dan containerd.

### 2.3 Level 1 — Docker Swarm

Profile ini hanya relevan jika Docker Swarm digunakan. Jika Swarm tidak digunakan, rekomendasi Swarm dapat ditandai sebagai **not applicable**. Jika digunakan, aspek yang penting adalah:

- jumlah manager node;
- enkripsi overlay network;
- secret management;
- auto-lock;
- rotasi key dan certificate;
- pemisahan management plane dan data plane.

---

## 3. Bab 1 — Host Configuration

Bab ini membahas keamanan host Linux tempat Docker berjalan. Prinsip dasarnya: **container tidak boleh dianggap sebagai pengganti hardening host**. Jika host lemah, Docker juga lemah.

### 3.1 Separate Partition untuk `/var/lib/docker`

Docker menyimpan image, container layer, volume, dan metadata secara default di `/var/lib/docker`. Jika direktori ini bercampur dengan root filesystem, container yang banyak, image besar, atau log yang tidak terkendali dapat memenuhi disk host.

Konsep keamanannya:

- membatasi dampak storage exhaustion;
- memudahkan monitoring kapasitas;
- memisahkan data container dari sistem utama;
- mendukung strategi backup dan recovery yang lebih rapi.

Dalam lingkungan produksi, `/var/lib/docker` sebaiknya berada pada partisi atau logical volume terpisah.

### 3.2 Trusted Users untuk Docker Daemon

Akses ke Docker daemon hampir setara dengan akses root. User yang menjadi anggota group `docker` dapat menjalankan container dengan mount host filesystem, misalnya memetakan `/` host ke container. Dengan cara itu, user dapat memodifikasi filesystem host dan memperoleh privilege tinggi.

Karena itu, group `docker` bukan group biasa. Ia harus diperlakukan seperti group administrator.

Prinsipnya:

- hanya admin yang benar-benar dipercaya boleh masuk group `docker`;
- akses Docker socket harus dibatasi;
- jangan memberikan akses Docker kepada user umum;
- jangan menganggap Docker group sebagai akses “non-root”.

### 3.3 Audit Docker Files dan Directories

Docker daemon berjalan dengan privilege tinggi dan bergantung pada banyak file penting, misalnya:

- `/usr/bin/dockerd`
- `/run/containerd`
- `/var/lib/docker`
- `/etc/docker`
- `docker.service`
- `docker.socket`
- `docker.sock`
- `/etc/docker/daemon.json`
- `/etc/containerd/config.toml`
- `/usr/bin/containerd`
- `/usr/bin/runc`

Audit penting karena perubahan pada file-file tersebut dapat mengubah perilaku daemon, runtime, socket, container storage, atau policy keamanan. Dalam konteks forensik, audit membantu menjawab:

- siapa yang mengubah konfigurasi Docker;
- kapan Docker daemon dimodifikasi;
- apakah socket atau runtime binary pernah disentuh;
- apakah konfigurasi containerd berubah;
- apakah ada indikasi persistence atau privilege abuse.

### 3.4 Host Harus Di-hardening

Docker host tetap Linux host. Maka hardening umum tetap dibutuhkan:

- patch OS;
- firewall;
- SSH hardening;
- auditd;
- logging;
- AppArmor/SELinux;
- permission file penting;
- user/group management;
- filesystem hardening.

Docker tidak boleh ditempatkan di host yang tidak aman. Container isolation bukan batas keamanan absolut.

### 3.5 Docker Version Harus Up to Date

Docker Engine memiliki rilis yang memperbaiki bug, vulnerability, dan perilaku runtime. Versi lama dapat menyimpan celah keamanan yang sudah diketahui. Namun update juga harus mempertimbangkan kompatibilitas aplikasi.

Prinsipnya:

- monitor rilis Docker;
- lakukan patching sesuai kebijakan organisasi;
- uji update di staging;
- jangan menunda patch keamanan kritis;
- dokumentasikan jika ada aplikasi yang harus memakai versi lama.

---

## 4. Bab 2 — Docker Daemon Configuration

Bab ini sangat penting karena konfigurasi Docker daemon memengaruhi **semua container** yang dibuat oleh daemon tersebut.

### 4.1 Rootless Mode

Rootless mode menjalankan Docker daemon dan container dalam user namespace tanpa privilege root penuh di host. Secara konsep, ini mengurangi dampak jika daemon atau runtime dieksploitasi.

Kelebihan:

- mengurangi risiko root-level compromise;
- memperkuat isolation;
- cocok untuk beberapa workload development atau non-privileged.

Keterbatasan:

- konfigurasi file berubah lokasi;
- ada batasan networking;
- ada batasan privileged ports;
- tidak semua workload cocok.

Rootless mode bukan solusi universal, tetapi sangat kuat untuk skenario yang kompatibel.

### 4.2 Inter-Container Communication

Secara default, container pada default bridge dapat berkomunikasi. Ini berisiko jika container tidak seharusnya saling melihat traffic atau service.

Konsepnya mirip segmentasi jaringan:

- container hanya boleh bicara dengan container yang memang diperlukan;
- jangan memakai default bridge untuk semua workload;
- buat custom network per aplikasi;
- matikan komunikasi bebas antar container pada default bridge jika tidak dibutuhkan.

### 4.3 Log Level Docker Daemon

Log level `info` dianggap baseline yang baik. Log terlalu rendah dapat menghilangkan informasi penting, sedangkan `debug` dapat menghasilkan terlalu banyak data dan berisiko mengekspos informasi sensitif.

Tujuannya:

- cukup detail untuk investigasi;
- tidak berlebihan;
- mendukung audit dan troubleshooting.

### 4.4 Docker dan iptables

Docker membuat aturan iptables/nftables untuk networking container. CIS menyarankan Docker tetap boleh mengelola aturan tersebut, karena jika dimatikan tanpa desain firewall yang matang, konektivitas container bisa kacau.

Namun ini tidak berarti firewall boleh diabaikan. Admin tetap harus memahami bahwa Docker dapat mengubah jalur network dan expose port.

### 4.5 Insecure Registries

Registry tanpa TLS atau dengan certificate tidak valid berisiko terhadap interception dan tampering. Jika image ditarik dari registry tidak aman, attacker dapat mengganti image atau layer.

Prinsipnya:

- gunakan registry dengan TLS;
- gunakan CA certificate yang dipercaya;
- hindari `insecure-registries`;
- validasi image dan artifact.

### 4.6 Storage Driver

CIS menyoroti agar tidak menggunakan storage driver lama atau deprecated seperti:

- `aufs`
- `devicemapper`

Driver yang deprecated berisiko karena:

- tidak lagi disarankan;
- bisa tidak kompatibel dengan versi baru;
- memiliki riwayat stabilitas atau maintenance yang buruk;
- mempersulit upgrade.

Pada banyak platform modern, `overlay2` menjadi pilihan default yang umum.

### 4.7 TLS untuk Remote Docker Daemon

Secara default Docker daemon memakai Unix socket lokal. Jika daemon diekspos lewat TCP tanpa TLS, siapa pun yang bisa mengakses port itu berpotensi mengontrol Docker daemon dan host.

Prinsipnya:

- jangan expose Docker daemon ke jaringan jika tidak perlu;
- jika wajib remote API, gunakan TLS mutual authentication;
- lindungi CA, certificate, dan private key;
- batasi IP yang boleh mengakses.

### 4.8 Default ulimit

`ulimit` mengatur batas resource proses. Tanpa batas yang tepat, container atau proses di dalamnya dapat menghabiskan resource host.

Contoh risiko:

- fork bomb;
- terlalu banyak file descriptor;
- exhaustion terhadap process table;
- DoS lokal.

### 4.9 User Namespace Remapping

User namespace remapping membuat user root di dalam container dipetakan menjadi UID non-root di host. Ini penting karena root di container tidak boleh otomatis bermakna root di host.

Konsepnya:

- root container ≠ root host;
- mengurangi dampak escape;
- membantu defense in depth;
- perlu pengujian karena tidak semua fitur kompatibel.

### 4.10 Cgroup

Cgroup membatasi resource CPU, memory, process, dan lain-lain. Docker memakai cgroup untuk mengontrol resource container.

Hardening cgroup berarti memastikan container tidak memakai resource host secara tidak terkendali.

### 4.11 Authorization Plugin

Secara default, Docker authorization bersifat “all or nothing”. Jika user boleh mengakses Docker daemon, ia dapat menjalankan banyak perintah. Authorization plugin menambah kontrol granular.

Konsepnya:

- membatasi command Docker berdasarkan policy;
- mendukung least privilege;
- cocok untuk environment multi-admin atau shared platform;
- butuh desain dan testing.

### 4.12 Centralized Logging

Log container dan daemon sebaiknya tidak hanya tersimpan lokal. Centralized logging membantu:

- investigasi insiden;
- korelasi antar host;
- mendeteksi anomali;
- mencegah kehilangan log saat host rusak.

### 4.13 No New Privileges

`no-new-privileges` mencegah proses di container memperoleh privilege tambahan melalui mekanisme seperti SUID/SGID. Ini penting untuk menurunkan risiko escalation di dalam container.

### 4.14 Live Restore

`live-restore` menjaga container tetap berjalan ketika Docker daemon restart. Dari sisi security, ini membantu availability dan memudahkan patching daemon tanpa downtime aplikasi.

### 4.15 Userland Proxy

Userland proxy dapat menambah attack surface. Jika hairpin NAT tersedia, proxy ini sering tidak diperlukan. Menonaktifkannya dapat mengurangi komponen tambahan yang berjalan.

### 4.16 Seccomp

Seccomp membatasi system call yang bisa digunakan proses. Karena container tetap berbagi kernel host, membatasi syscall adalah lapisan penting untuk mengurangi kernel attack surface.

Docker memiliki default seccomp profile. Custom profile bisa digunakan jika organisasi tahu syscall yang dibutuhkan aplikasi, tetapi salah konfigurasi dapat merusak aplikasi.

### 4.17 Experimental Features

Fitur experimental tidak seharusnya aktif di production karena:

- API belum stabil;
- fitur belum matang;
- belum tentu diuji penuh;
- dapat menambah risiko operasional.

---

## 5. Bab 3 — Docker Daemon Configuration Files

Bab ini membahas ownership dan permission file penting Docker. Prinsip utamanya adalah: **file yang mengontrol daemon harus hanya dapat dimodifikasi oleh root atau pihak yang benar-benar berwenang.**

### 5.1 Systemd Unit dan Socket

File seperti `docker.service` dan `docker.socket` menentukan bagaimana Docker daemon berjalan. Jika file ini dapat dimodifikasi user biasa, attacker dapat mengubah parameter daemon, membuka remote API, mematikan kontrol keamanan, atau memuat konfigurasi berbahaya.

Baseline umum:

- owner: `root:root`
- permission: `644` atau lebih ketat

### 5.2 `/etc/docker`

Direktori `/etc/docker` dapat berisi:

- `daemon.json`;
- certificate;
- registry trust;
- TLS key;
- konfigurasi daemon.

Karena isinya sensitif, owner harus `root:root`, dan permission direktori harus tidak writable oleh user biasa.

### 5.3 Registry Certificate dan TLS Key

Certificate publik dapat dibaca lebih longgar, tetapi private key harus sangat ketat. Private key Docker server yang bocor dapat dipakai untuk impersonation atau akses tidak sah.

Konsepnya:

- CA/server certificate: read-only;
- private key: hanya root yang boleh membaca;
- jangan menyimpan key di lokasi world-readable;
- jangan commit certificate/key ke repository aplikasi.

### 5.4 Docker Socket

`/var/run/docker.sock` adalah salah satu objek paling sensitif di host Docker. Siapa pun yang bisa write ke socket ini pada dasarnya dapat mengontrol Docker daemon.

Baseline umum:

- owner: `root:docker`
- permission: `660`
- group `docker` harus sangat terbatas.

### 5.5 `daemon.json`

`/etc/docker/daemon.json` menyimpan policy daemon. Jika dimodifikasi, attacker dapat:

- menonaktifkan `no-new-privileges`;
- mengaktifkan insecure registry;
- membuka remote API;
- mengubah logging;
- mengubah namespace/cgroup behavior.

File ini harus hanya dapat ditulis root.

---

## 6. Bab 4 — Container Images and Build File Configuration

Bab ini fokus pada keamanan sebelum container dijalankan: image, base image, Dockerfile, package, patch, dan secrets.

### 6.1 Non-root User di Container

Default container sering berjalan sebagai root. Root di container tidak selalu sama dengan root host, tetapi tetap berisiko, terutama jika ada mount, capability, atau runtime misconfiguration.

Prinsip:

- buat user non-root di Dockerfile;
- gunakan directive `USER`;
- pastikan permission aplikasi sesuai;
- hindari menjalankan service sebagai root jika tidak perlu.

### 6.2 Trusted Base Images

Base image adalah fondasi. Jika base image tidak tepercaya, semua layer di atasnya ikut berisiko.

Praktik konseptual:

- gunakan image resmi atau internal registry terpercaya;
- pin versi image;
- scan image;
- hindari image anonim dari internet;
- dokumentasikan sumber image.

### 6.3 Minimize Packages

Semakin banyak package dalam image, semakin besar attack surface. Image produksi sebaiknya hanya berisi runtime yang diperlukan.

Risiko package berlebihan:

- lebih banyak CVE;
- lebih banyak binary yang dapat disalahgunakan;
- image lebih besar;
- patching lebih rumit.

### 6.4 Image Scanning dan Rebuild

Image harus dipindai dan dibangun ulang ketika ada patch keamanan. Container bukan artefak statis yang boleh dipakai selamanya.

Siklus yang sehat:

1. build image;
2. scan image;
3. deploy;
4. monitor vulnerability;
5. rebuild dari base image terbaru;
6. redeploy.

### 6.5 Content Trust dan Signed Artifacts

Content trust memastikan image/artifact yang digunakan memiliki integritas dan berasal dari sumber yang dipercaya. Dalam supply chain security, tanda tangan digital membantu mencegah image palsu atau dimodifikasi.

### 6.6 HEALTHCHECK

`HEALTHCHECK` memberi sinyal apakah aplikasi di dalam container sehat. Ini bukan hanya fitur availability, tetapi juga membantu operasi keamanan karena container yang gagal dapat dideteksi lebih cepat.

### 6.7 Jangan Gunakan Update Instruction Sendirian

Dockerfile yang hanya menjalankan update tanpa install atau pinning dapat menghasilkan build yang tidak reproducible. Update sebaiknya dilakukan secara terkontrol dalam proses build dan patch lifecycle.

### 6.8 Hapus SUID/SGID

SUID/SGID binary dapat menjadi jalan privilege escalation. Dalam container minimal, SUID/SGID biasanya tidak diperlukan.

### 6.9 COPY vs ADD

`ADD` memiliki perilaku tambahan seperti ekstraksi archive dan dukungan URL. Jika hanya menyalin file lokal, `COPY` lebih eksplisit dan lebih aman.

### 6.10 Secrets Tidak Boleh Disimpan di Dockerfile

Secrets yang ditulis di Dockerfile akan masuk ke image layer dan dapat dibaca kembali melalui image history atau registry. Secret harus dikelola melalui secret manager, environment injection yang aman, atau runtime secret mechanism.

### 6.11 Verified Packages dan Signed Artifacts

Package yang diinstall dalam image harus berasal dari repository tepercaya dan diverifikasi. Artifact aplikasi juga sebaiknya memiliki checksum atau signature.

---

## 7. Bab 5 — Container Runtime Configuration

Runtime configuration menentukan bagaimana container benar-benar berjalan. Banyak kontrol keamanan Docker justru berada di command `docker run`, Compose, atau orchestrator.

### 7.1 Swarm Mode Tidak Aktif Jika Tidak Dibutuhkan

Swarm membuka port dan fitur cluster. Jika tidak digunakan, lebih aman tidak diaktifkan.

### 7.2 AppArmor atau SELinux

Mandatory Access Control membatasi apa yang dapat dilakukan proses container meskipun proses tersebut berhasil mengeksploitasi aplikasi. AppArmor dan SELinux adalah lapisan defense in depth.

### 7.3 Capabilities

Linux capabilities memecah privilege root menjadi kemampuan kecil. Container sebaiknya tidak diberi capability yang tidak perlu. Strategi yang baik adalah:

- `--cap-drop=ALL`;
- tambahkan hanya capability yang dibutuhkan;
- hindari capability berisiko seperti `SYS_ADMIN`.

### 7.4 Privileged Container

`--privileged` memberi container akses sangat luas ke host. Ini sering setara dengan melemahkan isolasi container. Gunakan hanya jika benar-benar disetujui dan terdokumentasi.

### 7.5 Host Directory Mount

Mount direktori sensitif seperti `/`, `/etc`, `/var/run`, `/proc`, atau `/sys` ke container dapat membuka akses langsung ke host. Volume harus dibatasi pada path yang benar-benar diperlukan.

### 7.6 SSH di Dalam Container

Container idealnya menjalankan satu proses aplikasi dan dikelola dari luar, bukan melalui SSH daemon di dalam container. Menjalankan `sshd` menambah attack surface dan menyulitkan audit.

### 7.7 Port Mapping

Port host di bawah 1024 adalah privileged ports. Mapping container langsung ke port privileged harus dikendalikan dan didokumentasikan. Gunakan reverse proxy atau load balancer jika lebih sesuai.

### 7.8 Host Network, PID, IPC, UTS, dan User Namespace

Container tidak boleh berbagi namespace host kecuali benar-benar perlu. Berbagi namespace memperlemah isolasi:

- `--network=host` membuka isolasi network;
- `--pid=host` memungkinkan melihat proses host;
- `--ipc=host` membuka IPC host;
- `--uts=host` berbagi hostname/domain;
- user namespace host mengurangi manfaat user remapping.

### 7.9 Resource Limit

Memory, CPU, ulimit, dan PIDs limit mencegah satu container menghabiskan resource host.

Kontrol ini penting untuk:

- mencegah DoS;
- menjaga stabilitas host;
- memisahkan resource antar aplikasi;
- membantu kapasitas operasional.

### 7.10 Read-only Root Filesystem

Root filesystem container sebaiknya read-only jika aplikasi mendukung. Temporary data dapat diarahkan ke `tmpfs` atau volume khusus.

Manfaat:

- mengurangi persistence attacker;
- mencegah perubahan file aplikasi;
- memaksa desain aplikasi yang lebih bersih.

### 7.11 Bind Traffic ke Interface Tertentu

Jika container expose service, bind ke interface tertentu. Jangan otomatis expose ke semua interface jika tidak diperlukan.

### 7.12 Restart Policy

Restart policy `on-failure:5` membatasi restart loop. Restart tanpa batas dapat menyembunyikan crash berulang atau menyebabkan resource churn.

### 7.13 Jangan Mount Docker Socket ke Container

Mount `/var/run/docker.sock` ke container memberi container kemampuan mengontrol Docker host. Ini adalah salah satu anti-pattern paling berbahaya.

### 7.14 Hindari Default Bridge `docker0`

Default bridge tidak ideal untuk segmentasi. Custom network memberi kontrol lebih baik terhadap isolasi aplikasi.

---

## 8. Bab 6 — Docker Security Operations

### 8.1 Image Sprawl

Image sprawl terjadi ketika terlalu banyak image lama, tidak dipakai, atau tidak diketahui sumbernya tertinggal di host/registry.

Risiko:

- image rentan tetap tersedia;
- kebingungan versi;
- storage penuh;
- deployment memakai image salah.

### 8.2 Container Sprawl

Container sprawl terjadi ketika banyak container stopped, orphaned, atau tidak terdokumentasi. Ini mengganggu audit dan bisa menyimpan konfigurasi atau data sensitif.

Prinsip operasional:

- inventory container;
- hapus yang tidak dipakai;
- gunakan label;
- dokumentasikan owner aplikasi;
- kontrol lifecycle container.

---

## 9. Bab 7 — Docker Swarm Configuration

Swarm hanya relevan jika fitur cluster Docker digunakan.

### 9.1 Minimum Manager Nodes

Manager node memegang state cluster. Jumlah manager harus cukup untuk availability, tetapi tidak terlalu banyak sehingga memperluas attack surface.

### 9.2 Bind Swarm Services ke Interface Tertentu

Service Swarm sebaiknya tidak listen di semua interface tanpa alasan. Bind ke interface yang memang dipakai.

### 9.3 Encrypted Overlay Network

Overlay network Swarm secara default tidak selalu mengenkripsi traffic antar container. Enkripsi mengurangi risiko sniffing antar node.

### 9.4 Docker Secrets

Secrets dalam Swarm harus dikelola dengan mekanisme `docker secret`, bukan environment variable biasa atau file di image.

### 9.5 Auto-lock

Auto-lock melindungi manager key saat daemon restart. Tanpa auto-lock, key material dapat lebih mudah dipakai ketika host compromise.

### 9.6 Rotasi Unlock Key, Node Certificate, dan CA Certificate

Rotasi key dan certificate penting untuk membatasi dampak kebocoran credential jangka panjang.

### 9.7 Pisahkan Management Plane dan Data Plane

Traffic manajemen cluster sebaiknya tidak bercampur dengan traffic aplikasi. Pemisahan ini mengurangi risiko lateral movement dan sniffing.

---

## 10. Pola Risiko Umum Docker

| Risiko | Penyebab | Dampak |
|---|---|---|
| Docker group terlalu luas | User biasa masuk group `docker` | Privilege escalation ke host |
| Docker socket dimount ke container | Container diberi akses `/var/run/docker.sock` | Container dapat mengontrol host |
| Container privileged | `--privileged` | Isolasi melemah drastis |
| Image tidak tepercaya | Pull dari registry publik tanpa validasi | Supply chain compromise |
| Secrets di Dockerfile | Secret masuk layer image | Secret bocor di registry/history |
| Tidak ada resource limit | Memory/CPU/PID tidak dibatasi | DoS host |
| Default bridge dipakai semua aplikasi | Segmentasi lemah | Lateral movement antar container |
| No audit | Perubahan Docker tidak tercatat | Sulit forensik |
| Insecure registry | Tidak memakai TLS | Image tampering |
| Swarm aktif tanpa kebutuhan | Port cluster terbuka | Attack surface bertambah |

---

## 11. Prioritas Belajar

Untuk memahami Docker hardening, urutan belajar yang disarankan:

1. Pahami bahwa akses Docker daemon ≈ akses root.
2. Pahami hubungan Docker host, Docker daemon, image, dan container.
3. Pahami namespace, cgroup, capability, seccomp, AppArmor/SELinux.
4. Pahami Dockerfile security.
5. Pahami runtime flags.
6. Pahami audit dan logging.
7. Pahami Swarm hanya jika memang digunakan.

---

## 12. Kesimpulan

CIS Docker Benchmark menekankan bahwa keamanan Docker tidak cukup hanya dengan “install Docker lalu jalankan container”. Docker harus diamankan dari beberapa sisi sekaligus:

- host Linux harus aman;
- daemon harus dikonfigurasi hati-hati;
- socket dan file konfigurasi harus dilindungi;
- image harus dibangun dari sumber tepercaya;
- container harus dijalankan dengan privilege minimal;
- operasi container harus dikontrol;
- Swarm harus diamankan jika digunakan.

Inti konseptualnya adalah **least privilege, attack surface reduction, auditability, integrity, isolation, dan operational discipline**.
