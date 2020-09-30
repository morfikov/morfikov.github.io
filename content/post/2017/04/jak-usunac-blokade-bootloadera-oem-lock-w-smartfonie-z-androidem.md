---
author: Morfik
categories:
- Android
date: "2017-04-30T19:17:15Z"
date_gmt: 2017-04-30 17:17:15 +0200
published: true
status: publish
tags:
- smartfon
- mediatek
- bootloader
title: Jak usunąć blokadę bootloader'a (OEM lock) w smartfonie z Androidem
---

Eksperymentując ostatnimi czasy ze smartfonami mającymi na pokładzie system Android nie zdarzyło mi
się jeszcze, by jakoś poważniej taki telefon uszkodzić. Oczywiście wiele razy złapałem soft brick'a
(bootloop i inne takie) ale w zasadzie bez większego problemu szło z takiej sytuacji wybrnąć. Dziś
jednak sprawa była nieco bardziej poważna, bo mój [Neffos
X1](http://www.neffos.com/en/product/details/X1) nie chciał się po prostu uruchomić, a konkretnie to
pojawiało się logo TP-LINK i Android i telefon na tym ekranie startowym się zwyczajnie zawieszał.
Pikanterii dodaje jeszcze fakt, że przed sprawdzeniem czy telefon działa poprawnie, zablokowałem
bootloader przez `fastboot oem lock` . Naturalnie bootloader można odblokować też przy użyciu
`fastboot` ale po zresetowaniu urządzenia, ta opcja, którą się przełącza w ustawieniach
deweloperskich automatycznie wraca do pozycji zablokowanej. W taki sposób, by odblokować bootloader
ponownie, trzeba wejść w te opcje jeszcze raz i tam ściągnąć pierw blokadę OEM, a dopiero później
można mówić o bawieniu się `fastboot` . A jak niby mamy wejść w te ustawienia jeśli system nie chce
wystartować, a my mamy stock'owy firmware producenta smartfona? Czy taki stan rzeczy oznacza trwałe
uszkodzenie telefonu?

<!--more-->
## Tryb recovery, reset ustawień do fabrycznych i SP Flash Tool

W pewnej części smartfonów prawdopodobnie nie damy rady nic zrobić w przypadku, gdy system takiego
urządzenia zawiesza się podczas jego startu. Można oczywiście próbować odratować telefon przez tryb
recovery, o ile ten w ogóle działa. W moim przypadku jednak zresetowanie ustawień Neffos'a X1 do
fabrycznych nie naprawiało problemu i w dalszym ciągu telefon zawieszał się na ekranie z animacją.

Jako, że Neffos X1 jest wyposażony w SoC od Mediatek, to na samym początku jak się tylko zacząłem
nim bawić zrobiłem backup całej przestrzeni flash'a w tym smartfonie. Mając zrobiony kompletny
backup, myślałem, że przywrócenie kilku partycji przyniesie jakieś efekty. Ostatecznie skończyło się
na tym, że przywróciłem cały ROM, a Neffos X1 w dalszym ciągu odmawiał współpracy. Nie wiem co mogło
się popsuć, że po przywróceniu całego fabrycznego ROM'u telefon nie startuje.

## TWRP recovery

Oczywiście mając możliwość skorzystania z SP Flash Tool, można z pominięciem tej blokady OEM wgrywać
obrazy na dowolne partycje. Można zatem poszukać na necie obrazu TWRP i wgrać go za pomocą tego
narzędzia na partycję recovery. Trzeba tylko posiadać odpowiedni plik `scatter.txt` . Niestety w
przypadku tych niezbyt popularnych smartfonów, możemy mieć problem ze znalezieniem czy to pliku
`scatter.txt` , czy też obrazu TWRP. Tak właśnie sprawa wygląda w przypadku tego Neffos'a X1.

Oczywiście ja sobie już przygotowałem odpowiedni plik `scatter.txt` oraz obraz TWRP i udało mi się
odratować smartfon przez ADB sideload wgrywając paczkę ZIP pobraną ze strony TP-LINK/Neffos.
Niemniej jednak, ten wpis dotyczy jedynie ściągania samej blokady OEM w przypadku braku dostępu do
systemu. Chcąc nie chcąc, przy okazji testowania kilku rozwiązań udało mi się tę blokadę dość prosto
ściągnąć i dlatego właśnie powstał ten wpis.

## Jak działa blokada OEM

Opcje deweloperskie, do których można uzyskać dostęp w ustawieniach Androida przez parokrotne
stuknięcie w numer kompilacji, oferują nam opcję "Zdjęcie blokady OEM". Jeśli system w naszym
telefonie posiada taką pozycję, to smartfon ma wykrojoną dedykowaną partycję, na której status tej
blokady jest przechowywany. Nazwa tej partycji może być różna ale można ją odczytać z pliku
`build.prop` , który jest zawarty w ROM'ie. Poniżej przykład z mojego Neffos'a X1:

    $ cat /system/build.prop | grep -i frp
    ro.frp.pst=/dev/block/platform/mtk-msdc.0/11230000.msdc0/by-name/frp

Ta partycja zwrócona w `ro.frp.pst` jest zmieniana za sprawą kilku czynników. Są tutaj przechowywane
też ustawienia [blokady FPR
Lock](/post/factory-reset-protection-frp-w-smartfonach-z-androidem/) (tej od konta
Google). Nas jednak interesuje ostatni bajt na tej partycji, bo zgodnie z [informacjami jakie
znalazłem na forum
XDA](https://forum.xda-developers.com/nexus-6/help/info-nexus-6-nexus-9-enable-oem-unlock-t3113539),
ten ostatni bajt jest przepisywany w zależności od położenia przełącznika opcji "Zdjęcie blokady
OEM" w ustawieniach deweloperskich.

W zasadzie są dwa ustawienia: 0 (blokada aktywna) oraz 1 (blokada zdjęta). Przestawiając przełącznik
tak by zdjąć blokadę, system przepisuje ten ostatni bajt ale tylko na czas jednego restartu
smartfona. Jeśli uruchomimy system ponownie i nie wprowadzimy żadnych zmian (nie ściągniemy blokady
z bootloader'a via `fastboot` ), to ten przełącznik wraca do pozycji wyjściowej, tj. blokada jest
automatycznie zakładana.

## Jak przepisać ostatni bajt na partycji frp

Wiemy zatem, na jakiej partycji i w którym miejscu jest zapisywane ustawienie blokady OEM. Nasuwa
się automatycznie pytanie o możliwość przepisania tego bajta, tak by z 0 zrobić 1. W jaki sposób
tego typu operację możemy przeprowadzić?

Potrzebny nam będzie obraz partycji `frp` , ewentualnie można przy pomocy `dd` stworzyć obraz
wypełniony zerami o odpowiedniej długości. Jak sobie wykroiłem tę partycję z backup'u flash'a. W
tym przypadku partycja `frp` zajmuje 1 MiB. Ten obraz trzeba załadować do edytora HEX. Jak korzystam
z [wxhexeditor](http://www.wxhexeditor.org/) . Po załadowaniu obrazu w tym programiku powinniśmy
ujrzeć poniższe okienko:

![](/img/2017/04/001-oem-lock-frp-smartfon-android-bootloader-hex.png#huge)

Jak widać wyżej ostatnie dwie cyferki (bajt) mają `00` , zatem blokada jest założona. Trzeba tutaj
ten bajt przepisać do postaci `01` .

![](/img/2017/04/002-oem-lock-frp-smartfon-android-bootloader-hex.png#huge)

Następnie zapisujemy plik i ładujemy go w SP Flash Tool i wgrywamy na partycję `frp`. Po wgraniu,
pamiętajmy, by smartfon uruchomić w trybie bootloader'a (zwykle VolDown+Power, ewentualnie przez
tryb recovery).
