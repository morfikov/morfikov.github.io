---
author: Morfik
categories:
- Linux
date: "2020-03-02T03:08:00Z"
published: true
status: publish
tags:
- debian
- kernel
- efi
- uefi
- luks
- lvm
GHissueID: 26
title: Linux kernel EFI boot stub i zaszyfrowany Debian (LUKS+LVM)
---

Szukając informacji na temat uruchamiania mojego zaszyfrowanego Debiana (LUKSv2+LVM) na laptopie z
EFI/UEFI, natrafiłem na dość ciekawy mechanizm zwany [kernel EFI boot stub][1], czasem też zwany
kernel EFISTUB. Zadaniem tego mechanizmu jest uruchomienie linux'a bezpośrednio z poziomu firmware
EFI z pominięciem czy też bez potrzeby stosowania dodatkowych menadżerów rozruchu (rEFInd) czy
bootloader'ów (grub/grub2/syslinux/extlinux). Jakby nie patrzeć bardzo ciekawa alternatywa, która
wymaga, by się z nią zapoznać i ocenić jej przydatność pod kątem użyteczności.

<!--more-->
## Czym jest linux kernel EFI boot stub

Zgodnie z tym co można wyczytać w dokumentacji kernela linux, EFI boot stub to umiejętność
tworzenia obrazu kernela w taki sposób, by podawał się on za obraz PE/COFF ([Portable
Executable][2]/[Common Object File Format][3]). Firmware EFI widząc taki plik interpretuje go jako
wykonywalny plik bootloader'a i go uruchamia. W ten sposób można zrezygnować z tradycyjnego
podejścia przy uruchamianiu systemu, który zakłada wykorzystanie czy to jakiegoś menadżera rozruchu,
czy też pełnowymiarowego bootloader'a. W zasadzie z racji, że EFI boot stub przeprowadza szereg
zadań charakterystycznych dla programu rozruchowego, to można by pokusić się o nazwanie go
bootloader'em.

## Konfiguracja kernela

By być w stanie wygenerować obraz kernela, który ma być uruchomiony przez firmware EFI, musimy mieć
w kernelu włączoną opcję `CONFIG_EFI_STUB` .  Warto też włączyć opcję `CONFIG_EFI_VARS` , jako że
umożliwi ona zarządzanie zmiennymi EFI za sprawą narzędzia `efibootmgr`. Jeśli korzystamy z
dystrybucyjnego kernela dostarczanego przez deweloperów dystrybucji Debian, to on ma już stosowne
opcje włączone i nic nie musimy dodatkowo konfigurować. Niemniej jednak, jeśli budujemy własny
kernel, to trzeba te opcje mieć na uwadze.

## Brak wsparcia dla systemu plików ext[,2,3,4]

Pierwsza niedogodność płynąca z uruchamiania systemu za sprawą EFI boot stub to brak wsparcia dla
natywnego systemu plików linux'a, tj. ext2, ext3 czy ext4. Do tego EFI boot stub może w zasadzie
uruchomić jedynie pliki z partycji ESP.

Warto w tym miejscu zaznaczyć, że partycja ESP może zostać zamontowana jako `/boot/` zamiast
standardowo jako `/efi/` . W takim układzie linux będzie widział partycję `/efi/` jako jako
partycję `/boot/` i standardowo będzie umieszczał w niej obrazy kernela i initrd przy
instalacji/aktualizacji jądra operacyjnego. Niemniej jednak, będziemy ograniczeni przez system
plików FAT, a ma to znaczenie choćby w przypadku [tworzenia dowiązań symbolicznych do plików
vmlinuz i initrd.img][4], by uniknąć ciągłej zmiany konfiguracji ilekroć zostanie wypuszczony nowy
kernel. W systemie plików FAT możemy zapomnieć o dowiązaniach, bo zwyczajnie on ich nie wspiera.

Mamy zatem do wyboru dwie opcje. W pierwszej z nich używamy dwóch partycji, tj. `/efi/`
sformatowana jako `fat16/fat32` i `/boot/` sformatowana jako `ext4` i kopiujemy obraz
kernela/initrd między nimi. W drugiej opcji używamy jednej partycji `/boot/` sformatowanej jako
`fat32` i przepisujemy nazwy plików obrazu kernela/initrd za każdym razem jak tylko w systemie
pojawi się aktualizacją jądra operacyjnego. Nie ważne którą opcję wybierzemy, to raczej bez
dodatkowej automatyzacji tego procesu się nie obejdzie. W moim przypadku wykorzystywana była osobna
partycja `/efi/` .

## Obraz z końcówką .efi

W Debianie obraz kernela znajduje się standardowo na partycji `/boot/` pod nazwą
`vmlinuz-x.x.x-x-amd64` . Po skopiowaniu go na partycję ESP trzeba mu zmienić nazwę dopisując na
jej końcu rozszerzenie `.efi` .

Ja nie miałem pojęcia o tym i skopiowałem jedynie obraz kernela na partycje ESP i uruchomiłem
system. Kernel się załadował, a system uruchomił się bez problemu. Wygląda na to, że niekoniecznie
trzeba dopisywać do nazwy obrazu kernela końcówkę `.efi` . Może to kwestia implementacji EFI, kto
wie. Tak czy inaczej, jeśli w naszym przypadku system nie chce się uruchomić, to sprawdźmy czy ta
końcówka coś pomoże.

## Ograniczona liczba wpisów

W przypadku mojego laptopa było zdefiniowanych 24 różnych wpisów w konfiguracji EFI. Poniżej
znajduje się pełna ich lista:

    # efibootmgr -v
    BootCurrent: 000A
    Timeout: 0 seconds
    BootOrder: 000A,0000,0001,0002,0003,000D,0007,0008,0009,000B,000C,000E,000F,0010,0011,0012
    Boot0000  Setup	FvFile(721c8b66-426c-4e86-8e99-3457c46ab0b9)
    Boot0001  Boot Menu	FvFile(126a762d-5758-4fca-8531-201a7f57f850)
    Boot0002  Diagnostic Splash Screen	FvFile(a7d8d9a6-6ab0-4aeb-ad9d-163e59a7a380)
    Boot0003  Lenovo Diagnostics	FvFile(3f7e615b-0d45-4f80-88dc-26b234958560)
    Boot0004  Startup Interrupt Menu	FvFile(f46ee6f4-4785-43a3-923d-7f786c3c8479)
    Boot0005  ME Configuration Menu	FvFile(82988420-7467-4490-9059-feb448dd1963)
    Boot0006  Rescue and Recovery	FvFile(665d3f60-ad3e-4cad-8e26-db46eee9f1b5)
    Boot0007  USB CD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,86701296aa5a7848b66cd49dd3ba6a55)
    Boot0008  USB FDD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,6ff015a28830b543a8b8641009461e49)
    Boot0009  ATAPI CD0	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,aea2090adfde214e8b3a5e471856a35401)
    Boot000A* ATA HDD0	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f600)
    Boot000B* ATA HDD1	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f601)
    Boot000C* ATA HDD2	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f602)
    Boot000D* USB HDD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,33e821aaaf33bc4789bd419f88c50803)
    Boot000E  PCI LAN	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,78a84aaf2b2afc4ea79cf5cc8f3d3803)
    Boot000F  ATAPI CD1	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,aea2090adfde214e8b3a5e471856a35404)
    Boot0010  Other CD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,aea2090adfde214e8b3a5e471856a35406)
    Boot0011* ATA HDD3	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f604)
    Boot0012  Other HDD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f606)
    Boot0013* IDER BOOT CDROM	PciRoot(0x0)/Pci(0x16,0x2)/Ata(0,1,0)
    Boot0014* IDER BOOT Floppy	PciRoot(0x0)/Pci(0x16,0x2)/Ata(0,0,0)
    Boot0015* ATA HDD	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,91af625956449f41a7b91f4f892ab0f6)
    Boot0016* ATAPI CD:	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,aea2090adfde214e8b3a5e471856a354)
    Boot0017* PCI LAN	VenMsg(bc7838d2-0f82-4d60-8316-c068ee79d25b,78a84aaf2b2afc4ea79cf5cc8f3d3803)

Do końca jeszcze nie znam przeznaczenia ich wszystkich ale jeśli chcemy mieć możliwość odpalania
linux'a bezpośrednio przez EFI, to trzeba dodać kolejny wpis.

W tym przypadku mogłem dodać maksymalnie 4 nowe wpisy. To bardzo niewiele. Na starym laptopie w
konfiguracji syslinux/extlinux mam tych wpisów ponad 10, każdy ma nieco inny zestaw parametrów,
tak bym mógł w przypadku zaistnienia różnych niecodziennych zdarzeń bez problemu odzyskać swój
system.

Pewnie nie wszystkie wpisy są potrzebne i kilka slotów można by odzyskać usuwając zbędne pozycje
ale trzeba też zaznaczyć tutaj fakt, że część wpisów nie może zostać usunięta. Po tym jak się je
usunie, to są tworzone na nowo po restarcie maszyny. Można zatem przyjąć, że w przypadku tego
laptopa, użytkownik ma do dyspozycji tylko 4 pozycje na określenie systemu, który chce bezpośrednio
uruchomić.

## Parametry kernela

Gdy w grę wchodzi standardowy bootloader, parametry kernela zwykłe podaje się bezpośrednio w
konfiguracji samego bootloader'a lub też w plikach systemowych, z których te parametry zostaną
podebrane przez bootloader. W przypadku EFI boot stub parametry kernela trzeba na sztywno
zdefiniować we wpisie, który dodawany jest za sprawą narzędzia `efibootmgr` z uruchomionego
systemu. By zmienić parametry kernela, trzeba usunąć cały wpis i stworzyć go na nowo dodając lub
zmieniając pożądane parametry i ich opcje.

Możemy też zapomnieć o tymczasowej zmianie parametrów przez edycję kernel cmd podczas rozruchu
systemu, tak jak to jest standardowo odstępne przez wciśnięcie klawisza `Tab` w okienku
bootloader'a.

## Bardzo niewygodna edycja wpisów

W przypadku bootloader'a, zwykle dodanie nowego wpisu nie stanowi problemu. Wystarczy edytować
konkretny plik, skopiować zwrotkę odpowiedzialną za jeden wpis wyświetlany w fazie startu i
przerobić go zgodnie z potrzebą. Z wpisami w EFI boot stub nie ma tak łatwo i trzeba używać tylko i
wyłączenie poleceń shell'owych, w których to trzeba zawsze definiować wszystkie parametry. Takie
polecenia mogą być naprawdę długie. Poniżej przykład:

    # efibootmgr --disk /dev/sda --part 5 --create --label "Debian EFISTUB" --loader \
                 '\EFI\BOOT\vmlinuz-5.4.0-4-amd64' --unicode 'root=/dev/mapper/sd_debian-root \
                 ro initrd=\EFI\BOOT\initrd.img-5.4.0-4-amd64' --verbose

To bardzo podstawowy przykład, do odszyfrowania mojego systemu na bazie LUKSv2 + LVM i nie zawiera
całej masy opcji kernela, których na co dzień używam w swojej starej maszynie. Gdybym je wszystkie
musiał tutaj określić, to ta linijka była by trzy razy dłuższa.

## Czasami parametr initrd= nie zadziała

Mój system jest w pełni zaszyfrowany i by móc go odpalić wymagany jest obraz initrd. Używając
tradycyjnego bootloader'a mogę wskazać dowolny plik/pliki za sprawą parametru `initd=` i problem
mam z głowy. A jak wygląda sprawa w przypadku EFI boot stub? Ano tak, że ten parametr `initrd=`
jest specyficzny dla firmware EFI i niekoniecznie może być wspierany w tej konkretnej implementacji.
Akurat w przypadku mojego laptopa jest. Zatem mój system da radę odszyfrować wykorzystując ten
mechanizm. Niemniej jednak, trzeba mieć na uwadze, że nie wszędzie i nie zawsze się to uda zrobić.

## Odwrócone ukośniki

Z reguły w linux'ach do określania ścieżek korzysta się ze slash'a ( `/` ) , np. `/home/morfik/`
ale do określania wpisów w EFI boot stub trzeba używać odwróconych ukośników, tj. backslash'a
( `\` ). Może ktoś uzna, że się czepiam ale popatrzcie na to powyższe polecenie dodawania wpisów. W
niektórych parametrach są normalne ukośniki, a w innych te odwrócone -- ten kto zaprojektował ten
mechanizm był idiotą... :]

## Interpretacja dodanych wpisów

Gdy już dodamy kilka wpisów na listę możliwych do uruchomienia systemów, to czasem chcielibyśmy
podejrzeć tę listę, by się upewnić, że zostanie załadowany ten kernel, który jest nam potrzebny.
Czasami wpisy mogą się różnić jedynie parametrami i zwykle nazwa, która nadaliśmy wpisowi, nam nic
nie powie. Z okna wyboru systemu podczas startu za bardzo nic się nie dowiemy, bo tam widnieje
jedynie, tylko krótka nazwa i podejrzeć parametrów nie sposób. W zasadzie to widać tylko tyle co na
poniższym obrazku:

![kernel-linux-efi-boot-stub-efistub-menu](/img/2020/03/001-kernel-linux-efi-boot-stub-efistub-menu.jpg#huge)

Ale nie o tym chciałem pisać, bo przecie istnieje narzędzie `efibootmgr` , które jest nam w stanie
zwrócić ładnie przeformatowaną listę opcji rozruchu, tak jak to było widać na początku, prawda? No
chyba, że się tam doda wpis z linux. Wtedy to wyjście wyglądać zaczyna tak:

    Boot0018* Debian EFISTUB	HD(5,GPT,9d090ca2-e968-4399-9af1-adb80ba4b924,0x38b4000,0xff800)/File(\EFI\BOOT\vmlinuz-5.4.0-4-amd64)r.o.o.t.=./.d.e.v./.m.a.p.p.e.r./.s.d._.d.e.b.i.a.n.-.r.o.o.t. .r.o. .i.n.i.t.r.d.=.\.E.F.I.\.B.O.O.T.\.i.n.i.t.r.d...i.m.g.-.5...4...0.-.4.-.a.m.d.6.4.

Co się tam pan czepiasz, przecie system idzie uruchomić...

Próbowałem znaleźć jakieś rozwiązanie tego problemu ale póki co mi się nie udało, a że ten
mechanizm EFI boot stub ma już i tak całą masę wad, to raczej wątpię, że przeznaczę jeszcze czas
na znalezienie rozwiązania problemów, których w normalnych menadżerach rozruchu się nie spotyka,
nie mówiąc już o bohaterskich próbach ich przezwyciężenia...

## Podsumowanie

Mi jakoś niespecjalnie ten cały EFI boot stub przypadł do gustu. Może znajdzie on zastosowanie w
przypadku przeciętnego Kowalskiego, który chce jedynie podnieść swój system jak najszybciej i bez
zbędnych pytań czy zagłębiania się w aspekty konfiguracyjne. No dobrze, że taki mechanizm istnieje.
Jego zaletą jest na pewno fakt, że jest on za free, nie wymaga instalowania żadnego dodatkowego
oprogramowania i działa OOTB nie oferując przy tym w zasadzie żadnej konfiguracji. No ja jednak
potrzebuję czegoś bardziej zaawansowanego i zwyczajnie EFI boot stub nie spełnia moich oczekiwań i
tych nawet podstawowych wymagań. Jeśli jednak ktoś korzysta z jednego systemu i nie miesza w nim,
to ten EFI boot stub jak najbardziej może mu się przydać ale jeśli tylko coś bardziej
zaawansowanego będziemy chcieli robić z naszym systemem, to trzeba rozejrzeć się za pełnoprawnym
menadżerem rozruchu czy bootloader'em.


[1]: https://www.kernel.org/doc/Documentation/efi-stub.txt
[2]: https://en.wikipedia.org/wiki/Portable_Executable
[3]: https://en.wikipedia.org/wiki/COFF
[4]: /post/jak-przepisac-linki-initrd-img-old-i-vmlinuz-old-do-boot/
