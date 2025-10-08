# Otus
## PostgreSQL Advanced
### Настройка PostgreSQL
#### Рукомойкин Андрей

1. Создайте виртуальную машину с Ubuntu 20.04 и установите PostgreSQL 15 или выше.

* VM configurating:
CPU: 2
RAM: 2GB
hostname:     compute-vm-2-2-20-ssd-1759907864379
Login name: otus
Login details: ssh -l otus 130.193.43.82

1.1 Подключение
ssh -i /var/lib/postgresql/.ssh/hw02t otus@89.169.184.38

otus@compute-vm-2-2-20-ssd-1759907864379:~$ uname -a
Linux compute-vm-2-2-20-ssd-1759907864379 6.8.0-85-generic #85-Ubuntu SMP PREEMPT_DYNAMIC Thu Sep 18 15:26:59 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
otus@compute-vm-2-2-20-ssd-1759907864379:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.3 LTS
Release:        24.04

2. Создайте таблицу с данными о перевозках.

2.1 Установка postgres

sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc

root@compute-vm-2-2-20-ssd-1759907864379:~# su - postgres
postgres@compute-vm-2-2-20-ssd-1759907864379:~$ psql
psql (18.0 (Ubuntu 18.0-1.pgdg24.04+3))
Type "help" for help.

postgres=# \conninfo
           Connection Information
      Parameter       |        Value
----------------------+---------------------
 Database             | postgres
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5432
 Options              |
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 4875
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)

2.2 создать базу данных и таблицу

postgres=# create database wh03;
postgres=# \c wh03;
postgres=# create table wh03table(cargo text);
postgres=# insert into wh03table values('1');

wh03=# select * from wh03table ;
 cargo
-------
 1
(1 row)

2.3 Остановим PostgreSQL
root@compute-vm-2-2-20-ssd-1759907864379:~# sudo -u postgres pg_ctlcluster 18 main stop

3. Добавьте внешний диск к виртуальной машине и перенесите туда базу данных. (https://yandex.cloud/ru/docs/compute/operations/vm-control/vm-attach-disk?from=int-console-help-center-or-nav)

https://console.yandex.cloud/folders/b1gbr66hl3ujn4jhi7r3/compute/instance/epdu8tcr1n2caof1k0v7/disks

Добавление дополнительного диска:
Имя: disk-otus-hw03
Тип: SSD
Размер: 20G

otus@compute-vm-2-2-20-ssd-1759907864379:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     253:0    0   20G  0 disk
├─vda1  253:1    0 19.4G  0 part /
├─vda14 253:14   0    4M  0 part
└─vda15 253:15   0  600M  0 part /boot/efi
vdb     253:16   0   20G  0 disk
otus@compute-vm-2-2-20-ssd-1759907864379:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           197M  1.1M  196M   1% /run
/dev/vda1        19G  2.4G   17G  13% /
tmpfs           984M  1.1M  983M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda15      599M  6.2M  593M   2% /boot/efi
tmpfs           197M  8.0K  197M   1% /run/user/1000

3.1 Проверьте, подключен ли диск как устройство, и узнайте его путь в системе:

root@compute-vm-2-2-20-ssd-1759907864379:~# ls -la /dev/disk/by-id
total 0
drwxr-xr-x 2 root root 140 Oct  8 22:43 .
drwxr-xr-x 8 root root 160 Oct  8 22:43 ..
lrwxrwxrwx 1 root root   9 Oct  8 22:43 virtio-epd718pm9v3a6d2kd6aa -> ../../vda
lrwxrwxrwx 1 root root  10 Oct  8 22:43 virtio-epd718pm9v3a6d2kd6aa-part1 -> ../../vda1
lrwxrwxrwx 1 root root  11 Oct  8 22:43 virtio-epd718pm9v3a6d2kd6aa-part14 -> ../../vda14
lrwxrwxrwx 1 root root  11 Oct  8 22:43 virtio-epd718pm9v3a6d2kd6aa-part15 -> ../../vda15
lrwxrwxrwx 1 root root   9 Oct  8 22:43 virtio-epdugith1o4oftgffa88 -> ../../vdb

3.2 Разметьте диск. 

root@compute-vm-2-2-20-ssd-1759907864379:~# fdisk /dev/vdb

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0xc522d240.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (8192-41943039, default 8192):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (8192-41943039, default 41943039):

Created a new partition 1 of type 'Linux' and of size 20 GiB.

Command (m for help): p
Disk /dev/vdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4194304 bytes
Disklabel type: dos
Disk identifier: 0xc522d240

Device     Boot Start      End  Sectors Size Id Type
/dev/vdb1        8192 41943039 41934848  20G 83 Linux
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.


3.3 Отформатируйте раздел в нужную файловую систему.

root@compute-vm-2-2-20-ssd-1759907864379:~# mkfs.ext4 /dev/vdb1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 5241856 4k blocks and 1310720 inodes
Filesystem UUID: 81f4396d-ce73-43a5-afda-e8ab06987c56
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

3.4 Смонтируйте раздел диска с помощью утилиты mount. 

root@compute-vm-2-2-20-ssd-1759907864379:~# mkdir /data/hw03 &&  mount /dev/vdb1 /data/hw03
root@compute-vm-2-2-20-ssd-1759907864379:~# cd /data/hw03
root@compute-vm-2-2-20-ssd-1759907864379:/data/hw03# df -h .
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdb1        20G   24K   19G   1% /data/hw03

3.5 Настройте разрешения на чтение и запись в разделе с помощью утилиты chmod

chmod a+w /data/hw03

3.6 Настройте автоматическое монтирование раздела в директорию mnt/new_disk при запуске ВМ:

nano /etc/fstab
~~~~~~~~~~~~~~~~~~
UUID=81f4396d-ce73-43a5-afda-e8ab06987c56 /mnt/new_disk ext4 defaults 0 2

3.7 Restart и проверка

root@compute-vm-2-2-20-ssd-1759907864379:~# df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        19G  2.4G   17G  13% /

3.8 Выдача прав

root@compute-vm-2-2-20-ssd-1759907864379:~# chown -R postgres:postgres /mnt/new_disk

3.9 перенесем кластер целиком 

mv /var/lib/postgresql/18 /mnt/data

3.10 Попытаюсь стартануть. 

root@compute-vm-2-2-20-ssd-1759907864379:/mnt/new_disk/18/main# sudo -u postgres pg_ctlcluster 18 main start
Error: /var/lib/postgresql/18/main is not accessible or does not exist

4. Настройте PostgreSQL для работы с новым диском.

root@compute-vm-2-2-20-ssd-1759907864379:/etc/postgresql/18/main# grep data_directory  postgresql.conf
data_directory = '/var/lib/postgresql/18/main'          # use data in another directory

root@compute-vm-2-2-20-ssd-1759907864379:/etc/postgresql/18/main# nano postgresql.conf

root@compute-vm-2-2-20-ssd-1759907864379:/etc/postgresql/18/main# grep data_directory  postgresql.conf
data_directory = '/mnt/new_disk/18/main'                # use data in another directory

4.1 Start postgres

root@compute-vm-2-2-20-ssd-1759907864379:/etc/postgresql/18/main# sudo -u postgres pg_ctlcluster 18 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@18-main
Removed stale pid file.
root@compute-vm-2-2-20-ssd-1759907864379:/etc/postgresql/18/main# pg_lsclusters
Ver Cluster Port Status Owner    Data directory        Log file
18  main    5432 online postgres /mnt/new_disk/18/main /var/log/postgresql/postgresql-18-main.log

5. Проверьте, что данные сохранились и доступны.

postgres@compute-vm-2-2-20-ssd-1759907864379:~$ psql
psql (18.0 (Ubuntu 18.0-1.pgdg24.04+3))
Type "help" for help.

postgres=# \l
                                                 List of databases
   Name    |  Owner   | Encoding | Locale Provider | Collate |  Ctype  | Locale | ICU Rules |   Access privileges
-----------+----------+----------+-----------------+---------+---------+--------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           |
 template0 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |         |         |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |         |         |        |           | postgres=CTc/postgres
 wh03      | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           |
(4 rows)

postgres=# \c wh03
You are now connected to database "wh03" as user "postgres".
wh03=# \dt
            List of tables
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | wh03table | table | postgres
(1 row)













