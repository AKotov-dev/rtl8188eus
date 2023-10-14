## rtl8188eus v5.3.9

# Realtek rtl8188eus &amp; rtl8188eu &amp; rtl8188etv WiFi drivers

[![Monitor mode](https://img.shields.io/badge/monitor%20mode-supported-brightgreen.svg)](#)
[![Frame Injection](https://img.shields.io/badge/frame%20injection-supported-brightgreen.svg)](#)
[![MESH Mode](https://img.shields.io/badge/mesh%20mode-supported-brightgreen.svg)](#)
[![GitHub issues](https://img.shields.io/github/issues/aircrack-ng/rtl8188eus.svg)](https://github.com/aircrack-ng/rtl8188eus/issues)
[![GitHub forks](https://img.shields.io/github/forks/aircrack-ng/rtl8188eus.svg)](https://github.com/aircrack-ng/rtl8188eus/network)
[![GitHub stars](https://img.shields.io/github/stars/aircrack-ng/rtl8188eus.svg)](https://github.com/aircrack-ng/rtl8188eus/stargazers)
[![GitHub license](https://img.shields.io/github/license/aircrack-ng/rtl8812au.svg)](https://github.com/aircrack-ng/rtl8188eus/blob/master/LICENSE)<br>
[![Android](https://img.shields.io/badge/android%20(8)-supported-brightgreen.svg)](#)
[![aircrack-ng](https://img.shields.io/badge/aircrack--ng-supported-blue.svg)](#)


# Supports
* Android 12/13
* MESH Support
* Monitor mode
* Frame injection
* Up to kernel v6.5+
... And a bunch of various wifi chipsets

# Howto build/install (Mageia-9)
Модуль ядра rtl8xxxu для WiFi-адаптеров Realtek rtl8188eus, rtl8188eu и rtl8188etv не обеспечивает их работу в режиме точки доступа (AP). Нужно заменить модуль rtl8xxxu на 8188eu...

#Будут получены: модуль ядра `8188eu.ko.xz` и конфиг отключения модуля по дефолту - `realtek.conf`
/usr/lib/modules/6.4.9-desktop-4.mga9/kernel/drivers/net/wireless/8188eu.ko
/etc/modprobe.d/realtek.conf
```
#Обновляем источники + ставим нужные пакеты для сборки модуля
urpmi.update -a
urpmi --auto kernel-devel kernel-source make gcc automake bison flex git

#Ссылка; Требуется для сборки (разница Makefile)
ln -s /usr/src/kernel-$(uname -r) /lib/modules/$(uname -r)/build

#Создаём конфиг ядра:
cd /usr/src/kernel-$(uname -r); make defconfig; make modules_prepare

#Скачиваем исходники модуля 8188eus, собираем и устанавливаем
mkdir /123 && cd /123 && git clone https://github.com/gglluukk/rtl8188eus.git
cd rtl8188eus
make && make install
#Сжимаем модуль и обновляем список модулей
xz /usr/lib/modules/$(uname -r)/kernel/drivers/net/wireless/8188eu.ko; depmod -a

#Забываем драйвер rtl8xxxu, делаем 8188eu основным и перезагружаемся
echo 'blacklist r8188eu' | tee -a '/etc/modprobe.d/realtek.conf'
echo 'blacklist rtl8xxxu' | tee -a '/etc/modprobe.d/realtek.conf'
reboot
```
```
#Проверка точки доступа (Access Point)
> iwconfig
wlp0s11u1  IEEE 802.11bgn  ESSID:"MYWIFI"  Nickname:"<WIFI@REALTEK>"
          Mode:Master  Frequency:2.412 GHz  Access Point: 54:E4:BD:C6:3D:67   
          Bit Rate:72.2 Mb/s   Sensitivity:0/0  
          Retry:off   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=61/100  Signal level=51/100  Noise level=0/100
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

> nmcli d
DEVICE             TYPE      STATE                 CONNECTION    
wlp0s11u1          wifi      подключено            MYWIFI        
enp0s3             ethernet  подключено            System enp0s3 
tun2socks          tun       подключено (внешнее)  tun2socks     
p2p-dev-wlp0s11u1  wifi-p2p  отключено             --            
lo                 loopback  без управления        --           
```

# MONITOR MODE howto
Use these steps to enter monitor mode.
```
sudo airmon-ng check kill
sudo ip link set <interface> down
sudo iw dev <interface> set type monitor
```
Frame injection test may be performed with
(after kernel v5.2 scanning is slow, run a scan or simply an airodump-ng first!)
```
sudo aireplay-ng -9 <interface>
```

# NetworkManager configuration
Add these lines below to "NetworkManager.conf" and ADD YOUR ADAPTER MAC below [keyfile]
This will make the Network-Manager ignore the device, and therefore don't cause problems.
```
[device]
wifi.scan-rand-mac-address=no

[ifupdown]
managed=false

[connection]
wifi.powersave=0

[main]
plugins=keyfile

[keyfile]
unmanaged-devices=mac:A7:A7:A7:A7:A7
```

# Credits
Realtek       - https://www.realtek.com<br>
Alfa Networks - https://www.alfa.com.tw<br>
aircrack-ng.  - https://www.aircrack-ng.org<br>
<br>
And all those who may be using or contributing to it of anykind. Thanks!<br>
