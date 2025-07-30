## Step-by-step 
### Create User & Database
```bash
root@barak:~# mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 48
Server version: 10.11.13-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE wordpress_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'PasswordKuat123';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> EXIT;
Bye
root@barak:~#
```

### Download Wordpress
```bash
root@barak:~# cd /tmp
root@barak:/tmp# wget https://wordpress.org/latest.tar.gz
--2025-07-29 03:44:01--  https://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 26925441 (26M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

latest.tar.gz                     100%[==========================================================>]  25.68M  8.15MB/s    in 4.2s

2025-07-29 03:44:06 (6.10 MB/s) - ‘latest.tar.gz’ saved [26925441/26925441]

root@barak:/tmp# tar -xzf latest.tar.gz
root@barak:/tmp# sudo mv wordpress/* /var/www/app2.klandestin.site/public_html
```

### Hak Akses
```bash
root@barak:~# chown -R www-data:www-data /var/www/app2.klandestin.site/public_html/
root@barak:~# chmod -R 755 /var/www/app2.klandestin.site/public_html/
```

### Konfigurasi Wordpress
```bash
root@barak:~# cd /var/www/app2.klandestin.site/public_html/
root@barak:/var/www/app2.klandestin.site/public_html# cp wp-config-sample.php wp-config.php
root@barak:/var/www/app2.klandestin.site/public_html# nano wp-config.php
```

isi
```bash
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db' );

/** Database username */
define( 'DB_USER', 'wp_user' );

/** Database password */
define( 'DB_PASSWORD', 'PasswordKuat123' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

### Reload Service
```bash
systemctl reload nginx
```

### Testing
**Akses `https://app2.klandestin.site`**

<img width="1819" height="846" alt="image" src="https://github.com/user-attachments/assets/6aa12b9f-5974-4b1d-af77-b7e2821bb5d5" />

<img width="1819" height="846" alt="image" src="https://github.com/user-attachments/assets/4076b4d1-bddd-4d5d-a427-616cf05101bb" />

<img width="1819" height="851" alt="image" src="https://github.com/user-attachments/assets/88023667-4913-496a-ac7f-6f99404d9ac7" />

<img width="1819" height="848" alt="image" src="https://github.com/user-attachments/assets/349cbe86-4ad5-47fa-a56c-dea1509cabb3" />

<img width="1819" height="854" alt="image" src="https://github.com/user-attachments/assets/8c1b0b8d-8c6f-4c67-9deb-77a94dac0978" />

