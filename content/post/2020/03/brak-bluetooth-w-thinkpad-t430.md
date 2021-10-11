---
author: Morfik
categories:
- Linux
date: "2020-03-11T21:03:00Z"
published: true
status: publish
tags:
- debian
- bluetooth
- lenovo
- thinkpad
- t430
GHissueID: 27
title: Brak bluetooth w ThinkPad T430 (BCM20702A1)
---

W moim laptopie Lenovo ThinkPad T430 jest obecny bluetooth `Broadcom Corp. BCM20702 Bluetooth 4.0`
o vid/pid `0a5c:21e6` . Niestety to urządzenie nie działa za dobrze na Debianie z powodu braku
odpowiedniego firmware (plik `BCM20702A1-0a5c-21e6.hcd` ), którego jak na złość nie ma w
repozytorium tej dystrybucji. Przydałoby się coś na to poradzić, by zmusić tę kartę/adapter do
pracy pod linux.

<!--more-->
## Brak pliku firmware

Podczas startu Debiana, na ekranie monitora (czy też w logu systemowym) można doszukać się
poniższych komunikatów świadczących o braku potrzebnego firmware:

	kernel: usb 1-1.4: new full-speed USB device number 4 using ehci-pci
	kernel: usb 1-1.4: New USB device found, idVendor=0a5c, idProduct=21e6, bcdDevice= 1.12
	kernel: usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
	kernel: usb 1-1.4: Product: BCM20702A0
	kernel: usb 1-1.4: Manufacturer: Broadcom Corp
	kernel: usb 1-1.4: SerialNumber: 74E54320FEA9
	kernel: usb 1-1.4: Device is not authorized for usage
	kernel: usb 1-1.4: authorized to connect
	kernel: Bluetooth: hci0: BCM: chip id 63
	kernel: Bluetooth: hci0: BCM: features 0x07
	mtp-probe[127320]: checking bus 1, device 4: "/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.4"
	kernel: Bluetooth: hci0: BCM20702A
	kernel: Bluetooth: hci0: BCM20702A1 (001.002.014) build 0000
	kernel: bluetooth hci0: firmware: failed to load brcm/BCM20702A1-0a5c-21e6.hcd (-2)
	kernel: bluetooth hci0: firmware: failed to load brcm/BCM20702A1-0a5c-21e6.hcd (-2)
	kernel: bluetooth hci0: Direct firmware load for brcm/BCM20702A1-0a5c-21e6.hcd failed with error -2
	kernel: Bluetooth: hci0: BCM: Patch brcm/BCM20702A1-0a5c-21e6.hcd not found

Z reguły by zmusić jakiś sprzęt cierpiący na brak firmware do działania pod Debianem, wystarczy
doinstalować mu stosowny pakiet mający w nazwie `firmware-*` . Nawet jeśli nie wiadomo, który do
końca plik wgrać, to zawsze można spróbować przeszukać pakiety przy pomocy `apt-file` i ustalić tę
paczkę, której potrzebujemy. Niestety w przypadku tego adaptera wbudowanego w mój laptop,
`apt-file` zwraca pusty wynik, czyli wygląda na to, że w repozytorium Debiana nie ma stosownego
pakietu, który by zawierał plik `BCM20702A1-0a5c-21e6.hcd` .

Poniżej znajdują się jeszcze dokładne informacje na temat tego adaptera bluetooth otrzymane via `lsusb` :

	# lsusb -vvv -d 0a5c:21e6

	Bus 001 Device 007: ID 0a5c:21e6 Broadcom Corp. BCM20702 Bluetooth 4.0 [ThinkPad]
	Device Descriptor:
	  bLength                18
	  bDescriptorType         1
	  bcdUSB               2.00
	  bDeviceClass          255 Vendor Specific Class
	  bDeviceSubClass         1
	  bDeviceProtocol         1
	  bMaxPacketSize0        64
	  idVendor           0x0a5c Broadcom Corp.
	  idProduct          0x21e6 BCM20702 Bluetooth 4.0 [ThinkPad]
	  bcdDevice            1.12
	  iManufacturer           1 Broadcom Corp
	  iProduct                2 BCM20702A0
	  iSerial                 3 74E54320FEA9
	  bNumConfigurations      1
	  Configuration Descriptor:
		bLength                 9
		bDescriptorType         2
		wTotalLength       0x00da
		bNumInterfaces          4
		bConfigurationValue     1
		iConfiguration          0
		bmAttributes         0xe0
		  Self Powered
		  Remote Wakeup
		MaxPower                0mA
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        0
		  bAlternateSetting       0
		  bNumEndpoints           3
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x81  EP 1 IN
			bmAttributes            3
			  Transfer Type            Interrupt
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0010  1x 16 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x82  EP 2 IN
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0040  1x 64 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x02  EP 2 OUT
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0040  1x 64 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       0
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0000  1x 0 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0000  1x 0 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       1
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0009  1x 9 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0009  1x 9 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       2
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0011  1x 17 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0011  1x 17 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       3
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0019  1x 25 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0019  1x 25 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       4
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0021  1x 33 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0021  1x 33 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        1
		  bAlternateSetting       5
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass      1
		  bInterfaceProtocol      1
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x83  EP 3 IN
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0031  1x 49 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x03  EP 3 OUT
			bmAttributes            1
			  Transfer Type            Isochronous
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0031  1x 49 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        2
		  bAlternateSetting       0
		  bNumEndpoints           2
		  bInterfaceClass       255 Vendor Specific Class
		  bInterfaceSubClass    255 Vendor Specific Subclass
		  bInterfaceProtocol    255 Vendor Specific Protocol
		  iInterface              0
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x84  EP 4 IN
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0020  1x 32 bytes
			bInterval               1
		  Endpoint Descriptor:
			bLength                 7
			bDescriptorType         5
			bEndpointAddress     0x04  EP 4 OUT
			bmAttributes            2
			  Transfer Type            Bulk
			  Synch Type               None
			  Usage Type               Data
			wMaxPacketSize     0x0020  1x 32 bytes
			bInterval               1
		Interface Descriptor:
		  bLength                 9
		  bDescriptorType         4
		  bInterfaceNumber        3
		  bAlternateSetting       0
		  bNumEndpoints           0
		  bInterfaceClass       254 Application Specific Interface
		  bInterfaceSubClass      1 Device Firmware Update
		  bInterfaceProtocol      1
		  iInterface              0
		  Device Firmware Upgrade Interface Descriptor:
			bLength                             9
			bDescriptorType                    33
			bmAttributes                        5
			  Will Not Detach
			  Manifestation Tolerant
			  Upload Unsupported
			  Download Supported
			wDetachTimeout                   5000 milliseconds
			wTransferSize                      64 bytes
			bcdDFUVersion                   1.10
	can't get device qualifier: Resource temporarily unavailable
	can't get debug descriptor: Resource temporarily unavailable
	Device Status:     0x0001
	  Self Powered

## Pozyskanie pliku BCM20702A1-0a5c-21e6.hcd

Na szczęście nie jest tak źle jak mogłoby się wydawać, [bo plik firmware][1] jak najbardziej
istnieje, tylko póki co nikt z Debiana nie pokusił się by zrobić stosowną paczkę i udostępnić ją
ludziom do łatwej instalacji w ich systemach. Dlatego też musimy ten plik pozyskać manualnie i
wgrać go w stosowne miejsce, by bluetooth zaczął nam działać. Odpalamy zatem terminal, logujemy
się na root, przechodzimy do katalogu `/lib/firmware/` i pobieramy potrzebny nam plik przy
pomocy `wget` :

	# cd /lib/firmware/
	# mkdir brcm/ && cd brcm/
	# wget https://github.com/winterheart/broadcom-bt-firmware/blob/master/brcm/BCM20702A1-0a5c-21e6.hcd

I to w zasadzie cała robota. Trzeba by oczywiście jeszcze zrestartować maszynę, by ten firmware
został załadowany, ewentualnie przeładować moduł `btusb` . Można też przełączyć hardware kill
switch (przełącznik sprzętowy) od WiFi/BT, który jest po prawej stronie obudowy tego ThinkPad'a.
Tak czy inaczej, teraz ten firmware powinien zostać już poprawnie załadowany, co można zobaczyć w
logu:

	kernel: usb 1-1.4: new full-speed USB device number 7 using ehci-pci
	kernel: usb 1-1.4: New USB device found, idVendor=0a5c, idProduct=21e6, bcdDevice= 1.12
	kernel: usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
	kernel: usb 1-1.4: Product: BCM20702A0
	kernel: usb 1-1.4: Manufacturer: Broadcom Corp
	kernel: usb 1-1.4: SerialNumber: 74E54320FEA9
	kernel: usb 1-1.4: Device is not authorized for usage
	kernel: usb 1-1.4: authorized to connect
	mtp-probe[135654]: checking bus 1, device 7: "/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.4"
	mtp-probe[135654]: bus: 1, device: 7 was not an MTP device
	mtp-probe[135657]: checking bus 1, device 7: "/sys/devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.4"
	mtp-probe[135657]: bus: 1, device: 7 was not an MTP device
	systemd[1]: Reached target bluetooth.target.
	kernel: Bluetooth: hci0: BCM: chip id 63
	kernel: Bluetooth: hci0: BCM: features 0x07
	kernel: Bluetooth: hci0: BCM20702A
	kernel: Bluetooth: hci0: BCM20702A1 (001.002.014) build 0000
	kernel: bluetooth  hci0: firmware: direct-loading firmware brcm/BCM20702A1-0a5c-21e6.hcd
	kernel: Bluetooth: hci0: BCM20702A1 (001.002.014) build 1757
	kernel: Bluetooth: hci0: Broadcom Bluetooth Device

Z racji, że instalowaliśmy ten firmware manualnie, to trzeba pamiętać o jego aktualizacji bo jakiś
czas.


[1]: https://github.com/winterheart/broadcom-bt-firmware
