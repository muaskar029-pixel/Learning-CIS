# Pembahasan Konseptual CIS Debian Linux 12 Benchmark

## B. Bab 1 — Initial Setup

**Sumber utama:** *CIS Debian Linux 12 Benchmark v2.0.0*  
**Ruang lingkup file ini:** Bab 1 — *Initial Setup*, khususnya bagian 1.1 sampai 1.7.  
**Catatan penting:** File ini berisi penjelasan konseptual, bukan panduan praktik menjalankan perintah. Tujuannya adalah membantu memahami mengapa rekomendasi CIS dibuat, apa risiko yang ingin dikurangi, dan bagaimana setiap subbagian berhubungan dengan hardening sistem Debian Linux 12.

---

## Gambaran Umum Bab 1 — Initial Setup

Bab **Initial Setup** dalam CIS Debian Linux 12 Benchmark membahas konfigurasi awal yang sebaiknya dipikirkan sejak tahap instalasi atau sebelum sistem digunakan secara serius. Disebut *initial setup* karena sebagian rekomendasi di bagian ini jauh lebih mudah diterapkan ketika sistem masih baru, belum berisi banyak aplikasi, belum menjadi server produksi, dan belum memiliki data pengguna yang besar.

Bagian ini mencakup hal-hal mendasar seperti filesystem, partisi, repository paket, Mandatory Access Control, bootloader, perlindungan proses, banner login, dan pengaturan login grafis GNOME Display Manager. Secara konseptual, Bab 1 ingin membangun fondasi keamanan sebelum sistem masuk ke konfigurasi layanan, jaringan, firewall, akses pengguna, logging, auditing, dan maintenance.

Dalam hardening, fondasi awal sangat penting. Sistem yang sudah telanjur berjalan lama tanpa perencanaan awal biasanya lebih sulit diamankan karena perubahan pada partisi, bootloader, module kernel, atau mekanisme akses bisa memengaruhi aplikasi dan pengguna. Karena itu, Bab 1 dapat dipahami sebagai tahap membentuk struktur dasar sistem agar lebih aman sejak awal.

---

# 1.1 Filesystem

Filesystem adalah lapisan yang mengatur bagaimana data disimpan, dibaca, ditulis, diberi izin akses, dan dikelola di media penyimpanan. Dalam Linux, filesystem bukan sekadar tempat menyimpan file, tetapi juga bagian dari model keamanan. Izin file, kepemilikan, special file, mount option, direktori sementara, direktori log, serta module filesystem semuanya dapat memengaruhi keamanan sistem.

Pada Bab 1.1, CIS membagi pembahasan filesystem menjadi dua area besar:

1. **Configure Filesystem Kernel Modules** — mengatur module filesystem di kernel agar filesystem yang tidak diperlukan tidak tersedia atau tidak dapat digunakan.
2. **Configure Filesystem Partitions** — mengatur pemisahan partisi dan mount option agar direktori tertentu memiliki batasan keamanan yang sesuai dengan fungsinya.

Secara sederhana, bagian filesystem menjawab dua pertanyaan besar:

- Apakah sistem benar-benar perlu mendukung semua jenis filesystem?
- Apakah semua direktori penting aman jika diletakkan dalam satu filesystem yang sama tanpa pembatasan khusus?

Jawaban CIS adalah: tidak semua filesystem perlu diaktifkan, dan tidak semua direktori sebaiknya dicampur dalam satu ruang filesystem tanpa kontrol tambahan.

---

# 1.1.1 Configure Filesystem Kernel Modules

## Konsep filesystem kernel module

Di Linux, dukungan terhadap jenis filesystem tertentu dapat disediakan oleh kernel secara langsung atau melalui **kernel module**. Kernel module adalah komponen yang dapat dimuat ke kernel untuk menambahkan kemampuan tertentu tanpa harus membangun ulang seluruh kernel. Dalam konteks filesystem, module memungkinkan sistem mengenali dan menggunakan format filesystem tertentu.

Contohnya, sistem Linux dapat mendukung filesystem native seperti ext4, xfs, atau btrfs, tetapi juga dapat mendukung filesystem lain yang dibuat untuk kebutuhan khusus, sistem operasi lain, media optik, perangkat flash, atau lingkungan container. Dukungan ini memang fleksibel, tetapi dari sudut pandang keamanan, setiap kemampuan tambahan juga berarti potensi permukaan serangan tambahan.

CIS memandang bahwa kemampuan yang tidak diperlukan sebaiknya tidak tersedia. Jika server Debian tidak pernah menggunakan filesystem tertentu, maka tidak ada alasan kuat untuk membiarkan module filesystem tersebut dapat dimuat. Ini sejalan dengan prinsip **least functionality**, yaitu sistem hanya menyediakan fungsi yang benar-benar dibutuhkan.

## Kenapa filesystem yang tidak diperlukan harus dinonaktifkan

Filesystem yang tidak diperlukan perlu dinonaktifkan karena setiap parser, driver, dan module filesystem adalah kode tambahan yang berjalan di level rendah. Jika terdapat kelemahan pada implementasi filesystem tersebut, penyerang dapat memanfaatkannya melalui file image, media eksternal, mount tertentu, atau mekanisme lain yang memicu pemrosesan filesystem tersebut.

Meskipun sebuah filesystem jarang digunakan, bukan berarti aman. Justru filesystem lama, niche, atau kurang populer sering kali memiliki risiko tambahan karena mungkin tidak mendapat perhatian pengujian dan pemeliharaan sebesar filesystem utama. Dari sudut pandang hardening, lebih aman menutup kemampuan yang tidak diperlukan daripada membiarkannya terbuka hanya karena “mungkin suatu saat dibutuhkan”.

Menonaktifkan filesystem yang tidak diperlukan juga membantu mengurangi kemungkinan penyalahgunaan oleh user lokal. Misalnya, jika user dapat membawa media tertentu, membuat image filesystem tertentu, atau memicu mount melalui mekanisme otomatis, dukungan filesystem yang tidak perlu dapat menjadi celah tambahan.

## Konsep attack surface lokal

**Attack surface** adalah seluruh titik yang dapat digunakan penyerang untuk menyerang sistem. Dalam konteks ini, **local attack surface** berarti permukaan serangan yang dapat dimanfaatkan dari sisi lokal, misalnya oleh user yang sudah memiliki akun, aplikasi yang berjalan di sistem, perangkat fisik yang ditancapkan, atau proses yang dapat memicu module tertentu.

Filesystem kernel module termasuk local attack surface karena:

- module berjalan dekat dengan kernel;
- kesalahan parsing filesystem dapat berdampak serius;
- beberapa module dapat dipicu melalui media eksternal atau file image;
- sebagian module mendukung fitur lama atau kebutuhan khusus yang tidak relevan untuk server modern;
- user lokal dapat mencoba mengeksploitasi fitur filesystem yang tidak dibatasi.

Dengan menonaktifkan module yang tidak diperlukan, sistem mengurangi jumlah kode kernel yang dapat dipakai atau dieksploitasi. Ini bukan berarti sistem menjadi kebal, tetapi risiko berkurang karena opsi serangan menjadi lebih sedikit.

## Perbedaan native dan non-native filesystem

**Native filesystem** adalah filesystem yang secara umum dirancang dan umum digunakan pada Linux, misalnya ext4, xfs, atau btrfs. Filesystem seperti ini biasanya lebih selaras dengan model permission, ownership, special file, symbolic link, dan mekanisme keamanan Linux.

**Non-native filesystem** adalah filesystem yang berasal dari sistem operasi lain, media khusus, atau kebutuhan tertentu. Contohnya hfs/hfsplus untuk Mac OS, freevxfs yang terkait dengan Veritas/HP-UX, atau udf untuk media optik. Non-native filesystem dapat membawa perilaku yang berbeda dari ekspektasi Linux. Perbedaan tersebut dapat menimbulkan konsekuensi keamanan atau fungsionalitas yang tidak selalu mudah diprediksi.

CIS menekankan bahwa filesystem non-native perlu digunakan dengan hati-hati. Alasannya bukan karena semua non-native filesystem pasti berbahaya, tetapi karena model keamanan dan pola penggunaannya bisa berbeda dari sistem Linux standar.

## Risiko filesystem lama, niche, atau jarang dipakai

Filesystem lama, niche, atau jarang dipakai memiliki beberapa risiko konseptual:

1. **Kurang diuji di lingkungan modern**  
   Filesystem yang jarang digunakan mungkin tidak diuji sebanyak filesystem utama. Bug yang tersembunyi bisa bertahan lebih lama.

2. **Kebutuhan bisnis tidak jelas**  
   Jika sistem tidak membutuhkan filesystem tersebut, dukungannya hanya menambah kompleksitas tanpa manfaat langsung.

3. **Potensi eksploitasi parser**  
   Kernel harus membaca struktur metadata filesystem. Jika implementasi parser bermasalah, file image yang dibuat secara jahat dapat memicu bug.

4. **Kompabilitas lintas sistem**  
   Beberapa filesystem dibuat untuk sistem operasi atau media tertentu. Perbedaan model permission dapat menghasilkan perilaku yang tidak sesuai dengan kebijakan keamanan Linux.

5. **Sulit dipantau**  
   Karena jarang digunakan, administrator mungkin tidak menyadari bahwa module tersebut tersedia atau termuat.

## Konsep blacklist module dan module availability secara konseptual

Dalam hardening kernel module, ada dua konsep penting:

- **Module availability** berarti apakah module tersedia di sistem dan dapat dimuat.
- **Blacklist module** berarti membuat konfigurasi agar module tidak dimuat secara otomatis atau tidak dapat digunakan dengan mudah.

Namun secara konseptual, blacklist saja tidak selalu sama dengan menghapus kemampuan secara total. Module bisa saja masih ada di disk, tetapi dicegah agar tidak dimuat. CIS biasanya melihat beberapa kondisi: apakah module tersedia, apakah sedang termuat, dan apakah module dapat dimuat lagi.

Hardening yang baik tidak hanya memastikan module “tidak sedang aktif”, tetapi juga memastikan module “tidak mudah diaktifkan kembali”. Inilah perbedaan antara sekadar mengecek kondisi saat ini dan membuat konfigurasi yang bertahan setelah reboot.

---

## Poin Filesystem Kernel Modules yang Dibahas CIS

### 1. cramfs

**cramfs** adalah compressed read-only filesystem yang biasanya digunakan pada sistem kecil atau embedded. Karena bersifat read-only dan terkompresi, cramfs cocok untuk image kecil, tetapi pada server Debian modern umumnya tidak diperlukan.

Secara keamanan, cramfs dinonaktifkan jika tidak digunakan karena dukungan terhadap filesystem tambahan berarti kernel perlu menyediakan kemampuan parsing tambahan. Jika server tidak memakai image cramfs, membiarkan module ini tersedia tidak memberi manfaat langsung.

CIS memasukkan cramfs dalam profil Level 1 untuk Server dan Workstation. Artinya, rekomendasi ini dianggap praktis, memiliki manfaat keamanan jelas, dan umumnya tidak mengganggu fungsi normal sistem.

### 2. freevxfs

**freevxfs** adalah implementasi bebas dari filesystem Veritas yang berkaitan dengan lingkungan HP-UX. Pada sistem Debian umum, filesystem ini sangat jarang dibutuhkan.

Risiko utamanya adalah keberadaan dukungan filesystem yang tidak relevan dengan kebutuhan sistem. Jika Debian tidak digunakan untuk membaca media atau volume freevxfs, module ini sebaiknya tidak tersedia. Prinsipnya tetap sama: fungsi yang tidak diperlukan sebaiknya dimatikan agar attack surface lebih kecil.

### 3. hfs

**hfs** adalah filesystem lama yang digunakan oleh Mac OS. Pada server Debian, terutama server produksi, kebutuhan untuk mount filesystem Mac OS biasanya sangat kecil.

Jika hfs tersedia, sistem memiliki kemampuan membaca struktur filesystem yang berasal dari platform lain. Hal ini tidak otomatis berbahaya, tetapi menambah kode dan perilaku yang tidak diperlukan. Karena itu, CIS merekomendasikan agar module hfs tidak tersedia jika tidak ada kebutuhan bisnis yang jelas.

### 4. hfsplus

**hfsplus** adalah penerus hfs yang juga berkaitan dengan filesystem Mac OS. Secara konseptual, alasannya mirip dengan hfs: jika Debian tidak digunakan untuk membaca atau mengelola media Mac OS, dukungan ini sebaiknya dinonaktifkan.

hfsplus termasuk non-native filesystem. Dengan menonaktifkannya, sistem mengurangi kemungkinan interaksi dengan media atau image yang membawa struktur filesystem dari luar ekosistem Linux.

### 5. jffs2

**jffs2** atau *Journaling Flash File System 2* adalah filesystem yang digunakan pada perangkat flash memory. Filesystem ini lebih dekat dengan kebutuhan embedded atau perangkat khusus, bukan server Debian umum.

Jika server tidak menggunakan jffs2, module ini sebaiknya tidak tersedia. Secara konseptual, jffs2 adalah contoh filesystem niche: berguna pada konteks tertentu, tetapi tidak perlu dibuka pada semua sistem.

### 6. overlay

**overlay** atau OverlayFS adalah filesystem union mount yang memungkinkan satu filesystem ditumpuk di atas filesystem lain sehingga terlihat sebagai satu tampilan terpadu. Overlay sangat penting dalam banyak teknologi container, misalnya Docker, Kubernetes, Podman, atau LXC.

CIS menempatkan overlay pada profil Level 2, bukan Level 1. Ini penting secara konseptual. Level 2 berarti rekomendasi ini lebih ketat dan bisa berdampak pada fungsi sistem. Jika overlay dinonaktifkan pada host yang menjalankan container, workload container dapat terganggu serius.

Dengan demikian, overlay adalah contoh bagus bahwa rekomendasi CIS bukan hukum mutlak. Untuk server yang memang menjalankan container, menonaktifkan overlay bisa bertentangan dengan fungsi utama server. Dalam kondisi seperti itu, pendekatan yang benar adalah membuat pengecualian terdokumentasi, bukan memaksa rekomendasi secara buta.

Alasan keamanan untuk membatasi overlay berkaitan dengan pengurangan attack surface dan riwayat kerentanan pada OverlayFS. Namun keputusan akhirnya harus mempertimbangkan peran sistem.

### 7. squashfs

**squashfs** adalah compressed read-only filesystem. Ia sering digunakan untuk image yang terkompresi dan juga digunakan oleh Snap packages.

CIS menempatkan squashfs pada Level 2. Artinya, rekomendasi ini lebih ketat dan dapat mengganggu beberapa penggunaan. Jika sistem menggunakan Snap, menonaktifkan squashfs dapat membuat aplikasi Snap gagal berjalan.

Secara konseptual, squashfs menunjukkan keseimbangan antara keamanan dan kompatibilitas. Dari sisi keamanan, module yang tidak diperlukan sebaiknya dimatikan. Namun dari sisi operasional, jika aplikasi bergantung pada squashfs, administrator harus menilai apakah rekomendasi ini cocok diterapkan.

### 8. udf

**udf** atau *Universal Disk Format* digunakan untuk media optik seperti DVD dan format disk tertentu. Dalam sistem server modern, kebutuhan untuk menulis atau membaca media optik sering kali tidak ada.

CIS menempatkan udf pada Level 2. Ada catatan penting bahwa lingkungan tertentu seperti Microsoft Azure dapat membutuhkan udf. Artinya, rekomendasi ini tidak boleh diterapkan tanpa memahami platform tempat Debian berjalan.

Secara konseptual, udf menegaskan bahwa konteks infrastruktur sangat penting. Debian yang berjalan di laptop, bare metal, virtual machine lokal, atau cloud provider tertentu bisa memiliki kebutuhan kernel module yang berbeda.

### 9. firewire-core

**firewire-core** berkaitan dengan IEEE 1394 atau FireWire, yaitu standar bus serial berkecepatan tinggi. Pada perangkat modern, FireWire jarang digunakan, terutama pada server.

Risiko keamanan FireWire berkaitan dengan eksploitasi implementasi atau penyalahgunaan akses perangkat fisik. Karena tidak banyak sistem modern membutuhkan FireWire, menonaktifkannya umumnya aman untuk server.

CIS menempatkan firewire-core pada Level 1 untuk Server dan Level 2 untuk Workstation. Perbedaan ini masuk akal: server biasanya tidak memerlukan FireWire, sedangkan workstation mungkin masih memiliki kebutuhan perangkat tertentu walaupun jarang.

### 10. usb-storage

**usb-storage** memungkinkan sistem menggunakan perangkat penyimpanan USB seperti flashdisk atau external drive. Fitur ini sangat berguna, tetapi juga membawa risiko keamanan besar.

USB storage dapat digunakan untuk:

- membawa malware;
- mengambil data tanpa izin;
- memindahkan file sensitif;
- memperkenalkan file atau executable dari luar lingkungan yang terkontrol;
- menjadi jalur awal infiltrasi jaringan.

CIS merekomendasikan pembatasan usb-storage terutama pada server. Untuk workstation, rekomendasi ini lebih ketat karena pengguna mungkin memang membutuhkan USB untuk pekerjaan tertentu. Jika organisasi masih mengizinkan USB storage, pendekatan yang lebih seimbang dapat berupa kontrol seperti USBGuard, kebijakan whitelist perangkat, enkripsi media, atau prosedur persetujuan.

### 11. unused filesystem kernel modules

Rekomendasi **unused filesystem kernel modules** bersifat Manual karena daftar filesystem yang tidak digunakan dapat berbeda antara satu organisasi dan organisasi lain. CIS tidak mungkin mencantumkan seluruh filesystem yang ada dan menentukan semuanya untuk semua lingkungan.

Secara konseptual, rekomendasi ini meminta organisasi melakukan review mandiri. Administrator perlu memahami kebutuhan sistem, aplikasi, media, cloud, container, backup, dan perangkat yang digunakan. Setelah itu, module filesystem yang tidak relevan dapat dikategorikan sebagai tidak diperlukan.

Manual di sini penting karena automated tool tidak selalu bisa mengetahui konteks bisnis. Tool bisa melihat apakah module ada atau termuat, tetapi tidak selalu tahu apakah module itu benar-benar dibutuhkan.

---

# 1.1.2 Configure Filesystem Partitions

## Konsep pemisahan partisi dalam hardening

Pemisahan partisi adalah praktik menempatkan direktori tertentu pada filesystem atau mount point yang terpisah. Tujuannya bukan hanya kerapian struktur disk, tetapi juga kontrol keamanan.

Dalam CIS, direktori seperti `/tmp`, `/dev/shm`, `/home`, `/var`, `/var/tmp`, `/var/log`, dan `/var/log/audit` mendapat perhatian khusus karena direktori-direktori tersebut memiliki pola penggunaan yang berbeda. Ada direktori yang world-writable, ada yang berisi data user, ada yang menampung log, ada yang menampung audit, dan ada yang dapat tumbuh sangat besar.

Jika semua direktori dicampur dalam root filesystem, maka satu area yang bermasalah dapat memengaruhi seluruh sistem. Misalnya, jika log tumbuh sampai penuh dan berada di root filesystem, sistem dapat gagal menulis file penting, service berhenti, atau proses login terganggu.

Pemisahan partisi membantu menerapkan prinsip containment: masalah pada satu area tidak langsung merusak area lain.

## Kenapa `/tmp`, `/var`, `/home`, `/var/log`, dan `/var/log/audit` dipisahkan

Direktori-direktori tersebut dipisahkan karena fungsinya berbeda:

- `/tmp` digunakan untuk file sementara dan biasanya dapat ditulis oleh banyak user atau aplikasi.
- `/dev/shm` digunakan untuk shared memory berbasis tmpfs dan dapat dipakai oleh proses untuk komunikasi sementara.
- `/home` menyimpan data pengguna dan file yang dibuat user.
- `/var` menyimpan data variabel seperti cache, spool, database tertentu, dan state aplikasi.
- `/var/tmp` menyimpan file sementara yang biasanya lebih persisten daripada `/tmp`.
- `/var/log` menyimpan log sistem dan aplikasi.
- `/var/log/audit` menyimpan audit log yang sangat penting untuk investigasi dan kepatuhan.

Masing-masing direktori memiliki risiko yang berbeda. Direktori sementara berisiko digunakan untuk menyimpan payload atau menjalankan file. Direktori user berisiko berisi file tidak tepercaya. Direktori log berisiko memenuhi disk. Direktori audit berisiko hilang atau tidak bisa ditulis saat terjadi insiden.

Dengan memisahkan partisi, administrator dapat memberikan batas ukuran, mount option, dan pengendalian yang sesuai untuk setiap direktori.

## Hubungan partisi dengan containment

**Containment** berarti membatasi dampak masalah agar tidak menyebar ke seluruh sistem. Dalam konteks filesystem, containment dicapai dengan memisahkan area yang berisiko.

Contoh konseptual:

- Jika `/tmp` penuh tetapi berada di partisi sendiri, root filesystem tidak ikut penuh.
- Jika user mengisi `/home` dengan file besar, direktori sistem seperti `/etc` atau `/usr` tidak langsung terdampak.
- Jika aplikasi menghasilkan log sangat banyak di `/var/log`, sistem tetap memiliki ruang untuk komponen inti lain.
- Jika audit log dipisah di `/var/log/audit`, audit trail lebih terlindungi dari gangguan log aplikasi biasa.

Containment juga membantu respons insiden. Ketika terjadi masalah, administrator dapat lebih mudah melihat area mana yang terdampak dan membatasi kerusakan.

## Konsep `nodev`, `nosuid`, dan `noexec`

Mount option adalah opsi yang menentukan bagaimana filesystem digunakan setelah dipasang. Tiga mount option penting dalam CIS adalah `nodev`, `nosuid`, dan `noexec`.

### `nodev`

`nodev` membuat sistem tidak memperlakukan device special file di filesystem tersebut sebagai perangkat valid. Ini penting untuk partisi yang tidak seharusnya berisi device file, seperti `/tmp`, `/home`, atau `/var/log`.

Secara konseptual, user tidak perlu membuat device file di direktori sementara atau direktori home untuk fungsi normal. Jika device file dapat dibuat dan dipakai, risiko penyalahgunaan meningkat.

### `nosuid`

`nosuid` membuat bit SUID dan SGID tidak berlaku pada filesystem tersebut. SUID/SGID dapat membuat file executable berjalan dengan privilege pemilik atau grup tertentu. Jika penyerang bisa menaruh atau memanfaatkan file SUID di area yang dapat ditulis, ini dapat menjadi jalur eskalasi privilege.

Direktori seperti `/tmp`, `/dev/shm`, `/home`, `/var/tmp`, `/var/log`, dan `/var/log/audit` umumnya tidak membutuhkan SUID/SGID untuk fungsi normal. Karena itu, `nosuid` membantu menutup risiko eskalasi privilege dari area tersebut.

### `noexec`

`noexec` mencegah eksekusi file langsung dari filesystem tersebut. Ini sangat berguna untuk direktori sementara dan log, karena direktori tersebut tidak seharusnya menjadi tempat menjalankan program.

Namun `noexec` bukan pengganti semua kontrol eksekusi. Beberapa interpreter masih bisa membaca file sebagai input. Meski begitu, `noexec` tetap berguna sebagai lapisan pertahanan tambahan karena mengurangi kemudahan menjalankan payload langsung dari direktori yang berisiko.

## Risiko jika direktori sementara atau log bercampur dengan root filesystem

Jika direktori sementara atau log bercampur dengan root filesystem, beberapa risiko muncul:

1. **Resource exhaustion**  
   File sementara atau log dapat memenuhi root filesystem. Jika root penuh, sistem bisa gagal boot, gagal login, gagal update, atau gagal menjalankan service.

2. **Penyalahgunaan area world-writable**  
   `/tmp` dan `/var/tmp` sering dapat ditulis oleh banyak user. Jika tidak dibatasi, area ini dapat digunakan untuk menyimpan payload, script, atau file berbahaya.

3. **Gangguan logging dan audit**  
   Jika log memenuhi disk, sistem mungkin berhenti mencatat kejadian penting. Ini merugikan investigasi insiden.

4. **Kesulitan pemulihan**  
   Jika semua data bercampur, membersihkan satu area berisiko mengganggu area lain.

5. **Tidak bisa menerapkan mount option spesifik**  
   Jika `/tmp`, `/home`, dan `/var/log` berada di filesystem yang sama dengan root, sulit memberi kebijakan berbeda sesuai fungsi masing-masing direktori.

---

## Subbagian Filesystem Partitions

### Configure `/tmp`

`/tmp` adalah direktori sementara yang digunakan oleh sistem dan aplikasi. Direktori ini biasanya bersifat world-writable, artinya banyak user dan proses dapat menulis ke sana. Karena itu, `/tmp` sangat sensitif dari sisi keamanan.

CIS mendorong agar `/tmp` menggunakan tmpfs atau partisi terpisah. Tujuannya adalah agar administrator dapat memberikan batasan seperti `nodev`, `nosuid`, dan `noexec`. Dengan begitu, `/tmp` tetap bisa digunakan untuk file sementara, tetapi tidak mudah dipakai untuk menjalankan payload atau menyalahgunakan special file.

Secara konsep, `/tmp` tidak boleh diperlakukan seperti direktori biasa. Ia adalah area transit, bukan tempat penyimpanan permanen dan bukan tempat eksekusi program.

### Configure `/dev/shm`

`/dev/shm` adalah area shared memory berbasis tmpfs. Direktori ini digunakan proses untuk berbagi data secara cepat melalui memori. Karena sifatnya sementara dan dapat digunakan oleh berbagai proses, `/dev/shm` juga perlu dibatasi.

Mount option seperti `nodev`, `nosuid`, dan `noexec` membantu memastikan bahwa shared memory tidak berubah menjadi area eksekusi atau penyalahgunaan privilege. Secara konseptual, `/dev/shm` diperlukan untuk komunikasi antarproses, bukan untuk menyimpan device file atau menjalankan program.

### Configure `/home`

`/home` berisi data pengguna. Pada workstation, direktori ini sangat penting karena menyimpan dokumen, konfigurasi aplikasi user, script, dan file pribadi. Pada server, `/home` mungkin lebih terbatas, tetapi tetap dapat berisi file user lokal.

Memisahkan `/home` memberikan beberapa manfaat:

- data user tidak memenuhi root filesystem;
- backup dan recovery lebih mudah;
- mount option seperti `nodev` dan `nosuid` dapat diterapkan;
- risiko dari file user dapat dibatasi agar tidak langsung memengaruhi area sistem.

CIS tidak memperlakukan `/home` persis sama dengan `/tmp`. Misalnya, tidak semua kebijakan `noexec` diterapkan secara umum pada `/home` karena beberapa lingkungan mungkin membutuhkan user menjalankan script atau program dari home directory. Namun `nodev` dan `nosuid` tetap relevan karena user normal tidak seharusnya membuat device file atau menjalankan file SUID dari home directory.

### Configure `/var`

`/var` menyimpan data yang berubah selama sistem berjalan. Di dalamnya bisa terdapat cache, spool, state aplikasi, database tertentu, log, mail queue, dan data runtime lainnya. Karena isi `/var` dapat tumbuh dinamis, memisahkannya dari root filesystem membantu mencegah data variabel memenuhi ruang sistem utama.

Secara keamanan, `/var` adalah area yang sering disentuh service. Jika aplikasi menulis data besar atau mengalami error yang menghasilkan file berlebihan, partisi `/var` dapat penuh. Jika `/var` menyatu dengan root filesystem, efeknya bisa lebih luas.

Mount option seperti `nodev` dan `nosuid` juga membantu karena `/var` tidak seharusnya menjadi tempat device file atau file SUID yang digunakan untuk operasi normal.

### Configure `/var/tmp`

`/var/tmp` mirip dengan `/tmp`, tetapi biasanya file di dalamnya diharapkan bertahan lebih lama, bahkan bisa bertahan setelah reboot. Karena itu, risikonya bisa lebih besar: file yang ditanam di `/var/tmp` dapat bertahan lebih lama daripada file di `/tmp`.

CIS mendorong pemisahan `/var/tmp` dan penerapan `nodev`, `nosuid`, serta `noexec`. Secara konseptual, `/var/tmp` adalah area sementara persisten, bukan tempat eksekusi program atau penyimpanan file privilege.

Perbedaan utama:

- `/tmp` untuk data sangat sementara dan bisa dibersihkan saat reboot.
- `/var/tmp` untuk data sementara yang mungkin perlu bertahan lebih lama.

Keduanya tetap perlu dibatasi karena keduanya dapat menjadi tempat penyerang menyimpan file.

### Configure `/var/log`

`/var/log` menyimpan log sistem dan aplikasi. Log penting untuk monitoring, troubleshooting, forensik, audit, dan deteksi insiden. Jika `/var/log` tidak dipisah, pertumbuhan log dapat memenuhi root filesystem.

Memisahkan `/var/log` membantu menjaga agar ledakan log tidak menghentikan sistem inti. Selain itu, mount option seperti `nodev`, `nosuid`, dan `noexec` memperjelas bahwa direktori log hanya untuk menyimpan catatan, bukan untuk menjalankan program atau menyimpan file khusus.

Secara konseptual, direktori log harus diperlakukan sebagai area bukti. Ia perlu tersedia, terlindungi, dan tidak mudah dimanipulasi.

### Configure `/var/log/audit`

`/var/log/audit` menyimpan audit log, yaitu catatan yang lebih spesifik untuk aktivitas keamanan dan perubahan penting pada sistem. Audit log lebih sensitif daripada log biasa karena sering digunakan untuk kepatuhan, investigasi insiden, dan pembuktian aktivitas.

Memisahkan `/var/log/audit` membantu memastikan audit log tidak terganggu oleh log aplikasi biasa. Jika `/var/log` penuh karena aplikasi menulis log berlebihan, audit log tetap memiliki ruang sendiri jika dipisahkan. Sebaliknya, jika audit log tumbuh besar, ia tidak langsung memenuhi seluruh `/var/log` atau root filesystem.

Mount option seperti `nodev`, `nosuid`, dan `noexec` memperkuat posisi `/var/log/audit` sebagai area penyimpanan catatan keamanan, bukan area eksekusi.

---

# 1.2 Package Management

Package management adalah proses mengelola software yang dipasang, diperbarui, dan dihapus dari sistem. Pada Debian, package management sangat erat dengan APT, repository, metadata paket, signature, dan keyring.

Dalam keamanan sistem, package management adalah salah satu titik paling kritis. Jika repository tidak tepercaya, key salah, file konfigurasi repository bisa dimodifikasi, atau update tidak berjalan, maka sistem dapat menerima paket yang salah, paket berbahaya, atau tetap menjalankan software rentan.

CIS membagi bagian ini menjadi:

1. Configure Package Repositories
2. Configure Package Updates

---

# 1.2.1 Configure Package Repositories

## Konsep repository terpercaya

Repository adalah sumber paket software. Dalam Debian, repository menyediakan paket, metadata, dependency, dan update. Karena sistem mengambil software dari repository, maka repository menjadi bagian dari rantai kepercayaan.

Repository terpercaya berarti:

- sumbernya jelas;
- metadata paket diverifikasi;
- paket ditandatangani atau berasal dari sumber yang dapat divalidasi;
- key yang digunakan sesuai;
- konfigurasi repository tidak dapat diubah sembarang user;
- tidak ada repository tidak resmi yang dimasukkan tanpa penilaian risiko.

Jika repository tidak terpercaya, seluruh sistem dapat dikompromikan. Package manager biasanya berjalan dengan privilege tinggi, sehingga paket berbahaya yang dipasang dapat memiliki dampak sangat serius.

## Signed repository

Repository yang ditandatangani memungkinkan sistem memverifikasi bahwa metadata dan paket berasal dari sumber yang sah. Di Debian, mekanisme seperti `apt-secure` dan opsi `Signed-By` membantu memastikan repository diverifikasi menggunakan key tertentu.

Secara konseptual, `Signed-By` penting karena membatasi key mana yang dipercaya untuk repository tertentu. Tanpa pembatasan, sistem bisa terlalu luas mempercayai key yang ada di keyring global. Dengan pembatasan key per repository, dampak kompromi satu key atau kesalahan konfigurasi dapat dikurangi.

Signed repository bukan hanya formalitas. Ia adalah mekanisme untuk mencegah package tampering, repository spoofing, dan distribusi paket palsu.

## GPG key

GPG key digunakan untuk memverifikasi tanda tangan repository atau metadata paket. Key ini adalah bagian dari rantai kepercayaan. Jika key salah, bocor, diganti, atau dapat dimodifikasi oleh pihak tidak berwenang, maka verifikasi paket menjadi tidak berarti.

Karena itu, CIS menaruh perhatian pada akses terhadap file key seperti keyring di `/usr/share/keyrings`, konfigurasi di `trusted.gpg.d`, dan file terkait repository. File-file ini harus memiliki permission yang membatasi perubahan hanya kepada pihak yang berwenang.

Konsep pentingnya adalah: bukan hanya paket yang harus tepercaya, tetapi juga file konfigurasi dan key yang menentukan siapa yang dipercaya.

## Risiko package tampering

**Package tampering** adalah kondisi ketika paket, metadata, atau jalur distribusi paket dimodifikasi oleh pihak tidak sah. Risikonya sangat besar karena paket yang dipasang dapat menjalankan script instalasi dengan privilege tinggi.

Dampak package tampering dapat berupa:

- instalasi backdoor;
- penggantian binary sistem;
- pencurian kredensial;
- perubahan konfigurasi keamanan;
- penonaktifan logging atau auditing;
- penyebaran malware ke sistem lain.

Mekanisme signature, keyring, permission file repository, dan kebijakan update semuanya bertujuan menjaga rantai distribusi software tetap tepercaya.

## Kenapa akses file konfigurasi APT harus dibatasi

File konfigurasi APT menentukan dari mana sistem mengambil software dan key apa yang dipercaya. Jika user biasa atau proses tidak tepercaya dapat mengubah file tersebut, penyerang dapat mengarahkan sistem ke repository berbahaya.

File yang perlu dipahami secara konseptual meliputi:

- `/etc/apt/sources.list`
- `/etc/apt/sources.list.d/`
- file `.sources`
- file keyring GPG
- `/etc/apt/trusted.gpg.d/`
- `/usr/share/keyrings/`
- `/etc/apt/auth.conf.d/`

`auth.conf.d` juga sensitif karena dapat menyimpan informasi autentikasi untuk repository privat. Jika aksesnya terlalu longgar, kredensial repository dapat bocor.

Secara hardening, repository bukan hanya daftar URL. Ia adalah pintu masuk software ke sistem. Karena itu, kontrol akses terhadap file repository harus ketat.

## Weak dependencies secara konseptual

Beberapa package manager dapat memasang paket tambahan yang sifatnya direkomendasikan atau tidak wajib. Dalam konteks hardening, pemasangan paket tambahan yang tidak benar-benar diperlukan dapat memperluas attack surface.

Konsepnya sederhana: semakin banyak paket, semakin banyak binary, service, library, dependency, dan potensi vulnerability. Karena itu, kebijakan package management sebaiknya mendorong pemasangan komponen yang memang diperlukan, bukan semua komponen tambahan secara otomatis tanpa pertimbangan.

---

# 1.2.2 Configure Package Updates

## Pentingnya update, patch, dan security software

Update dan patch adalah bagian utama dari cyber hygiene. Sistem yang tidak diperbarui akan menyimpan vulnerability yang sudah diketahui publik. Setelah vendor merilis patch, informasi tentang kelemahan yang diperbaiki sering kali juga menjadi lebih mudah diketahui. Penyerang dapat menggunakan informasi tersebut untuk menyerang sistem yang belum diperbarui.

CIS menekankan bahwa organisasi perlu memiliki proses patch management. Detail prosesnya dapat berbeda-beda: ada organisasi yang memakai update server internal, ada yang langsung memakai repository distribusi, ada yang update otomatis, dan ada yang update manual setelah pengujian.

Yang penting bukan hanya “pernah update”, tetapi ada proses yang jelas:

- bagaimana update dipantau;
- kapan patch diuji;
- kapan patch diterapkan;
- siapa yang bertanggung jawab;
- bagaimana rollback dilakukan jika patch bermasalah;
- bagaimana sistem kritis diprioritaskan.

## Hubungan patching dengan vulnerability management

Vulnerability management adalah proses mengidentifikasi, menilai, memprioritaskan, dan memperbaiki kelemahan keamanan. Patching adalah salah satu bentuk remediation dalam vulnerability management.

Hubungannya dapat dipahami sebagai berikut:

- vulnerability scan menemukan kelemahan;
- vendor merilis patch;
- organisasi menilai dampak patch;
- patch diuji pada lingkungan non-produksi;
- patch diterapkan ke sistem produksi;
- sistem diverifikasi ulang.

Tanpa patching, hardening lain bisa menjadi kurang efektif. Misalnya, firewall dan AppArmor membantu mengurangi risiko, tetapi software yang rentan tetap perlu diperbaiki. Sebaliknya, patching tanpa hardening juga belum cukup karena konfigurasi buruk tetap dapat membuka risiko.

---

# 1.3 Mandatory Access Control

## Apa itu Mandatory Access Control

**Mandatory Access Control** atau MAC adalah model kontrol akses yang memberi lapisan pembatasan tambahan di atas permission biasa. Dalam Linux tradisional, kontrol akses banyak bergantung pada **Discretionary Access Control** atau DAC, yaitu izin file berdasarkan owner, group, dan mode permission.

Pada DAC, jika user memiliki akses ke file, proses yang berjalan sebagai user tersebut biasanya juga memiliki akses. Masalahnya, jika aplikasi yang dijalankan user berhasil dieksploitasi, penyerang dapat menggunakan hak akses aplikasi tersebut untuk membaca atau mengubah file yang boleh diakses user.

MAC menambahkan kebijakan yang lebih ketat: meskipun DAC mengizinkan, tindakan tetap bisa ditolak jika kebijakan MAC tidak mengizinkan.

## Apa itu AppArmor

**AppArmor** adalah sistem Mandatory Access Control yang digunakan untuk membatasi kemampuan aplikasi berdasarkan profile. AppArmor menentukan file, direktori, kapabilitas, dan resource apa yang boleh diakses oleh sebuah program.

Ciri penting AppArmor adalah pendekatannya berbasis path. Berbeda dengan beberapa sistem MAC lain yang menggunakan label atau security context, AppArmor banyak bekerja dengan path file. Ini membuatnya relatif lebih mudah dipahami oleh administrator karena kebijakannya terlihat dekat dengan struktur direktori.

AppArmor membantu membatasi dampak jika aplikasi terkena exploit. Misalnya, jika sebuah service web diretas, AppArmor dapat membatasi file apa saja yang dapat dibaca atau ditulis oleh service tersebut.

## Perbedaan DAC dan MAC

### DAC — Discretionary Access Control

DAC adalah model permission standar Linux. Akses ditentukan oleh owner, group, dan permission seperti read, write, execute. Pemilik file dapat mengubah izin file miliknya.

Contoh konsep:

- user memiliki file;
- user menentukan siapa yang boleh membaca atau menulis;
- proses yang berjalan sebagai user tersebut mengikuti hak akses user.

Kelemahannya, jika proses milik user dikompromikan, proses tersebut mewarisi akses user.

### MAC — Mandatory Access Control

MAC adalah kebijakan wajib yang tidak sekadar bergantung pada keputusan pemilik file. Kebijakan ditentukan oleh sistem atau administrator keamanan. Aplikasi hanya boleh melakukan tindakan yang diizinkan oleh profile atau policy.

Konsep pentingnya: agar suatu akses diizinkan, DAC dan MAC harus sama-sama mengizinkan. Jika salah satu menolak, akses ditolak.

## Konsep profile enforcing

Dalam AppArmor, profile dapat berada dalam beberapa mode, tetapi mode paling penting untuk hardening adalah **enforcing**. Dalam mode enforcing, pelanggaran terhadap policy benar-benar diblokir.

Ada juga mode complain yang biasanya digunakan untuk pengujian atau pembuatan profile. Pada mode complain, pelanggaran dicatat tetapi tidak diblokir. Mode ini berguna saat tuning, tetapi belum memberikan perlindungan penuh.

CIS menekankan profile enforcing karena hardening membutuhkan pembatasan yang aktif, bukan sekadar pencatatan.

## Kenapa AppArmor penting untuk membatasi aplikasi

AppArmor penting karena banyak serangan modern tidak langsung menyerang seluruh sistem, tetapi mengeksploitasi satu aplikasi. Jika aplikasi tersebut tidak dibatasi, penyerang dapat bergerak lebih jauh sesuai privilege aplikasi.

Dengan AppArmor:

- aplikasi hanya dapat mengakses resource yang dibutuhkan;
- kerusakan akibat exploit dapat dibatasi;
- akses tidak perlu dikendalikan hanya dengan permission file;
- prinsip least privilege diterapkan pada level proses;
- sistem memiliki lapisan keamanan tambahan jika aplikasi rentan.

Namun AppArmor juga perlu dikonfigurasi dengan hati-hati. Profile yang terlalu ketat dapat mengganggu fungsi aplikasi, sedangkan profile yang terlalu longgar tidak memberi perlindungan optimal.

---

# 1.4 Configure Bootloader

## Kenapa bootloader perlu dilindungi

Bootloader adalah komponen yang memuat kernel dan memulai proses boot sistem operasi. Pada Debian, bootloader umum adalah GRUB. Karena bootloader berjalan sebelum sistem operasi sepenuhnya aktif, ia memiliki posisi yang sangat sensitif.

Jika bootloader tidak dilindungi, orang yang memiliki akses fisik atau console dapat mencoba mengubah parameter boot. Misalnya, seseorang dapat mencoba masuk ke mode single-user, mengubah parameter kernel, menonaktifkan kontrol keamanan tertentu, atau mem-boot sistem dengan konfigurasi yang tidak semestinya.

Hardening bootloader bertujuan mencegah perubahan tidak sah pada proses boot.

## Risiko akses fisik terhadap bootloader

Akses fisik adalah risiko serius. Banyak kontrol keamanan jaringan tidak relevan jika seseorang dapat berdiri di depan mesin, reboot, dan mengubah parameter boot. Karena itu, server fisik harus berada di lokasi aman, tetapi perlindungan fisik saja tidak cukup.

Risiko akses fisik terhadap bootloader meliputi:

- mengubah parameter kernel;
- masuk ke recovery mode;
- mencoba reset password root;
- menonaktifkan security module tertentu;
- mem-boot dari media eksternal;
- membaca atau memodifikasi disk jika tidak dienkripsi.

Bootloader password tidak menyelesaikan semua risiko fisik, tetapi menjadi salah satu lapisan perlindungan.

## Password bootloader

Password bootloader mencegah user tidak sah mengubah entri boot atau parameter kernel. Ini penting terutama untuk server, lab, atau workstation yang dapat diakses banyak orang.

Secara konseptual, password bootloader adalah kontrol terhadap perubahan boot-time. Ia tidak dimaksudkan menggantikan enkripsi disk, kontrol BIOS/UEFI, Secure Boot, atau keamanan ruang server. Ia adalah bagian dari pertahanan berlapis.

## Proteksi konfigurasi bootloader

Selain password, file konfigurasi bootloader juga harus dilindungi. Jika file konfigurasi dapat dibaca atau diubah oleh user tidak berwenang, penyerang dapat mempersiapkan perubahan boot tanpa perlu mengeditnya secara langsung saat boot.

Proteksi konfigurasi bootloader mencakup:

- kepemilikan file oleh root;
- permission yang ketat;
- mencegah user biasa membaca hash password atau konfigurasi sensitif;
- mencegah perubahan entri boot oleh proses tidak sah.

Intinya, keamanan bootloader bukan hanya saat menu boot tampil, tetapi juga saat file konfigurasinya tersimpan di sistem.

---

# 1.5 Configure Additional Process Hardening

## Konsep hardening proses

Process hardening adalah upaya membatasi cara proses berinteraksi dengan kernel, memori, file, dan proses lain. Tujuannya adalah mengurangi dampak exploit dan menutup teknik umum yang sering digunakan penyerang untuk eskalasi privilege atau pengumpulan informasi.

Bagian ini berisi kontrol kernel dan konfigurasi sistem seperti hardlink protection, symlink protection, ptrace restriction, core dump control, dmesg restriction, kernel pointer restriction, ASLR, dan systemd-coredump.

Secara konseptual, process hardening tidak selalu terlihat oleh user, tetapi sangat penting karena banyak serangan terjadi pada level proses dan memori.

## Perlindungan hardlink dan symlink

### Hardlink protection

Hardlink adalah nama tambahan yang menunjuk ke inode file yang sama. Dalam beberapa skenario, hardlink dapat disalahgunakan untuk menyerang file sensitif, terutama jika terkait dengan program SUID atau perubahan permission.

`fs.protected_hardlinks` membantu mencegah user membuat hardlink ke file yang tidak semestinya, terutama file yang bukan miliknya atau memiliki privilege tertentu. Ini mengurangi risiko serangan berbasis hardlink di direktori world-writable.

### Symlink protection

Symlink adalah file khusus yang menunjuk ke path lain. Symlink sering dipakai secara normal, tetapi juga dapat dimanfaatkan dalam serangan race condition. Misalnya, program yang menulis ke file sementara dapat diarahkan melalui symlink ke file sensitif.

`fs.protected_symlinks` membantu mencegah proses mengikuti symlink dalam kondisi yang berbahaya, terutama di direktori world-writable seperti `/tmp`.

Kedua kontrol ini sangat relevan dengan keamanan direktori sementara.

## ptrace restriction

`ptrace` adalah mekanisme yang memungkinkan satu proses mengamati atau mengontrol proses lain. Ini digunakan oleh debugger, profiler, dan tool diagnostik. Namun dari sisi keamanan, ptrace juga dapat disalahgunakan untuk membaca memori proses, mencuri rahasia, atau menyuntikkan instruksi.

`kernel.yama.ptrace_scope` membatasi kemampuan ptrace agar tidak sembarang proses dapat menelusuri proses lain. Ini penting karena banyak kredensial, token, atau data sensitif dapat berada di memori proses.

Konsepnya adalah membatasi debugging antarput proses hanya pada kondisi yang diperlukan dan sah.

## Core dump

Core dump adalah file yang berisi snapshot memori proses saat crash. File ini berguna untuk debugging, tetapi dapat mengandung informasi sensitif seperti password, token, session key, data user, atau isi file yang sedang diproses.

CIS mengatur beberapa aspek core dump:

- apakah program SUID boleh menghasilkan core dump;
- ukuran core file;
- bagaimana systemd-coredump menyimpan dump;
- apakah dump disimpan permanen atau tidak.

Dalam lingkungan produksi, core dump harus dikendalikan. Debugging memang penting, tetapi tidak boleh mengorbankan kerahasiaan data.

## dmesg restriction

`dmesg` menampilkan pesan kernel. Pesan kernel dapat berisi informasi tentang hardware, driver, alamat memori, error, module, dan detail sistem lainnya. Informasi ini berguna untuk administrator, tetapi juga berguna untuk penyerang yang ingin melakukan fingerprinting atau eksploitasi.

`kernel.dmesg_restrict` membatasi akses ke dmesg agar tidak dapat dibaca sembarang user. Ini mengurangi kebocoran informasi internal kernel.

## kernel pointer restriction

Kernel pointer adalah alamat memori kernel. Jika alamat ini bocor, penyerang dapat menggunakannya untuk melemahkan proteksi seperti ASLR atau menyusun exploit yang lebih akurat.

`kernel.kptr_restrict` membantu menyembunyikan atau membatasi tampilan alamat pointer kernel kepada user yang tidak berwenang. Ini adalah kontrol anti-information disclosure.

## ASLR

ASLR atau **Address Space Layout Randomization** adalah teknik yang mengacak lokasi memori proses. Tujuannya adalah membuat exploit lebih sulit karena penyerang tidak dapat dengan mudah memprediksi alamat fungsi, stack, heap, atau library.

`kernel.randomize_va_space` mengatur tingkat randomisasi address space. Dengan ASLR aktif, exploit memory corruption menjadi lebih sulit, walaupun tidak mustahil.

ASLR adalah contoh pertahanan probabilistik: ia tidak menghapus bug, tetapi membuat eksploitasi lebih sulit dan kurang andal.

## prelink

Prelink adalah mekanisme lama untuk mempercepat loading program dengan memodifikasi binary agar library lebih cepat dipetakan. Namun modifikasi binary dapat mengganggu mekanisme integritas file dan melemahkan beberapa manfaat randomisasi memori.

Karena keuntungan performanya tidak lagi sebanding dengan risikonya pada sistem modern, CIS mendorong agar prelink tidak digunakan.

## automatic error reporting

Automatic error reporting dapat mengirim atau menyimpan laporan crash. Laporan semacam ini bisa membantu troubleshooting, tetapi juga dapat berisi informasi sensitif tentang aplikasi, path, konfigurasi, environment, atau data yang sedang diproses.

Dalam hardening, error reporting harus dikendalikan. Sistem produksi tidak seharusnya mengirim detail crash tanpa kebijakan yang jelas.

## systemd-coredump

`systemd-coredump` menangani core dump pada sistem berbasis systemd. CIS memperhatikan pengaturan seperti ukuran maksimum dump dan apakah dump disimpan.

Konsepnya adalah membatasi risiko data sensitif tersimpan akibat crash. Jika core dump diperlukan untuk debugging, harus ada prosedur aman: akses terbatas, retensi jelas, dan pembersihan setelah analisis.

---

# 1.6 Configure Command Line Warning Banners

## Fungsi login banner

Login banner adalah pesan yang ditampilkan sebelum atau setelah proses login. Dalam keamanan, banner biasanya digunakan untuk:

- memberi pemberitahuan bahwa sistem hanya untuk pengguna berwenang;
- menyampaikan bahwa aktivitas dapat dimonitor;
- mendukung kebutuhan legal atau audit;
- menegaskan kebijakan akses organisasi.

Banner bukan kontrol teknis yang mencegah exploit, tetapi ia memiliki fungsi administratif, legal, dan kepatuhan. Dalam beberapa organisasi, banner menjadi bagian dari pembuktian bahwa pengguna telah diberi peringatan sebelum mengakses sistem.

## Perbedaan `/etc/motd`, `/etc/issue`, `/etc/issue.net`, dan SSH banner

### `/etc/motd`

`/etc/motd` adalah *message of the day*. Biasanya ditampilkan setelah user berhasil login. Karena muncul setelah login, ia lebih cocok untuk informasi internal, pengumuman, atau pengingat kebijakan.

Namun dalam hardening, isi motd tetap harus diperhatikan agar tidak membocorkan informasi sistem.

### `/etc/issue`

`/etc/issue` biasanya ditampilkan sebelum login pada console lokal. Karena muncul sebelum autentikasi, isinya harus sangat hati-hati. Jangan menampilkan detail yang membantu penyerang mengenali sistem.

### `/etc/issue.net`

`/etc/issue.net` digunakan untuk banner pada akses jaringan tertentu. Dalam konteks SSH, file ini dapat dipakai sebagai sumber banner jika dikonfigurasi pada SSH daemon.

Karena tampil sebelum autentikasi remote, isi `/etc/issue.net` harus berupa peringatan umum, bukan detail sistem.

### SSH warning banner

SSH warning banner adalah pesan yang ditampilkan kepada pengguna sebelum login SSH. Ini penting karena SSH adalah salah satu pintu akses administrasi paling umum pada server Linux.

Banner SSH sebaiknya berisi legal notice yang jelas, misalnya sistem hanya untuk pengguna berwenang dan aktivitas dapat dimonitor.

## Hubungan banner dengan legal notice dan audit

Banner mendukung legal notice karena menyatakan batasan penggunaan sistem. Dalam audit, banner dapat menunjukkan bahwa organisasi memiliki kebijakan akses dan memberi peringatan kepada pengguna.

Namun banner harus disusun oleh organisasi sesuai kebijakan internal dan kebutuhan hukum. Administrator teknis tidak sebaiknya membuat teks legal sembarangan tanpa koordinasi dengan pihak terkait.

## Risiko banner yang membocorkan informasi sistem

Banner tidak boleh membocorkan informasi seperti:

- nama distribusi dan versi detail;
- versi kernel;
- arsitektur sistem;
- hostname sensitif;
- nama aplikasi internal;
- detail environment produksi;
- informasi organisasi yang tidak perlu diketahui sebelum login.

Informasi tersebut dapat membantu fingerprinting. Penyerang yang mengetahui versi OS atau kernel dapat mencari exploit yang sesuai. Karena itu, CIS mendorong banner yang informatif secara kebijakan, tetapi tidak informatif secara teknis bagi penyerang.

## Proteksi file banner

Selain isi banner, file banner juga perlu permission yang tepat. Jika user tidak berwenang dapat mengubah banner, ia dapat:

- menghapus legal warning;
- menambahkan pesan menyesatkan;
- memasukkan informasi palsu;
- menggunakan banner untuk rekayasa sosial.

Karena itu, file seperti `/etc/motd`, `/etc/issue`, `/etc/issue.net`, file pam_motd, dan file banner SSH perlu dilindungi dari modifikasi tidak sah.

---

# 1.7 Configure GNOME Display Manager

## Hardening login GUI

GNOME Display Manager atau GDM adalah komponen login grafis pada sistem GNOME. Pada server minimal, GDM biasanya tidak dipasang. Namun pada workstation Debian atau server yang memiliki GUI, GDM menjadi titik awal autentikasi pengguna.

Hardening GDM penting karena login grafis dapat membocorkan informasi user, memudahkan akses fisik, atau menjalankan fitur desktop seperti automount dan autorun yang berisiko.

## Login banner di GDM

Seperti banner command line, banner login GDM memberi peringatan sebelum pengguna masuk. Tujuannya adalah memberi legal notice dan kebijakan akses pada antarmuka grafis.

Jika organisasi mewajibkan peringatan akses, maka banner harus konsisten di berbagai jalur login: console, SSH, dan GUI.

## Disable user list

Secara default, login manager kadang menampilkan daftar user. Ini memudahkan pengguna sah, tetapi juga membocorkan username kepada pihak yang melihat layar.

Menonaktifkan user list membantu mencegah username enumeration. Penyerang harus mengetahui username terlebih dahulu, bukan memilih dari daftar yang sudah disediakan.

Konsepnya sederhana: sistem login tidak perlu membantu penyerang dengan menampilkan daftar identitas pengguna.

## Screen lock

Screen lock mencegah akses tidak sah ketika user meninggalkan workstation dalam keadaan login. Ini sangat penting pada lingkungan kampus, kantor, lab, atau ruang publik.

Hardening screen lock mencakup:

- mengaktifkan lock otomatis;
- menentukan waktu idle yang wajar;
- mencegah user menonaktifkan lock jika kebijakan organisasi melarang;
- memastikan sesi terkunci saat tidak digunakan.

Screen lock adalah kontrol sederhana tetapi penting karena banyak insiden terjadi bukan melalui exploit teknis, melainkan karena sesi dibiarkan terbuka.

## Automount

Automount adalah fitur yang otomatis memasang media eksternal seperti USB drive ketika ditancapkan. Dari sisi kenyamanan, ini berguna. Dari sisi keamanan, ini berisiko karena media tidak tepercaya dapat langsung diproses oleh sistem.

Menonaktifkan automount mengurangi risiko:

- akses otomatis ke media berbahaya;
- eksekusi atau pemrosesan file dari perangkat asing;
- kebocoran data ke perangkat eksternal;
- serangan berbasis removable media.

## Autorun

Autorun adalah perilaku menjalankan sesuatu secara otomatis ketika media dimasukkan. Ini lebih berbahaya daripada automount karena dapat mengarah pada eksekusi otomatis.

CIS mendorong pengaturan agar autorun tidak terjadi. Secara konseptual, sistem tidak boleh menjalankan program hanya karena sebuah media eksternal dimasukkan.

## XDMCP

XDMCP adalah protokol lama untuk login grafis jarak jauh menggunakan X Display Manager. Dalam banyak lingkungan modern, XDMCP tidak diperlukan dan memiliki risiko keamanan karena membuka permukaan akses grafis remote.

Menonaktifkan XDMCP mengurangi risiko akses remote yang tidak diperlukan. Jika organisasi membutuhkan remote GUI, sebaiknya memakai mekanisme yang lebih aman dan terkontrol.

## Xwayland

Xwayland adalah lapisan kompatibilitas agar aplikasi X11 dapat berjalan di lingkungan Wayland. Kompatibilitas ini berguna, tetapi juga membawa model keamanan X11 yang secara historis lebih longgar dibanding Wayland.

Mengatur Xwayland berarti organisasi perlu menentukan apakah kompatibilitas X11 diperlukan. Jika tidak diperlukan, membatasinya dapat mengurangi risiko. Jika masih ada aplikasi yang bergantung pada Xwayland, maka perlu penilaian dampak sebelum menerapkan pembatasan.

---

# Ringkasan Konseptual Bab 1

Bab 1 CIS Debian Linux 12 Benchmark membangun dasar hardening sistem. Fokus utamanya bukan pada satu service tertentu, tetapi pada fondasi sistem operasi.

Inti pembahasannya dapat diringkas sebagai berikut:

1. **Filesystem kernel module**  
   Module filesystem yang tidak diperlukan sebaiknya tidak tersedia karena menambah local attack surface.

2. **Filesystem partitions**  
   Direktori dengan fungsi berbeda perlu dipisahkan agar resource exhaustion, penyalahgunaan direktori sementara, dan gangguan log dapat dibatasi.

3. **Mount options**  
   `nodev`, `nosuid`, dan `noexec` membantu membatasi penyalahgunaan filesystem berdasarkan fungsi direktori.

4. **Package management**  
   Repository, GPG key, dan update process adalah bagian dari rantai kepercayaan software. Jika rantai ini rusak, sistem dapat menerima paket berbahaya.

5. **AppArmor dan MAC**  
   AppArmor memberi pembatasan tambahan terhadap aplikasi sehingga exploit pada satu aplikasi tidak otomatis memberi akses luas.

6. **Bootloader**  
   Bootloader perlu dilindungi karena perubahan parameter boot dapat melewati atau melemahkan kontrol keamanan.

7. **Process hardening**  
   Kontrol kernel seperti hardlink protection, symlink protection, ptrace restriction, core dump control, ASLR, dan pembatasan informasi kernel membantu mengurangi dampak exploit.

8. **Warning banners**  
   Banner mendukung legal notice dan audit, tetapi tidak boleh membocorkan informasi sistem.

9. **GDM hardening**  
   Login grafis juga perlu diamankan melalui banner, penyembunyian daftar user, screen lock, pembatasan automount/autorun, XDMCP, dan Xwayland.

Secara keseluruhan, Bab 1 mengajarkan bahwa hardening bukan hanya mematikan service berbahaya. Hardening dimulai dari desain awal sistem: module apa yang tersedia, partisi bagaimana dibentuk, software dari mana diperoleh, proses bagaimana dibatasi, dan akses awal sistem bagaimana dikendalikan.

---

# Catatan Belajar

Untuk memahami Bab 1 secara matang, jangan menghafal setiap rekomendasi sebagai daftar perintah. Pahami pola pikirnya:

- Kurangi fitur yang tidak diperlukan.
- Pisahkan area yang berisiko.
- Batasi hak akses sesuai fungsi.
- Verifikasi sumber software.
- Terapkan pertahanan berlapis.
- Uji dampak sebelum menerapkan perubahan besar.
- Dokumentasikan pengecualian jika fungsi bisnis membutuhkan konfigurasi yang berbeda.

Dengan pola pikir ini, CIS Benchmark tidak hanya menjadi checklist, tetapi menjadi kerangka berpikir dalam membangun sistem Linux yang lebih aman.
