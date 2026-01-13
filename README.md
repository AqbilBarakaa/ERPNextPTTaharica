# ERPNext PT Taharica

Development environment untuk ERPNext PT Taharica menggunakan Docker.

## Apa Ini?

Repository ini menangani containerization dari Frappe stack untuk PT Taharica, termasuk application server, database (MariaDB), Redis, dan supporting services. Repository ini menyediakan development environment untuk mengembangkan dan menjalankan ERPNext.

### Komponen Utama

| Folder | Deskripsi |
|--------|-----------|
| `docs/` | Dokumentasi lengkap |
| `devcontainer-example/` | Template konfigurasi Docker dev container |
| `development/` | Environment untuk development |
| `images/` | Dockerfiles untuk building Frappe images |
| `overrides/` | Docker Compose configurations untuk berbagai skenario |
| `resources/` | Helper scripts dan configuration templates |

---

## Prerequisites

Pastikan Anda sudah menginstall:

1. **Git** - [Download Git](https://git-scm.com/downloads)
2. **Docker Desktop** - [Download Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - Untuk Windows: Pastikan WSL2 sudah terinstall
   - Alokasikan minimal **4GB RAM** untuk Docker

### Verifikasi Instalasi

```bash
git --version
docker --version
docker compose version
```

---

## Quick Demo (Opsional)

Jika Anda hanya ingin mencoba ERPNext dengan cepat tanpa development setup:

```bash
git clone https://github.com/AqbilBarakaa/ERPNextPTTaharica.git
cd ERPNextPTTaharica
docker compose -f pwd.yml up -d
```

Tunggu beberapa menit, lalu akses http://localhost:8080
- Username: `Administrator`
- Password: `admin`

> **Catatan:** Demo ini hanya untuk evaluasi cepat. Untuk development, ikuti langkah-langkah di bawah.

---

## Setup Development Environment

Ikuti langkah-langkah berikut secara berurutan untuk setup ERPNext dari awal.

### Step 1: Clone Repository

```bash
git clone https://github.com/AqbilBarakaa/ERPNextPTTaharica.git
cd ERPNextPTTaharica
```

### Step 2: Copy Konfigurasi Docker dan VSCode

Folder konfigurasi tidak ter-track di git, jadi perlu di-copy dari template:

**Windows (Command Prompt):**
```cmd
xcopy "devcontainer-example" ".devcontainer" /E /I /Y
xcopy "development\vscode-example" "development\.vscode" /E /I /Y
```

**Windows (PowerShell):**
```powershell
Copy-Item -Path "devcontainer-example" -Destination ".devcontainer" -Recurse -Force
Copy-Item -Path "development\vscode-example" -Destination "development\.vscode" -Recurse -Force
```

**Linux/Mac:**
```bash
cp -R devcontainer-example .devcontainer
cp -R development/vscode-example development/.vscode
```

### Step 3: Start Docker Containers

```bash
docker compose -f .devcontainer/docker-compose.yml up -d
```

Tunggu hingga semua container berjalan:
```bash
docker ps
```

Anda harus melihat 4 container berjalan:
- `devcontainer-frappe-1`
- `devcontainer-mariadb-1`
- `devcontainer-redis-cache-1`
- `devcontainer-redis-queue-1`

### Step 4: Masuk ke Container

```bash
docker exec -it devcontainer-frappe-1 bash
```

Setelah masuk, Anda akan melihat prompt seperti ini:
```
frappe@xxxxxxxx:/workspace/development$
```

### Step 5: Inisialisasi Bench dengan Python 3.10

Jalankan perintah berikut di dalam container:

```bash
cd /workspace/development
PYENV_VERSION=3.10.13 bench init --skip-redis-config-generation --frappe-branch version-15 frappe-bench
```

Tunggu proses ini selesai (bisa memakan waktu beberapa menit).

### Step 6: Konfigurasi Database dan Redis

```bash
cd frappe-bench
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

### Step 7: Buat Site Baru

```bash
bench new-site --db-root-password 123 --admin-password admin --mariadb-user-host-login-scope=% development.localhost
```

### Step 8: Aktifkan Developer Mode

```bash
bench --site development.localhost set-config developer_mode 1
bench --site development.localhost clear-cache
bench use development.localhost
```

### Step 9: Download dan Install ERPNext

```bash
bench get-app --branch version-15 --resolve-deps erpnext
bench --site development.localhost install-app erpnext
```

Proses ini memakan waktu cukup lama (5-15 menit).

### Step 10: Jalankan ERPNext

```bash
bench start
```

### Step 11: Akses ERPNext di Browser

Buka browser dan akses:

- **URL:** http://development.localhost:8000
- **Username:** `Administrator`
- **Password:** `admin`

Selamat! ERPNext sudah berjalan.

---

## Menjalankan ERPNext (Setelah Setup Selesai)

Setelah setup pertama kali selesai, Anda hanya perlu menjalankan langkah-langkah berikut untuk memulai ERPNext:

### Dari Windows/Mac Terminal:

```bash
cd ERPNextPTTaharica
docker compose -f .devcontainer/docker-compose.yml up -d
docker exec -it devcontainer-frappe-1 bash
```

### Di Dalam Container:

```bash
cd /workspace/development/frappe-bench
bench start
```

### Akses ERPNext:

- **URL:** http://development.localhost:8000
- **Username:** `Administrator`
- **Password:** `admin`

---

## Menghentikan ERPNext

### Menghentikan Server Bench

Tekan `Ctrl+C` di terminal yang menjalankan `bench start`

### Keluar dari Container

```bash
exit
```

### Menghentikan Semua Container

```bash
docker compose -f .devcontainer/docker-compose.yml down
```

---

## Troubleshooting

### Container tidak bisa start

```bash
docker compose -f .devcontainer/docker-compose.yml down
docker compose -f .devcontainer/docker-compose.yml up -d
```

### Port 8000 sudah digunakan

Hentikan aplikasi lain yang menggunakan port 8000, atau ubah port di konfigurasi.

### Error "No module named 'frappe'"

Pastikan menggunakan Python 3.10 saat inisialisasi bench:
```bash
PYENV_VERSION=3.10.13 bench init ...
```

### Database sudah ada

Gunakan flag `--force`:
```bash
bench new-site --force --db-root-password 123 --admin-password admin --mariadb-user-host-login-scope=% development.localhost
```

### Scheduler disabled / bench start langsung berhenti

```bash
bench --site development.localhost enable-scheduler
bench start
```

### frappe-bench folder tidak ditemukan di container

Restart container:
```bash
# Dari Windows terminal (bukan di dalam container)
docker compose -f .devcontainer/docker-compose.yml down
docker compose -f .devcontainer/docker-compose.yml up -d
```

---

## Struktur Folder

```
ERPNextPTTaharica/
├── .devcontainer/          # Konfigurasi Docker (copy dari devcontainer-example)
├── devcontainer-example/   # Template konfigurasi Docker
├── development/            # Development environment
│   ├── frappe-bench/       # Instalasi Frappe & ERPNext (dibuat saat setup)
│   │   ├── apps/           # Aplikasi (frappe, erpnext)
│   │   ├── sites/          # Site configurations
│   │   └── ...
│   ├── .vscode/            # VS Code config (copy dari vscode-example)
│   └── vscode-example/     # Template VS Code config
├── docs/                   # Dokumentasi tambahan
└── README.md               # File ini
```

---

## Informasi Default

| Item | Nilai |
|------|-------|
| Site Name | development.localhost |
| Admin Username | Administrator |
| Admin Password | admin |
| MariaDB Root Password | 123 |
| Frappe Version | 15 |
| ERPNext Version | 15 |
| Python Version | 3.10.13 |

---

## Dokumentasi Tambahan

Repository ini berbasis [frappe_docker](https://github.com/frappe/frappe_docker). Untuk dokumentasi lebih lanjut:

- [Getting Started Guide](docs/getting-started.md)
- [Development Documentation](docs/05-development/01-development.md)
- [Deployment Methods](docs/01-getting-started/01-choosing-a-deployment-method.md)
- [Container Setup Overview](docs/02-setup/01-overview.md)
- [Frequently Asked Questions](https://github.com/frappe/frappe_docker/wiki/Frequently-Asked-Questions)

---

## Resources

- [Frappe Framework](https://github.com/frappe/frappe)
- [ERPNext](https://github.com/frappe/erpnext)
- [Frappe Bench](https://github.com/frappe/bench)
- [Docker Documentation](http://docs.docker.com)

---

## License

This repository is licensed under the MIT License.
