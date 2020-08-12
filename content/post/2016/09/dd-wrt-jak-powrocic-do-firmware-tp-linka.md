---
author: Morfik
categories:
- DD-WRT
date: "2016-09-16T19:10:41Z"
date_gmt: 2016-09-16 17:10:41 +0200
published: true
status: publish
tags:
- router
- tp-link
- dd-wrt
title: 'DD-WRT: Jak powrócić do firmware TP-LINK''a'
---

[DD-WRT](https://www.dd-wrt.com/site/) nie każdemu może przypaść do gustu. Może się zdarzyć też tak,
że pewne rzeczy na routerze za sprawą tego firmware przestaną nam zwyczajnie działać, bo nie każde
urządzenie jest w pełni wspierane przez to oprogramowanie. W takim przypadku jedyną opcją będzie
powrót do oryginalnego firmware, który oferuje producent routera. Ja dysponuję [routerem model
TL-WDR3600](http://www.tp-link.com.pl/products/details/TL-WDR3600.html) od TP-LINK. On akurat jest w
pełni wspierany przez DD-WRT. Niemniej jednak, na jego przykładzie zostanie pokazane jak powrócić do
oryginalnego oprogramowania posługując się panelem webowym dostępnym DD-WRT.

<!--more-->
## Skąd pobrać oryginalny firmware TP-LINK'a

Musimy pokusić się o pobranie odpowiednie obrazu z oryginalnym firmware producenta. Na forum DD-WRT
jest wątek poświęcony routerom TP-LINK, gdzie znajduje się również odpowiedni [plik dla routera
TL-WDR3600](https://www.dd-wrt.com/phpBB2/viewtopic.php?t=85237&postdays=0&postorder=asc&start=30).
Pobieramy tę paczkę na dysk i wypakowujemy. Po wypakowaniu powinniśmy mieć na dysku plik
`wdr3600v1_webrevert.bin` i to ten plik musimy załadować na router.

## Wgrywanie firmware TP-LINK'a przez panel admina

Po pozyskaniu odpowiedniego obrazu, logujemy się do panelu administracyjnego przez przeglądarkę
wpisując adres `http://192.168.1.1/` i przechodzimy na zakładkę Administration =\> Firmware Upgrade.
Tam wskazujemy lokalizację pobranego pliku:

![]({{< baseurl >}}/img/2016/09/2.dd-wrt-powrot-do-oryginalnego-firmware-tp-link.png)

I teraz już wystarczy tylko wcisnąć przycisk Upgrade.

![]({{< baseurl >}}/img/2016/09/3.dd-wrt-powrot-do-oryginalnego-firmware-tp-link.png)

Po chwili proces flash'owania oryginalnym firmware zostanie ukończony:

![]({{< baseurl >}}/img/2016/09/4.dd-wrt-powrot-do-oryginalnego-firmware-tp-link.png)

Oczywiście nie mamy po co już klikać przycisku Continue, bo na routerze siedzi oprogramowanie
TP-LINK'a. Zmianie uległ także adres routera (192.168.0.1) i cała adresacja sieci (192.168.0.0/24).
By się połączyć z routerem prawdopodobnie będzie potrzebny restart połączenia sieciowego na
komputerze. Dopiero po uzyskaniu nowej adresacji możemy wbić do panelu admina TP-LINK'a wpisując w
przeglądarce nowy adres routera:

![]({{< baseurl >}}/img/2016/09/5.tp-link-panel-admina-stara-wersja-firmware.png)

Wyżej widzimy, że wersja firmware to `3.13.23 Build 120820 Rel.73549n` , która jest dość leciwa.
Dlatego też przydałoby się zaktualizować to oprogramowanie. W tym celu odwiedzamy stronę TP-LINK'a i
pobieramy [najnowszą wersję obrazu dla routera
TL-WDR3600](http://www.tp-link.com/en/download/TL-WDR3600.html#Firmware). Pamiętajmy, by pobrać plik
dla określonej wersji routera. TL-WDR3600 ma tylko wersję v1.

Pobraną paczkę wypakowujemy. W niej powinien znajdować się plik
`wdr3600v1_en_3_14_3_up_boot(150518).bin` , który trzeba wgrać na router. Przechodzimy zatem do
panelu administracyjnego i w menu wybieramy System Tools =\> Firmware Upgrade i wskazujemy
wypakowany plik:

![]({{< baseurl >}}/img/2016/09/7.tp-link-aktualizacja-firmware.png)

Klikamy przycisk Upgrade i po chwili proces aktualizacji powinien dobiec końca, a router powinien
się samoczynnie zrestartować. Logujemy się jeszcze na router, by sprawdzić wersję firmware:

![]({{< baseurl >}}/img/2016/09/8.tp-link-nowa-wersja-firmware.png)

Mamy zatem `3.14.3 Build 150518 Rel.72050n` , czyli aktualną wersję dla tego routera.
