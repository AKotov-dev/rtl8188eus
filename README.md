### rtl8188eus v5.3.9

## RTL8188EUS, RTL8188EU, RTL8188ETV WiFi drivers

## Howto build/install (Mageia-9)
The `rtl8xxxu` kernel module for Realtek `rtl8188eus`, `rtl8188eu` and `rtl8188etv` WiFi adapters does not ensure their operation in access point (AP) mode. You need to replace the `rtl8xxxu` module with `8188eu`.

You will receive: kernel module `8188eu.ko.xz` and config for disabling the module by default - `realtek.conf`:
```
/usr/lib/modules/$(uname -r)/kernel/drivers/net/wireless/8188eu.ko.xz  
/etc/modprobe.d/realtek.conf
```
```
#Update sources + install the necessary packages to build the module
urpmi.update -a
urpmi --auto kernel-devel kernel-source make gcc automake bison flex git

#Link; Required for assembly (Makefile difference)
ln -s /usr/src/kernel-$(uname -r) /lib/modules/$(uname -r)/build

#Creating a kernel config
cd /usr/src/kernel-$(uname -r); make defconfig; make modules_prepare

#Download the sources of the 8188eus module, assemble and install
mkdir /123 && cd /123 && git clone https://github.com/AKotov-dev/rtl8188eus.git
cd rtl8188eus
make && make install
#Compress the module and update the list of modules
xz /usr/lib/modules/$(uname -r)/kernel/drivers/net/wireless/8188eu.ko; depmod -a

#We forget the rtl8xxxu driver, make 8188eu the main one and reboot
echo 'blacklist r8188eu' | tee -a '/etc/modprobe.d/realtek.conf'
echo 'blacklist rtl8xxxu' | tee -a '/etc/modprobe.d/realtek.conf'
reboot
```
## AP MODE howto
```
iwconfig
wlp0s11u1  IEEE 802.11bgn  ESSID:"MYWIFI"  Nickname:"<WIFI@REALTEK>"
          Mode:Master  Frequency:2.412 GHz  Access Point: 54:E4:BD:C6:3D:67   
          Bit Rate:72.2 Mb/s   Sensitivity:0/0  
          Retry:off   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=61/100  Signal level=51/100  Noise level=0/100
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

nmcli d
DEVICE             TYPE      STATE                 CONNECTION    
wlp0s11u1          wifi      подключено            MYWIFI        
enp0s3             ethernet  подключено            System enp0s3 
tun2socks          tun       подключено (внешнее)  tun2socks     
p2p-dev-wlp0s11u1  wifi-p2p  отключено             --            
lo                 loopback  без управления        --           
```
## MONITOR MODE howto
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

## NetworkManager configuration
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

## Credits
Realtek       - https://www.realtek.com<br>
Alfa Networks - https://www.alfa.com.tw<br>
aircrack-ng.  - https://www.aircrack-ng.org<br>
<br>
And all those who may be using or contributing to it of anykind. Thanks!<br>
