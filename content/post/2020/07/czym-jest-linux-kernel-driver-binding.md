---
author: Morfik
categories:
- Linux
date: "2020-07-28T19:39:00Z"
published: true
status: publish
tags:
- debian
- kernel
- moduły-kernela
title: Czym jest linux kernel driver binding
---

Bawiąc się ostatnio QEMU/KVM na swoim laptopie z zainstalowanym Debianem natrafiłem na ciekawe
zagadnienie związane z wirtualizacją, tj. z PCI passthrough. Nie chodzi mi tutaj o samą technikę
PCI passthrough ale o dobór sterowników do urządzeń działających pod kontrolą linux. Każdy sprzęt,
który ma działać w systemie, musi mieć załadowany w pamięci RAM stosowny moduł kernela. Te moduły
zwykle są ładowane automatycznie podczas pracy systemu, np. gdy podłączamy nowy sprzęt do komputera
(można też te moduły ładować i ręcznie via `modprobe` ). Gdy nasz linux z jakiegoś powodu dobierze
niewłaściwy (z naszego punktu widzenia) moduł dla jakiegoś urządzenia, to możemy to urządzenie
odłączyć od komputera, a moduł bez problemu wyładować, po czym dokonać stosownych poprawek w
systemie. Problem zaczyna się w sytuacji, gdy mamy do czynienia ze sprzętem, którego nie da się od
komputera fizycznie odłączyć, np. wbudowana w płytę główną karta dźwiękowa, czy też wbudowana
grafika bezpośrednio w CPU. Podobnie sprawa wygląda w przypadku wkompilowania modułów na stałe w
kernel -- jak wyładować moduł, którego się nie da wyładować? By w takich sytuacjach zmienić
przypisany urządzeniu sterownik trzeba dodać parę plików w katalogach `/etc/modules-load.d/` /
`/etc/modprobe.d/` oraz zrestartować maszynę, tak by podczas fazy boot kernel dobrał sprzętowi
pożądane przez nas moduły i ich konfigurację. Niemniej jednak, istnieje prostszy sposób na zmianę
sterownika działającego w systemie sprzętu i to bez potrzeby fizycznego restartowania maszyny.
Chodzi o mechanizm ręcznego przypisywania urządzeń do konkretnych sterowników ([manual driver
binding and unbinding][3]).

<!--more-->
## Identyfikacja urządzenia PCI w linux

Na potrzeby tego artykułu weźmy sobie wbudowaną w płytę główną mojego ThinkPad'a T430 kartę
dźwiękową ( `vid:8086` oraz `pid:1e20` ). Rzućmy okiem na katalog `/sys/` w miejsce, gdzie
ulokowana jest struktura tej karty dźwiękowej, choć jej odszukanie może być ździebko problematyczne.
Generalnie to interesuje nas  katalog `/sys/devices/pci0000:00/0000:00:1b.0/` . Szukamy urządzenia,
dlatego katalog `/sys/devices/` . Szukane urządzenie jest z gatunku PCI, więc `/sys/devices/pci/` .
Problemy możemy napotkać w przypadku widocznych dalej w ścieżce numerów.

Jeśli chodzi o `pci0000:00` oraz `0000:00:1b.0`, to te numerki możemy odczytać z wyjścia `lspci` :

    $ lspci -nnk -d 8086:1e20
    00:1b.0 ...

Każde urządzenie PCI obecne w systemie [jest identyfikowane][2] przez numer szyny ( `00` ), numer
urządzenia ( `1b` ) i numer funkcji urządzenia ( `.0` ). Specyfikacja PCI pozwala maszynie posiadać
do 256 szyn, choć w większych systemach ta wartość może być niewystarczająca. Dlatego też linux
wspiera domeny PCI:

    $ lspci -vt
    -[0000:00]-+-...
               +-...
               +-1b.0  Intel Corporation 7 Series/C216 Chipset Family High Definition Audio Controller
               +-...
               +-...

Każda domena (w tym przypadku jest tylko jedna, tj, `0000` ) może pomieścić do 256 szyn
( `0000:00` ), każda szyna z kolei do 32 urządzeń, a każde urządzenie może mieć kilka funkcji (np.
karta dźwiękowa w połączeniu z CD-ROM), maksymalnie 8.

## Sterownik sprzętu

By sprawdzić jaki moduł kernela jest oddelegowany do obsługi danego sprzętu, wystarczy posłużyć się
ponownie poleceniem `lspci` . Poniżej jest przykład dla tej wbudowanej w laptop karty dźwiękowej:

    $ lspci -nnk -d 8086:1e20
    00:1b.0 Audio device [0403]: Intel Corporation 7 Series/C216 Chipset Family High Definition Audio Controller [8086:1e20] (rev 04)
            Subsystem: Lenovo 7 Series/C216 Chipset Family High Definition Audio Controller [17aa:21f3]
            Kernel driver in use: snd_hda_intel

Ta karta, jak widać, korzysta z modułu `snd_hda_intel` .

Załóżmy teraz, że coś nam z tym modułem nie pasuje i chcielibyśmy się go pozbyć. Zwykle do głowy w
takiej sytuacji jako pierwsze przychodzi nam to poniższe polecenie:

    # modprobe -r snd_hda_intel
    modprobe: FATAL: Module snd_hda_intel is builtin.

Jak widać, użytku z `modprobe` nie da się zbytnio zrobić, gdy moduły są wbudowane bezpośrednio w
kernel.

Podobnie sprawa wygląda, gdy mamy co prawda do czynienia z modułami ale jakiś sprzęt akurat z tego
modułu korzysta:

    # modprobe -r snd_hda_intel
    modprobe: FATAL: Module snd_hda_intel is in use.

Nie da się zatem wyładować sterownika, gdy ten jest używany przez urządzenie, przynajmniej nie
powinno się tego robić, zwłaszcza w sposób siłowy.

## Pliki bind i unbind

To w jaki niby sposób powinniśmy się zabrać do wyładowania takiego modułu? Trzeba poszukać w
katalogu `/sys/` plików `bind` i `unbind` dla sterownika `snd_hda_intel` . Interesuje nas ścieżka
`/sys/bus/pci/drivers/snd_hda_intel/` :

    # ls -al  /sys/bus/pci/drivers/snd_hda_intel/
    total 0
    drwxr-xr-x  2 root root    0 2020-07-27 10:28:45 ./
    drwxr-xr-x 26 root root    0 2020-07-27 10:28:45 ../
    lrwxrwxrwx  1 root root    0 2020-07-27 11:29:18 0000:00:1b.0 -> ../../../../devices/pci0000:00/0000:00:1b.0/
    lrwxrwxrwx  1 root root    0 2020-07-27 11:22:36 module -> ../../../../module/snd_hda_intel/
    --w-------  1 root root 4096 2020-07-27 11:22:36 bind
    --w-------  1 root root 4096 2020-07-27 03:20:45 new_id
    --w-------  1 root root 4096 2020-07-27 11:29:18 remove_id
    --w-------  1 root root 4096 2020-07-27 11:29:18 uevent
    --w-------  1 root root 4096 2020-07-27 11:20:50 unbind

Jak widać są dostępne szukane przez nas pliki `bind` i `unbind` . To właśnie przy pomocy tych
plików możemy przypisać jakieś urządzenie do konkretnego sterownika. Jest też link z numerkiem
urządzenia, które aktualnie korzysta z tego sterownika, tj. `0000:00:1b.0` (wspomniana wcześniej
karta dźwiękowa).

Ten mechanizm przypisania urządzenia do sterownika działa na zasadzie przesłania identyfikatora
konkretnego urządzenia do  pliku `bind` lub `unbind` , w zależności czy chcemy przypisać
urządzenie do sterownika, czy też usunąć to przypisanie. Przykładowo, by usunąć przypisanie
urządzenia `0000:00:1b.0` do sterownika `snd_hda_intel` , wpisujemy w terminalu poniższe polecenie:

    # echo -n "0000:00:1b.0" > /sys/bus/pci/drivers/snd_hda_intel/unbind

Jeśli teraz sprawdzimy ponownie wyjście `lspci` , to zauważymy, że nasza karta dźwiękowa nie ma już
informacji o tym by to urządzenie korzystało z jakiegokolwiek modułu:

    # lspci -nnk -d 8086:1e20
    00:1b.0 Audio device [0403]: Intel Corporation 7 Series/C216 Chipset Family High Definition Audio Controller [8086:1e20] (rev 04)
            Subsystem: Lenovo 7 Series/C216 Chipset Family High Definition Audio Controller [17aa:21f3]

Jeśli zajrzymy ponownie do katalogu `/sys/bus/pci/drivers/snd_hda_intel/` , to link z ID
`0000:00:1b.0` nie będzie już tam figurował:

    # ls -al /sys/bus/pci/drivers/snd_hda_intel/
    total 0
    drwxr-xr-x  2 root root    0 2020-07-27 10:28:45 ./
    drwxr-xr-x 26 root root    0 2020-07-27 10:28:45 ../
    lrwxrwxrwx  1 root root    0 2020-07-27 11:22:36 module -> ../../../../module/snd_hda_intel/
    --w-------  1 root root 4096 2020-07-27 11:22:36 bind
    --w-------  1 root root 4096 2020-07-27 03:20:45 new_id
    --w-------  1 root root 4096 2020-07-27 11:29:18 remove_id
    --w-------  1 root root 4096 2020-07-27 11:29:18 uevent
    --w-------  1 root root 4096 2020-07-27 11:34:29 unbind

W taki sposób zniknie nam też urządzenie audio z systemu, co możemy sprawdzić w pliku
`/proc/asound/cards` :

    # cat /proc/asound/cards

By ta karta dźwiękowa ponownie zaczęła poprawnie funkcjonować, trzeba dodać jej identyfikator do
pliku `bind` sterownika `snd_hda_intel` :

    # echo -n "0000:00:1b.0" > /sys/bus/pci/drivers/snd_hda_intel/bind

Po wydaniu tego powyższego polecenia, w logu systemowym pojawią się poniższe komunikaty:

    kernel: snd_hda_intel 0000:00:1b.0: bound 0000:00:02.0 (ops i915_audio_component_bind_ops)
    kernel: snd_hda_codec_realtek hdaudioC0D0: autoconfig for ALC3202: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
    kernel: snd_hda_codec_realtek hdaudioC0D0:    speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
    kernel: snd_hda_codec_realtek hdaudioC0D0:    hp_outs=2 (0x15/0x1b/0x0/0x0/0x0)
    kernel: snd_hda_codec_realtek hdaudioC0D0:    mono: mono_out=0x0
    kernel: snd_hda_codec_realtek hdaudioC0D0:    inputs:
    kernel: snd_hda_codec_realtek hdaudioC0D0:      Mic=0x18
    kernel: snd_hda_codec_realtek hdaudioC0D0:      Dock Mic=0x19
    kernel: snd_hda_codec_realtek hdaudioC0D0:      Internal Mic=0x12
    kernel: input: HDA Intel PCH Mic as /devices/pci0000:00/0000:00:1b.0/sound/card0/input38
    kernel: input: HDA Intel PCH Dock Mic as /devices/pci0000:00/0000:00:1b.0/sound/card0/input39
    kernel: input: HDA Intel PCH Headphone as /devices/pci0000:00/0000:00:1b.0/sound/card0/input40
    kernel: input: HDA Intel PCH Dock Headphone as /devices/pci0000:00/0000:00:1b.0/sound/card0/input41
    kernel: input: HDA Intel PCH HDMI/DP,pcm=3 as /devices/pci0000:00/0000:00:1b.0/sound/card0/input42
    kernel: input: HDA Intel PCH HDMI/DP,pcm=7 as /devices/pci0000:00/0000:00:1b.0/sound/card0/input43
    kernel: input: HDA Intel PCH HDMI/DP,pcm=8 as /devices/pci0000:00/0000:00:1b.0/sound/card0/input44

Jeśli teraz zajrzymy do pliku `/proc/asound/cards` , to powinniśmy ujrzeć tam kartę dźwiękową,
którą system właśnie dodał:

    # cat /proc/asound/cards
     0 [PCH            ]: HDA-Intel - HDA Intel PCH
                          HDA Intel PCH at 0xbfa00000 irq 32

## Wyładowanie modułu będącego w użyciu

Gdy już usuniemy przypisanie urządzenia do sterownika, to taki moduł kernela przestanie być przez
to urządzenie używany, a my bez problemu będziemy w stanie ten moduł wyładować:

    # echo -n "0000:00:1b.0" > /sys/bus/pci/drivers/snd_hda_intel/unbind

    # modprobe -r -v snd_hda_intel
    rmmod snd_hda_intel
    rmmod snd_intel_dspcfg

Trzeba jednak pamiętać, że jeden moduł kernela jest w stanie obsługiwać wiele urządzeń ale
konkretne urządzenie może korzystać w danej chwili tylko z jednego modułu.


[1]: {{< baseurl >}}/post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[2]: https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch12.html
[3]: https://lwn.net/Articles/143397/
