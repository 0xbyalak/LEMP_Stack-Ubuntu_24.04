## NGINX ?
Yo, Root Lords! NGINX (dibaca: "engine-ex") adalah web server open-source yang sangat ringan, cepat, dan efisien. Awalnya dirancang untuk mengatasi masalah C10k - yaitu kemampuan menangani 10.000 koneksi secara bersamaan - NGINX berkembang pesat dan kini digunakan tidak hanya sebagai web server, tapi juga sebagai reverse proxy, load balancer, dan bahkan proxy cache.

Berbeda dengan web server tradisional seperti Apache yang berbasis proses (process-based), NGINX menggunakan arsitektur event-driven non-blocking, yang memungkinkan satu proses menangani banyak koneksi sekaligus. Hasilnya? Penggunaan RAM lebih hemat dan performa lebih stabil, terutama saat trafik tinggi.

## Kenapa NGINX ?
Dalam konteks proyek ini LEMP Stack - kita memilih NGINX sebagai komponen utama karena alasan :

- Ringan dan Cepat: Ideal untuk VPS kecil seperti 1–2GB RAM. NGINX tidak membuat proses baru per koneksi, jadi sangat efisien.

- Aman Secara Default: Konfigurasi default-nya relatif aman, dan mudah di-hardening untuk memblokir serangan umum seperti directory listing, file hidden, dan request aneh.

- Support PHP via PHP-FPM: NGINX tidak menjalankan PHP secara langsung. Kita menghubungkannya ke PHP-FPM (FastCGI Process Manager) agar eksekusi script PHP lebih cepat dan terisolasi.

- Kompatibel dengan Let's Encrypt: NGINX mudah diintegrasikan dengan SSL dari Let's Encrypt, termasuk auto-renewal.

- Mendukung Static File Caching: File statis seperti gambar, CSS, dan JavaScript bisa di-cache langsung oleh NGINX tanpa perlu akses ke PHP, sehingga meningkatkan performa drastis.

## System Preparation
### Update & Upgrade
```bash
root@barak# apt update && apt upgrade -y
```

### Install Dependencies
```bash
root@barak:~# apt install curl wget gnupg2 software-properties-common apt-transport-https -y
```

## Instalasi NGINX
### Install
```bash
root@barak:~# apt install nginx -y
root@barak:~# systemctl enable --now nginx
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
root@barak:~# systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-07-28 21:38:32 WIB; 43s ago
       Docs: man:nginx(8)
   Main PID: 54990 (nginx)
      Tasks: 2 (limit: 2318)
     Memory: 1.7M (peak: 3.7M)
        CPU: 6ms
     CGroup: /system.slice/nginx.service
             ├─54990 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─54992 "nginx: worker process"

Jul 28 21:38:32 barak systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Jul 28 21:38:32 barak systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

### Firewall Setup
```bash
root@barak:~# ufw allow 'Nginx Full'
Rule added
Rule added (v6)
root@barak:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
2025/tcp                   ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
2025/tcp (v6)              ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

### Test
**cek ip**
```bash
root@barak:~# hostname -I
165.101.18.19
```

**Akses `http://165.101.18.19` di browser**

<img width="1819" height="850" alt="image" src="https://github.com/user-attachments/assets/9a4a39ef-a9ec-493e-abde-3824b03fb683" />

## Konfigurasi
### Hardening NGINX
#### edit
```bash
root@barak:~# nano /etc/nginx/nginx.conf
```

#### ganti dengan:
```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 64M;
    
    # Security Headers
    server_tokens off;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
    
    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=login:10m rate=10r/m;
    limit_req_zone $binary_remote_addr zone=general:10m rate=1r/s;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/javascript application/xml+rss application/json;
    
    # Include server configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```
### Penjelasan Konfigurasi `nginx.conf`
#### **Global Settings**

* `user www-data;` → Menjalankan proses NGINX sebagai user `www-data` (standar di Ubuntu).
* `worker_processes auto;` → Otomatis menyesuaikan jumlah proses kerja dengan jumlah CPU.
* `pid /run/nginx.pid;` → Lokasi file PID untuk proses utama NGINX.

#### **Events Block**

* `worker_connections 1024;` → Maksimal 1024 koneksi simultan per worker.
* `use epoll;` → Gunakan metode I/O event terbaik di Linux.
* `multi_accept on;` → Terima banyak koneksi sekaligus dalam satu waktu — meningkatkan performa.

#### **HTTP Block**

##### *Basic Settings*

* `sendfile`, `tcp_nopush`, `tcp_nodelay` → Optimasi transmisi file dan TCP latency.
* `keepalive_timeout 65;` → Waktu tunggu koneksi keep-alive.
* `client_max_body_size 64M;` → Maksimal ukuran file upload (64 MB).

##### *Security Headers*

* Menambah header keamanan seperti:

  * `X-Frame-Options` (blok clickjacking),
  * `X-Content-Type-Options` (hindari MIME sniffing),
  * `X-XSS-Protection` (proteksi XSS),
  * `Referrer-Policy` (kontrol header referer).

##### *Rate Limiting*

* `limit_req_zone` → Batas request per IP:

  * 10 request per menit (login)
  * 1 request per detik (umum)

##### *Logging*

* Format log `main` dibuat detail, termasuk IP, user agent, referer, dsb.
* Lokasi log:

  * Akses: `/var/log/nginx/access.log`
  * Error: `/var/log/nginx/error.log`

##### *MIME Types*

* `include /etc/nginx/mime.types` → Mapping ekstensi file ke jenis konten.
* `default_type` → Default ke `application/octet-stream` jika tidak dikenali.

##### *Gzip Compression*

* Aktifkan gzip untuk memperkecil ukuran transfer data (file teks, JSON, CSS, JS, dll) dan mempercepat loading.

##### *Server Config Include*

* `include /etc/nginx/conf.d/*.conf;` → Load config tambahan.
* `include /etc/nginx/sites-enabled/*;` → Load virtual host yang aktif.

### SSL Parameters Snippet (jika generate ssl manual)
SSL Parameters Snippet adalah file konfigurasi tambahan pada NGINX yang berisi pengaturan keamanan SSL/TLS tingkat lanjut, seperti pemilihan cipher suite, protokol TLS yang diizinkan, dan pengaktifan fitur keamanan seperti HSTS atau OCSP stapling. Snippet ini dibuat agar bisa digunakan ulang (include) di banyak virtual host, sehingga menjaga konsistensi dan mempermudah manajemen konfigurasi SSL secara terpusat dengan tetap mengikuti praktik keamanan terbaik.

#### Buat file
```bash
root@barak:~# nano /etc/nginx/snippets/ssl-params.conf
```

#### Isi dengan:
```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
ssl_ecdh_curve secp384r1;
ssl_session_timeout 10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
```

### Penjelasan Konfigurasi `ssl-params.conf`

* `ssl_protocols TLSv1.2 TLSv1.3;`
  
  Hanya mengizinkan protokol TLS versi aman (1.2 dan 1.3), menolak versi lama yang rentan.

* `ssl_prefer_server_ciphers on;`
  
  Server memprioritaskan cipher yang lebih aman daripada yang ditawarkan klien.

* `ssl_ciphers ...;`
  
  Mendefinisikan cipher suite kuat berbasis AES-GCM dan ECDHE/DHE untuk forward secrecy.

* `ssl_ecdh_curve secp384r1;`
  
  Menentukan kurva ECDH untuk keamanan pertukaran kunci.

* `ssl_session_timeout 10m;`
  
  Waktu maksimal sesi SSL yang bisa dipakai ulang.

* `ssl_session_cache shared:SSL:10m;`
  
  Mengaktifkan cache sesi SSL untuk mempercepat koneksi berulang.

* `ssl_session_tickets off;`
  
  Menonaktifkan session ticket untuk menghindari risiko reuse key material.

* `ssl_stapling on;`
  
  Mengaktifkan OCSP stapling untuk validasi sertifikat real-time dari CA.

* `ssl_stapling_verify on;`
  
  Memastikan validasi OCSP dilakukan dengan aman.

* `resolver 8.8.8.8 8.8.4.4 valid=300s;`
  
  Mengatur DNS resolver untuk validasi OCSP.

* `resolver_timeout 5s;`
  
  Timeout maksimal untuk query DNS.

* `add_header Strict-Transport-Security ...;`
  
  Mengaktifkan HSTS: memaksa browser hanya menggunakan HTTPS selama 2 tahun untuk domain dan subdomain (termasuk preload list).

## NGINX Server Block
NGINX Server Block adalah bagian dari konfigurasi NGINX yang menentukan bagaimana server merespons permintaan berdasarkan nama domain, IP, atau port tertentu. Setiap server {} block bisa dianggap sebagai virtual host yang menangani satu atau lebih domain, dan di dalamnya lo bisa atur root folder, SSL, redirect, proxy, log, hingga rule keamanan. Server block ini memungkinkan satu NGINX instance untuk melayani banyak situs dengan konfigurasi berbeda-beda.

### Buat Direktori Web
Dalam setup ini, aku membuat dua server block NGINX yang masing-masing melayani direktori website berbeda berdasarkan subdomain, contoh: app1.domain.com dan app2.domain.com. Tiap subdomain diarahkan ke folder terpisah (/var/www/app1.domain.com/public_html dan /var/www/app2.domain.com/public_html), sehingga isolasi file dan log. Pendekatan ini bikin manajemen banyak web app dalam satu VPS lebih rapi, aman, dan scalable.

```bash
root@barak:~# mkdir -p /var/www/{app1.klandestin.site,app2.klandestin.site}/{public_html,logs}
root@barak:~# chown -R www-data:www-data /var/www/
root@barak:~# chmod -R 755 /var/www/
root@barak:~# cat << EOF > /var/www/app1.klandestin.site/public_html/index.html
<h1>Welcome to app1 site.</h1>
EOF
root@barak:~# cat << EOF > /var/www/app2.klandestin.site/public_html/index.html
<h1>Welcome to app2 site.</h1>
EOF
```

### Konfigurasi Server Block app1
#### buat file
```bash
root@barak:~# nano /etc/nginx/sites-available/app1
```

#### isi dengan:
```bash
server {
    listen 80;
    server_name app1.klandestin.site;
    
    root /var/www/app1.klandestin.site/public_html;
    index index.php index.html index.htm;
    
    # Security
    location ~ /\. {
        deny all;
    }
    
    location ~* \.(git|svn|hg|bzr) {
        deny all;
    }
    
    # Rate limiting
    location /wp-login.php {
        limit_req zone=login burst=5 nodelay;
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    
    location / {
        limit_req zone=general burst=10 nodelay;
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors on;
    }
    
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    access_log /var/www/app1.klandestin.site/logs/access.log;
    error_log /var/www/app1.klandestin.site/logs/error.log;
}
```

#### Symlink
```bash
root@barak:/var/www# ln -s /etc/nginx/sites-available/app1 /etc/nginx/sites-enabled/
```

### Konfigurasi Server Block app2
#### buat file
```bash
root@barak:~# nano /etc/nginx/sites-available/app2
```

#### isi dengan:
```bash
server {
    listen 80;
    server_name app2.klandestin.site;
    
    root /var/www/app2.klandestin.site/public_html;
    index index.php index.html index.htm;
    
    # Security
    location ~ /\. {
        deny all;
    }
    
    location ~* \.(git|svn|hg|bzr) {
        deny all;
    }
    
    # Rate limiting
    location /wp-login.php {
        limit_req zone=login burst=5 nodelay;
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    
    location / {
        limit_req zone=general burst=10 nodelay;
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors on;
    }
    
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    access_log /var/www/app2.klandestin.site/logs/access.log;
    error_log /var/www/app2.klandestin.site/logs/error.log;
}
```

#### Symlink
```bash
root@barak:/var/www# ln -s /etc/nginx/sites-available/app2 /etc/nginx/sites-enabled/
```

### Cek konfigurasi & Reload
```bash
root@barak:~# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@barak:~# systemctl reload nginx
```

## Create & Pointer Domain
### Create A Record
Pergi DNS Manager

**app1.klandestin.site**

<img width="1257" height="602" alt="image" src="https://github.com/user-attachments/assets/6516d830-e295-4333-b876-af5f0c8e5cd8" />

**app2.klandestin.site**

<img width="1254" height="609" alt="image" src="https://github.com/user-attachments/assets/4dc23237-097b-458f-9940-bb085a06e2db" />

### Testing
**app1.klandestin.site**

<img width="1819" height="849" alt="image" src="https://github.com/user-attachments/assets/91f503dd-d8a9-4c5c-a129-20a417196f99" />

**app2.klandestin.site**

<img width="1819" height="845" alt="image" src="https://github.com/user-attachments/assets/25af07ca-80bb-4d7c-a3bd-3439d84c0c0f" />

## SSL dengan Let's Encrypt
### Install Certbot
```bash
root@barak:/var/www# apt install certbot python3-certbot-nginx -y
```

### Generate SSL
```bash
root@barak:/var/www# certbot --nginx -d app1.klandestin.site
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): admin@klandestin.site

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
Account registered.
Requesting a certificate for app1.klandestin.site

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/app1.klandestin.site/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/app1.klandestin.site/privkey.pem
This certificate expires on 2025-10-26.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for app1.klandestin.site to /etc/nginx/sites-enabled/app1
Congratulations! You have successfully enabled HTTPS on https://app1.klandestin.site

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
**Lakukan juga pada `app2.klandestin.site`**
