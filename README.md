# homework 2
1. добавить в Vagrantfile еще дисков
2. сломать/починить raid;
3. собрать R0/R5/R10 на выбор;
4. прописать собранный рейд в конфиг, чтобы рейд собирался при загрузке;
5. создать GPT раздел и 5 партиций. В качестве проверки принимаются - измененный Vagrantfile, скрипт для создания рейда, конф для автосборки рейда при загрузке.
6. Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. После перезагрузки стенда разделы должны автоматически примонтироваться.
7. перенести работающую систему с одним диском на RAID 1. Даунтайм на загрузку с нового диска предполагается. В качестве проверки принимается вывод команды lsblk до и после и описание хода решения (можно воспользоваться утилитой Script).

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

 Большую часть выполнил по методичке, так что она врядли будет сильно отличаться.
 Проверяем наличие дисков командами
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
