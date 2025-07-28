## PHP-FPM ?
PHP-FPM (FastCGI Process Manager) adalah sebuah alternative PHP FastCGI implementation dengan fitur tambahan yang dirancang untuk aplikasi web modern. PHP-FPM memungkinkan PHP untuk berjalan sebagai proses backend yang efisien dan terpisah dari web server (seperti Nginx), sehingga menghasilkan performa yang lebih tinggi, manajemen proses yang lebih baik, dan konsumsi resource yang lebih optimal.

## Kenapa PHP-FPM ?
- Process Management
  
  Mendukung pengaturan jumlah worker process, max request, timeout, dan graceful restart untuk mengelola beban server dengan baik.
- Security Isolation
  
  Tiap aplikasi (atau pool) bisa dijalankan dengan user dan konfigurasi yang berbeda, sehingga meningkatkan isolasi dan keamanan antar situs/aplikasi.
- Performance Boost
  
  Karena proses PHP sudah siap menerima permintaan, eksekusi skrip menjadi lebih cepat dibandingkan metode tradisional.
- Resource Efficiency
  
  Dengan fitur-fitur seperti pm.max_children dan pm.max_requests, PHP-FPM memberikan kontrol granular terhadap pemakaian CPU dan RAM.

## Installasi PHP & Extensions
### Tambahkan PPA ondrej/php
```bash
root@barak:~# add-apt-repository ppa:ondrej/php -y
root@barak:~# apt update
```

### Install
```bash
root@barak:~# apt install php8.1-fpm php8.1-mysql php8.1-mbstring php8.1-xml php8.1-gd php8.1-curl php8.1-zip php8.1-intl php8.1-bcmath php8.1-soap php8.1-redis php8.1-imagick -y
```

## Secure Konfigurasi
### Edit
```bash
root@barak:~# nano /etc/php/8.1/fpm/php.ini
```

### Ubah konfigurasi
```bash
expose_php = Off
max_execution_time = 300
max_input_vars = 3000
memory_limit = 256M
post_max_size = 64M
upload_max_filesize = 64M
date.timezone = Asia/Jakarta
session.cookie_httponly = 1
session.cookie_secure = 1
session.use_strict_mode = 1
```
**Gunakan `ctrl + w` untuk memudahkan pencarian**

## PHP-FPM Pool
### Edit
```bash
root@barak:~# nano /etc/php/8.1/fpm/pool.d/www.conf
```

### Ubah konfigurasi
```bash
[www]
user = www-data
group = www-data
listen = /run/php/php8.1-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 20
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
pm.max_requests = 500

security.limit_extensions = .php
php_admin_value[disable_functions] = exec,passthru,shell_exec,system,proc_open,popen
```
**Gunakan `ctrl + w` untuk memudahkan pencarian**

## Enable service
```bash
root@barak:~# systemctl enable php8.1-fpm
Synchronizing state of php8.1-fpm.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable php8.1-fpm
root@barak:~# systemctl status php8.1-fpm
● php8.1-fpm.service - The PHP 8.1 FastCGI Process Manager
     Loaded: loaded (/usr/lib/systemd/system/php8.1-fpm.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-07-29 00:34:23 WIB; 50min ago
       Docs: man:php-fpm8.1(8)
   Main PID: 77315 (php-fpm8.1)
     Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
      Tasks: 3 (limit: 2318)
     Memory: 11.7M (peak: 12.4M)
        CPU: 97ms
     CGroup: /system.slice/php8.1-fpm.service
             ├─77315 "php-fpm: master process (/etc/php/8.1/fpm/php-fpm.conf)"
             ├─77316 "php-fpm: pool www"
             └─77317 "php-fpm: pool www"

Jul 29 00:34:23 barak systemd[1]: Starting php8.1-fpm.service - The PHP 8.1 FastCGI Process Manager...
Jul 29 00:34:23 barak systemd[1]: Started php8.1-fpm.service - The PHP 8.1 FastCGI Process Manager.
```

## Testing
**Ubah index.html app1 jadi info.php**

```bash
root@barak:~# mv /var/www/app1.klandestin.site/public_html/index.html /var/www/app1.klandestin.site/public_html/info.php
```

**Ganti isi**

```bash
root@barak:~# cat << EOF > /var/www/app1.klandestin.site/public_html/info.php
<?php
phpinfo();
?>
EOF
```

**Reload NGINX**

```bash
root@barak:~# systemctl reload nginx
```

**Akses di Browser `https://app1.klandestin.site`**

<img width="1819" height="850" alt="image" src="https://github.com/user-attachments/assets/6f5b9f37-8647-415f-81c4-0cfd983b6b1c" />
