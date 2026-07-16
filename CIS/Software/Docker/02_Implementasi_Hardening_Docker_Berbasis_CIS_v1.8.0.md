# Implementasi Hardening Docker Berbasis CIS Docker Benchmark v1.8.0

> **Sumber utama:** CIS Docker Benchmark v1.8.0 — 07-24-2025  
> **Fokus:** instalasi dan post-installation hardening Docker Engine pada Linux.  
> **Target contoh:** Ubuntu/Debian style server. Pada RHEL/CentOS/Alma/Rocky, sesuaikan package manager dan path tertentu seperti `/etc/sysconfig/docker`.

---

## 0. Peringatan Penting

Dokumentasi ini adalah panduan implementasi berbasis CIS. Jangan langsung menerapkan ke production tanpa pengujian.

Urutan aman:

1. Terapkan di lab.
2. Uji aplikasi container.
3. Terapkan ke staging.
4. Terapkan ke sebagian kecil production.
5. Monitor error, log, performa, dan connectivity.
6. Baru lakukan deployment luas.

Beberapa kontrol dapat memutus aplikasi, terutama:

- rootless mode;
- user namespace remapping;
- `icc=false`;
- custom seccomp;
- AppArmor/SELinux;
- read-only root filesystem;
- drop capabilities;
- larangan mount host path;
- larangan `docker.sock` di container;
- pembatasan resource.

---

## 1. Variabel Lingkungan

Sesuaikan nilai berikut sebelum praktik.

```bash
export ADMIN_USER="kautsar"
export DOCKER_DATA_ROOT="/var/lib/docker"
export DOCKER_AUDIT_RULES="/etc/audit/rules.d/99-docker.rules"
export DOCKER_DAEMON_JSON="/etc/docker/daemon.json"
```

Cek identitas dan OS:

```bash
id
cat /etc/os-release
uname -a
```

Gunakan root atau sudo:

```bash
sudo -v
```

---

## 2. Tahap Perencanaan

Sebelum instalasi Docker, tentukan:

| Item | Keputusan |
|---|---|
| Docker untuk production/lab | production / lab |
| Mode daemon | rootful hardened / rootless |
| Data root Docker | `/var/lib/docker` atau path lain |
| AppArmor/SELinux | AppArmor untuk Ubuntu/Debian, SELinux untuk RHEL family |
| Registry | Docker Hub / private registry / internal mirror |
| Remote API | sebaiknya tidak aktif |
| Swarm | aktif hanya jika benar-benar dibutuhkan |
| Logging | local json-file / syslog / fluentd / journald |
| Image scanner | sesuai tooling organisasi |
| Backup | volume, registry, compose file, secret, konfigurasi daemon |

---

## 3. Hardening Host Sebelum Docker

Docker host harus lebih dulu aman. Minimal:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y ca-certificates curl gnupg lsb-release auditd audispd-plugins apparmor apparmor-utils ufw jq
```

Aktifkan auditd dan AppArmor:

```bash
sudo systemctl enable --now auditd
sudo systemctl enable --now apparmor
sudo aa-status || true
```

Aktifkan firewall host dasar. Contoh ini hanya mengizinkan SSH. Sesuaikan port aplikasi:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose
```

---

## 4. Partisi Terpisah untuk `/var/lib/docker`

CIS menyarankan storage Docker terpisah agar image/container tidak memenuhi root filesystem.

Cek kondisi sekarang:

```bash
df -h
mountpoint -- /var/lib/docker || true
```

Jika memakai LVM, contoh konseptual:

```bash
sudo lvcreate -L 50G -n lv_docker vg0
sudo mkfs.ext4 /dev/vg0/lv_docker
sudo mkdir -p /var/lib/docker
```

Tambahkan ke `/etc/fstab`:

```bash
echo '/dev/vg0/lv_docker /var/lib/docker ext4 defaults,nodev,nosuid 0 2' | sudo tee -a /etc/fstab
sudo mount -a
mountpoint -- /var/lib/docker
```

Catatan:

- Jangan gunakan `noexec` secara sembarang pada `/var/lib/docker`, karena container storage dapat membutuhkan eksekusi file dalam layer.
- Jika Docker sudah pernah berjalan, lakukan migrasi data dengan maintenance window.

---

## 5. Instalasi Docker Engine

Gunakan repository resmi atau repository organisasi yang tepercaya. Contoh umum Ubuntu/Debian:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Lalu ikuti prosedur repository resmi organisasi/vendor. Setelah repository tersedia:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Aktifkan service:

```bash
sudo systemctl enable --now docker
sudo systemctl enable --now containerd
```

Validasi:

```bash
docker version
docker info
systemctl status docker --no-pager
```

---

## 6. Batasi User yang Mengontrol Docker Daemon

Cek anggota group `docker`:

```bash
getent group docker
```

Tambahkan hanya admin tepercaya:

```bash
sudo usermod -aG docker "$ADMIN_USER"
```

Hapus user tidak tepercaya dari group `docker`:

```bash
sudo gpasswd -d nama_user docker
```

Catatan penting:

> Anggota group `docker` harus diperlakukan seperti administrator/root, karena akses ke Docker daemon dapat dipakai untuk mengakses host.

---

## 7. Audit Rules untuk Docker

Buat file audit rules:

```bash
sudo tee "$DOCKER_AUDIT_RULES" >/dev/null <<'EOF'
# CIS Docker Benchmark - Docker audit rules

# Docker daemon
-w /usr/bin/dockerd -k docker

# Runtime/container storage and config
-w /run/containerd -k docker
-w /var/lib/docker -k docker
-w /etc/docker -k docker

# systemd unit/socket if present
-w /usr/lib/systemd/system/docker.service -k docker
-w /usr/lib/systemd/system/docker.socket -k docker
-w /lib/systemd/system/docker.service -k docker
-w /lib/systemd/system/docker.socket -k docker

# Docker and containerd sockets/config
-w /var/run/docker.sock -k docker
-w /run/containerd/containerd.sock -k docker
-w /etc/docker/daemon.json -k docker
-w /etc/containerd/config.toml -k docker

# Distro-specific daemon config
-w /etc/default/docker -k docker
-w /etc/sysconfig/docker -k docker

# Container runtime binaries
-w /usr/bin/containerd -k docker
-w /usr/bin/containerd-shim -k docker
-w /usr/bin/containerd-shim-runc-v1 -k docker
-w /usr/bin/containerd-shim-runc-v2 -k docker
-w /usr/bin/runc -k docker
EOF
```

Muat ulang rules:

```bash
sudo augenrules --load || sudo systemctl restart auditd
sudo auditctl -l | grep docker
```

Catatan:

- Beberapa file mungkin tidak ada pada distro tertentu. Itu dapat ditandai **not applicable**.
- Jika audit log terlalu ramai, lakukan tuning sesuai kebijakan logging.

---

## 8. Pilih Mode Docker Daemon

Ada dua pendekatan utama:

### Opsi A — Rootless Mode

Rootless lebih kuat dari sisi privilege reduction, tetapi tidak selalu cocok.

Audit:

```bash
ps -fe | grep '[d]ockerd'
docker info --format '{{ .SecurityOptions }}'
```

Implementasi rootless harus mengikuti dokumentasi resmi Docker untuk distro yang dipakai. Setelah rootless aktif, path konfigurasi dan service berbeda dari rootful mode.

Gunakan rootless jika:

- workload kompatibel;
- tidak butuh privileged ports;
- tidak butuh fitur kernel tertentu;
- lingkungan mendukung user namespace.

### Opsi B — Rootful Hardened Mode

Jika tetap memakai rootful mode, lanjutkan hardening daemon berikut.

---

## 9. Konfigurasi `/etc/docker/daemon.json`

Backup konfigurasi lama:

```bash
sudo mkdir -p /etc/docker
[ -f /etc/docker/daemon.json ] && sudo cp -a /etc/docker/daemon.json /etc/docker/daemon.json.bak.$(date +%F-%H%M%S)
```

Contoh baseline daemon rootful hardened:

```bash
sudo tee /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "icc": false,
  "log-level": "info",
  "iptables": true,
  "storage-driver": "overlay2",
  "userns-remap": "default",
  "no-new-privileges": true,
  "live-restore": true,
  "userland-proxy": false,
  "experimental": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5"
  },
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65535,
      "Soft": 65535
    },
    "nproc": {
      "Name": "nproc",
      "Hard": 4096,
      "Soft": 2048
    }
  }
}
EOF
```

Validasi JSON:

```bash
jq . /etc/docker/daemon.json
```

Restart Docker:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Validasi:

```bash
docker info
docker info --format 'SecurityOptions={{ .SecurityOptions }}'
docker info --format 'LoggingDriver={{ .LoggingDriver }}'
docker info --format 'LiveRestore={{ .LiveRestoreEnabled }}'
docker version --format 'Server experimental={{ .Server.Experimental }}'
```

Catatan penting:

- `userns-remap` dapat mengubah ownership storage dan mengganggu workload lama.
- `icc=false` hanya berlaku untuk default bridge; tetap gunakan custom network.
- Jika aplikasi rusak, rollback dari backup `daemon.json`.

---

## 10. Jangan Gunakan Insecure Registry

Audit:

```bash
docker info | sed -n '/Insecure Registries:/,/Live Restore Enabled:/p'
```

Pastikan tidak ada registry insecure selain loopback default seperti `127.0.0.0/8`.

Jika ada konfigurasi seperti ini, hapus kecuali ada exception terdokumentasi:

```json
{
  "insecure-registries": ["registry.example.local:5000"]
}
```

Gunakan registry dengan TLS dan CA valid.

---

## 11. TLS untuk Remote Docker API

Prinsip terbaik:

> Jangan expose Docker daemon lewat TCP kecuali benar-benar perlu.

Cek apakah daemon listen pada TCP:

```bash
ss -lntp | grep -E 'dockerd|2375|2376' || true
ps -ef | grep '[d]ockerd'
```

Jika remote API tidak diperlukan, pastikan tidak ada konfigurasi seperti:

```bash
-H tcp://0.0.0.0:2375
```

Jika remote API wajib, gunakan TLS mutual authentication, bukan TCP plaintext. Parameter yang harus ada:

```bash
--tlsverify
--tlscacert=/path/ca.pem
--tlscert=/path/server-cert.pem
--tlskey=/path/server-key.pem
```

Port umum TLS Docker adalah `2376`. Batasi firewall hanya dari host administrasi.

---

## 12. Permission dan Ownership File Docker

### 12.1 `docker.service`

Cari path:

```bash
systemctl show -p FragmentPath docker.service
```

Jika ada:

```bash
DOCKER_SERVICE_PATH="$(systemctl show -p FragmentPath --value docker.service)"
sudo chown root:root "$DOCKER_SERVICE_PATH"
sudo chmod 644 "$DOCKER_SERVICE_PATH"
stat -c '%U:%G %a %n' "$DOCKER_SERVICE_PATH"
```

### 12.2 `docker.socket`

```bash
DOCKER_SOCKET_UNIT="$(systemctl show -p FragmentPath --value docker.socket 2>/dev/null || true)"
if [ -n "$DOCKER_SOCKET_UNIT" ] && [ -e "$DOCKER_SOCKET_UNIT" ]; then
  sudo chown root:root "$DOCKER_SOCKET_UNIT"
  sudo chmod 644 "$DOCKER_SOCKET_UNIT"
  stat -c '%U:%G %a %n' "$DOCKER_SOCKET_UNIT"
fi
```

### 12.3 `/etc/docker`

```bash
sudo chown root:root /etc/docker
sudo chmod 755 /etc/docker
stat -c '%U:%G %a %n' /etc/docker
```

### 12.4 `/etc/docker/daemon.json`

```bash
if [ -f /etc/docker/daemon.json ]; then
  sudo chown root:root /etc/docker/daemon.json
  sudo chmod 644 /etc/docker/daemon.json
  stat -c '%U:%G %a %n' /etc/docker/daemon.json
fi
```

### 12.5 Docker Socket

```bash
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
stat -c '%U:%G %a %n' /var/run/docker.sock
```

### 12.6 Registry Certificate

Jika memakai `/etc/docker/certs.d`:

```bash
if [ -d /etc/docker/certs.d ]; then
  sudo find /etc/docker/certs.d -type f -exec chown root:root {} \;
  sudo find /etc/docker/certs.d -type f -exec chmod 0444 {} \;
  sudo find /etc/docker/certs.d -type f -exec stat -c '%U:%G %a %n' {} \;
fi
```

### 12.7 Private Key Docker TLS

Jika ada private key:

```bash
sudo chown root:root /path/to/server-key.pem
sudo chmod 400 /path/to/server-key.pem
```

---

## 13. Dockerfile Hardening

Contoh Dockerfile yang lebih aman:

```Dockerfile
FROM debian:stable-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /usr/sbin/nologin appuser

WORKDIR /app

COPY --chown=appuser:appuser ./app /app

RUN find / -perm /6000 -type f -exec chmod a-s {} + 2>/dev/null || true

USER appuser

HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD curl -f http://127.0.0.1:8080/health || exit 1

EXPOSE 8080

CMD ["./start.sh"]
```

Prinsip yang diterapkan:

- base image minimal;
- package seperlunya;
- cache package dibersihkan;
- user non-root;
- `COPY`, bukan `ADD`;
- tidak ada secret di Dockerfile;
- `HEALTHCHECK`;
- SUID/SGID dikurangi.

---

## 14. Jangan Simpan Secrets di Dockerfile

Jangan lakukan:

```Dockerfile
ENV DB_PASSWORD=rahasia
RUN echo "token-api" > /root/token.txt
```

Alternatif:

- Docker secrets jika Swarm;
- secret manager organisasi;
- mounted secret file dengan permission ketat;
- environment variable dari CI/CD secret store;
- `.env` lokal yang tidak pernah masuk Git.

Cek image history:

```bash
docker history nama-image:tag
```

---

## 15. Build dan Scan Image

Build:

```bash
docker build -t appku:1.0.0 .
```

Cek user efektif:

```bash
docker run --rm appku:1.0.0 id
```

Scan image dengan tooling organisasi. Contoh umum:

```bash
trivy image appku:1.0.0
```

Jika organisasi memakai signing:

```bash
export DOCKER_CONTENT_TRUST=1
docker pull nama-image:tag
```

Catatan:

- Gunakan mekanisme signing yang disetujui organisasi.
- Validasi artifact, checksum, atau signature sebelum deploy.

---

## 16. Custom Network, Bukan Default `docker0`

Buat network per aplikasi:

```bash
docker network create --driver bridge app_net
docker network inspect app_net
```

Jalankan container pada custom network:

```bash
docker run -d \
  --name appku \
  --network app_net \
  appku:1.0.0
```

Audit container pada default bridge:

```bash
docker network inspect bridge --format '{{json .Containers}}' | jq .
```

---

## 17. Runtime Hardening dengan `docker run`

Contoh baseline aman:

```bash
docker run -d \
  --name appku \
  --network app_net \
  --user 1000:1000 \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m \
  --cap-drop ALL \
  --security-opt no-new-privileges:true \
  --security-opt apparmor=docker-default \
  --pids-limit 256 \
  --memory 512m \
  --cpus 1.0 \
  --restart on-failure:5 \
  --health-cmd "curl -f http://127.0.0.1:8080/health || exit 1" \
  --health-interval 30s \
  --health-timeout 5s \
  --health-retries 3 \
  -p 127.0.0.1:8080:8080 \
  appku:1.0.0
```

Jika aplikasi butuh bind port 80/443, lebih baik gunakan reverse proxy di host atau load balancer. Jika tetap harus map privileged port, dokumentasikan exception.

---

## 18. Runtime yang Harus Dihindari

Hindari:

```bash
docker run --privileged ...
docker run -v /:/host ...
docker run -v /var/run/docker.sock:/var/run/docker.sock ...
docker run --network host ...
docker run --pid host ...
docker run --ipc host ...
docker run --uts host ...
docker exec --privileged ...
docker exec -u root ...
```

Audit container berjalan:

```bash
docker ps --quiet | xargs -r docker inspect --format '
Name={{.Name}}
Privileged={{.HostConfig.Privileged}}
NetworkMode={{.HostConfig.NetworkMode}}
PidMode={{.HostConfig.PidMode}}
IpcMode={{.HostConfig.IpcMode}}
ReadonlyRootfs={{.HostConfig.ReadonlyRootfs}}
CapAdd={{.HostConfig.CapAdd}}
CapDrop={{.HostConfig.CapDrop}}
SecurityOpt={{.HostConfig.SecurityOpt}}
Binds={{.HostConfig.Binds}}
'
```

Cek container yang mount Docker socket:

```bash
docker ps --quiet | xargs -r docker inspect --format '{{.Name}} {{.HostConfig.Binds}}' | grep docker.sock || true
```

---

## 19. Docker Compose Hardening

Contoh `compose.yaml`:

```yaml
services:
  app:
    image: appku:1.0.0
    container_name: appku
    user: "1000:1000"
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,nodev,size=64m
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
      - apparmor=docker-default
    pids_limit: 256
    mem_limit: 512m
    cpus: 1.0
    restart: "on-failure:5"
    ports:
      - "127.0.0.1:8080:8080"
    networks:
      - app_net
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://127.0.0.1:8080/health || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3

networks:
  app_net:
    driver: bridge
```

Jalankan:

```bash
docker compose up -d
docker compose ps
```

---

## 20. Resource Limit

Audit resource container:

```bash
docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Memory={{.HostConfig.Memory}} NanoCPUs={{.HostConfig.NanoCpus}} PidsLimit={{.HostConfig.PidsLimit}}'
```

Contoh limit saat run:

```bash
docker run -d \
  --memory 512m \
  --memory-swap 512m \
  --cpus 1.0 \
  --pids-limit 256 \
  --ulimit nofile=1024:2048 \
  --name appku \
  appku:1.0.0
```

---

## 21. AppArmor / SELinux

### Ubuntu/Debian — AppArmor

Cek:

```bash
sudo aa-status
docker info --format '{{ .SecurityOptions }}'
```

Jalankan container dengan profile:

```bash
docker run --security-opt apparmor=docker-default appku:1.0.0
```

Untuk profile custom, buat dan uji di lab sebelum production.

### RHEL Family — SELinux

Cek:

```bash
sestatus
docker info --format '{{ .SecurityOptions }}'
```

Gunakan label SELinux sesuai kebijakan host. Jangan campur AppArmor dan SELinux secara sembarang pada satu host.

---

## 22. Seccomp

Cek default seccomp:

```bash
docker info --format '{{ .SecurityOptions }}'
```

Jangan disable seccomp:

```bash
# Hindari:
docker run --security-opt seccomp=unconfined ...
```

Jika memakai custom seccomp profile:

```bash
docker run --security-opt seccomp=/path/seccomp-profile.json appku:1.0.0
```

Uji aplikasi secara lengkap karena seccomp dapat memblokir syscall yang dibutuhkan.

---

## 23. Swarm Mode

Jika Swarm tidak dipakai:

```bash
docker info --format '{{ .Swarm }}'
docker swarm leave --force
```

Jika Swarm dipakai, lakukan hardening berikut.

### 23.1 Bind Swarm ke Interface Tertentu

```bash
docker swarm init --advertise-addr 10.10.10.10 --listen-addr 10.10.10.10:2377
```

### 23.2 Enkripsi Overlay Network

```bash
docker network create \
  --driver overlay \
  --opt encrypted \
  secure_overlay
```

Audit:

```bash
docker network ls --filter driver=overlay --quiet | xargs -r docker network inspect --format '{{.Name}} {{.Options}}'
```

### 23.3 Gunakan Docker Secrets

```bash
printf 'password-rahasia' | docker secret create db_password -
docker service create \
  --name appku \
  --secret db_password \
  --network secure_overlay \
  appku:1.0.0
```

### 23.4 Auto-lock Swarm Manager

```bash
docker swarm update --autolock=true
docker swarm unlock-key
```

Rotasi unlock key:

```bash
docker swarm unlock-key --rotate
```

### 23.5 Rotasi CA dan Node Certificate

```bash
docker swarm ca --rotate
docker swarm update --cert-expiry 720h
```

### 23.6 Pisahkan Management Plane dan Data Plane

Prinsip implementasi:

- gunakan interface/VLAN/subnet berbeda;
- batasi port manajemen Swarm;
- firewall hanya mengizinkan node yang sah;
- jangan expose management API ke internet.

---

## 24. Logging Container

Default `json-file` boleh dipakai untuk lab, tetapi production sebaiknya punya log rotation dan/atau centralized logging.

Cek logging driver:

```bash
docker info --format '{{ .LoggingDriver }}'
```

Contoh `json-file` dengan rotasi sudah dimasukkan di `daemon.json`:

```json
"log-driver": "json-file",
"log-opts": {
  "max-size": "10m",
  "max-file": "5"
}
```

Contoh syslog:

```json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "tcp://192.0.2.10:514"
  }
}
```

Pastikan remote logging aman, idealnya memakai TLS atau jaringan manajemen yang terlindungi.

---

## 25. Operasi Keamanan: Hindari Image dan Container Sprawl

### 25.1 Inventory

```bash
docker images
docker ps -a
docker volume ls
docker network ls
```

### 25.2 Hapus Resource Tidak Terpakai

Lakukan setelah verifikasi agar tidak menghapus data penting:

```bash
docker container prune
docker image prune
docker network prune
docker volume prune
```

Untuk cleanup lebih agresif:

```bash
docker system prune -a
```

Gunakan dengan hati-hati karena dapat menghapus image yang belum dipakai tetapi masih dibutuhkan.

### 25.3 Labeling

Gunakan label untuk owner dan environment:

```bash
docker run -d \
  --label owner="perpustakaan" \
  --label env="production" \
  --label app="himmah" \
  appku:1.0.0
```

Audit label:

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Labels}}'
```

---

## 26. Checklist Audit Cepat

### 26.1 Host dan Daemon

```bash
mountpoint -- "$(docker info -f '{{ .DockerRootDir }}')"
getent group docker
docker version
docker info --format 'SecurityOptions={{ .SecurityOptions }}'
docker info --format 'LoggingDriver={{ .LoggingDriver }}'
docker info --format 'LiveRestore={{ .LiveRestoreEnabled }}'
docker version --format 'Experimental={{ .Server.Experimental }}'
docker info | sed -n '/Insecure Registries:/,/Live Restore Enabled:/p'
```

### 26.2 Audit Rules

```bash
sudo auditctl -l | grep docker
```

### 26.3 File Permission

```bash
stat -c '%U:%G %a %n' /etc/docker
[ -f /etc/docker/daemon.json ] && stat -c '%U:%G %a %n' /etc/docker/daemon.json
stat -c '%U:%G %a %n' /var/run/docker.sock
systemctl show -p FragmentPath docker.service
systemctl show -p FragmentPath docker.socket
```

### 26.4 Image

```bash
docker images
docker history nama-image:tag
docker inspect nama-image:tag
```

### 26.5 Runtime

```bash
docker ps
docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Privileged={{.HostConfig.Privileged}} Readonly={{.HostConfig.ReadonlyRootfs}} Network={{.HostConfig.NetworkMode}} Pids={{.HostConfig.PidsLimit}} Binds={{.HostConfig.Binds}}'
```

### 26.6 Swarm

```bash
docker info --format '{{ .Swarm }}'
docker network ls --filter driver=overlay
docker node ls
docker secret ls
```

---

## 27. Template Exception Register

Gunakan jika ada kontrol CIS yang tidak bisa diterapkan.

| Field | Isi |
|---|---|
| ID rekomendasi | contoh: 5.8 |
| Kontrol | Privileged ports tidak boleh dimap |
| Status | Exception |
| Alasan bisnis | Aplikasi perlu bind port 443 langsung |
| Risiko | Container terekspos pada privileged port |
| Mitigasi | Reverse proxy, firewall, TLS, monitoring |
| Owner | Nama admin/aplikasi |
| Tanggal review | YYYY-MM-DD |
| Target eliminasi | YYYY-MM-DD atau accepted risk |

---

## 28. Urutan Implementasi yang Disarankan

1. Hardening host Linux.
2. Buat partisi `/var/lib/docker`.
3. Install Docker dari sumber tepercaya.
4. Batasi group `docker`.
5. Aktifkan auditd dan audit rules Docker.
6. Konfigurasi `daemon.json`.
7. Kunci permission file Docker.
8. Bangun image aman.
9. Jalankan container dengan runtime flags aman.
10. Terapkan logging dan monitoring.
11. Bersihkan image/container sprawl.
12. Audit berkala.
13. Dokumentasikan exception.

---

## 29. Kesimpulan Implementasi

Implementasi CIS Docker tidak cukup dengan satu command. Hardening harus dilakukan berlapis:

- host aman;
- Docker daemon aman;
- socket dan konfigurasi dilindungi;
- image dibangun secara aman;
- runtime container dibatasi;
- log dan audit aktif;
- operasi harian terkendali;
- Swarm diamankan jika digunakan.

Kontrol paling prioritas untuk praktik awal:

1. Batasi group `docker`.
2. Jangan mount Docker socket ke container.
3. Jangan gunakan `--privileged`.
4. Jalankan container sebagai non-root.
5. Batasi capabilities.
6. Gunakan custom network.
7. Aktifkan audit rules.
8. Lindungi `/etc/docker`, `daemon.json`, dan `docker.sock`.
9. Scan dan rebuild image.
10. Dokumentasikan exception.
