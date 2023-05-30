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

recover file dari snapshot dengan copy

20. cp /data/.snapshots/dbs_files_20230527_073900/satu.txt /data/dbs/files/satu.txt
21. cat /data/dbs/files/satu.txt

mengupdate data terbaru ke snapshot

22. cp /data/dbs/files/dua.txt /data/.snapshots/dbs_files_20230527_073900/dua.txt
23. cat /data/.snapshots/dbs_files_20230527_073900/dua.txt

mengembalikan keseluruhan snapshot dengan rsync

24. cat "hello world newest" > /data/dbs/files/satu.txt
25. cat "Hello world different" > /data/dbs/files/tiga.txt
26. rsync -avz /data/.snapshots/dbs_files_20230527_073900/ /data/dbs/files/
27. tree -a /data

mengembalikan keseluruhan snapshot (mirror mode, ini akan menghapus tiga.txt)

28. rsync -avz --delete /data/.snapshots/dbs_files_20230527_073900/ /data/dbs/files/
29. tree -a /data

mengembalikan data dengan mount berdasarkan VolId

30. umount /data
31. btrfs subvolume list /data
32. catat id subvolume yang akan dikembalikan, misalkan 261
33. mkdir /data/dbs/files
34. mount -o subvolid=<id subvolume> /data/.snapshot/dbs_files_20230527_073900 /data/dbs/files
35. mount -o subvolid=361 /data/.snapshot/dbs_files_20230527_073900 /data/dbs/files
36. cat /data/dbs/files/satu.txt

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

