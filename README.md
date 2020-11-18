1) Клонирование Vagrantfile из репозитория https://github.com/erlong15/otus-linux
2) Добавление дисков:
    1.Добавляю в Vagrantfile следующие строки
    ,
    :sata5 => {

   :dfile => './sata5.vdi',
   :size => 250, # Megabytes
   :port => 5 
}
3) Создние Raid6: 

>sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}

4) Проверка результата: sudo mdadm -D /dev/md0
5) Создание файла mdadm.conf:

>sudo su

>mkdir /etc/mdadm/

>touch mdadm.conf

>echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

>mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
    
6) Вывод из строя массива: 

>mdadm /dev/md0 --fail /dev/sdc

7) Восстанновление рейда:

>mdadm --remove /dev/md0 /dev/sdc

>mdadm --add /dev/md0 /dev/sdc
    
8) Создание GPT раздела
>umount /dev/md0
>parted -s /dev/md0 mklabel gpt

>parted /dev/md0 mkpart primary ext4 0% 25%

>parted /dev/md0 mkpart primary ext4 25% 50%

>parted /dev/md0 mkpart primary ext4 50% 75%

>parted /dev/md0 mkpart primary ext4 75% 100%

9)  Редактирование Vagrantfile для автоматической сборки Raid6 - добавление в секцию box.vm.provision следующих строк:
>mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}

>mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}

>mkdir /etc/mdadm

>echo "DEVICE partitions" > /etc/mdadm/mdadm.conf

>mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

>parted -s /dev/md0 mklabel gpt

>parted /dev/md0 mkpart primary ext4 0% 25%

>parted /dev/md0 mkpart primary ext4 25% 50%

>parted /dev/md0 mkpart primary ext4 50% 75%

>parted /dev/md0 mkpart primary ext4 75% 100%

>for i in $(seq 1 4); do sudo mkfs.ext4 /dev/md0p$i; done

>mkdir -p /raid/part{1,2,3,4}

>for i in $(seq 1 4); do mount /dev/md0p$i /raid/part$i; done
