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

## 1. Базовая настройка

a) Настройте имена устройств согласно топологии  
&ensp; a.	Используйте полное доменное имя  
&ensp; b.	Сконфигурируйте адреса устройств на свое усмотрение. Для офиса HQ выделена сеть 10.0.10.0/24, для офиса BR выделена сеть 10.0.20.0/24. Данные сети необходимо разделить на подсети для каждого vlan.  
&ensp; c.	На SRV-HQ и SRV-BR, создайте пользователя sshuser с паролем P@ssw0rd  
&ensp; &ensp; 1.	Пользователь sshuser должен иметь возможность запуска утилиты sudo без дополнительной аутентификации.  
&ensp; &ensp; 2.	Запретите парольную аутентификацию. Аутентификация пользователя sshuser должна происходить только при помощи ключей.  
&ensp; &ensp; 3.	Измените стандартный ssh порт на 2023.  
&ensp; &ensp; 4.	На CLI-HQ сконфигурируйте клиент для автоматического подключения к SRV-HQ и SRV-BR под пользователем sshuser. При подключении автоматически должен выбираться корректный порт. Создайте пользователя sshuser на CLI-HQ для обеспечения такого сетевого доступа.  

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
usermod -aG wheel admin
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
lvcreate -i2 -l 100%FREE -n lvstreped vg01
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
## 3. Настройка коммутации

a)	В качестве коммутаторов используются SW-HQ и SW-BR.  
b)	В обоих офисах серверы должны находиться во vlan100, клиенты – во vlan200, management подсеть – во  vlan300.  
c)	Создайте management интерфейсы на коммутаторах.  
d)	Для каждого vlan рассчитайте подсети, выданные для офисов. Количество хостов в каждой подсети не должно превышать 30-ти.  

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

## 5.	Настройка динамической трансляции адресов

a)	Настройте динамическую трансляцию адресов для обоих офисов. Доступ к интернету необходимо разрешить со всех устройств.  

rtr-hq:
```
config
security zone WAN
exit
security zone LAN-1
exit
int te1/0/2
security-zone WAN
exit
int te1/0/3
security-zone LAN-1
exit
object-group network COMPANY
ip address-range 10.0.10.1-10.0.10.254
exit
object-group network WAN
ip address-range 11.11.11.2
exit
security zone-pair WAN LAN-1
rule 1
match source-address COMPANY
action permit
enable
exit
exit
nat source
pool WAN
ip address-range 11.11.11.2
exit
ruleset SNAT
to zone WAN
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
end
commit
confirm

```
rtr-br:
```
config
security zone WAN
exit
security zone LAN-2
exit
int te1/0/4
security-zone WAN
exit
int te1/0/3
security-zone LAN-2
exit
object-group network COMPANY
ip address-range 10.0.20.1-10.0.20.254
exit
object-group network WAN
ip address-range 22.22.22.2
exit
security zone-pair WAN LAN-2
rule 1
match source-address COMPANY
action permit
enable
exit
exit
nat source
pool WAN
ip address-range 22.22.22.2
exit
ruleset SNAT
to zone WAN
rule 1
match source-address COMPANY
action source-nat pool WAN
enable
end
commit
confirm

```
## 6.	Настройка протокола динамической конфигурации хостов

a)	Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI - RTR-HQ  
&ensp; i.	Адрес сети – согласно топологии  
&ensp; ii.	Адрес шлюза по умолчанию – адрес маршрутизатора RTR-HQ  
&ensp; iii.	DNS-суффикс – company.prof  
b)	Настройте протокол динамической конфигурации хостов для устройств в подсетях CLI RTR-BR  
&ensp; i.	Адрес сети – согласно топологии  
&ensp; ii.	Адрес шлюза по умолчанию – адрес маршрутизатора RTR-BR  
&ensp; iii.	DNS-суффикс – company.prof  

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

## 7.	Настройка DNS для SRV-HQ и SRV-BR

i.	Реализуйте основной DNS сервер компании на SRV-HQ  
&ensp; a.	Для всех устройств обоих офисов необходимо создать записи A и PTR.  
&ensp; b.	Для всех сервисов предприятия необходимо создать записи CNAME.  
&ensp; c.	Создайте запись test таким образом, чтобы при разрешении имени из левого офиса имя разрешалось в адрес SRV-HQ, а из правого – в адрес SRV-BR.  
&ensp; d.	Сконфигурируйте SRV-BR, как резервный DNS сервер. Загрузка записей с SRV-HQ должна быть разрешена только для SRV-BR.  
&ensp; e.	Клиенты предприятия должны быть настроены на использование внутренних DNS серверов  

## 8.	Настройка узла управления Ansible

a)	Настройте узел управления на базе SRV-BR  
&ensp; a.	Установите Ansible.  
b)	Сконфигурируйте инвентарь по пути /etc/ansible/inventory. Инвентарь должен содержать три группы устройств:  
&ensp; a.	Networking  
&ensp; b.	Servers  
&ensp; c.	Clients  
c)	Напишите плейбук в /etc/ansible/gathering.yml для сбора информации об IP адресах и именах всех устройств (и клиенты, и серверы, и роутеры). Отчет должен быть сохранен в /etc/ansible/output.yaml, в формате ПОЛНОЕ_ДОМЕННОЕ_ИМЯ – АДРЕС  

## 9.	Между маршрутизаторами RTR-HQ и RTR-BR сконфигурируйте защищенное соединение

a)	Все параметры на усмотрение участника.  
b)	Используйте парольную аутентификацию.  
c)	Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса.  
d)	Для обеспечения динамической маршрутизации используйте протокол OSPF.  

https://sysahelper.gitbook.io/sysahelper/main/telecom/main/vesr_greoveripsec

### RTR-HQ  
Разрешение ICMP из LAN:
```
security zone-pair LAN-1 self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```
из WAN:
```
security zone-pair WAN self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```
Prerouting:
```
security zone-pair LAN-1 WAN
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```

### RTR-BR  
Разрешение ICMP из LAN:
```
security zone-pair LAN-2 self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```
из WAN:
```
security zone-pair WAN self
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```
Prerouting:
```
security zone-pair LAN-2 WAN
  rule 1
    description "ICMP"
    action permit
    match protocol icmp
    enable
  exit
exit

```
RTR-HQ Туннель:
```
tunnel gre 1
  ttl 16
  security-zone WAN
  local address 11.11.11.2
  remote address 22.22.22.2
  ip address 10.20.30.1/30
  enable
exit
do commit
do confirm
```
RTR-BR Туннель:
```
tunnel gre 1
  ttl 16
  security-zone WAN
  local address 22.22.22.2
  remote address 11.11.11.2
  ip address 10.20.30.2/30
  enable
exit
do commit
do confirm
```

Маршрут по умолчанию для офиса HQ:
```
ip route 10.0.10.0/24 10.20.30.2
```

Маршрут по умолчанию для офиса BR:
```
ip route 10.0.20.0/24 10.20.30.1
```

RTR-HQ | RTR-BR Профиль IKE:
```
security ike proposal ike_prop1
  authentication algorithm md5
  encryption algorithm aes128
  dh-group 2
exit

```
RTR-HQ | RTR-BR Политика IKE:
```
security ike policy ike_pol1
  pre-shared-key ascii-text P@ssw0rd
  proposal ike_prop1
exit

```
RTR-HQ Шлюз IKE:
```
security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 11.11.11.2
  local network 11.11.11.2/32 protocol gre 
  remote address 22.22.22.2
  remote network 22.22.22.2/32 protocol gre 
  mode policy-based
exit

```

RTR-BR Шлюз IKE:
```
security ike gateway ike_gw1
  ike-policy ike_pol1
  local address 22.22.22.2
  local network 22.22.22.2/32 protocol gre 
  remote address 11.11.11.2
  remote network 11.11.11.2/32 protocol gre 
  mode policy-based
exit

```

RTR-HQ | RTR-BR Профиль безопасности для IPsec:
```
security ipsec proposal ipsec_prop1
  authentication algorithm md5
  encryption algorithm aes128
  pfs dh-group 2
exit

```
RTR-HQ | RTR-BR Политика IPsec:
```
security ipsec policy ipsec_pol1
  proposal ipsec_prop1
exit

```
RTR-HQ | RTR-BR IPsec VPN:
```
security ipsec vpn ipsec1
  ike establish-tunnel route
  ike gateway ike_gw1
  ike ipsec-policy ipsec_pol1
  enable
exit

```
RTR-HQ | RTR-BR Firewall:
```
security zone-pair WAN self
  rule 2
    description "GRE"
    action permit
    match protocol gre
    enable
  exit
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
### RTR-HQ | RTR-BR
Включение OSPF:
```
router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit
```
Добавление интерфейсов, туннель и подынтерфейсы:
```
int te1/0/3
ip ospf instance 1
ip ospf
exit
tunnel gre 1
ip ospf instance 1
ip ospf
exit
```

vlan 100:
```
int te1/0/3.100
ip ospf instance 1
ip ospf
exit
```

vlan 200:
```
int te1/0/3.200
ip ospf instance 1
ip ospf
exit
```

vlan 300:
```
int te1/0/3.300
ip ospf instance 1
ip ospf
exit
```

Разрешение трафика OSPF:
```
security zone-pair WAN self
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


## 10.	На сервере SRV-HQ сконфигурируйте основной доменный контроллер на базе FreeIPA

a)	Создайте 30 пользователей user1-user30.  
b)	Пользователи user1-user10 должны входить в состав группы group1.  
c)	Пользователи user11-user20 должны входить в состав группы group2.  
d)	Пользователи user21-user30 должны входить в состав группы group3.  
e)	Разрешите аутентификацию с использованием доменных учетных данных на ВМ CLI-HQ.  
f)	Установите сертификат центра сертификации FreeIPA в качестве доверенного на обоих клиентских ПК.  

## 11.	На SRV-BR сконфигурируйте proxy-сервер со следующими параметрами

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

# Модуль В.  (Обеспечение отказоустойчивости)
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
