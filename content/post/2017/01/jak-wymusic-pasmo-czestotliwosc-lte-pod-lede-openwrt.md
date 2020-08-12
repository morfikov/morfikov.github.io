---
author: Morfik
categories:
- LEDE
- OpenWRT
date: "2017-01-13T18:43:37Z"
date_gmt: 2017-01-13 17:43:37 +0100
published: true
status: publish
tags:
- lte
- router
- modem
- huawei
- e3372
- tp-link
- chaos-calmer
title: Jak wymusić pasmo/częstotliwość LTE pod LEDE/OpenWRT
---

Zainspirowany [wątkiem na forum
JDtech](http://forum.jdtech.pl/Watek-wybor-czestotliwosci-lte-na-przykladzie-huawei-e3372) na temat
testów transferów w konkretnych pasmach/częstotliwościach LTE, postanowiłem sprawdzić jak ta sprawa
wygląda w mojej okolicy. Generalnie ja obecnie u siebie mam modem Huawei E3372s-153 w wersji
NON-HiLink podpięty do [routera TP-LINK Archer
C2600](http://www.tp-link.com.pl/products/details/Archer-C2600.html). Oczywiście na tym routerze
jest wgrany alternatywny firmware LEDE/OpenWRT, bo inaczej nie miałbym możliwości skorzystać z tego
modemu. Standardowa konfiguracja LTE w LEDE/OpenWRT daje nam jedynie możliwość wyboru między
ustawieniami `auto` , `gsm` , `umts` , `lte` , `preferumts` oraz `preferlte` . W przypadku internetu
LTE, zwykle wybieramy tutaj tryb `auto` , ewentualnie też `lte` , by wymusić konkretny tryb pracy
modemu, co może mieć kolosalne znaczenie przy darmowym internecie od RBM/Play. Niemniej jednak,
nawet w przypadku wyboru `lte` , częstotliwość na jakiej będzie pracował modem w dalszym ciągu jest
dobierana automatycznie w oparciu o parametry sygnału docierającego z dostępnych w okolicy BTS'ów. W
przypadku modemu E3372 można jednak wymusić, by połączenie LTE było realizowane na konkretnej
częstotliwości, np. 2100/1800/2600/900/800 MHz i by taki stan rzeczy osiągnąć, trzeba nieco
przerobić konfigurację tego alternatywnego oprogramowania znajdującego się w naszym routerze WiFi.

<!--more-->
## Dostosowanie konfiguracji LEDE/OpenWRT na potrzeby LTE

Przede wszystkim, by móc [operować na modemie LTE z poziomu routera
WiFi]({{< baseurl >}}/post/modem-lte-pod-openwrt/) z wgranym firmware LEDE/OpenWRT, musimy pierw
zainstalować stosowne oprogramowanie. Nie będę tutaj opisywał tego zagadnienia, bo to zostało
zrobione już w osobnym wątku. Zakładam też, że nasz modem LTE działa bez większego problemu na
routerze i nie mamy problemów ze zmuszeniem go do pracy.

Nas tutaj bardziej interesować będzie konfiguracja modemu, a konkretnie plik `/etc/gcom/ncm.json` .
To w tym pliku jest zawarta instrukcja, tj. poszczególne polecenia, które są przesyłane do modemu w
celu jego konfiguracji. Jako, że my tutaj dysponujemy modemem LTE od Huawei, to interesuje nas
sekcja `"huawei": { }` . Tam z kolei mamy podsekcję `"modes": { }` i tutaj właśnie są zlokalizowane
konfiguracje trybów pracy modemu. Standardowo mamy tutaj te poniższe wpisy:

    "modes": {
        "preferlte": "AT^SYSCFGEX=\\"030201\\",3fffffff,2,4,7fffffffffffffff,,",
        "preferumts": "AT^SYSCFGEX=\\"0201\\",3fffffff,2,4,7fffffffffffffff,,",
        "lte": "AT^SYSCFGEX=\\"03\\",3fffffff,2,4,7fffffffffffffff,,",
        "umts": "AT^SYSCFGEX=\\"02\\",3fffffff,2,4,7fffffffffffffff,,",
        "gsm": "AT^SYSCFGEX=\\"01\\",3fffffff,2,4,7fffffffffffffff,,",
        "auto": "AT^SYSCFGEX=\\"00\\",3fffffff,2,4,7fffffffffffffff,,"
    },

Mając dostępne tylko te powyższe tryby, nie da rady wymusić konkretnego pasma LTE, bo każdy z tych
trybów ma `7fffffffffffffff` , co odpowiada za obsługę wszystkich pasm. Możemy jednak zmienić tę
wartość na taką, którą odpowiada za konkretną częstotliwość. Najprościej jest po prostu dodać kilka
dodatkowych wpisów i odpowiednio przerobić `7fffffffffffffff` , poniżej przykład:

    "lte-fdd-2100": "AT^SYSCFGEX=\\"03\\",3fffffff,2,1,1,,",
    "lte-fdd-1800": "AT^SYSCFGEX=\\"03\\",3fffffff,2,4,4,,",
    "lte-fdd-2600": "AT^SYSCFGEX=\\"03\\",3fffffff,2,4,40,,",
    "lte-fdd-900": "AT^SYSCFGEX=\\"03\\",3fffffff,2,4,80,,",
    "lte-fdd-800": "AT^SYSCFGEX=\\"03\\",3fffffff,2,4,80000,,"

Pierwsza wartość liczbowa w komendzie AT, czyli `03` , wymusza LTE, zatem modem ma pracować tylko w
tym trybie. Ostatnia wartość liczbowa, tj. `1` , `4` , `40` , `80` oraz `80000` , odpowiada kolejno
pasmom B1 (2100 MHz), B3 (1800 MHz), B7 (2600 MHz), B8 (900 MHz) i B20 (800 MHz) w technologi FDD.

Każdy taki wpis, za wyjątkiem ostatniego, ma być zakończony przecinkiem ( `,` ).

## Jakie pasma/częstotliwości LTE są dostępne w mojej okolicy

Dostosowanie konfiguracji dla modemu LTE to jedna rzecz ale trzeba także zrobić lekkie rozeznanie na
temat tego jakie częstotliwości LTE są dostępne w okolicy naszego miejsca zamieszkania/przebywania.
Tutaj nie ma prostej metody, by takie informacje zdobyć. Niby można posłużyć się serwisami w stylu
[BTSEARCH](http://beta.btsearch.pl/) ale zawarte w nich dane dotyczące konkretnych stacji bazowych
czasami są błędne lub też w ogóle ich tam nie znajdziemy. Możemy jednak przełączyć modem w każdy ze
zdefiniowanych wyżej trybów i sprawdzić czy uda się uzyskać połączenie w pasmach obsługiwanych przez
modem. Edytujemy zatem plik `/etc/config/network` na routerze. Interesuje nas sekcja konfigurująca
interfejs sieciowy przypisany modemowi LTE:

    config interface 'lte'
    ...
        option mode 'lte-fdd-2600'
    ...

Teraz już wystarczy tylko dostosować opcję `mode` wpisując nazwy zdefiniowane w pliku
`/etc/gcom/ncm.json` oraz przeprowadzić szereg pomiarów prędkości łącza internetowego, np. w
serwisie speedtest. Ja dla wygody testy robiłem z poziomu aplikacji na smartfona. Uzyskałem wyniki
dla 2100 MHz, 1800 MHz, 2600 MHz i 800 MHz. Niestety na 900 MHz modem nie był w stanie zrealizować
połączenia.

[![]({{< baseurl >}}/img/2017/01/001-czestotliwosc-pasmo-lte-test-openwrt-lede-660x293.png)]({{< baseurl >}}/img/2017/01/001-czestotliwosc-pasmo-lte-test-openwrt-lede.png)

Widać zatem, że największą prędkość udało się uzyskać w paśmie 2600 MHz i w zasadzie można tę
częstotliwość wymusić. Niemniej jednak, jeśli zmieniamy dość często miejsce pobytu, to lepiej jest
pozostać przy automatycznym doborze częstotliwości, bo nie zawsze będziemy w zasięgu, np. tego pasma
2600 MHz.
