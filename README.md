# Ansible Multi-Purpose Deployment

Ansible playbook dan roles untuk setup server dan deploy aplikasi (Laravel, Express, dll).

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
│   ├── bootstrap.yml           # 🚀 Setup lengkap server (MySQL, Nginx, PHP, Node.js, Composer)
│   ├── deploy-laravel.yml      # Deploy aplikasi Laravel
│   ├── php.yml                 # Setup PHP saja
│   └── webserver.yml           # Setup Nginx saja
├── roles/
│   ├── common/                 # Setup dasar server + security
│   ├── mysql/                  # Instalasi MySQL/MariaDB
│   ├── nginx/                  # Instalasi & konfigurasi Nginx
│   ├── php/                    # Instalasi PHP + extensions
│   ├── composer/               # Instalasi Composer
│   ├── node/                   # Instalasi Node.js + npm
│   ├── laravel/                # Deploy aplikasi Laravel
│   └── express/                # Deploy aplikasi Express.js
└── site.yml                    # Master playbook
```

## 🚀 Quick Start

### 1. Konfigurasi Inventory

Edit file `inventory/hosts` dengan data server Anda:

```ini
[webserver]
server1 ansible_host=YOUR_IP ansible_user=YOUR_USER ansible_ssh_private_key_file=~/.ssh/your_key
```

### 2. Bootstrap Server (Setup Semua Requirements)

```bash
# Setup lengkap: Common, MySQL, Nginx, PHP, Composer, Node.js
ansible-playbook playbooks/bootstrap.yml

# Skip MySQL jika sudah ada atau tidak perlu
ansible-playbook playbooks/bootstrap.yml --skip-tags mysql

# Hanya install PHP dan Composer
ansible-playbook playbooks/bootstrap.yml --tags php,composer

# Hanya install Node.js
ansible-playbook playbooks/bootstrap.yml --tags node
```

### 3. Deploy Laravel Application

```bash
# Deploy pertama kali
ansible-playbook playbooks/deploy-laravel.yml -e "first_install=true"

# Deploy update
ansible-playbook playbooks/deploy-laravel.yml
```

---

## 📦 Bootstrap Playbook

Bootstrap playbook menginstall semua requirements untuk menjalankan aplikasi web:

| Component | Deskripsi |
|-----------|-----------|
| **Common** | Update system, UFW firewall, Fail2ban, essential packages |
| **MySQL** | MariaDB server + client |
| **Nginx** | Web server |
| **PHP** | PHP 8.3 + extensions (fpm, mysql, xml, mbstring, zip, curl, gd, bcmath, intl, opcache) |
| **Composer** | PHP dependency manager |
| **Node.js** | Node.js v20 + npm + PM2 |

### Cara Penggunaan:

```bash
# Full bootstrap
ansible-playbook playbooks/bootstrap.yml

# Dengan custom variables
ansible-playbook playbooks/bootstrap.yml \
  -e "php_version=8.2" \
  -e "node_version=18" \
  -e "timezone=UTC"
```

### Bootstrap Variables:

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `timezone` | `Asia/Jakarta` | Timezone server |
| `upgrade_packages` | `true` | Upgrade semua packages |
| `php_version` | `8.3` | Versi PHP |
| `node_version` | `20` | Versi Node.js |
| `deploy_user` | `deploy` | User untuk deployment |
| `allow_http` | `true` | Allow port 80 di firewall |
| `allow_https` | `true` | Allow port 443 di firewall |

---

## 🔧 Deploy Laravel

### Variables:

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `project_name` | required | Nama project/folder |
| `domain` | required | Domain website |
| `repo_url` | required | Git repository URL |
| `git_branch` | `main` | Branch yang akan di-deploy |
| `deploy_user` | `www-data` | User untuk deploy |
| `first_install` | `false` | Set `true` untuk deploy pertama |
| `enable_maintenance_mode` | `true` | Maintenance mode saat deploy |
| `run_seeders` | `false` | Jalankan db:seed |
| `cache_config` | `true` | Cache Laravel config |
| `cache_routes` | `true` | Cache Laravel routes |
| `cache_views` | `true` | Cache Laravel views |

### Database Variables:

| Variable | Default | Deskripsi |
|----------|---------|-----------|
| `db_name` | required | Nama database |
| `db_user` | required | Username database |
| `db_pass` | required | Password database |
| `db_host` | `127.0.0.1` | Database host |
| `db_port` | `3306` | Database port |

### Contoh Playbook Variables:

```yaml
vars:
  project_name: "myapp"
  domain: "myapp.com"
  repo_url: "git@github.com:user/myapp.git"
  
  db_name: "myapp_db"
  db_user: "myapp_user"
  db_pass: "secure_password"
  
  app_name: "My Application"
  app_timezone: "Asia/Jakarta"
```

---

## 📝 Workflow

### Setup Server Baru:

```bash
# 1. Test koneksi
ansible webserver -m ping

# 2. Bootstrap server
ansible-playbook playbooks/bootstrap.yml

# 3. Deploy Laravel
ansible-playbook playbooks/deploy-laravel.yml -e "first_install=true"
```

### Deploy Update:

```bash
# Simple deploy
ansible-playbook playbooks/deploy-laravel.yml

# Dengan seeders
ansible-playbook playbooks/deploy-laravel.yml -e "run_seeders=true"

# Tanpa maintenance mode
ansible-playbook playbooks/deploy-laravel.yml -e "enable_maintenance_mode=false"
```

---

## 🏷️ Tags Reference

### Bootstrap Tags:
| Tag | Roles |
|-----|-------|
| `common`, `base` | Common role |
| `mysql`, `database` | MySQL role |
| `nginx`, `webserver` | Nginx role |
| `php` | PHP role |
| `composer` | Composer role |
| `node`, `nodejs` | Node.js role |

### Deploy Tags:
| Tag | Roles |
|-----|-------|
| `laravel`, `deploy` | Laravel role |

### Contoh Penggunaan Tags:

```bash
# Hanya install PHP dan Composer
ansible-playbook playbooks/bootstrap.yml --tags php,composer

# Skip database
ansible-playbook playbooks/bootstrap.yml --skip-tags mysql

# Hanya setup webserver
ansible-playbook playbooks/bootstrap.yml --tags nginx
```

---

## ⚠️ Troubleshooting

### Permission Denied saat Git Clone

```bash
# Generate SSH key di server
ssh-keygen -t ed25519 -C "deploy@server"

# Tambahkan ke GitHub Deploy Keys
cat ~/.ssh/id_ed25519.pub
```

### Composer Memory Error

```bash
# Tambah swap
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### MySQL Connection Error

```bash
# Login dengan socket
sudo mysql -u root

# Atau cek status
sudo systemctl status mariadb
```

---

## 📄 License

MIT License
