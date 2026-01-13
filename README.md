# ERPNext PT Taharica

Development environment untuk ERPNext PT Taharica menggunakan Docker.

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

## 🚀 Quick Start - Development Setup

### Step 1: Clone Repository

```bash
git clone https://github.com/AqbilBarakaa/ERPNextPTTaharica.git
cd ERPNextPTTaharica
```

### Step 2: Start Docker Containers

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

### Step 3: Masuk ke Container

```bash
docker exec -it devcontainer-frappe-1 bash
```

### Step 4: Jalankan ERPNext

Di dalam container, jalankan:

```bash
cd /workspace/development/frappe-bench
bench start
```

### Step 5: Akses ERPNext

Buka browser dan akses:

- **URL:** http://development.localhost:8000
- **Username:** `Administrator`
- **Password:** `admin`

---

## 📋 Setup Baru (Jika frappe-bench belum ada)

Jika folder `frappe-bench` belum ada atau Anda ingin setup dari awal:

### Step 1: Masuk ke Container

```bash
docker exec -it devcontainer-frappe-1 bash
```

### Step 2: Inisialisasi Bench dengan Python 3.10

```bash
cd /workspace/development
PYENV_VERSION=3.10.13 bench init --skip-redis-config-generation --frappe-branch version-15 frappe-bench
```

### Step 3: Konfigurasi Database & Redis

```bash
cd frappe-bench
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

### Step 4: Buat Site Baru

```bash
bench new-site --db-root-password 123 --admin-password admin --mariadb-user-host-login-scope=% development.localhost
```

### Step 5: Aktifkan Developer Mode

```bash
bench --site development.localhost set-config developer_mode 1
bench --site development.localhost clear-cache
bench use development.localhost
```

### Step 6: Install ERPNext

```bash
bench get-app --branch version-15 --resolve-deps erpnext
bench --site development.localhost install-app erpnext
```

### Step 7: Jalankan Server

```bash
bench start
```

---

## 🔄 Perintah Sehari-hari

### Menjalankan ERPNext

```bash
# Dari Windows/Mac terminal:
cd ERPNextPTTaharica
docker compose -f .devcontainer/docker-compose.yml up -d
docker exec -it devcontainer-frappe-1 bash

# Di dalam container:
cd /workspace/development/frappe-bench
bench start
```

### Menghentikan Server

Tekan `Ctrl+C` di terminal yang menjalankan `bench start`

### Menghentikan Semua Container

```bash
docker compose -f .devcontainer/docker-compose.yml down
```

### Melihat Log Container

```bash
docker logs devcontainer-frappe-1
```

---

## ⚠️ Troubleshooting

### Container tidak bisa start

```bash
docker compose -f .devcontainer/docker-compose.yml down
docker compose -f .devcontainer/docker-compose.yml up -d
```

### Port 8000 sudah digunakan

Hentikan aplikasi lain yang menggunakan port 8000, atau ubah port di konfigurasi.

### Error "No module named 'frappe'"

Pastikan menggunakan Python 3.10:
```bash
PYENV_VERSION=3.10.13 bench init ...
```

### Database sudah ada

Gunakan flag `--force`:
```bash
bench new-site --force --db-root-password 123 --admin-password admin development.localhost
```

### Scheduler disabled

```bash
bench --site development.localhost enable-scheduler
bench start
```

---

## 📁 Struktur Folder

```
ERPNextPTTaharica/
├── .devcontainer/          # Konfigurasi Docker dev container
├── development/            # Development environment
│   ├── frappe-bench/       # Instalasi Frappe & ERPNext
│   │   ├── apps/           # Aplikasi (frappe, erpnext)
│   │   ├── sites/          # Site configurations
│   │   └── ...
│   └── .vscode/            # VS Code debug config
├── docs/                   # Dokumentasi
└── README.md               # File ini
```

---

## 📞 Informasi Kontak

Untuk pertanyaan teknis, hubungi tim development PT Taharica.

---

## License

This repository is licensed under the MIT License.
