# btrfs_snapshot
Langkah-langkah membuat btrfs snapshot

1. Buat partisi baru
2. sudo mkfs.btrfs -L data /dev/sdb -f
3. mkdir /data
4. mount /dev/sdb /data
5. df -h /data
6. mkdir -v /data/dbs
7. btrfs subvolume create /data/dbs/files
8. cat "hello world" > /data/dbs/files/satu.txt
9. cat "hello world" > /data/dbs/files/dua.txt
10. mkdir -v /data/.snapshots

--- writable snapshot ---

11. btrfs subvolume snapshot /data/dbs/files /data/.snapshots/dbs_files_20230527_073900
12. ls /data/.snapshots
13. btrfs subvolume list /data
14. btrfs subvolume show /data/.snapshots/dbs_files_20230527_073900
15. tree -a /data
16. cat "hello world new" > /data/dbs/files/satu.txt
17. cat "hello world new" > /data/dbs/files/dua.txt
18. cat /data/.snapshots/dbs_files_20230527_073900/satu.txt
19. cat /data/.snapshots/dbs_files_20230527_073900/dua.txt

mengembalikan file dari snapshot dengan copy

20. cp /data/.snapshots/dbs_files_20230527_073900/satu.txt /data/dbs/files/satu.txt
21. cat /data/dbs/files/satu.txt

mengupdate data terbaru ke snapshot (pastikan sisa space > 50% kalau file besar)

22. cp /data/dbs/files/dua.txt /data/.snapshots/dbs_files_20230527_073900/dua.txt
23. cat /data/.snapshots/dbs_files_20230527_073900/dua.txt

mengembalikan keseluruhan snapshot dengan rsync (pastikan sisa space > 50% kalau file besar)

24. cat "hello world newest" > /data/dbs/files/satu.txt
25. cat "Hello world different" > /data/dbs/files/tiga.txt
26. rsync -avz /data/.snapshots/dbs_files_20230527_073900/ /data/dbs/files/
27. tree -a /data

mengembalikan keseluruhan snapshot (mirror mode, ini akan menghapus tiga.txt)

28. rsync -avz --delete /data/.snapshots/dbs_files_20230527_073900/ /data/dbs/files/
29. tree -a /data

mengembalikan data dengan mount berdasarkan VolId (Cepat)

31. btrfs subvolume list /data
32. catat id subvolume yang akan dikembalikan, misalkan 261
33. umount /data
35. mkdir /data/dbs/files
36. mount -o subvolid=<id subvolume> /dev/sdb /data/dbs/files
37. mount -o subvolid=361 /dev/sdb /data/dbs/files
38. cat /data/dbs/files/satu.txt

Kesimpulan, suatu snapshot writable merupakan suatu subvolume, sehingga kita dapat melakukan
perubahan dengan operasi copy biasa, rsync, maupun cukup melakukan mount dengan menggunakan
subvolid.

--- readonly snapshot (-r) ---

1. btrfs subvolume snapshot -r /data/dbs/files /data/.snapshots/dbs_files_20230527_133900
2. btrfs subvolume show /data/.snapshots/dbs_files_20230527_133900

--- menghapus snapshot ---

6. btrfs subvolume delete /data/.snapshots/dbs_files_20230527_073900
7. btrfs subvolume list /data
8. tree -a /data

--- auto mount ---
1. lsblk -o NAME,FSTYPE,UUID,MOUNTPOINTS
2. pico /etc/fstab
3. isikan baris baru UUID=\<UUID\> \<MOUNTPOINTS\> \<FSTYPE\>  defaults        0       0

#btrfs mariadb

sudo btrfs subvolume list /

#membuat subvolume btrfs
```
sudo systemctl stop mariadb
sudo mv /var/lib/mysql /var/lib/mysql.old
```
## buat subvolume baru
sudo btrfs subvolume create /var/lib/mysql
## copy dari old ke subvolume baru
sudo cp -a /var/lib/mysql.old/* /var/lib/mysql/
sudo rm -rf /var/lib/mysql.old
sudo systemctl start mariadb

#membuat snapshot harian
sudo systemctl stop mariadb
sudo btrfs subvolume snapshot -r /var/lib/mysql /btrfs-snapshots/mysql-$(date +%Y%m%d-%H%M%S)
sudo systemctl start mariadb

#menampilkan daftar subvolume snapshot
sudo btrfs subvolume list /

#menampilkan isi subvolume snapshot tertentu
ls /btrfs-snapshots/mysql-20251008-090000

#menghapus snapshot yang tidak dibutuhkan lagi (melepas kapasitas drive)
sudo btrfs subvolume delete /btrfs-snapshots/mysql-20251001-120000

#memeriksa kapasitas drive
sudo btrfs filesystem du -s /var/lib/mysql /btrfs-snapshots/*

#restore ke snapshot tertentu
sudo systemctl stop mariadb
sudo systemctl is-active mariadb
##backup subvolume saat ini
sudo btrfs subvolume snapshot -r /var/lib/mysql /btrfs-snapshots/mysql-before-rollback-$(date +%Y%m%d-%H%M%S)

##hapus subvolume saat ini
sudo btrfs subvolume delete /var/lib/mysql

##kembalikan dari snapshot ke /var/lib/mysql
sudo btrfs subvolume snapshot /btrfs-snapshots/mysql-20251008-090000 /var/lib/mysql

##pulihkan ownership jika perlu
sudo chown -R mysql:mysql /var/lib/mysql
sudo systemctl start mariadb
sudo systemctl start mariadb



