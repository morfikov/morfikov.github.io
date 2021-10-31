---
author: Morfik
categories:
- OpenWRT
date: "2016-04-27T18:24:12Z"
date_gmt: 2016-04-27 16:24:12 +0200
published: true
status: publish
tags:
- system-plików
- chaos-calmer
- router
GHissueID: 429
title: Zmiana rozmiaru katalogu /tmp/ pod OpenWRT
---

OpenWRT ma w swoich repozytoriach całe mnóstwo pakietów. By móc je zainstalować, potrzebne jest nam
miejsce na flash'u routera. Ten z kolei nie jest zbyt duży, często nie przekracza 16 MiB. Podczas
pracy, system operacyjny routera przeprowadza cały szereg operacji. Część z nich generuje jakieś
dane, np. tworzone są pliki konfiguracyjne, generowane statystyki czy pobierane z internetu pliki w
celu dalszego ich przetworzenia. Zwykle są to pliki, które wędrują do katalogu `/tmp/` . Gdybyśmy
chcieli zapisać wszystkie te informacje na flash'u routera, to zabrakłoby nam zwyczajnie miejsca.
Inną kwestią są problemy związane z zapisem samego flash'a, który ulega zużyciu. Dlatego też, szereg
operacji zapisu został przeniesiony do pamięci operacyjnej RAM. W ten sposób mamy do wykorzystania
nieco więcej miejsca ale standardowo nie więcej niż 50% wielkości pamięci operacyjnej. Wielkość tego
[RAMdysku](https://pl.wikipedia.org/wiki/Ramdysk) można dostosować i w tym wpisie zobaczymy jak to
zrobić.

<!--more-->
## RAMdysk czy extroot/whole_root

OpenWRT dysponuje [mechanizmem zwanym
extroot/whole_root](/post/extroot-whole_root-fullroot-pod-openwrt/). Przy jego
pomocy jesteśmy w stanie rozbudować flash routera do rzędu gigabajtów. Dlaczego zatem mielibyśmy
korzystać z RAMdysku, który standardowo nie jest większy niż 64-128 MiB? Przede wszystkim,
`extroot`/`whole_root` wymagają zewnętrznego urządzenia w postaci dysku czy pendrive. By je
podłączyć do routera, ten musi mieć co najmniej jeden port USB. Po drugie, system plików partycji
jest podatny na błędy, zwłaszcza w przypadku utraty zasilania. Zwiększa się także pobór mocy przez
router. Dodatkowo, prędkość transferu na takim urządzeniu USB podłączonym do routera nie zachwyca
zbytnio. W przypadku wykorzystania pamięci RAM, te wszystkie powyższe dolegliwości są niwelowane.
Oczywiście, trzeba także pamiętać, że dane w pamięci RAM ulatniają się, gdy router zostanie
pozbawiony zasilania. Dlatego też w dużej mierze nadaje się ona jedynie do przechowywania danych
tymczasowych. Nic jednak nie stoi na przeszkodzie, by co restart routera te dane wygenerować na nowo
i korzystać z nich po załadowaniu się systemu operacyjnego.

## System plików tmpfs i katalog /tmp/

Wiemy zatem, że OpenWRT wykorzystuje RAMdysk oraz, że w nim są przechowywane dane tymczasowe. Jakie
jednak katalogi są zamontowane w pamięci RAM? Możemy to ustalić za pomocą `df` :

    # df -h
    Filesystem                Size      Used Available Use% Mounted on
    rootfs                    4.3M    368.0K      3.9M   8% /
    /dev/root                 2.5M      2.5M         0 100% /rom
    tmpfs                    61.5M     68.0K     61.5M   0% /tmp
    /dev/mtdblock3            4.3M    368.0K      3.9M   8% /overlay
    overlayfs:/overlay        4.3M    368.0K      3.9M   8% /
    tmpfs                   512.0K         0    512.0K   0% /dev

Powyżej mamy dwa katalogi, które posiadają system plików `tmpfs` . Są to `/dev/` oraz `/tmp/` . W
tym pierwszym są tworzone pliki urządzeń i nas on w tej chwili nie interesuje. Przyjrzyjmy się
bliżej katalogowi `/tmp/` . Wyżej widzimy, że mamy dostępnych tam nieco ponad 61 MiB. Jest to
maksymalna ilość danych, które możemy zapisać w pamięci. Aktualnie wykorzystywanych jest 68 KiB,
zatem niewiele. W przypadku korzystania z dodatkowych pakietów, miejsce tutaj może się nam wyczerpać
i wtedy dostaniemy komunikat `No space left on device`. W standardowej konfiguracji OpenWRT, tego
typu sytuacja nam nie grozi. Jeśli przyjrzymy się tej poniższej fotce, zobaczymy jakie jest mniej
więcej wykorzystanie pamięci operacyjnej przez ten router:

![openwrt-statystyki-pamiec-ram-tmp](/img/2016/04/1.openwrt-statystyki-pamiec-ram-tmp.png#big)

Mamy zatem router [TP-LINK TL-WDR3600 v1](http://www.tp-link.com/en/download/TL-WDR3600.html), który
ma 123.1MB dostępnej pamięci, z czego 109.1 MiB jest wolnych. W przypadku, gdy potrzebowalibyśmy 100
MiB na katalog `/tmp/` , to standardowa konfiguracja tego RAMdysku nam nie pozwoli zapisać takiej
ilości informacji. Zatem mimo, że mamy do dyspozycji ponad 100 MiB, nie możemy wykorzystać więcej
niż 61.5 MiB. Możemy to zmienić.

## Dostosowanie rozmiaru katalogu /tmp/

W OpenWRT za montowanie zasobów odpowiada pakiet `block-mount` . Niemniej jednak, określenie w pliku
`/etc/config/fstab` poniższej konfiguracji nie przynosi oczekiwanych efektów:

    config 'mount'
          option 'target' '/tmp'
          option 'device' 'tmpfs'
          option 'fstype' 'tmpfs'
          option 'options' 'remount,rw,nosuid,nodev,noatime,size=100M'
          option 'enabled_fsck' '0'
          option 'enabled' '1'

Nie wiem czemu tak się dzieje. [Zgodnie z informacjami na wiki
OpenWRT](https://wiki.openwrt.org/doc/uci/fstab), opcje zdefiniowane w `options` powinny zostać
zaaplikowane na punkt montowania. Niemniej jednak, najzwyczajniej w świecie zostają one zignorowane.
Możemy jednak pominąć konfigurację za pomocą `block-mount` i ręcznie przemontować katalog tymczasowy
przy pomocy poniższego polecenia:

    # mount -t tmpfs -o remount,rw,nosuid,nodev,noatime,size=100M tmpfs /tmp/

Pożądany rozmiar nowego RAMdysku definiujemy w opcji `size=` . W tej chwili, system powinien już
zwiększyć limit miejsca na partycji `/tmp/` do 100 MiB, o czym możemy się przekonać z `df` :

    # df -h
    Filesystem                Size      Used Available Use% Mounted on
    ...
    tmpfs                   100.0M     68.0K     99.9M   0% /tmp
    ...

To, że katalog tymczasowy ma obecnie przydzielone 100 MiB z 123 MiB na dobrą sprawę o niczym nie
świadczy. W dalszym ciągu procesy routera są w stanie wykorzystać praktycznie całą przestrzeń RAM,
o ile ta będzie wolna.

## Przemontowanie katalogu /tmp/ w fazie boot

Ręczne przepisywanie rozmiaru katalogu tymczasowego mija się z celem. Możemy to zadanie zlecić
routerowi. W tym celu do pliku `/etc/rc.local` dopisujemy tę poniższą linijkę tuż przed `exit 0` :

    mount -t tmpfs -o remount,rw,nosuid,nodev,noatime,size=100M tmpfs /tmp/

Skrypt `rc.local` jest wywoływany na końcu procesu uruchamiania się routera. To taki swojego rodzaju
autostart, który ma postać zwykłego skryptu shell'owego i wszystkie polecenia, które jesteśmy w
stanie wpisać w terminalu, można dodawać do tego pliku.
