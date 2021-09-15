---
author: Morfik
categories:
- Android
date:    2019-02-02 06:33:44 +0100
lastmod: 2019-02-02 06:33:44 +0100
published: true
status: publish
tags:
- smartfon
- twrp
- recovery
title: Większy stopień kompresii pliku recovery.img (TWRP)
---

Ostatnio próbowałem zaktualizować obraz TWRP recovery dla jednego z moich telefonów. Ja generalnie
buduje te obrazy ze źródeł OMNI ROM, a tam jest dostępnych szereg gałęzi, np. 6.0, 7.1, 8.1 , etc,
które naturalnie pasują do odpowiadających im wersji Androida. Do tej pory budowałem w oparciu o
gałąź 7.1 ale po wydaniu polecenia `repo sync` , szereg aktualizacji w stosunku do repozytorium
`bootable/recovery` zostało pobranych, w tym też i jedna trefna, która uwalała proces kompilacji.
Ostatecznie [udało się problem namierzyć i zlikwidować][1] ale w międzyczasie próbowałem zbudować
obraz TWRP recovery z gałęzi 8.1. Wygląda na to, że im nowszy Android, tym obrazy recovery rosną w
objętość i 16M, które u mnie jest limitem, zostało przekroczone o jakieś 500K i to przy najbardziej
okrojonej funkcjonalności trybu recovery. Czy istnieje jakieś rozwiązanie, które by umożliwiło
zmniejszenie rozmiaru obrazu `ramdisk-recovery.img` , co przełożyłoby się również na wagę pliku
`recovery.img` ? Tak, trzeba tylko zmienić rodzaj kompresji z domyślnego `gzip` na `lzma`.

<!--more-->
## Wsparcie kernela dla lzma

Przede wszystkim, by móc myśleć o zmianie kompresji obrazu TWRP recovery z `gzip` na `lzma`, to
kernel, który obsługuje nasz smartfon, musi mieć wkompilowane wsparcie dla tego rodzaju kompresji.
Można ten fakt ustalić wpisując to poniższe polecenie:

    # </proc/kallsyms cut -d " " -f 3 | grep -e lzma

Jeśli wynik będzie pusty, to kernel nie posiada wsparcia dla `lzma`, a my możemy zapomnieć o
zmniejszeniu rozmiaru pliku `recovery.img` , no chyba, że dysponujemy źródłami kernela i możemy
ustawić sobie `CONFIG_RD_LZMA=y` .

Jeśli jednak to powyższe polecenie zwróci coś na wzór tego co widnieje poniżej:

    lzma_len
    lzma_main
    xz_dec_lzma2_run
    xz_dec_lzma2_create
    xz_dec_lzma2_reset
    xz_dec_lzma2_end
    unlzma

to jest dobrze.

## Plik custombootimg.mk

Wsparcie w kernelu dla `lzma` to jeszcze nie wszystko. Musimy odpowiednio skonfigurować proces
kompilacji, by ten obraz TWRP recovery został skompresowany przy wykorzystaniu `lzma` . Jedną z
potrzebnych nam rzeczy będzie własny skrypt kompresujący ramdysk. Wygląda on mniej więcej tak:

    LZMA_BIN := $(shell which lzma)


    $(INSTALLED_RECOVERYIMAGE_TARGET): $(MKBOOTIMG) \
            $(recovery_ramdisk) \
            $(recovery_kernel)
        @echo ----- Compressing recovery ramdisk with lzma ------
        rm -f $(recovery_uncompressed_ramdisk).lzma
        $(LZMA_BIN) $(recovery_uncompressed_ramdisk)
        $(hide) cp $(recovery_uncompressed_ramdisk).lzma $(recovery_ramdisk)
        @echo ----- Making recovery image ------
        $(MKBOOTIMG) $(INTERNAL_RECOVERYIMAGE_ARGS) $(BOARD_MKBOOTIMG_ARGS) --output $@
        @echo ----- Made recovery image -------- $@
        $(hide) $(call assert-max-image-size,$@,$(BOARD_RECOVERYIMAGE_PARTITION_SIZE),raw)


    $(INSTALLED_BOOTIMAGE_TARGET): $(MKBOOTIMG) $(INTERNAL_BOOTIMAGE_FILES)
        $(call pretty,"Target boot image: $@")
        $(hide) $(MKBOOTIMG) $(INTERNAL_BOOTIMAGE_ARGS) $(BOARD_MKBOOTIMG_ARGS) --output $@
        $(hide) $(call assert-max-image-size,$@,$(BOARD_BOOTIMAGE_PARTITION_SIZE),raw)

Plik zapisujemy pod nazwą `custombootimg.mk` w `/Android/omni-twrp-8.1/device/tp-link/tp903a/` .

## Plik BoardConfig.mk

Kolejnym krokiem jest dodanie stosownej konfiguracji do pliku `BoardConfig.mk` . Musimy w zasadzie
określić tylko zmienną `BOARD_CUSTOM_BOOTIMG_MK` tak, by wskazywała na ścieżkę do pliku utworzonego
wyżej, przykładowo:

    BOARD_CUSTOM_BOOTIMG_MK := $(LOCAL_PATH)/custombootimg.mk

I to tyle. Obraz TWRP recovery powinien zostać skompresowany przy użyciu `lzma` , co z pewnością
zobaczymy pod koniec procesu budowy obrazu.

Warto tutaj jeszcze zaznaczyć, że w przypadku załadowania takiego obrazu `recovery.img` na smartfon,
gdy kernel nie obsługuje kompresji `lzma` , to telefon nam się w trybie recovery nie uruchomi.
Oczywiście nie uwalimy sobie urządzenia w ten sposób, ale przy próbie odpalenia trybu recovery,
smartfon będzie się resetował i jeśli dodatkowo nie działa na system urządzenia, to możemy się
wpakować w problemy.


[1]: https://gerrit.omnirom.org/#/c/android_bootable_recovery/+/33485/
