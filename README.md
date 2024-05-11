# GGIGADEMO

| Имя устройства | Интерфейс | IPv4/IPv6 | Маска/префикс | Шлюз |
| ----------- | ----------- | ----------------- | ----- | --------------- |
| CLI         | ens160      | 10.12.10.1        |  24 / .0   |                 |
| HQ-R        | ens160      | 10.12.20.1        |  24 / .0   |                 |
|             | ens192      | 172.16.0.1        |  25 / .128 |                 |
|             | GRE1        | 10.0.0.1          |  24 / .0   |                 |
| BR-R        | ens160      | 10.12.30.1        |  24 / .0   |                 |
|             | ens192      | 172.16.0.129      |  27 / .224 |                 |
|             | GRE1        | 10.0.0.2          |  24 / .0   |                 |
| HQ-SRV      | ens160      | 172.16.0.126      |  25 / .128 | 172.16.0.1      |
| BR-SRV      | ens160      | 172.16.0.158      |  27 / .224 | 172.16.0.129    |

Чтобы поменять адрес интерфейса
```
mcedit /etc/net/ifaces/ens***/ipv4address
```
Чтобы поменять шлюз по умолчанию интерфейса
```
mcedit /etc/net/ifaces/ens***/ipv4route
```
Чтобы поменять настройки конфигурации
```
mcedit /etc/net/ifaces/ens***/options
```
# Поднимаем GRE-туннель средствами nmtui (бонусное задание)
# Выполнение:
Установление пакета
```
apt-get install -y NetworkMnager-{daemon,tui}
```
Включение в автозагрузку
```
systemctl enable --now NetworkManager
```
Вход в интерфейс
```
nmtui
```
![image](https://github.com/Ganibal-24/demo2024/assets/148868527/aa3f89b4-258d-416d-b640-841dc113e882)

Добавить IP tunnel

![image](https://github.com/Ganibal-24/demo2024/assets/148868527/7119312a-f41a-46c5-bf1e-7d04fe7f29ac)

![image](https://github.com/Ganibal-24/demo2024/assets/148868527/102808ba-b1f2-4e68-a05c-7b8afa776640)

Для HQ-R:
```
nmcli connection modify HQ-R ip-tunnel.ttl 64
ip r add 192.168.0.128/27 dev gre1
```
Для BR-R:
```
nmcli connection modify BR-R ip-tunnel.ttl 64
ip r add 192.168.0.0/25 dev gre1
```
# 2. Настройте внутреннюю динамическую маршрутизацию по средствам FRR
# Выполнение:
Установил пакет FRR:
```
apt-get install -y frr
```
Вошёл в файл:
```
mcedit /etc/frr/daemons
```
Изменил значение:
```
ospfd=yes
```
Включаю и добавляю в автозагрузку FRR:
```
systemctl enable --now frr
```
Вошёл в настройку маршрутизации:
```
vtysh
```
Вхожу в конфигурацию терминала, затем запускаю процесс и добавляю сети:
```
conf t
router ospf
passive-interface default
 network 192.168.0.0/25 area 0
 network 10.0.0.0/30 area 0
```
Просматриваю соседей:
```
show ip ospf neighbor
```
Далее нужно прописать sysctl -w net.ipv4.ip_forward=1. Затем раскомментировать сроку net.ipv4.ip_forward в файле /etc/sysctl.conf. Сохранить настройку в vtysh. Проверить настройку можно с помощью команды sysctl -a | grep forward.
# 3. Настройте автоматическое распределение IP-адресов на роутере HQ-R
# Выполнение:
Установка DHCP
```
apt-get install -y dhcp-server
```
Вошёл в файл
```
mcedit /etc/sysconfig/dhcpd 
```
Указал интерфейс который смотрит на HQ-SRV
```
DHCPDARGS=ens224
```
Далее следует настройка раздачи адресов, а для этого захожу в файл:
```
mcedit /etc/dhcp/dhcpd.conf
```
Прописываю конфиг
```
default-lease-time 6000;
max-lease-time 72000;

subnet 192.168.0.0 netmask 255.255.255.128 {
range 192.168.0.10 192.168.0.125;
option routers 192.168.0.1;
}
```
Повторяю конфиг в файле /etc/dhcp/dhcpd.conf.example
Запускаю и добавляю в автозагрузку слжубу 
```
systemctl enable --now dhcpd
```
Проверить DHCP можно при помощи HQ-SRV, включив на нём DHCP.
# 4. Настройте локальные учётные записи на всех устройствах
| Учётная запись | Пароль      | Примичание     |     
| -----------    | ----------- |------------    | 
| Admin          | toor        | CLI, HQ-SRV, HQ-R| 
| Branch admin   | toor        | BR-SRV, BR-R   |
| Network admin  | toor        | HQ-R, BR-R, BR-SRV|
# Выполнение:
Добавляю пользователя:
```
useradd admin -m -c "Admin" -U 
```
Поменять пароль:
```
passwd admin
```
Для CLI: пуск - центр управления - центр управления системой - локальные учётные записи. Комментарий - Admin; назначенные - users; группы в которые входит - Admin - применить.
# 5. Измерьте пропускную способность сети между двумя узлами
# Выполнение:
Устанавливаю пакет:
```
apt-get install -y iperf3
```
Выполняю запуск службы:
```
systemctl enable --now iperf3
```
Запускаю iperf3 в качестве клиента:
```
iperf3 -c 192.168.0.162
```
# 6. Составьте backup скрипты для сохранения конфигурации сетевых устройств
# Выполнение:
Создаем новую директорию, где будет храниться наш backup:
```
mkdir /etc/frrbackup
```
Нужно поменять редактор, поэтому:
```
vim ~/.bash_profile
```
Внутри файла прописываем:
```
export VISUAL="mcedit"
```
Запускаем файл:
```
. ~/.bash_profile
```
Перезагружаю машину. Захожу в файл crontab -e и пишу конфиг:
```
0 0 * * * rsync -avzh /etc/frr/frr.conf /etc/frrbackup
```
# 7. Настройте подключение по SSH для удалённого конфигурирования устройства
# Выполнение: 
Нужно настроить конфиг на HQ-SRV:
```
mcedit /etc/openssh/sshd_config
```
Меняем значение:
```
Port 22
PermitRootLogin yes
PasswordAuthentication yes
```
Перезагружаем sshd.

На HQ-R пишем правило:
```
iptables -t nat -A PREROUTING -p tcp --dport 2222 -g DNAT --to 172.16.0.5:22
```
Добавляем и включаем с помощью start и enable. Сохраняем правило командой iptables-save.

iptables-save > /etc/sysconfig/iptables               reboot                iptables-save < /etc/sysconfig/iptables 

Проверяем подключаясь с BR-R на HQ-SRV:
```
ssh -p 2222 admin@172.16.0.5
```
# 8. Настройте контроль доступа до HQ-SRV по SSH
# Выполнение:
На HQ-SRV пишем правило:
```
iptables -A INPUT -p tcp --dport 22 -s 10.10.10.10 -g DROP
```
Включаем и добавляем в автозагрузку. Сохраняем правило:
```
iptables-save.
iptables-save > /etc/sysconfig/iptables
reboot
iptables-save < /etc/sysconfig/iptables 
```
Пытаемся с CLI подключиться на HQ-SRV:
```
ssh -p 2222 admin@10.10.20.2
```
