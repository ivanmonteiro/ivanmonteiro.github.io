---
layout: post
title:  "MariaDB Backup and Restore with Docker"
categories: dotnet-core
---

In the realm of web development, safeguarding your data is paramount. Docker simplifies MariaDB database management, making backup and restore operations a breeze with just two commands.

## Backup MariaDB Database

Execute the following command to back up your MariaDB database:

`docker exec mariadb sh -c 'exec mysqldump -uroot -p"$MARIADB_ROOT_PASSWORD" mautic --default-character-set=utf8mb4' > dump.sql`

Replace your_database_name with your database's name. Enter the MariaDB root password when prompted. The command exports the database to backup.sql in the current directory.

## Restore MariaDB Database

To restore the database from the backup, use this command:
`cat dump.sql | docker exec -i mariadb /usr/bin/mysql -uroot -p"$MARIADB_ROOT_PASSWORD" replace_name_database`

Replace replace_name_database with your desired database name. This command reads and restores the database from backup.sql.

## Conclusion
Harnessing Docker for MariaDB management streamlines the backup and restore process. Whether you're a seasoned developer or new to Docker, these commands offer a simple yet powerful solution for efficient database operations. Tailor the commands to fit your specific setup and enjoy a hassle-free approach to database management.