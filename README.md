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
13. btrfs subvolume show /data/.snapshots/dbs_files_20230527_073900
14. tree -a /data
15. cat "hello world new" > /data/dbs/files/satu.txt
16. cat "hello world new" > /data/dbs/files/dua.txt
17. cat /data/.snapshots/dbs_files_20230527_073900/satu.txt
18. cat /data/.snapshots/dbs_files_20230527_073900/dua.txt

recover file dari snapshot
19. cp /data/.snapshots/dbs_files_20230527_073900/satu.txt /data/dbs/files/satu.txt
20. cat /data/dbs/files/satu.txt

mengupdate data terbaru ke snapshot
21. cp /data/dbs/files/dua.txt /data/.snapshots/dbs_files_20230527_073900/dua.txt
22. cat /data/.snapshots/dbs_files_20230527_073900/dua.txt


mengembalikan keseluruhan snapshot
23. cat "hello world newest" > /data/dbs/files/satu.txt
24. cat "Hello world different" > /data/dbs/files/tiga.txt
25. rsync -avz /data/.snapshots/dbs_files_20230527_073900 /data/dbs/files
26. tree -a /data

mengembalikan keseluruhan snapshot (mirror mode, ini akan menghapus tiga.txt)
27. rsync -avz --delete /data/.snapshots/dbs_files_20230527_073900 /data/dbs/files
23. tree -a /data

Kesimpulan, suatu snapshot writable merupakan suatu subvolume, sehingga kita dapat melakukan
perubahan dengan operasi copy biasa.

--- readonly snapshot (-r) ---
1. btrfs subvolume snapshot -r /data/dbs/files /data/.snapshots/dbs_files_20230527_133900
2. btrfs subvolume show /data/.snapshots/dbs_files_20230527_133900

-- menghapus snapshot ---
1. btrfs subvolume delete /data/.snapshots/dbs_files_20230527_073900
2. btrfs subvolume list /data
3. tree -a /data
