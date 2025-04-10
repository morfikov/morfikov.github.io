---
author: Morfik
categories:
- Linux
- RaspberryPi
date:    2024-06-16 16:30:00 +0200
lastmod: 2024-06-16 16:30:00 +0200
published: true
status: publish
tags:
- debian
- ssd
- trim
- discard
- system-plików
- ext4
- udev
- usb
GHissueID: 602
title: TRIM/UNMAP w dyskach SSD podłączonych via adapter USB-SATA na linux
---

Jakiś czas temu opisywałem [jak na Debianie włączyć obsługę mechanizmu TRIM][2] (realizowanego
przez polecenie `fstrim` ) na podpiętych do komputera dyskach SSD. Problem w tym, że dyski SSD, z
którymi będziemy wchodzić w interakcję, nie zawsze będą podłączane do dedykowanych portów
SATA/mSATA. Jak zachowa się zatem nasz linux, gdy będziemy chcieli podłączyć po portu USB
zewnętrzny dysk SSD? Przez zewnętrzny dysk USB nie mam na myśli dedykowanych zewnętrznych dysków
USB, bo te raczej nie powinny sprawiać kłopotów. Chodzi mi bardziej o wewnętrzne dyski SSD (np. ze
starego laptopa), które zamkniemy w dedykowanej obudowie USB, lub też będziemy taki dysk podłączać
na krótko za pomocą adaptera USB-SATA. W takich przypadkach zwykle kernel linux'a nie odważy się
włączyć wsparcia dla TRIM dla nośników SSD i tak właśnie się stało w przypadku mojego nowo
zakupionego dysku od Goodram, a konkretnie jest to model `SSDPR-CX400-02T-G2` , który to został
podłączony do portu USB3 mojego Raspberry PI (i Debiana) przy pomocy kabelka USB-SATA (Unitek
USB3.1 USB-A to 2.5" SATA6G). Przez kilka miesięcy dysk sprawował się bez zarzutu ale ostatnio przy
próbie wgrania na niego danych (przez sieć), transfer spadł do dosłownie pojedynczych MiB/s.
Poszukałem trochę informacji i okazało się, że dla tego typu nośników [trzeba ręcznie włączyć
TRIM][7], o ile będzie to w ogóle możliwe.

<!--more-->

## Czy mój dysk SSD na USB wspiera TRIM

Ten nowo zakupiony dysk SSD ma w zasadzie działać w roli storage pod filmy/seriale odtwarzane na TV
podpiętym do mojego Raspberry Pi 4B. Dlaczego dysk SSD (i to jeszcze w technologi SLC), a nie dysk
HDD? Bo dyski SSD nie mają głowic magnetycznych i przy takim zastosowaniu (w roli dysku pod TV),
nawet dysk SSD SLC będzie o wiele żywotniejszy niż dyski HDD, w których głowica parkuje dosłownie
co kilka sekund. Może i istnieją sposoby, [by sobie poradzić z tym parkowaniem głowicy w dyskach
HDD][4] ale podejście producentów tych nośników do klientów sprawiło, że postanowiłem w końcu
pokazać im środkowy palec i pod storage wybrać dysk SSD. Z racji, że w zasadzie rzadko kiedy z tego
dysku coś będzie usuwane (chyba że dysk ulegnie zapełnieniu), to tą niezbyt dużą ilością cykli
wymazywania/zapisywania komórek flash (około 350) nie ma zbytnio się co przejmować. Podobnie sprawa
wygląda w kwestii wydajności, przynajmniej przy odczycie danych.

Po kilku miesiącach użytkowania tego dysku SSD zaszła potrzebna, by go nieco przeczyścić i tym
samym zrobić nieco wolnego miejsca pod nowe filmy i seriale. Problem w tym, że transfer po sieci w
pewnym momencie spadł mi ze 110 MiB/s do około 1-2 MiB/s i tak wisiał w menadżerze plików, a
postępu w kopiowaniu danych nie było żadnych (no dobra prawie żadnych). Myślałem, że może coś się
stało z procesem kopiowania (został przerwany czy zawiesił się) ale ten funkcjonował poprawnie.
Podejrzenie padło więc na mechanizm TRIM. Okazało się, że ten dysk SSD ma TRIM zupełnie wyłączony,
choć sam nośnik wsparcie dla TRIM posiada. Jak to możliwe? Wygląda na to, że w przypadku dysków
SSD podpinanych do portów USB komputera, ten cały TRIM może nieść ze sobą bardzo niemiłe
konsekwencje i dlatego jest domyślnie wyłączony.

By sprawdzić czy TRIM dla dysku SSD podpiętego do portu USB komputera jest aktywny i działa, możemy
albo skorzystać z polecenia `fstrim` na systemie plików takiego nośnika, albo też możemy posłużyć
się poleceniem `lsblk` .

W tym przypadku, polecenia `fstrim` zwróciło komunikat o braku wsparcia dla TRIM:

    # fstrim -v /media/morfik/gdata
    fstrim: /media/morfik/gdata: the discard operation is not supported

Jeśli zaś zajrzymy do wyjścia polecenia `lsblk` , to zobaczymy wartość 0B w kolumnie `DISC-MAX` ,
co też jednoznacznie nam mówi, że kernel wsparcia dla TRIM dla tego dysku z jakiegoś powodu nie
włączył:

    # lsblk -D /dev/sdc
    NAME   DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
    sdc           0      512B       0B         0
    └─sdc1        0      512B       0B         0

Jest to o tyle dziwne, że model dysku SSD `SSDPR-CX400-02T-G2` wsparcie dla mechanizmu TRIM jak
najbardziej posiada, co możemy odczytać z raportu
SMART ( `TRIM Command: Available, deterministic, zeroed` ):

    # smartctl -x  /dev/sdc
    smartctl 7.4 2023-08-01 r5530 [x86_64-linux-6.9.4-amd64] (local build)
    Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF INFORMATION SECTION ===
    Device Model:     SSDPR-CX400-02T-G2
    Serial Number:    G44034700
    LU WWN Device Id: 5 000000 0000027b9
    Firmware Version: SN17595
    User Capacity:    2,048,408,248,320 bytes [2.04 TB]
    Sector Size:      512 bytes logical/physical
    Rotation Rate:    Solid State Device
    Form Factor:      2.5 inches
    TRIM Command:     Available, deterministic, zeroed
    Device is:        Not in smartctl database 7.3/5610
    ATA Version is:   ACS-4 (minor revision not indicated)
    SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
    Local Time is:    Sun Jun 16 10:39:14 2024 CEST
    SMART support is: Available - device has SMART capability.
    SMART support is: Enabled
    AAM feature is:   Unavailable
    APM feature is:   Unavailable
    Rd look-ahead is: Enabled
    Write cache is:   Enabled
    DSN feature is:   Unavailable
    ATA Security is:  Disabled, NOT FROZEN [SEC1]
    Wt Cache Reorder: Enabled

    === START OF READ SMART DATA SECTION ===
    SMART overall-health self-assessment test result: PASSED

    General SMART Values:
    Offline data collection status:  (0x00) Offline data collection activity
                                            was never started.
                                            Auto Offline Data Collection: Disabled.
    Self-test execution status:      (   0) The previous self-test routine completed
                                            without error or no self-test has ever
                                            been run.
    Total time to complete Offline
    data collection:                (   33) seconds.
    Offline data collection
    capabilities:                    (0x7b) SMART execute Offline immediate.
                                            Auto Offline data collection on/off support.
                                            Suspend Offline collection upon new
                                            command.
                                            Offline surface scan supported.
                                            Self-test supported.
                                            Conveyance Self-test supported.
                                            Selective Self-test supported.
    SMART capabilities:            (0x0003) Saves SMART data before entering
                                            power-saving mode.
                                            Supports SMART auto save timer.
    Error logging capability:        (0x01) Error logging supported.
                                            General Purpose Logging supported.
    Short self-test routine
    recommended polling time:        (   2) minutes.
    Extended self-test routine
    recommended polling time:        (  85) minutes.
    Conveyance self-test routine
    recommended polling time:        (   2) minutes.
    SCT capabilities:              (0x0031) SCT Status supported.
                                            SCT Feature Control supported.
                                            SCT Data Table supported.

    SMART Attributes Data Structure revision number: 20
    Vendor Specific SMART Attributes with Thresholds:
    ID# ATTRIBUTE_NAME          FLAGS    VALUE WORST THRESH FAIL RAW_VALUE
      5 Reallocated_Sector_Ct   PO--C-   100   100   010    -    0
      9 Power_On_Hours          -O--C-   100   100   000    -    273
     12 Power_Cycle_Count       -O--C-   100   100   000    -    100
    164 Unknown_Attribute       ------   100   100   000    -    21476540424
    165 Unknown_Attribute       ------   100   100   000    -    26
    166 Unknown_Attribute       ------   100   100   000    -    5
    167 Unknown_Attribute       -O---K   100   100   000    -    8
    194 Temperature_Celsius     -O---K   029   029   000    -    29 (Min/Max 24/40)
    199 UDMA_CRC_Error_Count    -O--C-   100   100   000    -    0
    241 Total_LBAs_Written      -O--CK   100   100   000    -    1776
    242 Total_LBAs_Read         -O--CK   100   100   000    -    396
                                ||||||_ K auto-keep
                                |||||__ C event count
                                ||||___ R error rate
                                |||____ S speed/performance
                                ||_____ O updated online
                                |______ P prefailure warning

    General Purpose Log Directory Version 1
    SMART           Log Directory Version 1 [multi-sector log support]
    Address    Access  R/W   Size  Description
    0x00       GPL,SL  R/O      1  Log Directory
    0x01           SL  R/O      1  Summary SMART error log
    0x02           SL  R/O     51  Comprehensive SMART error log
    0x03       GPL     R/O     64  Ext. Comprehensive SMART error log
    0x04       GPL,SL  R/O      8  Device Statistics log
    0x06           SL  R/O      1  SMART self-test log
    0x07       GPL     R/O      1  Extended self-test log
    0x09           SL  R/W      1  Selective self-test log
    0x10       GPL     R/O      1  NCQ Command Error log
    0x11       GPL     R/O      1  SATA Phy Event Counters log
    0x30       GPL,SL  R/O      9  IDENTIFY DEVICE data log
    0x80-0x9f  GPL,SL  R/W     16  Host vendor specific log
    0xe0       GPL,SL  R/W      1  SCT Command/Status
    0xe1       GPL,SL  R/W      1  SCT Data Transfer

    SMART Extended Comprehensive Error Log Version: 1 (64 sectors)
    No Errors Logged

    SMART Extended Self-test Log Version: 1 (1 sectors)
    Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
    # 1  Extended offline    Completed without error       00%         1         -

    SMART Selective self-test log data structure revision number 1
     SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
        1        0        0  Not_testing
        2        0        0  Not_testing
        3        0        0  Not_testing
        4        0        0  Not_testing
        5        0        0  Not_testing
    Selective self-test flags (0x0):
      After scanning selected spans, do NOT read-scan remainder of disk.
    If Selective self-test is pending on power-up, resume after 0 minute delay.

    SCT Status Version:                  3
    SCT Version (vendor specific):       1 (0x0001)
    Device State:                        Active (0)
    Current Temperature:                    32 Celsius
    Power Cycle Min/Max Temperature:      ?/32 Celsius
    Lifetime    Min/Max Temperature:      ?/ ? Celsius
    Under/Over Temperature Limit Count:   0/0

    SCT Temperature History Version:     2
    Temperature Sampling Period:         1 minute
    Temperature Logging Interval:        1 minute
    Min/Max recommended Temperature:     -127/127 Celsius
    Min/Max Temperature Limit:           -127/127 Celsius
    Temperature History Size (Index):    478 (369)

    Index    Estimated Time   Temperature Celsius
     370    2024-06-16 02:42     ?  -
     371    2024-06-16 02:43    30  ***********
    ...
     369    2024-06-16 10:39    32  *************

    SCT Error Recovery Control command not supported

    Device Statistics (GP Log 0x04)
    Page  Offset Size        Value Flags Description
    0x01  =====  =               =  ===  == General Statistics (rev 1) ==
    0x01  0x008  4             100  ---  Lifetime Power-On Resets
    0x01  0x010  4             273  ---  Power-on Hours
    0x01  0x018  6      3725681472  ---  Logical Sectors Written
    0x01  0x020  6        13529637  ---  Number of Write Commands
    0x01  0x028  6       832035662  ---  Logical Sectors Read
    0x01  0x030  6         3014905  ---  Number of Read Commands
    0x07  =====  =               =  ===  == Solid State Device Statistics (rev 1) ==
    0x07  0x008  1               5  N--  Percentage Used Endurance Indicator
                                    |||_ C monitored condition met
                                    ||__ D supports DSN
                                    |___ N normalized value

    Pending Defects log (GP Log 0x0c) not supported

    SATA Phy Event Counters (GP Log 0x11)
    ID      Size     Value  Description
    0x0001  2            0  Command failed due to ICRC error
    0x0003  2            0  R_ERR response for device-to-host data FIS
    0x0004  2            0  R_ERR response for host-to-device data FIS
    0x0006  2            0  R_ERR response for device-to-host non-data FIS
    0x0007  2            0  R_ERR response for host-to-device non-data FIS
    0x0008  2            0  Device-to-host non-data FIS retries
    0x0009  4            0  Transition from drive PhyRdy to drive PhyNRdy
    0x000a  4            1  Device-to-host register FISes sent due to a COMRESET
    0x000f  2            0  R_ERR response for host-to-device data FIS, CRC
    0x0010  2            0  R_ERR response for host-to-device data FIS, non-CRC
    0x0012  2            0  R_ERR response for host-to-device non-data FIS, CRC
    0x0013  2            0  R_ERR response for host-to-device non-data FIS, non-CRC

Zatem wsparcie dla TRIM w firmware dysku SSD jest ale pozostaje ono nieaktywne w systemie. Problem
tkwi w adapterze USB-SATA (czy też zewnętrznej obudowie USB), za pomocą którego ten nośnik SSD
został podpięty do portu USB komputera. Najwyraźniej z jakiegoś powodu taka konfiguracja upośledza
ten cały mechanizm TRIM.

## Problematyczny adapter USB-SATA (obudowa USB)

Zgodnie z tym, co [można wyczytać tutaj][5], taki adapter USB-SATA ma spełniać dwie główne funkcje.
Pierwszą z nich jest interfejs UAS, który jest interfejsem USB do kapsułkowania (enkapsulacji)
poleceń protokołu SCSI. Drugą zaś jest translacja, gdzie polecenia SCSI są konwertowane na
polecenia ATA. Zestaw poleceń określonych przez protokół SCSI jest spory i nie wszystkie
urządzenia SCSI wspierają wszystkie te polecenia. Dlatego też istnieje kilka poleceń, które mogą
zostać wykorzystane do UNMAP (TRIM to polecenie ATA, UNMAP to jego analog SCSI). Translator
SCSI-ATA może wspierać jedno z tych poleceń, może także wspierać kilka lub wszystkie ale też może
nie wspierać żadnego. W przypadku tego ostatniego nie będzie możliwy UNMAP bloku dysku SSD. Dlatego
też trzeba się zaopatrzyć w taki adapter USB-SATA, który wspiera choć jedno polecenie UNMAP.

Poniżej jest trochę informacji na temat samego adaptera USB-SATA (Unitek USB3.1 USB-A to 2.5"
SATA6G):

    # lsusb
    ...
    Bus 004 Device 003: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    ...

    # lsusb -t
    ...
    /:  Bus 004.Port 001: Dev 001, Class=root_hub, Driver=xhci_hcd/4p, 5000M
        |__ Port 001: Dev 003, If 0, Class=Mass Storage, Driver=uas, 5000M
    ...

    # lsusb -vvv -d 174c:55aa

    Bus 004 Device 003: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               3.10
      bDeviceClass            0 [unknown]
      bDeviceSubClass         0 [unknown]
      bDeviceProtocol         0
      bMaxPacketSize0         9
      idVendor           0x174c ASMedia Technology Inc.
      idProduct          0x55aa ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
      bcdDevice            1.00
      iManufacturer           2 asmedia
      iProduct                3 ASMT1153e
      iSerial                 1 123456789394
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength       0x0079
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xc0
          Self Powered
        MaxPower                0mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass         8 Mass Storage
          bInterfaceSubClass      6 SCSI
          bInterfaceProtocol     80 Bulk-Only
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst              15
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst              15
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       1
          bNumEndpoints           4
          bInterfaceClass         8 Mass Storage
          bInterfaceSubClass      6 SCSI
          bInterfaceProtocol     98
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst              15
            MaxStreams             32
            Data-in pipe (0x03)
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst              15
            MaxStreams             32
            Data-out pipe (0x04)
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst              15
            MaxStreams             32
            Status pipe (0x02)
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x04  EP 4 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst               0
            Command pipe (0x01)
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength       0x0016
      bNumDeviceCaps          2
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x0000f41e
          BESL Link Power Management (LPM) Supported
        BESL value     1024 us
        Deep BESL value    61440 us
      SuperSpeed USB Device Capability:
        bLength                10
        bDescriptorType        16
        bDevCapabilityType      3
        bmAttributes         0x00
        wSpeedsSupported   0x000e
          Device can operate at Full Speed (12Mbps)
          Device can operate at High Speed (480Mbps)
          Device can operate at SuperSpeed (5Gbps)
        bFunctionalitySupport   1
          Lowest fully-functional device speed is Full Speed (12Mbps)
        bU1DevExitLat          10 micro seconds
        bU2DevExitLat        2047 micro seconds
    can't get debug descriptor: Resource temporarily unavailable
    Device Status:     0x000d
      Self Powered
      U1 Enabled
      U2 Enabled

## Ryzyko utraty danych i uszkodzenia dysku SSD

Trzeba tutaj wyraźnie rozgraniczyć dwie rzeczy. To, że w firmware dysku SSD zaimplementowano
wsparcie dla TRIM nie oznacza z automatu, że kernel linux'a będzie do takiego nośnika przesyłał
takie żądania, przez co sam nośnik SSD może (i prawdopodobnie będzie) nam się zachowywać tak, jakby
wsparcia dla TRIM nie posiadał. Przyczyny mogą być różne, choć ta najpopularniejsza to błędy w
komunikacji między kernelem a dyskiem, co może prowadzić do uwalenia dysku lub utraty
zgromadzonych na nim danych. Dlatego też jeśli zaistnieją przesłanki, że TRIM nie powinien być
obsługiwany dla tego konkretnego dysku SSD, to kernel go nie włącza. Z reguły kernel linux'a ma
dobre powody by nie włączać na pewnych nośnika SSD obsługi mechanizmu TRIM. Niemniej jednak, w
pewnych przypadkach decyzja podjęta przez kernel może być niewłaściwa i trzeba będzie taki nośnik
SSD oznaczyć ręcznie, by kernel na nim TRIM włączył.

Jeśli nie jesteśmy przekonani co do włączania TRIM dla dysków SSD zamkniętych w zewnętrznych
obudowach USB czy też podłączanych za pomocą adaptera USB-SATA, to najlepszym rozwiązaniem dla nas
będzie wypięcie takiego nośnika z obudowy/adaptera i podłączenie go do slotu SATA w komputerze i
dopiero w takim przypadku zainicjowanie TRIM przez wydanie polecenia `fstrim` na podmontowanym
systemie plików. W taki sposób nie ma ryzyka utraty danych czy uszkodzenia samego dysku SSD,
niemniej jednak jeśli zapisujemy dużo danych na dysku, to takie ciągłe przepinanie mija się raczej
z celem.

## Strony VPD (Vital Product Data)

Czytając dalej ten podlinkowany wyżej artykuł, możemy wyłapać informację, że ze względu na bogactwo
zestawu poleceń SCSI, sterownik kernela linux musi wiedzieć jakie funkcje są obsługiwane przez dane
urządzenie. Odbywa się to za pomocą stron informacyjnych, zwanych stronami VPD (Vital Product Data),
zwracanych przez urządzenie do sterownika.

Pierwszą stroną jest `Supported VPD pages`, która zawiera listę stron obsługiwanych przez
dane urządzenie. Zwykle sterownik wysyła zapytanie o tę pierwszą stronę, a następnie o
interesujące go strony, jeśli są one obsługiwane. Jeśli chodzi o funkcję UNMAP, interesującą nas
stroną jest strona `Logical block provisioning` .

Na linux'ach jesteśmy w stanie wysłać zapytanie o strony `Supported VPD pages` oraz
`Logical block provisioning` przy pomocy narzędzia `sg_vpd` (z pakietu `sg3-utils` ). Musimy
jedynie określić dysk, do którego zapytania powędrują:

Dla strony `Supported VPD pages` :

    # sg_vpd -p bl /dev/sdc
    Block limits VPD page (SBC):
      Write same non-zero (WSNZ): 0
      Maximum compare and write length: 0 blocks [Command not implemented]
      Optimal transfer length granularity: 1 blocks
      Maximum transfer length: 65535 blocks
      Optimal transfer length: 65535 blocks
      Maximum prefetch transfer length: 65535 blocks
      Maximum unmap LBA count: 4194240
      Maximum unmap block descriptor count: 1
      Optimal unmap granularity: 1 blocks
      Unmap granularity alignment valid: false
      Unmap granularity alignment: 0 [invalid]
      Maximum write same length: 0 blocks [not reported]
      Maximum atomic transfer length: 0 blocks [not reported]
      Atomic alignment: 0 [unaligned atomic writes permitted]
      Atomic transfer length granularity: 0 [no granularity requirement
      Maximum atomic transfer length with atomic boundary: 0 blocks [not reported]
      Maximum atomic boundary size: 0 blocks [can only write atomic 1 block]

Dla strony `Logical block provisioning` :

    # sg_vpd --page=0xb2 /dev/sdc
    # sg_vpd -p lbpv /dev/sdc
    Logical block provisioning VPD page (SBC):
      Unmap command supported (LBPU): 1
      Write same (16) with unmap bit supported (LBPWS): 0
      Write same (10) with unmap bit supported (LBPWS10): 0
      Logical block provisioning read zeros (LBPRZ): 0
      Anchored LBAs supported (ANC_SUP): 0
      Threshold exponent: 0 [threshold sets not supported]
      Descriptor present (DP): 0
      Minimum percentage: 0 [not reported]
      Provisioning type: 0 (not known or fully provisioned)
      Threshold percentage: 0 [percentages not supported]

To co się rzuca w oczy, to fakt, że translator SCSI-ATA wspiera polecenie
UNMAP ( `Unmap command supported (LBPU)` ) ale nie wspiera [poleceń WRITE SAME][6],
tj. ( `Write same (16)` i `Write same (10)` ). Będzie to miało znaczenie w późniejszym procesie
konfiguracji TRIM dla tego nośnika SSD.

### Brak auto konfiguracji w oparciu o VPD

Normalnie sterownik odczytałby te strony i odpowiednio skonfigurował urządzenie. Niemniej jednak,
istnieje ryzyko, że próba odczytania niektórych stron VPD może doprowadzić do
zablokowania (lock-up) lub też wręcz uszkodzenia nośnika SSD. Dlatego też deweloperzy kernela linux
zdecydowali się [domyślnie nie odpytywać stron VPD][1] dla urządzeń SCSI podłączonych przez USB i
nie konfigurować zaawansowanych funkcji, takich jak UNMAP.

## Jak włączyć TRIM w dysku SSD na USB

W tym przypadku mamy do czynienia z dyskiem SSD od Goodram, konkretnie jest to
model `SSDPR-CX400-02T-G2` . Poniższe instrukcje zostały przetestowane tylko i wyłączenie z tym
dyskiem i jeśli nie mamy pewności co do obsługi TRIM/UNMAP w przypadku naszego dysku/adaptera
USB-SATA, to lepiej jest tych poniższych kroków nie przeprowadzać.

### Ręczne określenie wartości w provisioning_mode (konfiguracja UNMAP)

Dobra wiadomość jest taka, że te powyżej opisane funkcje można skonfigurować z przestrzeni
użytkownika używając do tego celu pliku `/sys/block/*/device/scsi_disk/*/provisioning_mode`
lub `/sys/devices/**/scsi_disk/**/provisioning_mode` . Gdy podłączymy dysk SSD za pomocą adaptera
USB-SATA do portu USB komputera, to wartość w tym pliku będzie prawdopodobnie wskazywać na `full` :

    # find /sys/ -name provisioning_mode -exec grep -H . {} + | sort
    /sys/devices/pci0000:00/0000:00:14.0/usb4/4-1/4-1:1.0/host6/target6:0:0/6:0:0:0/scsi_disk/6:0:0:0/provisioning_mode:full
    ...

Możemy jednak tę wartość przestawić ręcznie zapisując powyższy plik. W tym przypadku pozostaje nam
w zasadzie jedynie `unmap` ponieważ ten dysk podłączony przez adapter USB-SATA obsługuje jedynie
polecenie UNMAP. Można także określić wartości (jeśli są wspierane ) `writesame_16` , `writesame_10` , `writesame_zero` oraz `disabled` .

W przypadku, gdy mamy więcej niż jeden dysk, dobrze jest upewnić się, że ta ścieżka powyżej jest
odpowiednia:

    # lsscsi -liv
    ...
    [6:0:0:0]    disk    SSDPR-CX 400-02T-G2       0     /dev/sdc   SSDPR-CX_400-02T-G2_123456789394-0:0
      state=running queue_depth=30 scsi_level=7 type=0 device_blocked=0 timeout=30
      dir: /sys/bus/scsi/devices/6:0:0:0  [/sys/devices/pci0000:00/0000:00:14.0/usb4/4-1/4-1:1.0/host6/target6:0:0/6:0:0:0]
    list_ndevices: scandir: /sys/class/nvme/: No such file or directory
    NVMe module may not be loaded

Mając ustaloną ścieżkę oraz informację, że adapter USB-SATA wspiera jedynie polecenie UNMAP,
zapisujemy stosowny plik w poniższy sposób:

    # echo unmap > /sys/devices/pci0000:00/0000:00:14.0/usb4/4-1/4-1:1.0/host6/target6:0:0/6:0:0:0/scsi_disk/6:0:0:0/provisioning_mode

Sprawdzamy, czy wartość została poprawnie ustawiona:

    # cat /sys/devices/pci0000:00/0000:00:14.0/usb4/4-1/4-1:1.0/host6/target6:0:0/6:0:0:0/scsi_disk/6:0:0:0/provisioning_mode
    unmap

### Wartości dla discard_max_bytes i discard_max_hw_bytes

Sprawdzamy także co zostało ustawione w pliku `discard_max_bytes` i w `discard_max_hw_bytes` ,
które po włączeniu UNMAP zostaną automatycznie dostosowane (była tam wartość `0` ):

    # cat /sys/block/sdc/queue/discard_max_bytes
    4294966784

    # cat /sys/block/sdc/queue/discard_max_hw_bytes
    4294966784

Zgodnie z tym co [można wyczytać na wiki Gentoo][3], wartość którą widzimy tutaj, to ilość bajtów,
które mechanizm TRIM będzie w stanie wyczyścić w jednym podejściu. Możemy ją obliczyć posługując
się następującymi danymi:

    # sg_vpd -p bl /dev/sdc
    ...
      Maximum unmap LBA count: 4194240
    ...

oraz

    # sg_readcap -l /dev/sdc
    ...
       Logical block length=512 bytes
    ...

Mając informację o `Maximum unmap LBA count` oraz `Logical block length` , możemy ich wartości
pomnożyć przez siebie:

    # echo '4194240*512' | bc
    2147450880

Trochę to dziwne, że otrzymana wartość jest inna od tej domyślnie ustawionej. Nie wiem czy powinno
się zawierzyć tej automatycznie wykalkulowanej wartości, czy też tej obliczonej ręcznie. W każdym
razie jeśli nie chcemy zawierzać tej automatycznej, to wynik tego powyższego obliczenia możemy
przesłać do pliku `/sys/block/sdc/queue/discard_max_bytes` :

    # echo 2147450880 > /sys/block/sdc/queue/discard_max_bytes

Przeglądając pliki `discard_max_bytes` innych moich dysków SSD, wygląda na to, że jednak
wartość `2147450880` jest pewniejsza, bo inne nośniki mają ustawione właśnie `2147450880` . Jedyne
co mi przychodzi do głowy, to że ta różnica może wynikać ze sporo większego rozmiaru dysku
SSD (256GB vs. 2TB).

W zasadzie to te dwa pliki ( `discard_max_bytes` (RW) i `discard_max_hw_bytes` (RO) ) służą jedynie
do zmniejszenia ilości danych/bloków, które mogą zostać wyczyszczone w pojedynczej operacji TRIM.
Czasem może się zdarzyć tak, że gdy system będzie czyścił zbyt wiele bloków naraz, to wzrosną nam
dość znacznie opóźnienia w zapytaniach do dysku SSD, co zdegraduje jego wydajność. Dlatego
istnieje taki mechanizm bezpieczeństwa w kernelu, by ilość tych danych ograniczyć, co powinno
zredukować opóźnienia. Ja bym pozostawił wartość domyślną i jedynie w przypadku ewentualnej
odczuwalnej degradacji wydajności dysku SSD tę wartość w pliku `discard_max_bytes` bym próbował
przepisać.

Tak czy inaczej w wyjściu `lsblk` na pozycji `DISC-MAX` powinna już być widoczna jakaś wartość:

    # lsblk --discard /dev/sdc
    NAME   DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
    sdc           0      512B       4G         0
    └─sdc1        0      512B       4G         0

## Test TRIM dysku SSD na USB

W zasadzie jedyne co zrobiliśmy w przypadku tego dysku SSD to zapisaliśmy `unmap` w
pliku `provisioning_mode` i to powinno włączyć mechanizm TRIM dla tego nośnika SSD podpiętego przez
ten adapter USB-SATA do portu USB komputera. Czy tak się faktycznie stało, możemy ocenić montując
system plików dowolnej partycji dysku i wydając w terminalu polecenie `fstrim` :

    # fstrim -v /media/morfik/gdata
    /media/morfik/gdata: 482.5 GiB (518127964160 bytes) trimmed

Jak widać tym razem polecenie `fstrim` na tym dysku zadziałało. W zależności od ilości bloków
przeznaczonych do wyczyszczenia, to powyższe polecenie może zająć dłuższą chwilę. Po jego
wykonaniu dobrze jest dać dyskowi chwilę na ogarnięcie się. Z tego co zaobserwowałem, to
bezpośrednio po wykonaniu polecenia `fstrim` wydajność dysku nie powraca od razu do tej wyjściowej.
Potrzebnych jest kilka-kilkanaście dodatkowych minut, by transfer z paru MiB/s powrócił do tego, co
było przed zapełnieniem się dysku.

Po odczekaniu chwili, odmontowujemy partycję i ponownie ją montujemy. Jeśli jesteśmy w stanie
odczytać dane zgromadzone na dysku SSD, to znaczy, że wszystko jest w porządku i możemy przejść do
napisania reguły dla UDEV'a dla tego nośnika.

## Reguła dla UDEV

Sposób z zapisywaniem pliku `provisioning_mode` ma efekt jedynie tymczasowy i działa jedynie do
momentu odłączenia dysku SSD od komputera. Po ponownym podłączeniu trzeba by ponownie zapisać
wspomniany plik. Jest to mało praktyczne i przydałoby się napisać regułę dla UDEV'a, która za nas
automatycznie dostosuje ten plik po podpięciu dysku do portu USB.

Edytujemy (lub tworzymy) plik `/etc/udev/rules.d/10-trim.rules` i dodajemy do niego poniższą
zawartość:

    # SSD disk Goodram SSDPR-CX400-02T-G2 over Unitek USB3.1 USB-A to 2.5" SATA6G USB-SATA adapter
    #
    ACTION=="add|change", ATTRS{idVendor}=="174c", ATTRS{idProduct}=="55aa", SUBSYSTEM=="scsi_disk", ATTR{provisioning_mode}="unmap"

Przeładowujemy politykę UDEV'a i patrzymy czy plik `provisioning_mode` zawiera już odpowiednią
wartość (dla pewności dobrze jest odłączyć dysk od portu USB i podpiąć go ponownie):

    # udevadm control --reload-rules
    # udevadm trigger --type=devices --action=change

    # cat /sys/devices/pci0000:00/0000:00:14.0/usb4/4-1/4-1:1.0/host6/target6:0:0/6:0:0:0/scsi_disk/6:0:0:0/provisioning_mode
    unmap

Jak widać, `unmap` został już ustawiony automatycznie.

## Podsumowanie

Przyznam, że gdy kupowałem dysk SSD z zamiarem używania go po USB jako storage, nie byłem świadomy,
że TRIM w takiej sytuacji nie będzie działał z automatu oraz, że próba jego włączenia może okazać
się katastrofalna w skutkach. Na szczęście, w przypadku tego dysku SSD nic złego się nie wydarzyło,
dane na dysku są i można je odczytać, a sam nośnik działa do tej pory, z nieco większą wydajnością.
Dlatego też jeśli zamierzamy kupić dysk SSD i podłączać go przez adaptery/obudowy USB-SATA, to
lepiej upewnić się, że ta obudowa czy adapter wspierają TRIM/UNMAP, bo jeśli tak nie będzie, to
trzeba będzie liczyć się z transferami rzędu pojedynczych MiB z chwilą, gdy dysk po raz pierwszy
się zapełni.


[1]: https://git.kernel.org/pub/scm/linux/kernel/git/mkp/linux.git/commit/?h=5.18/discovery&id=916740efdd2208564decee40a6049674f2063811
[2]: /post/trim-discard-przy-luks-lvm-na-dysku-ssd-pod-debian-linux/
[3]:  https://wiki.gentoo.org/wiki/Discard_over_USB
[4]: /post/parkowanie-glowicy-w-dyskach-wstern-digital/
[5]: https://superuser.com/questions/1740543/no-trim-discard-with-a-sata-ssd-connected-through-an-uasp-enabled-usb-adapter
[6]: https://linux.die.net/man/8/sg_write_same
[7]: https://www.jeffgeerling.com/blog/2020/enabling-trim-on-external-ssd-on-raspberry-pi
