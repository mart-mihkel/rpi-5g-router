# RM500U-CNV hat router setup

crooked setup workflow for [RM500U-CNV](https://www.waveshare.com/wiki/RM500U-CNV) 5g hat router for raspberry pi 4.
working on debian 13.

## workflow

driver setup

```sh
modprobe option

echo 2c7c 0900 > /sys/bus/usb-serial/drivers/option1/new_id
echo 2c7c 0900 > /sys/bus/usb-serial/drivers/generic/new_id

wget https://files.waveshare.com/wiki/RM500U-CN/Driver/Linux/RM500U_FOR_RPI.zip
unzip RM500U_FOR_RPI.zip
cd RM500U_FOR_RPI

chmod +x ./install.sh
./install.sh

minicom -D /dev/ttyUSB2
```

in minicom

```minicom
AT+CPIN?
AT+QENG="servingcell"
AT+QCFG="ethernet",0
AT+QCFG="usbnet",1
AT+QNETDEVCTL=1,3,1
AT+QNETDEVCTL=1,1,1
AT+QCFG="nat",0
AT+CFUN=1,1
```

routing, enable nat, disable firewall

```sh
route add -net 0.0.0.0 usb0
iptables -t nat -A POSTROUTING -o usb0 -k MASQUERADE
iptables -A FORWARD -i wlan0 -o usb0 -j ACCEPT
iptables -A FORWARD -o wlan0 -i usb0 -j ACCEPT
```

copy configs, start services

```sh
cp -vr ./etc/* /etc

systemctl start dhcpcd
systemctl start dnsmasq
systemctl start hostapd
```
