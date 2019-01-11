# Проброс USB-устройств в Citrix XenServer

Теоретически, Citrix XenServer позволяет "пробрасывать" любые устройства из dom0 в domU. На практике дело обстоит немного иначе: требуется аппаратная поддержка VT-d и далеко не всегда устройство может определиться в гостевой системе. Но, тем не менее, в консоли гипервизора можно выполнить:
```
# lspci | grep USB
```
Получим что-то наподобие:
```
00:14.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB xHCI Host Controller (rev 04)
00:1a.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB Enhanced Host Controller #2 (rev 04)
00:1d.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB Enhanced Host Controller #1 (rev 04)
```
Значает у нас три USB-контроллера с идентификаторами: 00:14.0, 00:1a.0 и 00:1d.0. Узнать, на каком контроллере расположено наше устройство, можно командой `lsusb`. Однако по умолчанию usbutils в XenServer (в его урезаном RedHat) не установлен. Установка проверена на Citrix XenServer 6.5, в более поздних версяих может быть иначе:
```
# yum install --enablerepo=base usbutils
# lsusb
```
Получаем:
```
Bus 004 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 004 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 0665:5161 Cypress Semiconductor USB to Serial
Bus 003 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
Допустим, нас интересует *Cypress Semiconductor USB to Serial*. Значит третье устрйство расположено на третьем контроллере и имеет идентификатор 00:1d.0. Теперь необходимо узнать UUID виртуальной машины, в которую будет осуществлен проброс:
```
# xe vm-list name-label=test
uuid ( RO) : cd9c4255-d28a-c086–113c-c1716293449d
name-label ( RW): test
power-state ( RO): running
```
где: test - имя нашей ВМ.
Следовательно:
```
# xe vm-param-set other-config:pci=0/000:00:1d.0 uuid=cd9c4255-d28a-c086–113c-c1716293449d
```
Проверить настройки можно:
```
# xe vm-param-list uuid=cd9c4255-d28a-c086–113c-c1716293449d | grep other-config
```
Отключить проброс контроллера:
```
# xe vm-param-remove param-name=other-config param-key=pci uuid=cd9c4255-d28a-c086–113c-c1716293449d
```
