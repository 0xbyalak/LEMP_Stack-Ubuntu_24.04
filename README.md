## LEMP Stack - Web Hosting Platform

Proyek ini adalah bagian dari portofolio aku, dengan tujuan membangun **platform hosting berbasis LEMP Stack** (Linux, Nginx, MySQL/MariaDB, PHP) yang **aman, stabil, dan optimal**. Proyek ini aku kerjakan langsung di **VPS dengan Ubuntu Server 24.04**, sebagai latihan nyata untuk membangun web hosting platform.

### Tujuan Proyek

Menyediakan fondasi server hosting yang:

- Aman dari serangan umum (seperti brute-force SSH, serangan HTTP)
- Siap digunakan untuk deploy aplikasi PHP secara live
- Teroptimasi untuk performa tinggi pada server dengan resource terbatas (2GB RAM)
- Mudah dipantau dan dirawat melalui sistem monitoring dan backup otomatis

### Ruang Lingkup Implementasi

#### 1. **Base Setup & Hardening**
- Penerapan hardening dasar: update otomatis, SSH key login, disable root dan password login

#### 2. **LEMP Stack Deployment**
- Menggunakan software stack Linux + Nginx + MariaDB + PHP 8.2 (LEMP)
- Virtual host berbasis domain untuk multi-site hosting
- Proses konfigurasi dirancang untuk fleksibilitas dan keamanan

#### 3. **SSL Automation**
- Instalasi Certbot dan penerapan SSL dari Let's Encrypt
- Otomatisasi pembaruan sertifikat SSL menggunakan cron job

#### 4. **Security Layer**
- Konfigurasi firewall menggunakan UFW dengan aturan khusus hanya membuka port penting (web & SSH)
- Pemasangan Fail2ban untuk memblokir brute-force login secara otomatis
- SSH dikonfigurasi hanya menerima koneksi dari key-pair, tidak ada password login

#### 5. **Performance Optimization**
- Nginx dioptimalkan dengan:
  - Gzip compression untuk file statis
  - Static file caching
- PHP-FPM disesuaikan untuk server RAM 2GB agar efisien (pm.max_children, process idle timeout, dsb.)
- MariaDB dioptimalkan untuk beban ringan-menengah dengan query cache, slow query log, dsb.

#### 6. **Monitoring & Alerts**
- Sistem log terpusat via `journald` dan `logrotate`
- Dasar penggunaan `htop`, `df`, dan `du` sebagai tool monitoring lokal

#### 7. **Backup & Recovery System**
- Backup otomatis harian untuk direktori penting dan database
- Retensi backup dengan kebijakan rotasi mingguan

### Catatan

Proyek ini cocok untuk pemula yang ingin belajar membangun layanan hosting PHP secara serius, namun tetap memperhatikan **keamanan**, **performa**, dan **maintainability**. Dokumentasi ini bisa dikembangkan lebih lanjut untuk menambahkan fitur seperti Docker, CI/CD, atau WordPress multi-site.

> Semua proses dilakukan secara manual untuk melatih skill dasar Linux dan server administration.
