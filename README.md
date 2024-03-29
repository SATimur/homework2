# homework 2
1. добавить в Vagrantfile еще дисков
2. сломать/починить raid;
3. собрать R0/R5/R10 на выбор;
4. прописать собранный рейд в конфиг, чтобы рейд собирался при загрузке;
5. создать GPT раздел и 5 партиций. В качестве проверки принимаются - измененный Vagrantfile, скрипт для создания рейда, конф для автосборки рейда при загрузке.
6. Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. После перезагрузки стенда разделы должны автоматически примонтироваться.

## Добавить в Vagrantfile еще дисков

1. Берем тестовый стенд от сюда: [ https://github.com/erlong15/otus-linux ]
2. Добовляем еще два диска в Vagrantfile:
```  :box_name => "centos/7",
        :ip_addr => '192.168.56.2/21',
        :disks => {
                :sata1 => {
                        :dfile => './sata1.vdi',
                        :size => 500,
                        :port => 1
                },
                :sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 500, # Megabytes
                        :port => 2
                },
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 500,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 500, # Megabytes
                        :port => 4
                },
                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 500,
                        :port => 5
                },
                :sata6 => {
                        :dfile => './sata6.vdi',
                        :size => 500, # Megabytes
                        :port => 6
                }
```
3. Запускаем Vagrant и подключаемся к виртуалке с создаными дисками
   ``` vagrant ssh otuslinux ```
   
## собрать R0/R5/R10

1. Большую часть выполнил по методичке, так что она врядли будет сильно отличаться.
   Проверяем наличие дисков командами, кстати, у меня эти команды выполняются строго из под root пользователя
   с командой sudo работать отказываются, поэтому переключаемся на root и работаем из под него.
 ``` 
 [root@otuslinux mdadm]#  sudo lshw -short | grep disk
/0/100/1.1/0.0.0    /dev/sdg   disk        42GB VBOX HARDDISK
/0/100/d/0          /dev/sda   disk        524MB VBOX HARDDISK
/0/100/d/1          /dev/sdb   disk        524MB VBOX HARDDISK
/0/100/d/2          /dev/sdc   disk        524MB VBOX HARDDISK
/0/100/d/3          /dev/sdd   disk        524MB VBOX HARDDISK
/0/100/d/4          /dev/sde   disk        524MB VBOX HARDDISK
/0/100/d/5          /dev/sdf   disk        524MB VBOX HARDDISK 
```
```
[vagrant@otuslinux ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  500M  0 disk
sdc      8:32   0  500M  0 disk
sdd      8:48   0  500M  0 disk
sde      8:64   0  500M  0 disk
sdf      8:80   0  500M  0 disk
sdg      8:96   0  500M  0 disk
```
2. Зануляем блоки командой 
   ```
   mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
   ```
3. Создаем 6-й рейд командой 
   ```
   mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
   ```
4. Проверяем, собрался ли RAID командой 
   ```
   [root@otuslinux vagrant]# cat /proc/mdstat
      Personalities : [raid6] [raid5] [raid4]
      md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      1529856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]
   ```
5. Создаем запасной диск (Hot Spare)
   Если в массиве будет запасной диск для горячей замены, при выходе из строя одного из основных дисков, его место займет запасной.
   Диском Hot Spare станет тот, который просто будет добавлен к массиву:
   ```
   mdadm /dev/md0 --add /dev/sdg
   ```
   Проверяем наши диски:
      ```
          [root@otuslinux vagrant]# mdadm -D /dev/md0
          /dev/md0:
                   Version : 1.2
          Creation Time : Mon Feb 21 06:42:42 2022
          Raid Level : raid6
          Array Size : 1529856 (1494.00 MiB 1566.57 MB)
          Used Dev Size : 509952 (498.00 MiB 522.19 MB)
          Raid Devices : 5
          Total Devices : 6
          Persistence : Superblock is persistent

          Update Time : Mon Feb 21 06:59:44 2022
          State : clean
          Active Devices : 5
          Working Devices : 6
          Failed Devices : 0
          Spare Devices : 1

                    Layout : left-symmetric
                Chunk Size : 512K

          Consistency Policy : resync

                     Name : otuslinux:0  (local to host otuslinux)
                     UUID : 63d02509:462bd8ee:1c5ae473:189da4b0
                     Events : 18

         Number   Major   Minor   RaidDevice State
           0       8       16        0      active sync   /dev/sdb
           1       8       32        1      active sync   /dev/sdc
           2       8       48        2      active sync   /dev/sdd
           3       8       64        3      active sync   /dev/sde
           4       8       80        4      active sync   /dev/sdf

           5       8       96        -      spare   /dev/sdg
     ```   
6.Создание конфигурационного файла mdadm.conf
  Снаùала убедимся, что информаøиā верна:
  ```
   [root@otuslinux vagrant]# mdadm --detail --scan --verbose
   ARRAY /dev/md0 level=raid6 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=63d02509:462bd8ee:1c5ae473:189da4b0
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf
  ```
 Теперь создадим файл mdadm.conf
 ```
    [root@otuslinux vagrant]# echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
    [root@otuslinux vagrant]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
 ```
7. Сломать/починить RAID
   Ломаем один из дисков в нашем RAID-де, после чего на замену должен встать диск из Hot Spare
   ```
      mdadm /dev/md0 --fail /dev/sdb
   ```
   Проверям наш RAID:
   ```
      [root@otuslinux vagrant]# mdadm /dev/md0 --fail /dev/sdb
      Number   Major   Minor   RaidDevice State
       5       8       96        0      active sync   /dev/sdg
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf

       0       8       16        -      faulty   /dev/sdb
   ```
8. Удалим “сломанный” диск из массива:
   ````
       [root@otuslinux vagrant]# mdadm /dev/md0 --remove /dev/sdb
       mdadm: hot removed /dev/sdb from /dev/md0
   ````
   Добавляем его в RAID
   ````
       [root@otuslinux vagrant]# mdadm /dev/md0 --add /dev/sdb
       mdadm: added /dev/sdb
   ````
   Проверяем результат:
   ````
       [root@otuslinux vagrant]# mdadm -D /dev/md0
        Number   Major   Minor   RaidDevice State
       5       8       96        0      active sync   /dev/sdg
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf

       6       8       16        -      spare   /dev/sdb
   ````
   Видим счто диск успешно заменен. Двигаемся дальше.
   
## Создать GPT раздел, пять партиций и смонтировать их на диск   

1. Создаем раздел GPT на RAID
   ```
      [root@otuslinux vagrant]# mkfs.ext4 -F /dev/md0
      mke2fs 1.42.9 (28-Dec-2013)
      Filesystem label=
      OS type: Linux
      Block size=4096 (log=2)
      Fragment size=4096 (log=2)
      Stride=128 blocks, Stripe width=384 blocks
      95616 inodes, 382464 blocks
      19123 blocks (5.00%) reserved for the super user
      First data block=0
      Maximum filesystem blocks=392167424
      12 block groups
      32768 blocks per group, 32768 fragments per group
      7968 inodes per group
      Superblock backups stored on blocks:
                  32768, 98304, 163840, 229376, 294912

      Allocating group tables: done
      Writing inode tables: done
      Creating journal (8192 blocks): done
      Writing superblocks and filesystem accounting information: done
   ```
2. Создание каталога для монтирования и монтируем его:
   ```
      [root@otuslinux vagrant]# mkdir -p /mnt/md0
      [root@otuslinux vagrant]# mount /dev/md0 /mnt/md0
      [root@otuslinux vagrant]# df -h
      Filesystem      Size  Used Avail Use% Mounted on
      devtmpfs        489M     0  489M   0% /dev
      tmpfs           496M     0  496M   0% /dev/shm
      tmpfs           496M   20M  477M   4% /run
      tmpfs           496M     0  496M   0% /sys/fs/cgroup
      /dev/sda1        40G  8.7G   32G  22% /
      tmpfs           100M     0  100M   0% /run/user/1000
      /dev/md0        1.5G  4.4M  1.4G   1% /mnt/md0
   ```
3. Добавляем конфигурацию mdadm в автозапуск:
   ```
      mdadm --verbose --detail -scan > /etc/mdadm.conf
   ```
## Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. После перезагрузки стенда разделы должны автоматически примонтироваться.

1. Редактируем блок в файле Vagrantfile
   ```
         box.vm.provision "shell", inline: <<-SHELL
              mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
              yum install -y mdadm smartmontools hdparm gdisk
                  mdadm --create --verbose /dev/md0 --level=6 --raid-devices=5 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf
                  mdadm /dev/md0 --add /dev/sdg
                  mkfs.ext4 -F /dev/md0
                  mkdir /mnt/md0
                  mount /dev/md0 /mnt/md0
                  mdadm --verbose --detail -scan > /etc/mdadm.conf
                  echo "/dev/md0    /mnt/md0    ext4    defaults    0    1" >> /etc/fstab
          SHELL
    ```
2. Проверяем:
   ```
      [vagrant@otuslinux ~]$ df -h
      Filesystem      Size  Used Avail Use% Mounted on
      devtmpfs        489M     0  489M   0% /dev
      tmpfs           496M     0  496M   0% /dev/shm
      tmpfs           496M  6.7M  489M   2% /run 
      tmpfs           496M     0  496M   0% /sys/fs/cgroup
      /dev/sda1        40G  7.7G   33G  20% /
      /dev/md0        1.5G  4.4M  1.4G   1% /mnt/md0
      tmpfs           100M     0  100M   0% /run/user/1000     
   ```
  Перезагружаем и снова проверяем:
  ```
     [vagrant@otuslinux ~]$ df -h
     Filesystem      Size  Used Avail Use% Mounted on
     devtmpfs        489M     0  489M   0% /dev
     tmpfs           496M     0  496M   0% /dev/shm
     tmpfs           496M  6.7M  489M   2% /run
     tmpfs           496M     0  496M   0% /sys/fs/cgroup
     /dev/sda1        40G  7.7G   33G  20% /
     /dev/md0        1.5G  4.4M  1.4G   1% /mnt/md0
     tmpfs           100M     0  100M   0% /run/user/1000
  ```
  ```
     [root@otuslinux vagrant]# lsblk
     NAME   MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
     sda      8:0    0   40G  0 disk
     `-sda1   8:1    0   40G  0 part  /
     sdb      8:16   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
     sdc      8:32   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
     sdd      8:48   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
     sde      8:64   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
     sdf      8:80   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
     sdg      8:96   0  500M  0 disk
     `-md0    9:0    0  1.5G  0 raid6 /mnt/md0
  ```
  ```
      [root@otuslinux vagrant]# mdadm -D /dev/md0
        Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf

       5       8       96        -      spare   /dev/sdg
  ```
  
  
