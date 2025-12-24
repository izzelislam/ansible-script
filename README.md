# Ansible Laravel Deployment

Ansible playbook dan roles untuk deploy aplikasi Laravel ke server VPS.

## 📁 Struktur Project

```
ansible/
├── ansible.cfg                 # Konfigurasi Ansible
├── inventory/
│   ├── hosts                   # Daftar server/host
│   └── host_vars/              # Variables per host
├── group_vars/
│   └── all.yml                 # Variables global
├── playbooks/
│   ├── deploy-laravel.yml      # Playbook untuk deploy Laravel
│   ├── php.yml                 # Setup PHP saja
│   ├── webserver.yml           # Setup Nginx saja
│   └── bootstrap.yml           # Setup awal server
├── roles/
│   ├── common/                 # Setup dasar server
│   ├── php/                    # Instalasi PHP + extensions
│   ├── nginx/                  # Instalasi & konfigurasi Nginx
│   └── laravel/                # Deploy aplikasi Laravel
└── site.yml                    # Master playbook
```

## 🚀 Quick Start

### 1. Konfigurasi Inventory

Edit file `inventory/hosts` dengan data server Anda:

```ini
[webserver]
server1 ansible_host=YOUR_IP ansible_user=YOUR_USER ansible_ssh_private_key_file=~/.ssh/your_key
```

### 2. Update Variables di Playbook

Edit `playbooks/deploy-laravel.yml`:

```yaml
vars:
  project_name: "nama_project_anda"
  domain: "domain-anda.com"
  repo_url: "git@github.com:username/repo.git"
  
  db_name: "nama_database"
  db_user: "user_database"
  db_pass: "password_database"
```

### 3. Jalankan Playbook

**Deploy pertama kali (first install):**
```bash
ansible-playbook playbooks/deploy-laravel.yml -e "first_install=true"
```

**Deploy update (regular deployment):**
```bash
ansible-playbook playbooks/deploy-laravel.yml
```

**Deploy dengan SSL:**
```bash
ansible-playbook playbooks/deploy-laravel.yml -e "use_ssl=true"
```

## 📋 Variables yang Tersedia

### Laravel Role Variables

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `project_name` | required | Nama project/folder |
| `domain` | required | Domain website |
| `repo_url` | required | Git repository URL |
| `git_branch` | `main` | Branch yang akan di-deploy |
| `deploy_user` | `www-data` | User untuk deploy |
| `php_version` | `8.3` | Versi PHP |
| `first_install` | `false` | Set `true` untuk deploy pertama |
| `enable_maintenance_mode` | `true` | Aktifkan maintenance mode saat deploy |
| `run_seeders` | `false` | Jalankan db:seed |
| `cache_config` | `true` | Cache Laravel config |
| `cache_routes` | `true` | Cache Laravel routes |
| `cache_views` | `true` | Cache Laravel views |

### Database Variables

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `db_name` | required | Nama database |
| `db_user` | required | Username database |
| `db_pass` | required | Password database |
| `db_host` | `127.0.0.1` | Database host |
| `db_port` | `3306` | Database port |

### Nginx Variables

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `use_ssl` | `false` | Aktifkan konfigurasi SSL |
| `ssl_certificate` | auto | Path ke SSL certificate |
| `ssl_certificate_key` | auto | Path ke SSL private key |
| `client_max_body_size` | `64M` | Maksimal upload size |
| `static_cache_days` | `30` | Caching untuk file statis |

### Environment Variables (dalam .env)

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `app_name` | `Laravel` | APP_NAME |
| `app_env` | `production` | APP_ENV |
| `app_debug` | `false` | APP_DEBUG |
| `app_timezone` | `Asia/Jakarta` | APP_TIMEZONE |
| `session_driver` | `database` | SESSION_DRIVER |
| `queue_connection` | `sync` | QUEUE_CONNECTION |
| `cache_driver` | `file` | CACHE_DRIVER |

## 🔧 Contoh Penggunaan

### Deploy Project Baru

```bash
# 1. Test koneksi ke server
ansible webserver -m ping

# 2. Deploy dengan first_install
ansible-playbook playbooks/deploy-laravel.yml \
  -e "first_install=true" \
  -e "project_name=myapp" \
  -e "domain=myapp.com" \
  -e "repo_url=git@github.com:user/myapp.git"
```

### Deploy Update

```bash
ansible-playbook playbooks/deploy-laravel.yml --tags laravel
```

### Hanya Setup PHP

```bash
ansible-playbook playbooks/deploy-laravel.yml --tags php
```

### Hanya Setup Nginx

```bash
ansible-playbook playbooks/deploy-laravel.yml --tags nginx
```

### Dengan Password Database dari Vault

```bash
# Buat vault file untuk password
ansible-vault create group_vars/vault.yml

# Jalankan dengan vault
ansible-playbook playbooks/deploy-laravel.yml --ask-vault-pass
```

## 📝 Workflow Deploy

1. **Maintenance Mode ON** - Laravel masuk mode maintenance
2. **Git Pull** - Tarik kode terbaru dari repository
3. **Update .env** - Push file environment dari template
4. **Composer Install** - Install dependencies
5. **Database Migration** - Jalankan migration
6. **Clear Cache** - Bersihkan semua cache Laravel
7. **Optimize Cache** - Cache config, routes, views
8. **Fix Permissions** - Set permissions storage & bootstrap/cache
9. **Restart Services** - Restart PHP-FPM & Nginx
10. **Maintenance Mode OFF** - Laravel aktif kembali

## ⚠️ Troubleshooting

### Permission Denied saat Git Clone

Pastikan deploy user memiliki akses ke repository:
```bash
# Generate SSH key di server
ssh-keygen -t ed25519 -C "deploy@server"

# Tambahkan public key ke GitHub Deploy Keys
cat ~/.ssh/id_ed25519.pub
```

### Composer Memory Error

Tambahkan swap atau tingkatkan memory limit:
```bash
# Di server
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### MySQL Authentication Error

Pastikan PyMySQL terinstall:
```yaml
# Sudah ditangani di role laravel/tasks/setup-db.yml
- name: Install PyMySQL
  apt:
    name: python3-pymysql
    state: present
```

## 📄 License

MIT License
