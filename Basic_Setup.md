## Checklist Sebelum Memulai

```bash
# Yang perlu disiapkan sebelum mulai:
- Alamat IP VPS
- Password root dari provider
- Komputer lokal dengan SSH client
```

## 1. Koneksi & Cek sistem
**Koneksi ke VPS sebagai root**
```bash
PS C:\Users\dapit> ssh root@165.101.18.19

The authenticity of host '203.0.113.10 (203.0.113.10)' can't be established.
ED25519 key fingerprint is SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz1234567890abcd.
Are you sure you want to continue connecting (yes/no/[fingerprint])? *yes
Warning: Permanently added '165.101.18.19' (ED25519) to the list of known hosts.

root@165.101.18.19's password: *masukkan password dari provider
```

**Cek informasi sistem (opsional)**
```bash
uname -a               # Versi kernel dan arsitektur
lsb_release -a         # Detail versi Ubuntu
df -h                  # Penggunaan ruang disk
free -h                # Penggunaan memori
lscpu                  # Informasi CPU
ip a                   # Interface jaringan
```

### 2. Update Sistem & Paket Essential
**Update sistem**
```bash
root@klandestin:~# apt update && apt upgrade -y
```

**Install paket**
```bash
root@klandestin:~# apt install curl wget net-tools gnupg2 software-properties-common unzip -y
```

### 3. Pembuatan User Administrator
**Buat user administrator**
```bash
root@klandestin:~# adduser dapit
info: Adding user `dapit' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `dapit' (1000) ...
info: Adding new user `dapit' (1000) with group `dapit (1000)' ...
info: Creating home directory `/home/dapit' ...
info: Copying files from `/etc/skel' ...
New password: *masukkan password yang kuat
Retype new password: *password confirmation
passwd: password updated successfully
Changing the user information for dapit
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] *y
info: Adding new user `dapit' to supplemental / extra groups `users' ...
info: Adding user `dapit' to group `users' ...
root@klandestin:~#
```

**Tambahkan user ke grup sudo**
```bash
root@klandestin:~# usermod -aG sudo dapit
```

**Verifikasi akses sudo**
```bash
root@klandestin:~# su - dapit
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

dapit@klandestin:~$ sudo whoami
[sudo] password for dapit:
root
dapit@klandestin:~$ exit
logout
```

### 4. Setup secure SSH
**Backup konfigurasi SSH**
```bash
root@klandestin:~# cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

**Edit konfigurasi SSH**
```bash
root@klandestin:~# nano /etc/ssh/sshd_config
```
Dokumentasi dapat di lihat di [SSH Setup](https://github.com/0xbyalak/Sysadmin-Lab/blob/716ac673bfe73b3f98ad382f86a8e0d7a8c17599/00-Instalation/RemoteAkses-Ubuntu.md)

**Warning**: Selalu test koneksi SSH baru sebelum menutup sesi lama untuk menghindari terkunci dari server.

### 5. Konfigurasi Firewall
**Reset UFW**
```bash
root@klandestin:~# ufw --force reset
```

**Set default policies (deny input, allow output)**
```bash
root@klandestin:~# ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
root@klandestin:~# ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

**Allow port SSH**
```bash
root@klandestin:~# ufw allow 2025/tcp
Rules updated
Rules updated (v6)
```

**Enable firewall**
```bash
root@klandestin:~# ufw --force enable
Firewall is active and enabled on system startup
```

**Cek status**
```bash
root@klandestin:~# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
2025/tcp                   ALLOW IN    Anywhere
2025/tcp (v6)              ALLOW IN    Anywhere (v6)
```

### 6. Instalasi dan Konfigurasi Fail2Ban
**Install Fail2Ban**
```bash
root@klandestin:~# apt install fail2ban -y
```

**Buat konfigurasi custom**
```bash
root@klandestin:~# tee /etc/fail2ban/jail.local << EOF
[DEFAULT]
bantime = 3600          
findtime = 600          
maxretry = 3            
backend = systemd       

[sshd]
enabled = true
port = 2025
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF
```

**Start dan enable Fail2Ban**
```bash
root@klandestin:~# systemctl enable --now fail2ban
Synchronizing state of fail2ban.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable fail2ban
root@klandestin:~# systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled; preset: enabled)
     Active: active (running) since Sun 2025-07-27 17:19:43 UTC; 3min 34s ago
       Docs: man:fail2ban(1)
   Main PID: 3099 (fail2ban-server)
      Tasks: 5 (limit: 2318)
     Memory: 24.9M (peak: 25.1M)
        CPU: 149ms
     CGroup: /system.slice/fail2ban.service
             └─3099 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Jul 27 17:19:43 klandestin systemd[1]: Started fail2ban.service - Fail2Ban Service.
Jul 27 17:19:43 klandestin fail2ban-server[3099]: 2025-07-27 17:19:43,156 fail2ban.configreader
Jul 27 17:19:43 klandestin fail2ban-server[3099]: Server ready
```

**Check status**
```bash
root@klandestin:~# fail2ban-client status
Status
|- Number of jail:      1
`- Jail list:   sshd
root@klandestin:~# fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
```

### 7. Konfigurasi Timezone dan Hostname
**Set timezone**
```bash
root@klandestin:~# timedatectl set-timezone Asia/Jakarta
root@klandestin:~# timedatectl
               Local time: Mon 2025-07-28 00:30:39 WIB
           Universal time: Sun 2025-07-27 17:30:39 UTC
                 RTC time: Sun 2025-07-27 17:30:40
                Time zone: Asia/Jakarta (WIB, +0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

**Set locale**
```bash
root@klandestin:~# locale-gen en_US.UTF-8
root@klandestin:~# locale
LANG=C.UTF-8
LANGUAGE=
LC_CTYPE="C.UTF-8"
LC_NUMERIC="C.UTF-8"
LC_TIME="C.UTF-8"
LC_COLLATE="C.UTF-8"
LC_MONETARY="C.UTF-8"
LC_MESSAGES="C.UTF-8"
LC_PAPER="C.UTF-8"
LC_NAME="C.UTF-8"
LC_ADDRESS="C.UTF-8"
LC_TELEPHONE="C.UTF-8"
LC_MEASUREMENT="C.UTF-8"
LC_IDENTIFICATION="C.UTF-8"
LC_ALL=
```
**Set hostname**
```bash
root@klandestin:~# hostnamectl set-hostname barak
root@klandestin:~# exec bash
root@barak:~# hostname
barak
```

### 8. kernel hardening
**Backup file sysctl.conf**
```bash
root@barak:~# cp /etc/sysctl.conf /etc/sysctl.conf.backup
```

**Edit file**
```bash
root@klandestin:~# tee /etc/sysctl.conf << EOF

# Nonaktifkan IPv6 hanya jika memang tidak digunakan
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Perlindungan SYN flood
net.ipv4.tcp_syncookies = 1

# Tolak ICMP redirect (mencegah MITM)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Tolak source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Jangan kirim ICMP redirect (menghindari kebocoran informasi routing)
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Abaikan paket broadcast (hindari smurf attack)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Abaikan ICMP error palsu
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Log paket mencurigakan (bisa dimatikan jika log terlalu banyak)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Randomisasi alamat memori (ASLR)
kernel.randomize_va_space = 2

# Buffer jaringan umum
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 5000
EOF
```

### 9. Optimasi Memori (Swap Configuration)
**Buat swap file 2x RAM**
```bash
root@barak:~# fallocate -l 4G /swapfile
```

**Set permission**
```bash
root@barak:~# chmod 600 /swapfile
```

**Format sebagai swap**
```bash
root@barak:~# mkswap /swapfile
Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
no label, UUID=39ac02b6-273b-4818-aea2-be4f38928928
```

**Aktifkan swap**
```bash
root@barak:~# swapon /swapfile
```

**Buat swap permanen**
```bash
root@barak:~# echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
/swapfile none swap sw 0 0
```
- /swapfile → lokasi file swap
- none → tidak ada mount point (memang untuk swap)
- swap → tipe filesystem adalah swap
- sw → opsi (default untuk swap)
- 0 0 → tidak perlu dump dan tidak perlu fsck

**Optimasi penggunaan swap**
```bash
root@barak:~# echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
vm.swappiness=10
root@barak:~# echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
vm.vfs_cache_pressure=50
```
- **Swappiness=10**: Sistem akan lebih prefer menggunakan RAM, swap hanya untuk emergency
- **VFS Cache Pressure=50**: Balance antara caching dan memory pressure

**Apply perubahan**
```bash
root@barak:~# sysctl -p
```

**Verifikasi swap aktif**
```bash
root@barak:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       474Mi       957Mi       1.2Mi       697Mi       1.5Gi
Swap:          4.0Gi          0B       4.0Gi
root@barak:~# swapon --show
NAME      TYPE SIZE USED PRIO
/swapfile file   4G   0B   -2
```
### 10. Monitoring 
**Installasi**
```bash
root@barak:~# apt install htop iftop iotop ncdu lsof -y
```

**htop**
- Alternatif top dengan tampilan interaktif.
- Menampilkan proses, penggunaan CPU, RAM, swap, dan load system.
- Bisa kill proses langsung dengan tombol.
```bash
root@barak:~# htop
```
<img width="1730" height="856" alt="image" src="https://github.com/user-attachments/assets/6a119e29-28d9-42ef-8c71-75fa41667bce" />

**iftop**
- Menunjukkan bandwidth real-time per koneksi jaringan.
- Berguna untuk melihat siapa/apa yang memakai internet di server.
```bash
root@barak:~# iftop
```
<img width="1732" height="918" alt="image" src="https://github.com/user-attachments/assets/7d0bde7d-6c6c-45d2-bcc7-3eb0d895bb0f" />

**iotop**
- Memantau I/O disk (baca/tulis) per proses.
- Penting saat troubleshooting bottleneck disk.
```bash
root@barak:~# iotop
```
<img width="1709" height="901" alt="image" src="https://github.com/user-attachments/assets/fddc71ed-0ab4-4c00-a6c5-88350515586b" />

**ncdu**
- Menampilkan penggunaan disk per folder.
- Memudahkan mencari folder/file yang memakan banyak ruang.
```bash
root@barak:~# ncdu /
```
<img width="1731" height="920" alt="image" src="https://github.com/user-attachments/assets/98d75e83-10fb-4272-b521-40a30c401689" />

**lsof**
- "List Open Files", menampilkan file atau socket yang sedang dibuka proses.
- Bisa digunakan untuk:
    - Cek port apa yang sedang dipakai (lsof -i :2025)
    - Lihat file yang mengunci disk.
```bash
root@barak:~# lsof -i :2025
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd    1  root  217u  IPv4   6342      0t0  TCP *:2025 (LISTEN)
systemd    1  root  218u  IPv6   6346      0t0  TCP *:2025 (LISTEN)
sshd    1184  root    3u  IPv4   6342      0t0  TCP *:2025 (LISTEN)
sshd    1184  root    4u  IPv6   6346      0t0  TCP *:2025 (LISTEN)
sshd    1411  root    4u  IPv4   9588      0t0  TCP host-301247.serverjumbo.com:2025->103.226.232.164:11767 (ESTABLISHED)
sshd    1457 dapit    4u  IPv4   9588      0t0  TCP host-301247.serverjumbo.com:2025->103.226.232.164:11767 (ESTABLISHED)
```
