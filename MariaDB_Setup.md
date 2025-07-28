## MariaDB ?
Yo, Root Lords! MariaDB adalah sistem manajemen basis data relasional (RDBMS) yang bersifat open-source dan dikembangkan sebagai pengganti langsung dari MySQL. Proyek ini dibuat oleh pengembang asli MySQL setelah MySQL diakuisisi oleh Oracle, dengan tujuan menjaga agar basis data tetap bebas dan terbuka. MariaDB sepenuhnya kompatibel dengan MySQL, baik dari sisi perintah SQL, API, maupun struktur data, sehingga pengguna MySQL dapat migrasi ke MariaDB tanpa perubahan besar.

## Kenapa MariaDB ?
Dalam stack LEMP (Linux, NGINX, MariaDB, PHP), MariaDB berfungsi sebagai tulang punggung penyimpanan data — mulai dari menyimpan konfigurasi aplikasi, data pengguna, hingga transaksi penting. Kombinasi MariaDB dengan NGINX dan PHP menghasilkan sistem web server yang ringan, cepat, dan scalable. Biasanya MariaDB akan dikonfigurasi untuk menerima koneksi lokal dari PHP-FPM atau aplikasi yang berjalan di server yang sama

## Instalasi MariaDB
```bash
root@barak:~# apt update
root@barak:~# apt install mariadb-server mariadb-client -y
```

## Secure Installation
```bash
root@barak:~# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

## Secure Konfigurasi
### Edit
```bash
root@barak:~# nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

### Ubah/Tambahkan
```bash
[mysqld]
bind-address = 127.0.0.1
max_connections = 100
innodb_buffer_pool_size = 256M
innodb_log_file_size = 64M
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 2
query_cache_type = 1
query_cache_size = 32M
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

Berikut penjelasan singkat dan tepat untuk dokumentasi GitHub terkait konfigurasi `mysqld` pada MariaDB:

---

#### Penjelasan Konfigurasi

```ini
[mysqld]
bind-address = 127.0.0.1
```

* Membatasi MariaDB hanya menerima koneksi dari **localhost**, meningkatkan keamanan server database.

```ini
max_connections = 100
```

* Maksimum **100 koneksi klien** yang bisa terhubung secara bersamaan ke MariaDB.

```ini
innodb_buffer_pool_size = 256M
```

* Ukuran buffer utama untuk menyimpan data dan indeks InnoDB. Semakin besar nilainya, semakin baik performa (idealnya 60–80% dari total RAM pada server dedicated DB).

```ini
innodb_log_file_size = 64M
```

* Ukuran file log redo InnoDB. Berpengaruh pada recovery dan performa transaksi.

```ini
innodb_file_per_table = 1
```

* Setiap tabel InnoDB akan disimpan di file `.ibd` terpisah, membuat manajemen dan backup lebih fleksibel.

```ini
innodb_flush_log_at_trx_commit = 2
```

* Menulis log ke disk setiap detik, bukan setiap transaksi. Menyeimbangkan antara **keamanan data** dan **performa**.

```ini
query_cache_type = 1
query_cache_size = 32M
```

* Mengaktifkan cache query dan mengalokasikan 32MB untuk menyimpan hasil query SELECT, **mengurangi beban query berulang**. (Catatan: di MariaDB query cache masih didukung, berbeda dengan MySQL 8+ yang menghapus fitur ini.)

```ini
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

* Mengaktifkan pencatatan **slow queries** (query lambat) yang butuh waktu lebih dari 2 detik untuk dijalankan. Sangat berguna untuk **identifikasi bottleneck kinerja**.

## Restart Service
```bash
root@barak:~# systemctl enable --now mariadb
root@barak:~# systemctl status mariadb
● mariadb.service - MariaDB 10.11.13 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-07-29 00:16:53 WIB; 50s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 63384 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 63389 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 63391 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0>
    Process: 63463 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 63465 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 63451 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 12 (limit: 15299)
     Memory: 78.7M (peak: 81.6M)
        CPU: 133ms
     CGroup: /system.slice/mariadb.service
             └─63451 /usr/sbin/mariadbd

Jul 29 00:16:53 barak mariadbd[63451]: 2025-07-29  0:16:53 0 [Note] Plugin 'FEEDBACK' is disabled.
Jul 29 00:16:53 barak mariadbd[63451]: 2025-07-29  0:16:53 0 [Warning] You need to use --log-bin to make --expire-logs-days or --bin>
Jul 29 00:16:53 barak mariadbd[63451]: 2025-07-29  0:16:53 0 [Note] Server socket created on IP: '127.0.0.1'.
Jul 29 00:16:53 barak mariadbd[63451]: 2025-07-29  0:16:53 0 [Note] /usr/sbin/mariadbd: ready for connections.
Jul 29 00:16:53 barak mariadbd[63451]: Version: '10.11.13-MariaDB-0ubuntu0.24.04.1'  socket: '/run/mysqld/mysqld.sock'  port: 3306  >
Jul 29 00:16:53 barak mariadbd[63451]: 2025-07-29  0:16:53 0 [Note] InnoDB: Buffer pool(s) load completed at 250729  0:16:53
Jul 29 00:16:53 barak systemd[1]: Started mariadb.service - MariaDB 10.11.13 database server.
Jul 29 00:16:53 barak /etc/mysql/debian-start[63468]: Upgrading MariaDB tables if necessary.
Jul 29 00:16:53 barak /etc/mysql/debian-start[63479]: Checking for insecure root accounts.
Jul 29 00:16:53 barak /etc/mysql/debian-start[63483]: Triggering myisam-recover for all MyISAM tables and aria-recover for all Aria >
lines 1-28/28 (END)
```

## Testing
```bash
root@barak:~# mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 32
Server version: 10.11.13-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.000 sec)

MariaDB [(none)]>
```
