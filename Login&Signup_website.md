## Step-by-step Deploy Manual
### 1. Persiapkan Source Code Website

### 2. Copy ke server
```bash
C:\Users\iyhor\OneDrive\Desktop\Projek Sysadmin>scp -P 2025 -r "Website Sederhana"/* dapit@165.101.18.19:/home/dapit/WebsiteSederhana
dapit@165.101.18.19's password:
root@barak:~# cp -r /home/dapit/WebsiteSederhana/* /var/www/app1.klandestin.site/public_html/
root@barak:~# chown -R www-data:www-data /var/www/app1.klandestin.site/public_html/*
root@barak:~# chmod -R 755 /var/www/app1.klandestin.site/public_html/*
```

### 3. Create user & Database
```sql
root@barak:~# mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 31
Server version: 10.11.13-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE WebsiteSederhana_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> CREATE USER 'WebsiteSederhana'@'localhost' IDENTIFIED BY 'PasswordKuat123';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON WebsiteSederhana_db.* TO 'WebsiteSederhana'@'lo
calhost';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> EXIT;
Bye
root@barak:~#
```

### 4. Setting Connection.php
```bash
root@barak:~# nano /var/www/app1.klandestin.site/public_html/connection.php

<?php

$server = "localhost";
$username = "WebsiteSederhana";
$password = "PasswordKuat123";
$db = "WebsiteSederhana_db";

$conn = new mysqli($server, $username, $password, $db);

?>
```

### 5. Reload Service
```bash
root@barak:~# systemctl reload nginx
```

### 6. Testing
**Akses `https://app1.klandestin.site`**

<img width="1719" height="754" alt="image" src="https://github.com/user-attachments/assets/c380095e-68f5-4909-b6fe-74455376d0cc" />
