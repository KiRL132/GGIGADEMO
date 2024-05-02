# GGIGADEMO

| Имя устройства | Интерфейс | IPv4/IPv6 | Маска/префикс | Шлюз |
| ----------- | ----------- | ----------------- | ----- | --------------- |
| CLI         | ens160      | 10.12.10.1        |  24   |  10.12.10.2     |
| ISP         |             | DHCP              |  24   |                 |
|             |             | 10.12.10.2 DHCP   |  24   |                 |
|             |             | 10.12.20.2 DHCP   |  24   |                 |
|             |             | 10.12.30.2 DHCP   |  24   |                 |
| HQ-R        | ens160      | 10.12.20.1        |  24   |  10.12.20.2     |
|             | ens192      | 172.16.0.1        |  25   |    |
|             | GRE1        | 10.0.0.1          |  30   |    |
| BR-R        | ens160      | 10.12.30.1        |  24   |  10.12.30.2     |
|             | ens192      | 172.16.0.129      |  27   |    |
|             | GRE1        | 10.0.0.2          |  30   |    |
| HQ-SRV      | ens160      | 172.16.0.126      |  25   | 172.16.0.1      |
| BR-SRV      | ens160      | 172.16.0.158      |  27   | 172.16.0.129    |
