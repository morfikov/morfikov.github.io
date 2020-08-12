---
author: Morfik
categories:
- Linux
date: "2015-10-23T19:35:57Z"
date_gmt: 2015-10-23 17:35:57 +0200
published: true
status: publish
tags:
- moduły-kernela
title: Moduł kernela i wartości jego parametrów
---

Raczej na pewno spotkaliśmy się już z modułami kernela w linux'ie. Generalnie rzecz biorąc, taki
moduł może być ładowany dynamicznie i w sporej części przypadków niezależnie, choć z zwykle jest
pociągany przy zdarzeniach udev'a. Czasem jednak, dany moduł nie działa jak należy i może to być
wynikiem, np. problemów w samym module, lub też jego niewłaściwej konfiguracji, która konfliktuje z
podzespołami naszego komputera. Ten wpis będzie dotyczył tego jak ustalić parametry modułów i ich
domyślne wartości, tak by móc je sobie zmienić w późniejszym czasie.

<!--more-->
## Informacje o modułach w modinfo

Moduły z reguły ładujemy za pomocą `modprobe` , z kolei wyładowanie ich odbywa się przez `modprobe
-r` . Aktualnie załadowane moduły można ustalić przy pomocy `lsmod` , który często jest używany z
narzędziem `grep` , bo ilość załadowanych modułów w systemie może być dość spora. Każdy moduł
kernela można podejrzeć przy pomocy `modinfo` . Zwraca ono trochę informacji na temat danego modułu,
przykładowo (wyjście nieco przycięte dla czytelności):

    # modinfo i915
    filename:       /lib/modules/4.2.0-1-amd64/kernel/drivers/gpu/drm/i915/i915.ko
    license:        GPL and additional rights
    description:    Intel Graphics
    author: Morfik         Intel Corporation
    ..
    firmware:       i915/skl_dmc_ver1.bin
    alias:          pci:v00008086d00005A84sv*sd*bc03sc*i*
    ...
    depends:        drm_kms_helper,drm,video,button,i2c-algo-bit
    intree:         Y
    vermagic:       4.2.0-1-amd64 SMP mod_unload modversions
    parm:           modeset:Use kernel modesetting [KMS] (0=DRM_I915_KMS from .config, 1=on, -1=force vga console preference [default]) (int)
    parm:           panel_ignore_lid:Override lid status (0=autodetect, 1=autodetect disabled [default], -1=force lid closed, -2=force lid open) (int)
    parm:           semaphores:Use semaphores for inter-ring sync (default: -1 (use per-chip defaults)) (int)
    ...

W skład informacji o module zwracanych przez `modinfo` wchodzi:

  - `filename` określa nazwę pliku `.ko` oraz jego położenie w drzewie katalogów.
  - `license` to licencja na jakiej dany moduł został udostępniony.
  - `description` to krótki opis modułu.
  - `author` określa autora, z tym, że może być ich wielu.
  - `firmware` określa czy moduł do swojej pracy potrzebuje firmware. Pliki firmware są
    zlokalizowane w katalogu `/lib/firmware/` .
  - `alias` odnosi się do urządzeń, które po dopasowaniu mają być obsługiwane przez ten moduł.
  - `depends` definiuje moduły, które muszą zostać załadowane wcześniej. Jeśli z jakiegoś powodu nie
    są, to zostaną załadowane automatycznie przy ładowaniu tego modułu.
  - `intree` określa czy dany moduł został włączony do włączony do kernela czy też jest rozwijany
    poza nim.
  - `vermagic` określa opcje, którą podczas ładowania modułu muszą pasować do tych, które są
    ustawione w samym module. W przypadku gdy różnią się one, ładowanie modułu nie powiedzie się, no
    chyba, że skorzysta się z parametru `--force` .
  - `parm` parametr modułu, może ich być kilka.

Przydałoby się kilka słów wyjaśnienia odnośnie ciągu, który widnieje na pozycji `alias` . W tym
przypadku jest ma on postać `pci:v00008086d00005A84sv*sd*bc03sc*i*` . Jest to urządzenie `pci` o
numerach identyfikacyjnych `v00008086` (vendorID) oraz `d00005A84` (deviceID). Dalej mamy `sv*` oraz
`sd*` , to wersje podsystemu, odpowiednio dla producenta (vendor) i urządzenia (device). Z kolei
`bc03` to klasa podstawowa określająca rodzaj urządzenia, np. interfejsy IDE, kontrolery Ethernet
czy też USB. Natomiast w tym przypadku jest to kontroler wyświetlacza (Display controller). Na końcu
mamy już tylko `sc*` odpowiadający za podklasę urządzenia no i `i*` jego interfejs. W każdym
przypadku, `*` oznacza dopasowanie do czegokolwiek. Informacje na temat numerów identyfikacyjnych
można znaleźć na wiki debiana we wpisach poświęconych
[pci](https://wiki.debian.org/HowToIdentifyADevice/PCI) oraz
[usb](https://wiki.debian.org/HowToIdentifyADevice/USB).

Parametry modułu określone przez `parm` mogą być zmieniane przez administratora systemu za pomocą
plików w katalogu `/etc/modprobe.d/` . I tak dla przykładu jeśli chcielibyśmy zmienić kilka opcji
modułu `i915` , to musimy stworzyć plik o dowolnej nazwie z końcówką `.conf` i umieścić go we
wspomnianym katalogu. Jego treść powinna wyglądać mniej więcej tak:

    options i915 lvds_downclock=1 fastboot=1

## Wartości parametrów modułu

W parametrach modułów, które uzyskaliśmy za pomocą `lsub` mamy informacje dotyczące tego jaka
wartość jest domyślna dla każdego z nich. Nie mamy jednak żadnego info na temat faktycznej
konfiguracji załadowanego modułu. Jak zatem je ustalić? Do tego celu będzie nam potrzebny pakiet
`sysfsutils` , w którym to jest dostępne narzędzie `systool` . To właśnie ono jest nam w stanie
pokazać wartości aktualnie załadowanych w systemie modułów. Poniżej przykład:

    # systool -v -m i915
    Module = "i915"
    ...
      Parameters:
    ...
        fastboot            = "Y"
    ...
        lvds_downclock      = "1"
    ...
