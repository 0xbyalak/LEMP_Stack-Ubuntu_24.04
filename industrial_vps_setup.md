# 🛡️ Panduan Setup VPS Ubuntu 24.04 LTS Tingkat Industri

**Sistem Target**: Ubuntu 24.04 LTS Server  
**Hardware**: Ryzen 9950X 1C, 2GB RAM, 50GB SSD, 4TB Bandwidth  
**Level Keamanan**: Enterprise/Production Ready  
**Total Waktu**: 4-5 jam  

---

## 📋 **Checklist Sebelum Memulai**

```bash
# Yang perlu disiapkan sebelum mulai:
- Alamat IP VPS
- Password root (dari provider)
- Nama domain (opsional, untuk SSL)
- Komputer lokal dengan SSH client
- Pengetahuan dasar text editor (vim/nano)
```

---

## **Fase 1: Koneksi Awal & Penilaian Sistem**
*⏱️ Waktu: 15 menit*

### **Step 1.1: Koneksi Pertama & Penemuan Sistem**

```bash
# Koneksi ke VPS sebagai root (pertama kali saja)
ssh root@ALAMAT_IP_VPS_ANDA

# Pengumpulan informasi sistem
uname -a                # Versi kernel dan arsitektur
lsb_release -a         # Detail versi Ubuntu
df -h                  # Penggunaan ruang disk
free -h                # Penggunaan memori
lscpu                  # Informasi CPU
ip addr show           # Interface jaringan
```

**💡 Mengapa langkah ini penting:**
- **Penilaian Sistem**: Memahami keterbatasan hardware membantu perencanaan alokasi resource
- **Verifikasi Versi**: Memastikan bekerja dengan versi OS yang sesuai (Ubuntu 24.04)
- **Perencanaan Resource**: Kritis untuk keterbatasan 2GB RAM - setiap MB berharga
- **Penemuan Jaringan**: Mengidentifikasi interface yang tersedia untuk konfigurasi keamanan

### **Step 1.2: Update Sistem & Paket Essential**

```bash
# Update database paket dan paket yang terinstall
apt update && apt upgrade -y

# Install tools sistem essential
apt install -y \
    curl \              # HTTP client untuk download
    wget \              # File downloader
    vim \               # Text editor advanced
    htop \              # Process viewer interaktif
    tree \              # Viewer struktur direktori
    unzip \             # Ekstraksi arsip
    git \               # Version control
    software-properties-common \  # Manajemen PPA
    apt-transport-https \         # Support repository HTTPS
    ca-certificates \             # Validasi sertifikat SSL
    gnupg \                      # Enkripsi GPG
    lsb-release                  # Deteksi versi OS
```

**💡 Mengapa paket-paket ini:**
- **curl/wget**: Essential untuk download konfigurasi dan script
- **vim**: Lebih powerful dari nano untuk editing konfigurasi
- **htop**: Monitoring proses visual (krusial untuk 2GB RAM)
- **git**: Version control untuk manajemen konfigurasi
- **SSL tools**: Diperlukan untuk akses repository aman dan HTTPS

---

## **Fase 2: Fondasi Keamanan**
*⏱️ Waktu: 45 menit*

### **Step 2.1: Pembuatan User Administrator**

```bash
# Buat user administrator (ganti 'admin' dengan pilihan Anda)
adduser admin

# Tambahkan user ke grup sudo untuk hak administratif
usermod -aG sudo admin

# Verifikasi akses sudo
su - admin
sudo whoami    # Harus return 'root'
exit
```

**💡 Alasan keamanan:**
- **Prinsip Least Privilege**: Root tidak boleh digunakan untuk operasi harian
- **Audit Trail**: Akun user bernama memberikan logging yang lebih baik
- **Pemisahan Kredensial**: Akun user yang terkompromi ≠ root terkompromi
- **Standar Industri**: Semua environment enterprise menggunakan dedicated admin user

### **Step 2.2: Pengerasan Keamanan SSH**

```bash
# Backup konfigurasi SSH original
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Edit konfigurasi SSH
vim /etc/ssh/sshd_config
```

**Konfigurasi Keamanan SSH:**
```bash
# Ubah port default (security through obscurity)
Port 2222
# Alasan: Mengurangi serangan otomatis pada port 22 sampai ~90%

# Disable login root sepenuhnya
PermitRootLogin no
# Alasan: Menghilangkan vektor serangan dengan hak akses tertinggi

# Batasi akses user
AllowUsers admin
# Alasan: Pendekatan whitelist - hanya user yang diotorisasi bisa connect

# Disable autentikasi password (setelah setup key)
PasswordAuthentication no
PubkeyAuthentication yes
# Alasan: Key-based auth 1000x lebih aman dari password

# Setting keamanan lainnya
Protocol 2                    # Gunakan SSH protokol 2 (lebih aman)
MaxAuthTries 3               # Maksimal 3 percobaan login
ClientAliveInterval 300      # Timeout koneksi idle (5 menit)
ClientAliveCountMax 2        # Maksimal 2 heartbeat gagal
X11Forwarding no            # Disable X11 forwarding (tidak diperlukan server)
PermitEmptyPasswords no     # Larang password kosong
```

**💡 Mengapa konfigurasi ini:**
- **Port Non-Standard**: Mengurangi brute force attacks secara dramatis
- **No Root Login**: Eliminasi target utama attacker
- **Key Authentication**: Praktis tidak mungkin di-brute force
- **Connection Timeouts**: Mencegah session hijacking dan idle connections

### **Step 2.3: Setup SSH Key Authentication**

```bash
# Di komputer lokal Anda, generate SSH key
ssh-keygen -t ed25519 -C "email_anda@domain.com"

# Copy public key ke VPS
ssh-copy-id -p 2222 admin@ALAMAT_IP_VPS_ANDA

# Test login berbasis key
ssh -p 2222 admin@ALAMAT_IP_VPS_ANDA
```

**💡 Keunggulan ED25519:**
- **Performa Tinggi**: Lebih cepat dari RSA 2048-bit
- **Keamanan Superior**: Resistance terhadap side-channel attacks
- **Ukuran Key Kecil**: 256-bit tapi setara RSA 3072-bit
- **Modern Standard**: Direkomendasikan oleh cryptography experts

### **Step 2.4: Restart SSH dengan Konfigurasi Baru**

```bash
# Restart SSH service dengan konfigurasi baru
sudo systemctl restart ssh

# Verifikasi SSH berjalan di port baru
sudo ss -tlnp | grep :2222

# Test koneksi dari terminal baru (JANGAN TUTUP SESI LAMA DULU!)
ssh -p 2222 admin@ALAMAT_IP_VPS_ANDA
```

**⚠️ PENTING**: Selalu test koneksi SSH baru sebelum menutup sesi lama untuk menghindari terkunci dari server.

### **Step 2.5: Konfigurasi Firewall UFW**

```bash
# Reset UFW untuk memulai fresh
sudo ufw --force reset

# Set default policies (deny input, allow output)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH di port custom
sudo ufw allow 2222/tcp

# Allow HTTP dan HTTPS untuk web services
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw --force enable

# Check status
sudo ufw status verbose
```

**💡 Filosofi firewall:**
- **Default Deny**: Semua koneksi masuk diblokir secara default
- **Explicit Allow**: Hanya service yang dibutuhkan yang dibuka
- **Stateful Filtering**: UFW secara otomatis mengizinkan response connections
- **Logging**: Semua aktivitas firewall di-log untuk monitoring

---

## **Fase 3: Optimasi Sistem**
*⏱️ Waktu: 30 menit*

### **Step 3.1: Optimasi Memori (Swap Configuration)**

```bash
# Buat swap file 4GB (2x RAM untuk sistem 2GB)
sudo fallocate -l 4G /swapfile

# Set permission yang aman untuk swap file
sudo chmod 600 /swapfile

# Format sebagai swap
sudo mkswap /swapfile

# Aktifkan swap
sudo swapon /swapfile

# Buat swap permanen
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimasi penggunaan swap
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf

# Apply perubahan
sudo sysctl -p

# Verifikasi swap aktif
free -h
swapon --show
```

**💡 Alasan konfigurasi swap:**
- **Ukuran 4GB**: Rule of thumb 2x RAM untuk sistem dengan RAM terbatas
- **Swappiness=10**: Sistem akan lebih prefer menggunakan RAM, swap hanya untuk emergency
- **VFS Cache Pressure=50**: Balance antara caching dan memory pressure
- **Permission 600**: Hanya root yang bisa baca/tulis (keamanan)

### **Step 3.2: Tuning Performa Sistem**

```bash
# Edit konfigurasi limits
sudo vim /etc/security/limits.conf
```

**Tambahkan baris ini di akhir file:**
```bash
# Increase file descriptor limits
* soft nofile 65536
* hard nofile 65536
# Increase process limits  
* soft nproc 32768
* hard nproc 32768
```

**💡 Mengapa limits ini penting:**
- **File Descriptors**: Web servers dan databases membuka banyak file/connections
- **Process Limits**: Mencegah fork bombs dan resource exhaustion
- **65536 FD**: Cukup untuk aplikasi enterprise tanpa berlebihan
- **32768 Processes**: Balance antara fleksibilitas dan resource protection

### **Step 3.3: Optimasi Jaringan TCP**

```bash
# Tambahkan optimasi TCP ke sysctl
sudo tee -a /etc/sysctl.conf << EOF

# === Network Performance Optimization ===
# Increase TCP buffer sizes
net.core.rmem_max = 16777216          # Max receive buffer
net.core.wmem_max = 16777216          # Max send buffer
net.ipv4.tcp_rmem = 4096 12582912 16777216  # TCP receive buffer
net.ipv4.tcp_wmem = 4096 12582912 16777216  # TCP send buffer

# TCP performance improvements
net.ipv4.tcp_no_metrics_save = 1      # Don't save metrics on close
net.ipv4.tcp_moderate_rcvbuf = 1      # Auto-tune receive buffers
net.core.netdev_max_backlog = 5000    # Increase network device backlog

# Connection tracking optimization
net.netfilter.nf_conntrack_max = 262144
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
EOF

# Apply pengaturan jaringan
sudo sysctl -p
```

**💡 Optimasi jaringan ini:**
- **Buffer Sizes**: Meningkatkan throughput untuk high-bandwidth applications
- **TCP Tuning**: Mengurangi latency dan meningkatkan connection handling
- **Connection Tracking**: Penting untuk firewall performance
- **Conservative Values**: Tidak berlebihan untuk VPS 2GB RAM

---

## **Fase 4: Keamanan Tingkat Lanjut**
*⏱️ Waktu: 45 menit*

### **Step 4.1: Instalasi dan Konfigurasi Fail2Ban**

```bash
# Install Fail2Ban
sudo apt install -y fail2ban

# Buat konfigurasi custom
sudo tee /etc/fail2ban/jail.local << EOF
[DEFAULT]
# Default ban settings
bantime = 3600          # Ban selama 1 jam
findtime = 600          # Window time 10 menit
maxretry = 3            # Maksimal 3 percobaan gagal
backend = systemd       # Use systemd untuk log parsing

# Email notifications (opsional)
# destemail = admin@yourdomain.com
# sendername = Fail2Ban
# mta = sendmail

[sshd]
enabled = true
port = 2222             # Port SSH custom kita
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 6

[nginx-limit-req]
enabled = true  
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
findtime = 600
bantime = 3600
EOF

# Start dan enable Fail2Ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

**💡 Mengapa Fail2Ban penting:**
- **Intrusion Prevention**: Secara otomatis memblokir IP yang mencurigakan
- **Adaptive Defense**: Belajar dari pola serangan dan menyesuaikan
- **Multi-Service**: Melindungi SSH, web server, dan layanan lainnya
- **Temporary Bans**: Tidak permanent, memberikan kesempatan kedua untuk user legitimate

### **Step 4.2: Tools Keamanan Tambahan**

```bash
# Install security scanning tools
sudo apt install -y rkhunter chkrootkit lynis unattended-upgrades

# Konfigurasi automatic security updates
sudo dpkg-reconfigure -plow unattended-upgrades
# Pilih "Yes" untuk enable automatic updates

# Setup rkhunter untuk rootkit detection
sudo rkhunter --update
sudo rkhunter --propupd

# Buat script security scan
sudo tee /usr/local/bin/security-scan.sh << 'EOF'
#!/bin/bash
LOGFILE="/var/log/security-scan.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "=== Security Scan Started at $DATE ===" >> $LOGFILE

# Rkhunter scan
echo "Running rkhunter..." >> $LOGFILE
rkhunter --check --skip-keypress --report-warnings-only >> $LOGFILE 2>&1

# Chkrootkit scan  
echo "Running chkrootkit..." >> $LOGFILE
chkrootkit >> $LOGFILE 2>&1

# System integrity check
echo "Checking system integrity..." >> $LOGFILE
find /etc -type f -name "*.conf" -exec md5sum {} \; >> /var/log/config-checksums.log

echo "=== Security Scan Completed at $(date '+%Y-%m-%d %H:%M:%S') ===" >> $LOGFILE
echo "" >> $LOGFILE
EOF

sudo chmod +x /usr/local/bin/security-scan.sh

# Test script
sudo /usr/local/bin/security-scan.sh
```

**💡 Tools keamanan ini:**
- **rkhunter**: Deteksi rootkit dan malware yang canggih
- **chkrootkit**: Scanning tambahan untuk rootkit detection
- **lynis**: Security auditing dan hardening recommendations
- **unattended-upgrades**: Automatic security patches (krusial untuk server production)

### **Step 4.3: Manajemen Log dan Monitoring**

```bash
# Konfigurasi logrotate untuk custom logs
sudo tee /etc/logrotate.d/custom-security << EOF
/var/log/security-scan.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 root root
}

/var/log/config-checksums.log {
    weekly
    missingok
    rotate 12
    compress
    delaycompress
    notifempty
    create 644 root root
}
EOF

# Setup direktori untuk aplikasi logs
sudo mkdir -p /var/log/applications
sudo chown syslog:adm /var/log/applications

# Konfigurasi rsyslog untuk centralized logging
sudo tee /etc/rsyslog.d/50-applications.conf << EOF
# Application logs
local0.*    /var/log/applications/app.log
local1.*    /var/log/applications/security.log
local2.*    /var/log/applications/performance.log
EOF

sudo systemctl restart rsyslog
```

**💡 Manajemen log yang baik:**
- **Log Rotation**: Mencegah disk full dari log yang menumpuk
- **Centralized Logging**: Memudahkan monitoring dan troubleshooting
- **Log Retention**: Balance antara storage dan audit requirements
- **Structured Logging**: Memudahkan parsing dan analysis

---

## **Fase 5: Monitoring & Alerting**
*⏱️ Waktu: 30 menit*

### **Step 5.1: System Health Monitoring**

```bash
# Install tools monitoring
sudo apt install -y htop iotop nethogs ncdu duf

# Buat script health check comprehensive
sudo tee /usr/local/bin/health-check.sh << 'EOF'
#!/bin/bash
LOGFILE="/var/log/health-check.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')
HOSTNAME=$(hostname)

echo "[$DATE] [$HOSTNAME] Health Check Started" >> $LOGFILE

# Check disk usage
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "[$DATE] [CRITICAL] Disk usage is ${DISK_USAGE}%" >> $LOGFILE
    logger -p local1.crit "CRITICAL: Disk usage ${DISK_USAGE}%"
elif [ $DISK_USAGE -gt 70 ]; then
    echo "[$DATE] [WARNING] Disk usage is ${DISK_USAGE}%" >> $LOGFILE
    logger -p local1.warning "WARNING: Disk usage ${DISK_USAGE}%"
fi

# Check memory usage
MEM_USAGE=$(free | grep '^Mem' | awk '{printf "%.0f", ($3/$2)*100}')
if [ $MEM_USAGE -gt 90 ]; then
    echo "[$DATE] [CRITICAL] Memory usage is ${MEM_USAGE}%" >> $LOGFILE
    logger -p local1.crit "CRITICAL: Memory usage ${MEM_USAGE}%"
elif [ $MEM_USAGE -gt 80 ]; then
    echo "[$DATE] [WARNING] Memory usage is ${MEM_USAGE}%" >> $LOGFILE
    logger -p local1.warning "WARNING: Memory usage ${MEM_USAGE}%"
fi

# Check load average
LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
if [ $(echo "$LOAD_AVG > 2.0" | bc -l) -eq 1 ]; then
    echo "[$DATE] [WARNING] Load average is $LOAD_AVG" >> $LOGFILE
    logger -p local1.warning "WARNING: Load average $LOAD_AVG"
fi

# Check available inodes
INODE_USAGE=$(df -i / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ $INODE_USAGE -gt 80 ]; then
    echo "[$DATE] [WARNING] Inode usage is ${INODE_USAGE}%" >> $LOGFILE
    logger -p local1.warning "WARNING: Inode usage ${INODE_USAGE}%"
fi

# Check critical services
SERVICES=("ssh" "nginx" "mysql" "fail2ban")
for service in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet $service 2>/dev/null; then
        echo "[$DATE] [CRITICAL] Service $service is not running" >> $LOGFILE
        logger -p local1.crit "CRITICAL: Service $service down"
    fi
done

# Check network connectivity
if ! ping -c 1 8.8.8.8 >/dev/null 2>&1; then
    echo "[$DATE] [CRITICAL] No internet connectivity" >> $LOGFILE
    logger -p local1.crit "CRITICAL: No internet connectivity"
fi

echo "[$DATE] [$HOSTNAME] Health Check Completed" >> $LOGFILE
EOF

sudo chmod +x /usr/local/bin/health-check.sh

# Test health check
sudo /usr/local/bin/health-check.sh
tail -20 /var/log/health-check.log
```

**💡 Health check ini monitor:**
- **Disk Usage**: Mencegah disk full yang bisa crash sistem
- **Memory Usage**: Critical untuk sistem 2GB RAM
- **Load Average**: Indikator performa sistem
- **Inode Usage**: Mencegah "No space left on device" error
- **Service Status**: Memastikan layanan kritis berjalan
- **Network Connectivity**: Monitoring konektivitas internet

### **Step 5.2: Automated Task Scheduling**

```bash
# Setup cron jobs untuk automated maintenance
sudo crontab -e
```

**Tambahkan cron jobs ini:**
```bash
# Health check setiap 15 menit
*/15 * * * * /usr/local/bin/health-check.sh

# Security scan harian jam 2 pagi
0 2 * * * /usr/local/bin/security-scan.sh

# Update sistem mingguan (Minggu jam 3 pagi)
0 3 * * 0 apt update && apt upgrade -y >> /var/log/auto-update.log 2>&1

# Cleanup package cache bulanan
0 4 1 * * apt autoremove -y && apt autoclean >> /var/log/cleanup.log 2>&1

# Rotate logs manual (backup untuk logrotate)
0 5 * * * find /var/log -name "*.log" -size +100M -exec gzip {} \;

# Check disk space dan alert jika >85%
*/30 * * * * [ $(df / | tail -1 | awk '{print $5}' | sed 's/%//') -gt 85 ] && echo "Disk usage critical: $(df -h /)" | logger -p local1.crit
```

**💡 Cron schedule rationale:**
- **15-minute Health Checks**: Balance antara responsiveness dan resource usage
- **2 AM Security Scans**: Low traffic time, results ready untuk review pagi
- **Weekly Updates**: Sunday 3 AM - minimal impact pada operations
- **Monthly Cleanup**: Prevent gradual disk space creep
- **30-minute Disk Checks**: Critical untuk sistem dengan storage terbatas

---

## **Fase 6: Backup & Recovery Strategy**
*⏱️ Waktu: 25 menit*

### **Step 6.1: Backup Infrastructure**

```bash
# Buat struktur direktori backup
sudo mkdir -p /backup/{daily,weekly,monthly,config}
sudo chown -R root:root /backup
sudo chmod -R 750 /backup

# Buat comprehensive backup script
sudo tee /usr/local/bin/backup-system.sh << 'EOF'
#!/bin/bash
BACKUP_ROOT="/backup"
DATE=$(date '+%Y%m%d_%H%M%S')
LOGFILE="/var/log/backup.log"
HOSTNAME=$(hostname)

echo "[$DATE] [$HOSTNAME] Backup started" >> $LOGFILE

# Function untuk backup dengan compression dan verification
backup_and_verify() {
    local SOURCE=$1
    local DEST=$2
    local NAME=$3
    
    echo "Backing up $NAME..." >> $LOGFILE
    
    if tar -czf "$DEST" "$SOURCE" 2>/dev/null; then
        # Verify backup integrity
        if tar -tzf "$DEST" >/dev/null 2>&1; then
            SIZE=$(du -h "$DEST" | cut -f1)
            echo "SUCCESS: $NAME backup completed ($SIZE)" >> $LOGFILE
        else
            echo "ERROR: $NAME backup verification failed" >> $LOGFILE
            rm -f "$DEST"
            return 1
        fi
    else
        echo "ERROR: $NAME backup failed" >> $LOGFILE
        return 1
    fi
}

# System configuration backup
backup_and_verify "/etc" "$BACKUP_ROOT/daily/etc_$DATE.tar.gz" "System Config"

# User configurations
backup_and_verify "/home" "$BACKUP_ROOT/daily/home_$DATE.tar.gz" "User Configs"

# Custom scripts and binaries
backup_and_verify "/usr/local" "$BACKUP_ROOT/daily/usr_local_$DATE.tar.gz" "Custom Scripts"

# Important logs (excluding large/rotated ones)
find /var/log -name "*.log" -size -10M -mtime -7 | tar -czf "$BACKUP_ROOT/daily/logs_$DATE.tar.gz" -T - 2>/dev/null
echo "Recent logs backed up" >> $LOGFILE

# Crontab backup
crontab -l > "$BACKUP_ROOT/config/crontab_$DATE.txt" 2>/dev/null
sudo crontab -l > "$BACKUP_ROOT/config/root_crontab_$DATE.txt" 2>/dev/null

# Package list backup
dpkg --get-selections > "$BACKUP_ROOT/config/packages_$DATE.txt"
apt-mark showmanual > "$BACKUP_ROOT/config/manual_packages_$DATE.txt"

# Network configuration backup
ip addr show > "$BACKUP_ROOT/config/network_config_$DATE.txt"
ss -tlnp > "$BACKUP_ROOT/config/listening_ports_$DATE.txt"

# Cleanup old daily backups (keep 7 days)
find $BACKUP_ROOT/daily -name "*.tar.gz" -mtime +7 -delete
find $BACKUP_ROOT/config -name "*.txt" -mtime +30 -delete

# Weekly backup (every Sunday) - more comprehensive
if [ $(date +%u) -eq 7 ]; then
    echo "Creating weekly backup..." >> $LOGFILE
    backup_and_verify "/" "$BACKUP_ROOT/weekly/full_system_$DATE.tar.gz" "Full System" \
        --exclude=/proc --exclude=/sys --exclude=/dev --exclude=/tmp \
        --exclude=/var/cache --exclude=/var/tmp --exclude=/backup
    
    find $BACKUP_ROOT/weekly -name "*.tar.gz" -mtime +28 -delete
fi

# Monthly backup (first day of month)
if [ $(date +%d) -eq 1 ]; then
    echo "Creating monthly backup..." >> $LOGFILE
    cp "$BACKUP_ROOT/weekly/full_system_$DATE.tar.gz" "$BACKUP_ROOT/monthly/" 2>/dev/null
    find $BACKUP_ROOT/monthly -name "*.tar.gz" -mtime +365 -delete
fi

# Backup space usage report
echo "Backup space usage:" >> $LOGFILE
du -h $BACKUP_ROOT/* >> $LOGFILE

echo "[$DATE] [$HOSTNAME] Backup completed" >> $LOGFILE
echo "" >> $LOGFILE
EOF

sudo chmod +x /usr/local/bin/backup-system.sh

# Test backup script
sudo /usr/local/bin/backup-system.sh
```

**💡 Backup strategy explanation:**
- **Incremental Approach**: Daily configs, weekly full, monthly archive
- **Verification**: Setiap backup di-verify integrity-nya
- **Retention Policy**: 7 daily, 4 weekly, 12 monthly backups
- **Selective Backup**: Exclude directories yang tidak perlu (proc, sys, cache)
- **Package Lists**: Memudahkan rebuild sistem jika diperlukan

### **Step 6.2: Disaster Recovery Procedures**

```bash
# Buat disaster recovery documentation script
sudo tee /usr/local/bin/disaster-recovery-info.sh << 'EOF'
#!/bin/bash
DR_DOC="/backup/disaster-recovery.txt"

cat > $DR_DOC << EODOC
# DISASTER RECOVERY PROCEDURES
# Generated: $(date)
# Hostname: $(hostname)
# IP Address: $(ip route get 8.8.8.8 | head -1 | awk '{print $7}')

## CRITICAL INFORMATION
SSH Port: $(grep '^Port' /etc/ssh/sshd_config | awk '{print $2}')
Admin User: admin
Backup Location: /backup/

## RECOVERY STEPS

### 1. Fresh System Recovery
1. Install Ubuntu 24.04 LTS
2. Create admin user: adduser admin && usermod -aG sudo admin
3. Configure SSH: Port 2222, disable root login
4. Restore /etc: tar -xzf backup/daily/etc_YYYYMMDD_HHMMSS.tar.gz -C /
5. Restore packages: apt install \$(cat backup/config/manual_packages_*.txt)
6. Restore custom scripts: tar -xzf backup/daily/usr_local_*.tar.gz -C /
7. Restore user configs: tar -xzf backup/daily/home_*.tar.gz -C /

### 2. Service Recovery Priority
1. SSH (critical for remote access)
2. Firewall (ufw enable)
3. Fail2Ban (systemctl start fail2ban)
4. Web services (nginx, php-fpm)
5. Databases (mysql)
6. Monitoring services

### 3. Verification Checklist
- [ ] SSH access working on port 2222
- [ ] Firewall rules active (ufw status)
- [ ] All critical services running
- [ ] Backup script working
- [ ] Health checks operational
- [ ] Log rotation configured

### 4. Emergency Contacts & Information
Provider: [VPS_PROVIDER_NAME]
Backup Frequency: Daily configs, Weekly full
RTO (Recovery Time Objective): 4 hours
RPO (Recovery Point Objective): 24 hours

EODOC

echo "Disaster recovery documentation updated: $DR_DOC"
EOF

sudo chmod +x /usr/local/bin/disaster-recovery-info.sh
sudo /usr/local/bin/disaster-recovery-info.sh
```

**💡 Disaster recovery preparation:**
- **Documentation**: Prosedur recovery yang jelas dan tested
- **RTO/RPO**: Target waktu recovery yang realistis
- **Priority Services**: Urutan service recovery berdasarkan kritikalitas
- **Verification Checklist**: Memastikan sistem fully recovered

---

## **Fase