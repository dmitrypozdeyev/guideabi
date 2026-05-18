         @page { size: 21.59cm 27.94cm; margin: 2cm } p { margin-bottom: 0.21cm; background: transparent } h5 { margin-top: 0.21cm; margin-bottom: 0.11cm; background: transparent; page-break-after: avoid } h5.western { font-family: "Nimbus Sans L", sans-serif; font-size: 12pt; font-weight: bold } h5.cjk { font-family: "DejaVu Sans"; font-size: 12pt; font-weight: bold } h5.ctl { font-family: "DejaVu Sans"; font-size: 12pt; font-weight: bold } a:link { color: #000080; text-decoration: underline } a:visited { color: #800000; text-decoration: underline }

Гайд по настройке технических и программных средств информационно-коммуникационных систем.

Выполнен студентом 2ИСП группы

Коротиным Игнатом.

1.  Настройка R1  
    1\. Заходим по деф. Ip 192.168.1.1
    

2.Меняем имя на r1.au.team

![](guide_html_5249e475.gif)

3\. Веб-морда у нас поменялась ( на keenetic), создаем зоны SRV и CAMS в моих сетях. Настраиваем ip и vlans.

![](guide_html_214447af.gif)

![](guide_html_84c37368.gif)

CAMS

![](guide_html_45b8e726.gif)

  

  

  

  

  

  

  

  

  

  

  

SERVERS

![](guide_html_7e9f5a64.gif)

4\. Заходим в пользователи и делаем разрешение  
![](guide_html_abfcaf38.png)

![](guide_html_f9d5a988.gif)

5\. Создаем в межсетевом экране протокол 22,222. (SSH, openWrt)

![](guide_html_b346ba33.gif)

\+ Протокол ICMP

![](guide_html_49b38db2.gif)

  

1.  2\. Настройка R2  
      
    
    1.Настройка vlan и ip
    

![](guide_html_dec7b2fd.gif)

![](guide_html_af56fb0.gif)

Так же создаем 22, 222 И ICMP протоколы.

![](guide_html_b43a6896.gif)

  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

  

Настройка DHCP и ip ADMINPC

P![](guide_html_a595f5cb.gif) RINT

![](guide_html_c27a2484.gif)

  

3\. Настройка GRE

( НА R2)

1\. Заходим в конфиг ( на нашем роутере через telnet) по айпишнику  
telnet 192.168.1.1

Login: admin

password: admin123456+

  

(config)> …

2\. В конфиге пишем:  
  
![](guide_html_91b4bd47.gif)

(Проверить конфиг можно с помощью (show interface Gre0))

4\. Настройка GRE  
На (R1)

1.заходим в конфиг через ssh

ssh [admin@192.168.1.1](mailto:admin@192.168.1.1)

password: admin123456+  
  
  

(config)> …  
2\. В конфиге пишем:  
![](guide_html_91b4bd47.gif)

  

3\. Сверяем ip ( Источник и назначение)

![](guide_html_64402d49.gif)

4\. Пингуем (Пинг есть, значит можем переходить к настройке bird)

  

4\. Настройка Bird (r2)

Заходим по ssh

ssh -p 222 [root@192.168.1.1](mailto:root@192.168.1.1)

password: keenetic

  

~ # …

opkg update

opkg install bird2

opkg install bird2c

  

Config (R2). (  /opt/etc/bird.conf)

  

  
log syslog all;

  

router id 10.10.10.2;

  

protocol device {

scan time 10;

}

  

protocol direct direct4 {

ipv4;

interface "\*";

}

  

filter export\_r2\_to\_ospf {

if net = 172.16.120.0/24 then accept;

if net = 172.16.100.0/24 then accept;

if net = 10.10.10.0/30 then accept;

reject;

}

  

filter import\_from\_ospf {

if net = 172.16.10.0/24 then accept;

if net = 172.16.110.0/24 then accept;

reject;

}

  

protocol kernel kernel4 {

ipv4 {

import none;

export all;

};

scan time 20;

persist;

}

  

protocol ospf v2 ospf\_gre {

ipv4 {

import filter import\_from\_ospf;

export filter export\_r2\_to\_ospf;

};

  

area 0.0.0.0 {

interface "ngre0" {

type ptp;

cost 10;

hello 10;

dead count 4;

authentication cryptographic;

password "abi2026" {

id 1;

};

};

};

}

  

Запускаем Bird

/opt/etc/init.d/S70bird restart

  
Проверяем соседство

birdc show ospf neighbors

  

Проверяем маршруты

birdc show route

  
Config (R1) (  /opt/etc/bird.conf)

  
  
log syslog all;

  

router id 10.10.10.1;

  

protocol device {

scan time 10;

}

  

protocol direct direct4 {

ipv4;

interface "\*";

}

  

filter export\_r1\_to\_ospf {

if net = 172.16.10.0/24 then accept;

if net = 172.16.110.0/24 then accept;

if net = 10.10.10.0/30 then accept;

reject;

}

  

filter import\_from\_ospf {

if net = 172.16.120.0/24 then accept;

if net = 172.16.100.0/24 then accept;

reject;

}

  

protocol kernel kernel4 {

ipv4 {

import none;

export all;

};

scan time 20;

persist;

}

  

protocol ospf v2 ospf\_gre {

ipv4 {

import filter import\_from\_ospf;

export filter export\_r1\_to\_ospf;

};

  

area 0.0.0.0 {

interface "ngre0" {

type ptp;

cost 10;

hello 10;

dead count 4;

authentication cryptographic;

password "abi2026" {

id 1;

};

};

};

}

![](guide_html_e220cf7.gif)

(Всё робит)

5.Настройка SRV

Заходим на сервер

login: root

password: 123

systemctl hostname srv.au.team

hostname

mcedit /etc/net/ ifaces/enp1s0/ipv4address

##### 172.16.110.50/24

mcedit /etc/net/ ifaces/enp1s0/ipv4route  
  

default via 172.16.110.254  
  
systemctl restart network

  

mcedit /etc/net/ ifaces/enp1s0/resolv.conf

  

nameserver 1.1.1.1

  

Проверим выход в интернет

ping ya.ru

  
Подключаемся по ssh [user@172.16.110.254](mailto:user@172.16.110.254)

password: 123

[root@srv](mailto:root@srv)~\]# apt-get update

[root@srv](mailto:root@srv)~\]# apt-get dist-upgrade

  
Установка Zoneminder alt linux по гайду:

Меняем язык и часовой пояс.  
  
Заходим через vlc (rtsp ссылка)  
rtsp://admin:password@172.16.10.10/Streaming/Channels/101

Меняем ip, а во время добавления камеры в Zoneminder, стираем admin:password и лишние символы, если будут ( в строчке пароль)

  

sD24H7v4a6  
  
6\. Установка Zabbix ( apt-get update)

1.Устанавливаем пакеты:

apt-get install git

apt-get install docker-engine

apt-get instal docker-compose

  

systemctl enable --now docker.service ( Запуск докера)

mcedit .env ( поправка Порта ( 80 →8080) )

  

Дальше установка по гайду:  
**zabbix docker compose**  
docker compose -f ./compose.yaml up -d ( поднятие)

  

Если будут ошибки снова возвращайся в mcedit .env и меняй необходимый протокол (с 443 на 843)  
Зайти на Zabbix через ip  
172.16.110.50:8080

\-------------------------------------------------------------------------

![](guide_html_53bcf47f.gif)

  

правила для маршрутизации

  

7\. Настройка CUPS

1\. Через гайд: Настройка принтера alt linux

![](guide_html_6b2769af.gif)

  

![](guide_html_4ce7e760.gif)

  

( На конкурсе будет другое подключение)  
epm play cnrdrvcups-ufr2

  

IPP everywhere

AirPrint

Generic PCL6