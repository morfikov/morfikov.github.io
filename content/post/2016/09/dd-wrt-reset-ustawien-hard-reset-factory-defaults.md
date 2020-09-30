---
author: Morfik
categories:
- DD-WRT
date: "2016-09-13T19:06:25Z"
date_gmt: 2016-09-13 17:06:25 +0200
published: true
status: publish
tags:
- router
- dd-wrt
title: 'DD-WRT: Reset ustawień (Hard Reset i factory defaults)'
---

Prawdopodobnie zdarzy się nam taka sytuacja, że nie będziemy w stanie zalogować się do panelu
administracyjnego naszego routera wyposażonego w firmware DD-WRT. Przyczyny mogą być różne, choć
zwykle leżą one gdzieś w konfiguracji urządzenia. Jeśli dodatkowo wystąpią problemy z dostępem do
sieci/internetu, to wybrnięcie z tej sytuacji może być dość kłopotliwe. DD-WRT posiada mechanizm
resetu, tzw. factory defaults, który jest w stanie zresetować router do ustawień fabrycznych i
przywrócić nam nad nim kontrolę. W tym wpisie zobaczymy jak przeprowadzić taki reset ustawień na
przykładzie mojego [routera TL-WDR3600][1] od TP-LINK.

<!--more-->
## Problemy z ustawieniami DD-WRT

Zmiany w ustawieniach firmware DD-WRT są przechowywane w [NVRAM][2] (Non-Volatile RAM). W ten
sposób wszystkie zmiany, które wprowadzamy w panelu administracyjnym mogą przetrwać restart routera
czy też aktualizację jego firmware.

W przypadku pojawienia się krytycznego błędu w tych ustawieniach, taki błąd również przetrwa restart
routera. Problemy z ustawieniami mogą się także pojawić, gdy chcemy wgrać nowszą wersję firmware na
router. Tutaj ustawienia szeregu aplikacji w nowszej wersji firmware mogą nie być wstecznie
kompatybilne ze zmianami, których my dokonaliśmy w konfiguracji routera.

W obu tych powyższych przypadkach możemy stracić kontrolę nad naszym routerem WiFi. Zwykle jesteśmy
w stanie ją odzyskać resetując ustawienia do domyślnych, czyli czyszcząc całą przestrzeń NVRAM.

## Reset ustawień z poziomu panelu administracyjnego

Reset ustawień z poziomu panelu administracyjnego można przeprowadzić odwiedzając w przeglądarce
adres `http://192.168.1.1/` i przechodząc na zakładkę =\> Administration =\> Factory Defaults. Tutaj
wystarczy zaznaczyć `Restore Factory Defaults` i zaaplikować ustawienia:

![](/img/2016/09/1.dd-wrt-factory-defaults-panel-admina-reset-ustawien.png#huge)

Za każdym razem, gdy będziemy aktualizować DD-WRT do nowszej wersji, przydałoby się zresetować
ustawienia firmware. Nie musimy jednak robić tego osobno w powyższej zakładce. Możemy przejść na
Administration => Firmware Upgrade i tam mamy opcję `After flashing, reset to` , która wyczyści
przestrzeń NVRAM tuż po procesie flash'owania routera:

![](/img/2016/09/2.dd-wrt-factory-defaults-panel-admina-reset-ustawien.png#huge)

## Reset ustawień przez przez przycisk na obudowie routera

Na obudowie routera WiFi mamy zwykle kilka przycisków. Model TL-WDR3600 ma między innymi przycisk
reset. Przy jego pomocy również jesteśmy w stanie zresetować ustawienia routera. Wystarczy ten
przycisk przycisnąć i przytrzymać przez około 10 sekund. Router po tym czasie powinien się
automatycznie zresetować i wyczyścić wszelkie zmiany, które wprowadziliśmy w konfiguracji.

## Reset ustawień przez SSH/telnet

W przypadku, gdy przycisk reset na obudowie routera nie działa nam z jakiegoś powodu i nie możemy
przy tym uzyskać dostępu do panelu administracyjnego, to musimy posiłkować się innym rozwiązaniem,
które pomoże nam zresetować router do ustawień fabrycznych.

Standardowo na routerze powinna nasłuchiwać usługa telnet lub/i SSH. Przy ich pomocy możemy się
zalogować do tego urządzenia z wykorzystaniem wiersza poleceń. Odpalamy zatem terminal i wydajemy w
nim komendę `telnet 192.168.2.1` . Adres oczywiście trzeba sobie dostosować do własnych potrzeb.
Zostaniemy poproszeni o login i hasło. Login to `root` , a hasło jest takie jak ustawialiśmy przez
panel administracyjny (domyślnie `admin` ).

![](/img/2016/09/2.dd-wrt-hard-reset-telnet.png#huge)

Teraz wystarczy już tylko wpisać w terminalu `erase nvram` i zresetować router poleceniem `reboot` :

![](/img/2016/09/3.dd-wrt-hard-reset-telnet.png#huge)

Po tych krokach, stracimy połączenie z routerem. Samo urządzenie zaś powinno się po chwili uruchomić
ponownie. Teraz już powinniśmy być w stanie zalogować się na router z poziomu panelu webowego
wpisując w przeglądarce adres `http://192.168.1.1/` .

## Factory defaults, a Hard Reset

Na wiki DD-WRT jest artykuł dotyczący czegoś co się nazywa [Hard Reset 30/30/30][3]. Możemy tam
wyczytać, że przed i po każdym procesie flash'owania routera nowym firmware powinniśmy resetować
router z wykorzystaniem tego mechanizmu Hard Reset. Mamy też tam informację, że Hard Reset nie jest
tym samym co factory defaults.

Okazuje się jednak, że ten cały Hard Reset to dość stary sposób na reset ustawień routerów i to
głównie tych wyposażonych w podzespoły producenta Broadcom. W nowszych routerach, zwłaszcza tych
opartych na układach Qualcomm Atheros, Hard Reset nie odnosi żadnego skutku i nie ma co sobie nim
głowy zawracać.

Generalnie rzecz biorąc, to reset ustawień przez panel administracyjny czy resetowanie routera za
pomocą przycisku na obudowie jak i wpisywanie w terminalu `nvram erase` , to dokładnie ta sama
czynność, którą można opisać obecnie jako Hard Reset 30/30/30. Nie musimy ciągle resetować ustawień
w sposób zalecany przez wiki DD-WRT. Po prostu instalujemy/aktualizujemy firmware, a jak wystąpią
jakieś problemy z konfiguracją, to przywracamy ją do ustawień domyślnych za pomocą jednego z powyżej
opisanych sposobów.

[1]: http://www.tp-link.com.pl/products/details/TL-WDR3600.html
[2]: https://pl.wikipedia.org/wiki/Nieulotna_pami%C4%99%C4%87_o_dost%C4%99pie_swobodnym
[3]: https://www.dd-wrt.com/wiki/index.php/Hard_reset_or_30/30/30
