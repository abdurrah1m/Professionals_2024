# Модуль Б. (Настройка технических и программных средств информационно-коммуникационных систем)

Время на выполнение модуля 5 часов.  
Доступ к ISP вы не имеете!!  

![image](https://github.com/abdurrah1m/Professionals_09.02.06/assets/148451230/3e4e187a-5fac-4c20-a3a3-ee0f2379b1f8)

| Название устройства | ОС |
|:-|:-|
| RTR-HQ | Eltex vESR |
| RTR-BR | Eltex vESR |
| SRV-HQ | Альт Сервер 10 |
| SRV-BR | Альт Сервер 10 (Допустима замена) Debian |
| CLI-HQ | Альт Рабочая станция 10 |
| CLI-BR | Альт Рабочая станция 10 (Допустима замена) |
| SW-HQ | Альт Сервер 10 |
| SW-BR | Альт Сервер 10 (Допустима замена) |
| CICD-HQ | Альт Сервер 10 |

### HQ

| Подсеть/VLAN | Префикс | Диапазон | Broadband | Размер |
|:-|:-|:-|:-|:-|
| 10.0.10.0 / vlan100 |/27 | 10.0.10.1-30 | 10.0.10.31 | 30 |
| 10.0.10.32 / vlan200 | /27 | 10.0.10.33-62 | 10.0.10.63 | 30 |
| 10.0.10.64 / vlan300 | /27 | 10.0.10.65-94 | 10.0.10.95 | 30 |

### BR

| Подсеть/VLAN | Префикс | Диапазон | Broadband | Размер |
|:-|:-|:-|:-|:-|
| 10.0.20.0 / vlan100 | /27 | 10.0.20.1-30 | 10.0.20.31 | 30 |
| 10.0.20.32 / vlan200 | /27 | 10.0.20.33-62 | 10.0.20.63 | 30 |
| 10.0.20.64 / vlan300 | /27 | 10.0.20.65-94 | 10.0.20.95 | 30 |

| Имя | NIC | IP | Default Gateway |
|:-:|:-:|:-:|:-:|
| RTR-HQ | ISP | 11.11.11.2/30 | 11.11.11.1 |
| | vlan100 | 10.0.10.1/27 | |
| | vlan200 | 10.0.10.33/27 | |
| | vlan300 | 10.0.10.65/27 | |
| RTR-BR | ISP | 22.22.22.2/30 | 22.22.22.1 |
| | vlan100 | 10.0.20.1/27 | |
| | vlan200 | 10.0.20.33/27 | |
| | vlan300 | 10.0.20.65/27 | |
| SW-HQ | vlan300 | 10.0.10.66/27 |
| SW-BR | vlan300 | 10.0.20.66/27 |
| SRV-HQ | vlan100 | 10.0.10.2/27 |
| SRV-BR | vlan100 | 10.0.20.2/27 |
| CLI-HQ | vlan200 | 10.0.10.32/27 |
| CLI-BR | vlan200 | 10.0.20.32/27 |
| CICD-HQ| vlan200 | 10.0.10.3/27 |

### Внеполосное управление виртуалками в Proxmox

<details>
  <summary>ТЫКНИ</summary>
  
Добавление serial порта в Гипервизоре:
```
qm set <VM ID> -serial0 socket
```
Хост `/etc/init/ttyS0.conf`:
```
# ttyS0 - getty
start on stopped rc RUNLEVEL=[12345]
stop on runlevel [!12345]
respawn
exec /sbin/getty -L 115200 ttyS0 vt102
```
Конфигурация `grub` `/etc/default/grub`:
```
GRUB_CMDLINE_LINUX ='console=tty0 console=ttyS0,115200'
```
Update:
```
update-grub
```
Включение serial порта:
```
systemctl enable serial-getty@ttyS0.service
```
Перезагружаемся и заходим через `xterm.js`. Теперь доступны скроллинг, вставка, копирование и произвольный размер окна.

</details>
  
## 1. Базовая настройка

a) Настройте имена устройств согласно топологии  
&ensp; a.	Используйте полное доменное имя  
&ensp; b.	Сконфигурируйте адреса устройств на свое усмотрение. Для офиса HQ выделена сеть 10.0.10.0/24, для офиса BR выделена сеть 10.0.20.0/24. Данные сети необходимо разделить на подсети для каждого vlan.  
&ensp; c.	На SRV-HQ и SRV-BR, создайте пользователя sshuser с паролем P@ssw0rd  
&ensp; &ensp; 1.	Пользователь sshuser должен иметь возможность запуска утилиты sudo без дополнительной аутентификации.  
&ensp; &ensp; 2.	Запретите парольную аутентификацию. Аутентификация пользователя sshuser должна происходить только при помощи ключей.  
&ensp; &ensp; 3.	Измените стандартный ssh порт на 2023.  
&ensp; &ensp; 4.	На CLI-HQ сконфигурируйте клиент для автоматического подключения к SRV-HQ и SRV-BR под пользователем sshuser. При подключении автоматически должен выбираться корректный порт. Создайте пользователя sshuser на CLI-HQ для обеспечения такого сетевого доступа. 

<details>
  <summary>ТЫКНИ</summary>

### RTR-HQ

Полное доменное имя:
```
config
hostname rtr-hq.company.prof
do commit
do confirm
```

Команды для просмотра интерфейсов:
```
sh ip int
sh int stat
```

Настройка ip-адреса (ISP-HQ):
```
interface te1/0/2
ip address 11.11.11.2/30
description ISP
exit
```

Параметры DNS:
```
domain name company.prof
domain name-server 11.11.11.1
```

Настройка шлюза:
```
ip route 0.0.0.0/0 11.11.11.1
```

Проверка доступа в Интернет:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/1cb14a1a-3e59-4120-a730-f8746de774f4)

Настройка подынтерфейсов (Router-on-a-stick) vlan100:
```
interface te1/0/3.100
ip firewall disable
description vlan100
ip address 10.0.10.1/27
exit
```

vlan200:
```
interface te1/0/3.200
ip firewall disable
description vlan200
ip address 10.0.10.33/27
exit
```

vlan300:
```
interface te1/0/3.300
ip firewall disable
description vlan300
ip address 10.0.10.65/27
exit
```

Сохранение:
```
do commit
do confirm
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/77c60f84-486f-45df-86d8-c10f6c756560)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/24fdd024-aa52-4b7a-9c7a-7a0a6d6b6d8e)

### RTR-BR
То же самое, кроме IP-адреса:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/5f79ec4d-41a3-4e92-aef8-533cb24e5954)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/8fcc282b-3736-49a6-8d57-bdd5cec8e770)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/2f309d19-26dd-41cb-a42f-a79cfed8cbfd)

### SW-HQ | SRV-HQ | CLI-HQ | CICD-HQ | SW-BR | SRV-BR | CLI-BR

Полное доменное имя:
```
hostnamectl set-hostname <VM_NAME>.company.prof;exec bash
```

Настройка интерфейсов:
```
sed -i 's/DISABLED=yes/DISABLED=no/g' /etc/net/ifaces/ens18/options
sed -i 's/NM_CONTROLLED=yes/NM_CONTROLLED=no/g' /etc/net/ifaces/ens18/options
echo 10.0.20.2/24 > /etc/net/ifaces/ens18/ipv4address
echo default via 10.0.20.1 > /etc/net/ifaces/ens18/ipv4route
systemctl restart network
ping -c 4 8.8.8.8
```

### SRV-HQ | SRV-BR

Создание пользователя sshuser:
```
adduser sshuser
passwd sshuser
P@ssw0rd
P@ssw0rd
echo "sshuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
usermod -aG wheel sshuser
sudo -i
```

### SSH

На серверах, к которым подключаемся:
```
mkdir /home/sshuser
```
```
chown sshuser:sshuser /home/sshuser
```
```
cd /home/sshuser
```
```
mkdir -p .ssh/
```
```
chmod 700 .ssh
```
```
touch .ssh/authorized_keys
```
```
chmod 600 .ssh/authorized_keys
```
```
chown sshuser:sshuser .ssh/authorized_keys
```

На машине с которого подключаемся к серверам:
```
ssh-keygen -t rsa -b 2048 -f srv_ssh_key
```
```
mkdir .ssh
```
```
mv srv_ssh_key* .ssh/
```
Конфиг для автоматического подключения `.ssh/config`:
```
Host srv-hq
        HostName 10.0.10.2
        User sshuser
        IdentityFile .ssh/srv_ssh_key
        Port 2023
Host srv-br
        HostName 10.0.20.2
        User sshuser
        IdentityFile .ssh/srv_ssh_key
        Port 2023
```
```
chmod 600 .ssh/config
```
Копирование ключа на удаленный сервер:
```
ssh-copy-id -i .ssh/srv_ssh_key.pub sshuser@10.0.10.2
```
```
ssh-copy-id -i .ssh/srv_ssh_key.pub sshuser@10.0.20.2
```
На сервере `/etc/ssh/sshd_config`:
```
AllowUsers sshuser
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
AuthorizedKeysFile .ssh/authorized_keys
Port 2023
```
```
systemctl restart sshd
```
Подключение:
```
ssh srv-hq
```

</details>

## 2. Настройка дисковой подсистемы
a)	На SRV-HQ настройте зеркалируемый LVM том  
&ensp; a.	Используйте два неразмеченных жестких диска.  
&ensp; b.	Настройте автоматическое монтирование логического тома.  
&ensp; c.	Точка монтирования /opt/data.  
b)	На SRV-BR сконфигурируйте stripped LVM том.  
&ensp; a.	Используйте два неразмеченных жестких диска.  
&ensp; b.	Настройте автоматическое монтирование тома.  
&ensp; c.	Обеспечьте шифрование тома средствами dm-crypt. Диск должен монтироваться при загрузке ОС без запроса пароля.  
&ensp; d.	Точка монтирования /opt/data.  

<details>
  <summary>ТЫКНИ</summary>

### SRV-HQ

Создаем разделы:
```
gdisk /dev/sdb
```
o - y  
n - 1 - enter - enter - 8e00  
p  
w - y  
То же самое с `sdc`  
Инициализация:
```
pvcreate /dev/sd{b,c}1
```
В группу:
```
vgcreate vg01 /dev/sd{b,c}1
```
Логический том-зеркало RAID1:
```
lvcreate -l 100%FREE -n lvmirror -m1 vg01
```
Создаём ФС:
```
mkfs.ext4 /dev/vg01/lvmirror
```
Директория:
```
mkdir /opt/data
```
Автозагрузка `/etc/fstab`:
```
echo "UUID=$(blkid -s UUID -o value /dev/vg01/lvmirror) /opt/data ext4 defaults 1 2" | tee -a /etc/fstab
```
Монтаж:
```
mount -av
```
Проверка:
```
df -h
```

### SRV-BR

Создаем разделы:
```
gdisk /dev/sdb
```
o - y  
n - 1 - enter - enter - 8e00  
p  
w - y  
То же самое с `sdc`  
Инициализация:
```
pvcreate /dev/sd{b,c}1
```
В группу:
```
vgcreate vg01 /dev/sd{b,c}1
```
Striped том:
```
lvcreate -i2 -l 100%FREE -n lvstriped vg01
```
> -i2 - количество полос  
> -l 100%FREE - занять всё место  
> -n lvstriped - имя 'lvstriped'  
> vg01 - имя группы

ФС:
```
mkfs.ext4 /dev/vg01/lvstriped
```
Создание ключа:
```
dd if=/dev/urandom of=/root/ext2.key bs=1 count=4096
```
Шифрование тома с помощью ключа:
```
cryptsetup luksFormat /dev/vg01/lvstriped /root/ext2.key
```
Открытие тома ключом:
```
cryptsetup luksOpen -d /root/ext2.key /dev/vg01/lvstriped lvstriped
```
ФС на расшифрованном томе:
```
mkfs.ext4 /dev/mapper/lvstriped
```
Директория монтирования:
```
mkdir /opt/data
```
Автозагрузка `/etc/fstab`
```
echo "UUID=$(blkid -s UUID -o value /dev/mapper/lvstriped) /opt/data ext4 defaults 0 0" | tee -a /etc/fstab
```
Монтаж:
```
mount -av
```
Автозагрузка зашифрованного тома с помощью ключа `/etc/crypttab`:
```
echo "lvstreped UUID=$(blkid -s UUID -o value /dev/vg01/lvstriped) /root/ext2.key luks" | tee -a /etc/crypttab
```
reboot, df -h, lsblk

</details>

## 3. Настройка коммутации

a)	В качестве коммутаторов используются SW-HQ и SW-BR.  
b)	В обоих офисах серверы должны находиться во vlan100, клиенты – во vlan200, management подсеть – во  vlan300.  
c)	Создайте management интерфейсы на коммутаторах.  
d)	Для каждого vlan рассчитайте подсети, выданные для офисов. Количество хостов в каждой подсети не должно превышать 30-ти.  

<details>
  <summary>ТЫКНИ</summary>

### SW-HQ

#### Внимание! в интерфейсах ens... не должно быть прочих файлов, кроме `options`, иначе порты не привяжутся

Временное назначение ip-адреса (смотрящего в сторону rtr-hq):
```
ip link add link ens18 name ens18.300 type vlan id 300
ip link set dev ens18.300 up
ip addr add 10.0.10.66/27 dev ens18.300
ip route add 0.0.0.0/0 via 10.0.10.65
echo nameserver 8.8.8.8 > /etc/resolv.conf
```

Обновление пакетов и установка `openvswitch`:
```
apt-get update && apt-get install -y openvswitch
```
Автозагрузка:
```
systemctl enable --now openvswitch
```

ens18 - rtr-hq  
ens19 - cicd-hq - vlan200
ens20 - srv-hq - vlan100
ens21 - cli-hq - vlan200

Создаем каталоги для ens19,ens20,ens21:
```
mkdir /etc/net/ifaces/ens{19,20,21}
```

Для моста:
```
mkdir /etc/net/ifaces/ovs0
```

Management интерфейс:
```
mkdir /etc/net/ifaces/mgmt
```

Не удалять настройки openvswitch:
```
sed -i "s/OVS_REMOVE=yes/OVS_REMOVE=no/g" /etc/net/ifaces/default/options
```
Мост `/etc/net/ifaces/ovs0/options`:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/6b3421b9-9260-47d7-a61e-a3ded4399557)

> TYPE - тип интерфейса, bridge;  
> HOST - добавляемые интерфейсы в bridge.  

mgmt `/etc/net/ifaces/mgmt/options`:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/8720b7e7-dd56-4116-aa44-13e586f31c70)

> TYPE - тип интерфейса (internal);  
> BOOTPROTO - статически;  
> CONFIG_IPV4 - использовать ipv4;  
> BRIDGE - определяет к какому мосту необходимо добавить данный интерфейс;  
> VID - определяет принадлежность интерфейса к VLAN.  

Поднимаем интерфейсы:
```
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens19/options
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens20/options
cp /etc/net/ifaces/ens18/options /etc/net/ifaces/ens21/options
```

Назначаем Ip, default gateway на mgmt:
```
echo 10.0.10.66/27 > /etc/net/ifaces/mgmt/ipv4address
```
```
echo default via 10.0.10.65 > /etc/net/ifaces/mgmt/ipv4route
```

Перезапуск network:
```
systemctl restart network
```

Проверка:
```
ip -c --br a
```
```
ovs-vsctl show
```

ens18 - rtr-hq делаем trunk и пропускаем VLANs:
```
ovs-vsctl set port ens18 trunk=100,200,300
```

ens19 - tag=200
```
ovs-vsctl set port ens19 tag=200
```

ens20 - tag=100:
```
ovs-vsctl set port ens20 tag=100
```

ens21 - tag=200
```
ovs-vsctl set port ens21 tag=200
```

Включаем инкапсулирование пакетов по 802.1q:
```
modprobe 8021q
```
Проверка включения

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/84767fce-7964-4ba4-9d33-15e6e2944c8c)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/693808bc-aa6e-406f-9381-cb6774874ca4)

</details>

## 4. Установка и настройка сервера баз данных

a)	В качестве серверов баз данных используйте сервера SRV-HQ и SRV-BR  
b)	Разверните сервер баз данных на базе Postgresql  
&ensp; a.	Создайте базы данных prod, test, dev  
&ensp; &ensp; i.	Заполните базы данных тестовыми данными при помощи утилиты pgbench. Коэффицент масштабирования сохраните по умолчанию.  
&ensp; b.	Разрешите внешние подключения для всех пользователей.  
&ensp; c.	Сконфигурируйте репликацию с SRV-HQ на SRV-BR  
&ensp; d.	Обеспечьте отказоустойчивость СУБД при помощи HAProxy.  
&ensp; &ensp; i.	HAProxy установите на SW-HQ.  
&ensp; &ensp; ii.	Режим балансировки – Hot-Standby: Активным необходимо сделать только SRV-HQ. В случае отказа SRV-HQ активным сервером должен становится SRV-BR.  
&ensp; &ensp; iii.	Выбор standby режима (RO/RW) остается на усмотрение участника.  
&ensp; &ensp; iv.	Обеспечьте единую точку подключения к СУБД по имени dbms.company.prof  

<details>
  <summary>ТЫКНИ</summary>

### SRV-HQ

Установка `postgresql`:
```
apt-get install -y postgresql16 postgresql16-server postgresql16-contrib
```
Развертываем БД:
```
/etc/init.d/postgresql initdb
```
Автозагрузка:
```
systemctl enable --now postgresql
```
Включаем прослушивание всех адресов:
```
nano /var/lib/pgsql/data/postgresql.conf
```

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/6cfbc559-b14f-49bd-a62e-5208bdb54aba)

Перезагружаем:
```
systemctl restart postgresql
```
Проверяем:
```
ss -tlpn | grep postgres
```

Заходим в БД под рутом:
```
psql -U postgres
```
Создаем базы данных "prod","test","dev":
```
CREATE DATABASE prod;
```
```
CREATE DATABASE test;
```
```
CREATE DATABASE dev;
```
Создаем пользователей "produser","testuser","devuser":
```
CREATE USER produser WITH PASSWORD 'P@ssw0rd';
```
```
CREATE USER testuser WITH PASSWORD 'P@ssw0rd';
```
```
CREATE USER devuser WITH PASSWORD 'P@ssw0rd';
```
Назначаем на БД владельца:
```
GRANT ALL PRIVILEGES ON DATABASE prod to produser;
```
```
GRANT ALL PRIVILEGES ON DATABASE test to testuser;
```
```
GRANT ALL PRIVILEGES ON DATABASE dev to devuser;
```
Заполняем БД с помощью `pgbench`:
```
pgqbench -U postgres -i prod
```
```
pgqbench -U postgres -i test
```
```
pgqbench -U postgres -i dev
```
Проверяем

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/555b8fd9-9c1e-4d94-9f17-799bcc40435f)

Настройка аутентификации для удаленного доступа:
```
nano /var/lib/pgsql/data/pg_hba.conf
```

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/514da535-fe56-4186-8e78-78fa5f1baf04)

Перезапускаем:
```
systemctl restart postgresql
```

### SRV-BR

Установка:
```
apt-get install -y postgresql16 postgresql16-server postgresql16-contrib
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/385d4826-5df8-475a-84e0-c6ccee7422e9)

## Репликация

### SRV-HQ

Конфиг
```
nano /var/lib/pgsql/data/postgresql.conf
```
```
wal_level = replica
max_wal_senders = 2
max_replication_slots = 2
hot_standby = on
hot_standby_feedback = on
```

> wal_level указывает, сколько информации записывается в WAL (журнал операций, который используется для репликации);
> max_wal_senders — количество планируемых слейвов;
> max_replication_slots — максимальное число слотов репликации; 
> hot_standby — определяет, можно ли подключаться к postgresql для выполнения запросов в процессе восстановления;
> hot_standby_feedback — определяет, будет ли сервер slave сообщать мастеру о запросах, которые он выполняет.

```
systemctl restart postgresql
```

### SRV-BR

Останавливаем:
```
systemctl stop postgresql
```

Удаляем каталог:
```
rm -rf /var/lib/pgsql/data/*
```

Запускаем репликацию (может длиться несколько десятков минут):
```
pg_basebackup -h 10.0.10.2 -U postgres -D /var/lib/pgsql/data --wal-method=stream --write-recovery-conf
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/dd67d507-8fe2-4e6e-b18a-55ebcb7286d9)


Назначаем владельца:
```
chown -R postgres:postgres /var/lib/pgsql/data/
```

Запускаем:
```
systemctl start postgresql
```

### SRV-HQ

Создаем тест:
```
psql -U postgres
```
```
CREATE DATABASE testik;
```

### SRV-BR

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/132c023d-ddcd-4350-9cb5-84e18fb99751)

## HAPROXY

### SW-HQ (test, not prod)

Установка haproxy:
```
apt-get install haproxy
```
Автозагрузка:
```
systemctl enable --now haproxy
```
Конфиг `/etc/haproxy/haproxy.cfg`:
```
listen stats
    bind 0.0.0.0:8989
    mode http
    stats enable
    stats uri /haproxy_stats
    stats realm HAProxy\ Statistics
    stats auth admin:toor
    stats admin if TRUE

frontend postgre
    bind 0.0.0.0:5432
    default_backend my-web

backend postgre
    balance first
    server srv-hq 10.0.10.2:5432 check
    server srv-br 10.0.20.2:5432 check
```

</details>

## 5.	Настройка динамической трансляции адресов

a)	Настройте динамическую трансляцию адресов для обоих офисов. Доступ к интернету необходимо разрешить со всех устройств.  

<details>
  <summary>ТЫКНИ</summary>

rtr-hq:
```
config
security zone public
exit
int te1/0/2
security-zone public
exit
object-group network COMPANY
ip address-range 10.0.10.1-10.0.10.254
exit
object-group network WAN
ip address-range 11.11.11.11
exit
nat source
pool WAN
ip address-range 11.11.11.11
exit
ruleset SNAT
to zone public
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
exit
exit
commit
confirm

```
rtr-br:
```
config
security zone public
exit
int te1/0/2
security-zone public
exit
object-group network COMPANY
ip address-range 10.0.20.1-10.0.20.254
exit
object-group network WAN
ip address-range 22.22.22.22
exit
nat source
pool WAN
ip address-range 22.22.22.22
exit
ruleset SNAT
to zone public
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
exit
exit
commit
confirm

```

</details>

## 6.	Настройка протокола динамической конфигурации хостов

a)	Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI - RTR-HQ  
&ensp; i.	Адрес сети – согласно топологии  
&ensp; ii.	Адрес шлюза по умолчанию – адрес маршрутизатора RTR-HQ  
&ensp; iii.	DNS-суффикс – company.prof  
b)	Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI RTR-BR  
&ensp; i.	Адрес сети – согласно топологии  
&ensp; ii.	Адрес шлюза по умолчанию – адрес маршрутизатора RTR-BR  
&ensp; iii.	DNS-суффикс – company.prof  

<details>
  <summary>ТЫКНИ</summary>

### RTR-HQ

DNS-сервер - srv-hq
```
configure terminal
ip dhcp-server pool COMPANY-HQ
network 10.0.10.32/27
default-lease-time 3:00:00
address-range 10.0.10.33-10.0.10.62
excluded-address-range 10.0.10.33                      
default-router 10.0.10.33 
dns-server 10.0.10.2
domain-name company.prof
exit
```

Включение DHCP-сервера:
```
ip dhcp-server
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/d1818095-9c71-4d89-b3b3-3f6a9d8ef41c)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/9a91aa12-c9d7-4b60-953a-398d714cbe39)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/a0fbac34-90c8-4aae-9e96-92d178ba806c)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/525dd722-2b8a-49f2-88e9-da0d2c05cb1d)

</details>

## 7.	Настройка DNS для SRV-HQ и SRV-BR

i.	Реализуйте основной DNS сервер компании на SRV-HQ  
&ensp; a.	Для всех устройств обоих офисов необходимо создать записи A и PTR.  
&ensp; b.	Для всех сервисов предприятия необходимо создать записи CNAME.  
&ensp; c.	Создайте запись test таким образом, чтобы при разрешении имени из левого офиса имя разрешалось в адрес SRV-HQ, а из правого – в адрес SRV-BR.  
&ensp; d.	Сконфигурируйте SRV-BR, как резервный DNS сервер. Загрузка записей с SRV-HQ должна быть разрешена только для SRV-BR.  
&ensp; e.	Клиенты предприятия должны быть настроены на использование внутренних DNS серверов  

<details>
  <summary>ТЫКНИ</summary>

### SRV-HQ

Установка bind и bind-utils:
```
apt-get update && apt-get install -y bind bind-utils
```
Конфиг:
```
nano /etc/bind/options.conf
```
```
listen-on { any; };
allow-query { any; };
allow-transfer { 10.0.20.2; }; 
```

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/12ad33f9-2df1-47e7-ad7a-1788b2276c88)

Включаем resolv:
```
nano /etc/net/ifaces/ens18/resolv.conf
```
```
systemctl restart network
```
Автозагрузка bind:
```
systemctl enable --now bind
```
Создаем прямую и обратные зоны:
```
nano /etc/bind/local.conf
```

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/74bbb72d-1577-413d-969c-bff4497d0b9c)

Копируем дефолты:
```
cp /etc/bind/zone/{localhost,company.db}
```
```
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/10.0.10.in-addr.arpa.db
```
```
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/20.0.10.in-addr.arpa.db
```
Назначаем права:
```
chown root:named /etc/bind/zone/company.db
```
```
chown root:named /etc/bind/zone/10.0.10.in-addr.arpa.db
```
```
chown root:named /etc/bind/zone/20.0.10.in-addr.arpa.db
```
Настраиваем зону прямого просмотра `/etc/bind/zone/company.prof`:

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/9d3bddf9-e8db-4cfc-8971-b3a6ff0f38ac)

Настраиваем зону обратного просмотра `/etc/bind/zone/10.0.10.in-addr.arpa.db`:

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/3fee1532-458b-4e10-967f-b858a4a43b63)

Настраиваем зону обратного просмотра `/etc/bind/zone/20.0.10.in-addr.arpa.db`:

![изображение](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/c9f18689-b324-4125-836d-91bdb23b1075)

Проверка зон:
```
named-checkconf -z
```
![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/d2de05e9-3830-4846-86a1-2974bb64dbf5)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/e1b6fb80-63df-4cf5-8dd6-2f3d46a1dc3b)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/21d53c0b-27d9-4af9-bbe3-b64035b0ffad)

### SRV-BR

Конфиг
```
vim /etc/bind/options.conf
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/e90d49ce-6735-4fdb-b44a-0b1c62b8305a)

Добавляем зоны

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/ea22291d-0b41-4044-a271-1fbb32f26185)

Резолв `/etc/net/ifaces/ens18/resolv.conf`:
```
search company.prof
nameserver 10.0.10.2
nameserver 10.0.20.2
```
Перезапуск адаптера:
```
systemctl restart network
```
Автозагрузка:
```
systemctl enable --now bind
```
SLAVE:
```
control bind-slave enabled
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/dc174c7e-e960-42c6-8b9d-7522df00989a)

Разрешение имени хоста test

### SRV-HQ

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/a73800a1-b7cf-48e3-8160-49b3b14a0060)

Копируем дефолт для зоны:
```
cp /etc/bind/zone/{localdomain,test.company.db}
```

Задаём права, владельца:
```
chown root:named /etc/bind/zone/test.company.db
```
Настраиваем зону:
```
vim /etc/bind/zone/test.company.db
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/ae1dcd6e-d5b9-4980-8105-d14b74905083)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/c8735610-56d5-419b-bdc8-3efc48a969b0)

Перезапускаем:
```
systemctl restart bind
```

### SRV-BR
Добавляем зону `/etc/bind/local.conf`:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/2d279b81-8bfd-453b-8efa-dc3d87476c40)

Задаём права, владельца:
```
chown root:named /etc/bind/zone/test.company.db
```

Редактируем зону `/etc/bind/zone/test.company.db`:

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/1aa398ac-dd3a-4887-b975-14ff7a3bf633)

Перезапускаем:
```
systemctl restart bind
```

</details>

## 8.	Настройка узла управления Ansible

a)	Настройте узел управления на базе SRV-BR  
&ensp; a.	Установите Ansible.  
b)	Сконфигурируйте инвентарь по пути /etc/ansible/inventory. Инвентарь должен содержать три группы устройств:  
&ensp; a.	Networking  
&ensp; b.	Servers  
&ensp; c.	Clients  
c)	Напишите плейбук в /etc/ansible/gathering.yml для сбора информации об IP адресах и именах всех устройств (и клиенты, и серверы, и роутеры). Отчет должен быть сохранен в /etc/ansible/output.yaml, в формате ПОЛНОЕ_ДОМЕННОЕ_ИМЯ – АДРЕС  

<details>
  <summary>ТЫКНИ</summary>

Установка:
```
apt-get install -y ansible sshpass
```
```
nano /etc/ansible/inventory.yml
```
```yml
Networking:
  hosts:
    rtr-hq:
      ansible_host: 10.0.10.1
      ansible_user: admin
      ansible_password: P@ssw0rd
      ansible_port: 22
    rtr-br:
      ansible_host: 10.0.20.1
      ansible_user: admin
      ansible_password: P@ssw0rd
      ansible_port: 22
Servers:
  hosts:
    srv-hq:
      ansible_host: 10.0.10.2
      ansible_ssh_private_key_file: ~/.ssh/srv_ssh_key
      ansible_user: sshuser
      ansible_port: 2023
    srv-br:
      ansible_host: 10.0.20.2
      ansible_ssh_private_key_file: ~/.ssh/srv_ssh_key
      ansible_user: sshuser
      ansible_port: 22
Clients:
  hosts:
    cli-hq:
      ansible_host: 10.0.10.34
      ansible_user: root
      ansible_password: P@ssw0rd
      ansible_port: 22
    cli-br:
      ansible_host: 10.0.20.34
      ansible_user: root
      ansible_password: P@ssw0rd
      ansible_port: 22
```
```
nano /etc/ansible/ansible.cfg
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/534a5b2d-380e-4659-8455-98883556eb9f)

```
nano /etc/ansible/gathering.yml
```
```yml
---
- name: show ip and hostname from routers
  hosts: Networking
  gather_facts: false
  tasks:
  - name: show ip
    esr_command:
      commands: show ip interfaces

- name: show ip and fqdn
  hosts: Servers, Clients
  tasks:
  - name: print ip and hostname
    debug:
      msg: "IP address: {{ ansible_default_ipv4.address }}, Hostname {{ ansible_hostname }}"
  - name: create file
    copy:
      content: ""
      dest: /etc/ansible/output.yml
      mode: '0644'

  - name: save output
    copy:
      content: "IP address: {{ ansible_default_ipv4.address }}, Hostname {{ ansible_hostname }}"
      dest: /etc/ansible/output.yml
```
```
cd /etc/ansible
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/a3c7e3de-d1fb-4a08-bc55-cc0765404a36)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/f2ccec52-702a-4ba5-8c61-ba3dd9c5d147)

</details>

## 9.	Между маршрутизаторами RTR-HQ и RTR-BR сконфигурируйте защищенное соединение

a)	Все параметры на усмотрение участника.  
b)	Используйте парольную аутентификацию.  
c)	Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса.  
d)	Для обеспечения динамической маршрутизации используйте протокол OSPF.  

https://sysahelper.gitbook.io/sysahelper/main/telecom/main/vesr_greoveripsec

<details>
  <summary>ТЫКНИ</summary>

```
tunnel gre 1
  ttl 16
  ip firewall disable
  local address 11.11.11.11
  remote address 22.22.22.22
  ip address 172.16.1.1/30
  enable
exit
```
```
security zone-pair public self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
  rule 2
    description "GRE"
    action permit
    match protocol gre
    enable
  exit
exit
```

```
security ike proposal ike_prop1
  authentication algorithm md5
  encryption algorithm aes128
  dh-group 2
exit
```

```
security ike policy ike_pol1
  pre-shared-key ascii-text P@ssw0rd
  proposal ike_prop1
exit
```

```
security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 11.11.11.11
  local network 11.11.11.11/32 protocol gre 
  remote address 22.22.22.22
  remote network 22.22.22.22/32 protocol gre 
  mode policy-based
exit
```

```
security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 22.22.22.22
  local network 22.22.22.22/32 protocol gre 
  remote address 11.11.11.11
  remote network 11.11.11.11/32 protocol gre 
  mode policy-based
exit
```

```
security ipsec proposal ipsec_prop1
  authentication algorithm md5
  encryption algorithm aes128
  pfs dh-group 2
exit
```

```
security ipsec policy ipsec_pol1
  proposal ipsec_prop1
exit
```
```
security ipsec vpn ipsec1
  ike establish-tunnel route
  ike gateway ike_gw1
  ike ipsec-policy ipsec_pol1
  enable
exit
```

```
security zone-pair public self
  rule 3
    description "ESP"
    action permit
    match protocol esp
    enable
  exit
  rule 4
    description "AH"
    action permit
    match protocol ah
    enable
  exit
exit
```

```
router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit
```

```
tunnel gre 1
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.100
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.200
  ip ospf instance 1
  ip ospf
exit

interface gigabitethernet 1/0/2.300
  ip ospf instance 1
  ip ospf
exit
```

```
security zone-pair public self
  rule 5
    description "OSPF"
    action permit
    match protocol ospf
    enable
  exit
exit
```

Проверка 
```
sh ip ospf neighbors
```
```
sh ip route
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/c77249a2-eec9-4b23-9992-a10e891f4a68)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/fa2b8166-cc1f-4d7d-a230-7c0f8f882e6c)

</details>

## 10.	[->](./10.md) На сервере SRV-HQ сконфигурируйте основной доменный контроллер на базе FreeIPA

a)	Создайте 30 пользователей user1-user30.  
b)	Пользователи user1-user10 должны входить в состав группы group1.  
c)	Пользователи user11-user20 должны входить в состав группы group2.  
d)	Пользователи user21-user30 должны входить в состав группы group3.  
e)	Разрешите аутентификацию с использованием доменных учетных данных на ВМ CLI-HQ.  
f)	Установите сертификат центра сертификации FreeIPA в качестве доверенного на обоих клиентских ПК.  

<details>
  <summary>ТЫКНИ</summary>

[->](./10.md)

</details>

# 11.	На SRV-BR сконфигурируйте proxy-сервер со следующими параметрами

a)	Пользователям group1 разрешен доступ на любые сервисы предприятия  
b)	Пользователям group2 разрешен доступ только к системе мониторинга  
c)	Пользователям group3 не разрешен доступ никуда, также, как и пользователям, не прошедшим аутентификацию  
d)	Любым пользователям компьютера CLI-HQ разрешен доступ в сеть Интернет и на все сервисы предприятия, кроме доменов vk.com, mail.yandex.ru и worldskills.org  
e)	Настройте клиент правого офиса на использование прокси сервера предприятия  
f)	Авторизация для proxy спрашивается браузером, SSO не ожидается  

| Запись | Тип записи |
|:-|:-|
|rtr-hq.company.prof|A|
|rtr-br.company.prof|A|
|sw-hq.company.prof|A|
|sw-br.company.prof|A|
|srv-hq.company.prof|A|
|srv-br.company.prof|A|
|cli-hq.company.prof|A|
|cli-br.company.prof|A|
|dbms.company.prof|CNAME|

<details>
  <summary>ТЫКНИ</summary>

## SRV-BR


## SRV-HQ

Установка:
```
apt-get install alterator-squid
```
Доступ осуществляется по `https://ip-address:8080/`

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/1def4702-a3ce-4862-8d03-ceb34e7b1fdd)


</details>

# Модуль В.  (Обеспечение отказоустойчивости)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/66f6a25e-db17-4434-b11f-7aa3ca613651)


&ensp; Время на выполнение модуля 5 часов  
&ensp; Данный модуль содержит задачи, основанные на практиках DevOps при разработке и эксплуатации информационных систем в сфере разработки современного программного обеспечения.  
&ensp; Главной задачей данного модуля является создание элементов автоматизированной инфраструктуры с помощью инструментов для работы с облачными средами и управления контейнерами. В рамках задания вам будет предоставлен доступ к инфраструктуре облачного провайдера. В качестве основной задачи необходимо подготовить инструкции для полностью автоматического развёртывания приложения вместе со всеми необходимыми службами в облаке.  
&ensp; Для выполнения задания на ваших локальных компьютерах будет обеспечен доступ к сети Интернет.  
ИНСТРУКЦИИ ДЛЯ УЧАСТНИКА  
&ensp; Доступ к облачному провайдеру и авторизацию на сервере выполняет экспертная группа. Главный эксперт сам определяет провайдера облачной инфраструктуры.  
&ensp; Все пункты задания необходимо реализовать на стороне провайдера облачной инфраструктуры. Эксперты будут ожидать наличие работоспособных элементов инфраструктуры согласно заданию.  
&ensp; Пункты задания, касающиеся настройки облака, потребуют от Вас написания автоматизированных сценариев развёртывания инфраструктуры согласно заданию. Конкретные инструменты выбираются на Ваше усмотрение. По завершении выполнения задания, ожидается, что будут полностью удалены автоматически созданные машины, сетевые и другие настройки в облачной инфраструктуре. Эксперты при проверке будут выполнять скрипт и проверять, что за отведённое (20 минут) время вся инфраструктура создаётся и работает должным образом.  
&ensp; Для проверки работоспособности скриптов автоматического развертывания инфраструктуры, эксперты могут проверять их работу в отдельной учётной записи того же облачного провайдера. Вам необходимо предусмотреть возможность указания необходимых параметров для работы скриптов с другой учетной записью облачного провайдера.  
&ensp; Проверка будет выполняться с инстанса ControlVM, вы можете установить все необходимые для вашей работы инструменты на указанную машину.  

## Подготовка машины ControlVM  

&ensp; 1. Вся проверка выполнения задания будет проводиться с машины ControlVM.  
&ensp; 2. НЕ удаляйте ControlVM по завершении задания.  
&ensp; 3. Создайте инстанс с именем ControlVM и подключите его к сети интернет.  
&ensp; &ensp; 1. Тип виртуальной машины: 2 vCPU, Доля vCPU 50%, 2 RAM.  
&ensp; &ensp; 2. Размер диска: 10 ГБ.  
&ensp; &ensp; 3. Тип диска: SSD.  
&ensp; &ensp; 4. Отключите мониторинг и резервное копирование.  
&ensp; &ensp; 5. Операционная система: ALT Linux 10.  
&ensp; &ensp; 6. Разрешите внешние подключения по протоколу SSH.  
&ensp; &ensp; 7. Сохраните ключевую пару для доступа на рабочем столе вашего локального ПК с расширением .pem  
&ensp; 4. Настройте внешнее подключение к ControlVM.  
&ensp; &ensp; 1. Установите на локальный ПК клиент SSH PuTTY.  
&ensp; &ensp; 2. Создайте в PuTTY профиль с именем YACloud.  
&ensp; &ensp; 3. Убедитесь в возможности установления соединения с ControlVM с локального ПК с помощью клиента PuTTY без ввода дополнительных параметров.  
&ensp; &ensp; 4. Используйте для подключения имя пользователя altlunxu и загруженную ключевую пару.  

<details>
  <summary>Кликни</summary>

Буду использовать Mobaxterm, который базирован на putty

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/d5e36f86-9a5e-452b-87e4-88b44ed16b4a)

Tools - MobaKeyGen

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/363a48d1-5411-4440-aab4-a26d336ef793)

Generate - копируем публичный ключ - Save private key - меняем расширение ключа на `pem`

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/e4390e4b-c7d2-4ed7-b887-956e73d2e6f6)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/6aebff10-f997-4026-bb77-27a56759f02e)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/99cc5ea4-1fd4-45b9-93d5-94dfa01ea772)

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/4ee74e84-096a-4a58-91ee-5070bea69f67)

```
sudo -i
```

```
apt-get update && apt-get install -y wget unzip
```

Зеркало Яндекса
```
wget https://hashicorp-releases.yandexcloud.net/terraform/1.7.2/terraform_1.7.2_linux_amd64.zip
```

Зеркало VK
```
wget https://hashicorp-releases.mcs.mail.ru/terraform/1.7.3/terraform_1.7.3_linux_amd64.zip
```

```
unzip terraform_1.7.2_linux_amd64.zip -d /usr/local/bin/
```

![image](https://github.com/abdurrah1m/Professionals_2024/assets/148451230/b8af500d-480f-4eab-9ad6-59d3a40cc252)

</details>

## Подготовка облачной инфраструктуры  

1. Подготовьте сценарий автоматизации развёртывания облачной инфраструктуры.  
&ensp; &ensp; 1. Виртуальные машины и сети должны быть созданы согласно Топологии.  
&ensp; &ensp; 2. Имена виртуальных машин и сетей должны соответствовать Топологии.  
&ensp; &ensp; 3. Обеспечьте подключение виртуальных машин к соответствующим сетям.  
&ensp; &ensp; 4. В случае предоставления внешнего доступа к созданным виртуальным машинам, он должен быть разрешён только по протоколу ssh.  
&ensp; &ensp; 5. Разрешите трафик по протоколу ICMP.  
&ensp; &ensp; 6. Вы можете назначить глобальные IP адреса для управления созданными виртуальными машинами.  
&ensp; &ensp; 7. Используйте аутентификацию на основе открытых ключей, аутентификация с использованием пароля должна быть отключена для SSH.  
&ensp; &ensp; 8. Создайте балансировщик нагрузки.  
&ensp; &ensp; &ensp; 1. Сохраните внешний адрес балансировщика нагрузки в файле /home/altlinux/lb.ip.  
&ensp; &ensp; &ensp; 2. Ограничьте внешний доступ протоколами http и https.  
&ensp; &ensp; &ensp; 3. Балансировка нагрузки должна использовать алгоритм round robin.  
&ensp; &ensp; &ensp; 4. При обращении на внешний адрес балансировщика нагрузки должен выводиться ответ от приложения на внутреннем сервере.  
2. Виртуальные машины должны соответствовать следующим характеристикам.  
&ensp; &ensp; 1. Операционная система: ALT Linux 10.  
&ensp; &ensp; 2. Количество vCPU: 1.  
&ensp; &ensp; 3. Объём оперативной памяти: 1024 МБ.  
&ensp; &ensp; 4. Объём диска: 15 ГБ.  
&ensp; &ensp; 5. Тип диска: HDD.  
&ensp; &ensp; 6. Разместите виртуальные машины в регионе Москва.  
&ensp; &ensp; 7. Разместите Web1 в зоне доступности ru-central1-a.  
&ensp; &ensp; 8. Разместите Web2 в зоне доступности ru-central1-b.  
3. На машине ControlVM создайте скрипт cloudinit.sh.  
&ensp; &ensp; 1. В качестве рабочей директории используйте путь /home/altlinux/bin.  
&ensp; &ensp; 2. Используйте файл /home/altlinux/bin/cloud.conf для указания настроек для подключения к облачному провайдеру.  
&ensp; &ensp; &ensp; 1. При выполнении проверки, эксперты могут изменить настройки только в файле cloud.conf. Другие файлы редактироваться не будут.  
&ensp; &ensp; &ensp; 2. Вы можете оставить любые понятные комментарии в файле cloud.conf.  
&ensp; &ensp; 3. Скрипт должен выполняться из любой директории без явного указания пути к исполняемому файлу.  
&ensp; &ensp; 4. Выполнение задания ожидается с использованием инструментов Terraform и/или OpenStack CLI. Однако, вы вправе выбрать другие инструменты, не противоречащие условиям задания и правилам соревнования.  

## Развертывание приложений в Docker  

1. На машине ControlVM.  
&ensp; &ensp; 1. Установите Docker и Docker Compose.  
&ensp; &ensp; 2. Создайте локальный Docker Registry.  
&ensp; &ensp; 3. В домашней директории хоста создайте файл name.txt и запишите в него строку experts.  
&ensp; &ensp; 4. Напишите Dockerfile для приложения HelloFIRPO.  
&ensp; &ensp; &ensp; 1. В качестве базового образа используйте alpine  
&ensp; &ensp; &ensp; 2. Сделайте рабочей директорию /hello и скопируйте в неё name.txt  
&ensp; &ensp; &ensp; 3. Контейнер при запуске должен выполнять команду echo, которая выводит сообщение "Hello, FIRPO! Greetings from " и затем содержимое файла name.txt, после чего завершать свою работу.  
&ensp; &ensp; 5. Соберите образ приложения App и загрузите его в ваш Registry.  
&ensp; &ensp; &ensp; 1. Используйте номер версии 1.0 для вашего приложения  
&ensp; &ensp; &ensp; 2. Образ должен быть доступен для скачивания и дальнейшего запуска на локальной машине.  
&ensp; &ensp; 6. Создайте в домашней директории пользователя ubuntu файл wiki.yml для приложения MediaWiki.  
&ensp; &ensp; &ensp; 1. Средствами docker compose должен создаваться стек контейнеров с приложением MediaWiki и базой данных.  
&ensp; &ensp; &ensp; &ensp; 1. Используйте два сервиса  
&ensp; &ensp; &ensp; &ensp; 2. Основной контейнер MediaWiki должен называться wiki и использовать образ mediawiki  
&ensp; &ensp; &ensp; &ensp; 3. Файл LocalSettings.php с корректными настройками должен находиться в домашней папке пользователя ubuntu и автоматически монтироваться в образ.  
&ensp; &ensp; &ensp; &ensp; 4. Контейнер с базой данных должен называться db и использовать образ mysql  
&ensp; &ensp; &ensp; &ensp; 5. Он должен создавать базу с названием mediawiki, доступную по стандартному порту, для пользователя wiki с паролем P@ssw0rd  
&ensp; &ensp; &ensp; &ensp; 6. База должна храниться в отдельном volume с названием dbvolume  
&ensp; &ensp; &ensp; &ensp; 7. База данных должна находиться в одной сети с приложением App2, но не должна быть доступна снаружи.  
&ensp; &ensp; &ensp; &ensp; 8. MediaWiki должна быть доступна извне через порт 80.  
&ensp; &ensp; 7. Настройте мониторинг с помощью NodeExporter, Prometheus и Grafana.  
&ensp; &ensp; &ensp; 1. Создайте в домашней директории пользователя ubuntu файл monitoring.yml для Docker Compose  
&ensp; &ensp; &ensp; &ensp; 1. Используйте контейнеры NodeExporter, Prometheus и Grafana для сбора, обработки и отображения метрик.  
&ensp; &ensp; &ensp; &ensp; 2. Настройте Dashboard в Grafana, в котором будет отображаться загрузка CPU, объём свободной оперативной памяти и места на диске.  
&ensp; &ensp; &ensp; &ensp; 3. Интерфейс Grafana должен быть доступен по внешнему адресу на порту 3000.  

## Развёртывания облачных сервисов  

1. На машине ControlVM создайте скрипт /home/ubuntu/bin/DeployApp.sh.  
&ensp; &ensp; 1. Скрипт должен выполняться из любой директории без явного указания пути к исполняемому файлу.  
2. Подготовьте web-приложение App1  
&ensp; &ensp; 1. Скачайте файлы app1.py и Dockerfile по адресу: https://github.com/auteam-usr/moscow39  
&ensp; &ensp; 2. Соберите образ приложения и загрузите его в любой репозиторий Docker на ваше усмотрение.  
3. Команда DeployApp.sh должна запускать средства автоматизации для настройки операционных систем.  
&ensp; &ensp; 1. Разверните web-приложение App1 из репозитория Docker на виртуальных машинах Web1 и Web2.  
&ensp; &ensp; 2. Обеспечьте балансировку нагрузки между Web1 и Web2.  
&ensp; &ensp; 3. Обеспечьте внешний доступ к web-приложению по протоколу https.  
&ensp; &ensp; 4. При обращении по протоколу http должно выполняться автоматическое перенаправления на протокол https.  
&ensp; &ensp; 5. Обеспечивать доверие сертификату не требуется.  

## По завершении рабочего времени  

•	Высвободите выделенные ресурсы облачного провайдера для автоматически созданных объектов.  
•	Удалите все автоматически созданные виртуальные машины, сети, объекты, ресурсы.  
•	НЕ удаляйте ControlVM и необходимые для её работы ресурсы.  
Обратите внимание, что при наличии в облачной инфраструктуре существующих объектов, за исключением объектов, необходимых для работы ControlVM, объектов по умолчанию проверка осуществляться не будет.  
