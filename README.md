# Raspberry Pi 4 5G router with RM500U-CNV hat

Crooked setup workflow for the RM500U-CNV hat

Tried on Debian 13

## Hat setup

Subsections from the [RM500U-CNV](https://www.waveshare.com/wiki/RM500U-CNV) wiki

```bash
wget https://files.waveshare.com/wiki/RM500U-CN/Driver/Linux/RM500U_FOR_RPI.zip
unzip RM500U_FOR_RPI.zip
cd RM500U_FOR_RPI

chmod +x ./install.sh
./install.sh
```

Run `minicom -D /dev/ttyUSB2` and send the commands

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

Exit minicom with CTRL-A X

## Router setup

```bash
route add -net 0.0.0.0 usb0                             # default route on usb0
iptables -t nat -A POSTROUTING -o usb0 -k MASQUERADE    # nat outgoing on usb0
iptables -A FORWARD -i wlan0 -o usb0 -j ACCEPT          # allow traffic between wlan0
iptables -A FORWARD -o wlan0 -i usb0 -j ACCEPT          # and usb0 to be forwarded
```

You should read all the configs in the [etc](./etc/) dir and modify where necessary

```bash
cp -vr ./etc/* /etc

systemctl start dhcpcd
systemctl start dnsmasq
systemctl start hostapd
```
