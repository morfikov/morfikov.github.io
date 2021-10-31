---
author: Morfik
categories:
- Android
date:    2016-11-24 21:02:26 +0100
lastmod: 2016-11-24 21:02:26 +0100
published: true
status: publish
tags:
- tp-link
- smartfon
- aplikacje
GHissueID: 505
title: Aplikacja Tether do zarządzania urządzeniami TP-LINK przez smartfon
---

TP-LINK ma w swojej ofercie szereg urządzeń, którymi można zarządzać z grubsza na dwa sposoby.
Pierwszym jest raczej znany nam wszystkim panel administracyjny dostępny z poziomu przeglądarki
internetowej zainstalowanej na dowolnym komputerze czy laptopie. Drugim ze sposobów jest
wykorzystanie smartfona i dedykowanej aplikacji Tether na Androida/iOS. Webowy panel administracyjny
zwykł udostępniać nam całą masę opcji, a jak jest w przypadku aplikacji Tether?

<!--more-->
## Lista kompatybilnych urządzeń

Aplikacja Tether od TP-LINK została zaprojektowana w celu zarządzania routerami WiFi i innymi
urządzeniami bezprzewodowymi tego producenta przy pomocy smartfonów z Androidem 4.0+ i iOS 7.0+. By
móc zarządzać jakimś sprzętem przy pomocy telefonu, musi ono być na liście wspieranych urządzeń.
Poniżej jest taka lista:

    ★ Kompatybilne Routery
    Archer C3200 V1 / Archer C2600 V1
    Archer C1900 V1
    Archer C9 V1 V2 / Archer C8 V1 V2
    Archer C7 V2 V3 / Archer C5 V2
    Archer C50 V1 / Archer C2 V1
    Archer C20 V1 / Archer C20i V1
    TL-WDR4300 V1 / TL-WDR3600 V1
    TL-WDR3500 V1 /TL-WR1043ND V3
    TL-WR941ND V5, V6 / TL-WR940N V2, V3
    TL-WR841ND V9, V10 V11
    TL-WR841N V9, V10 V11 / TL-WR741ND V5
    TL-WR740N V5 V6, etc.

    ★ Kompatybilne Routery xDSL
    Archer VR900v V1 / Archer VR200v V1
    Archer VR900 V1
    Archer D9 V1 / Archer D7 V1
    Archer D5 V1, V2
    Archer D2 V1 / Archer D20 V1, etc.

    ★ Kompatybilne ekstendery zasięgu
    RE590T V1 / RE580D V1
    RE450 V1 / RE355 V1
    TL-WA855RE V1 / TL-WA854RE V2
    TL-WA850RE V2 / TL-WA830RE V3

Posiadanie jednego z tych powyższych urządzeń nie zagwarantuje nam możliwości korzystania z
aplikacji Tether. Musimy bowiem posiadać w miarę aktualny firmware, który ma dodaną stosowną
poprawkę. Ja dysponuję [routerem Archer C9 v2][1] i najnowszy firmware do tego jak i innych
routerów zawsze można pobrać z [oficjalnej strony TP-LINK][2].

## Instalacja Tether na smartfonie z Androidem

Mając najnowszy firmware na routerze, który chcielibyśmy podpiąć pod aplikację TP-LINK Tether, pora
przejść do instalacji samego oprogramowania na smartfonie. Ja akurat mam na wyposażeniu [smartfon
Neffos C5][3] z Androidem 5.1 (Lollipop) i na nim ta aplikacja powinna działać bez zarzutu.
[Tether można pobrać z Google Play][4].

![tether-tp-link-smartfon-instalacja](/img/2016/11/001.tether-tp-link-smartfon-instalacja.png#huge)

Co ciekawe, w stosunku do innych aplikacji TP-LINK, np. [tpMiFi][5], [tpCamera][6] czy [KASA][7],
Tether cieszy się dość sporą popularnością. Aplikacja jest darmowa i nie zawiera reklam. Sprawdźmy
zatem co ta aplikacja jest nam w stanie zaoferować.

## Konfiguracja routera przez aplikację Tether

Po podłączeniu smartfona do domowej sieci WiFi, aplikacja Tether powinna rozpoznać nasz
bezprzewodowy router, o ile mamy na nim wgrany odpowiedni firmware. Poniżej przykład wykrycia mojego
routera Archer C9:

![tether-tp-link-smartfon-wykrywanie-routera](/img/2016/11/002.tether-tp-link-smartfon-wykrywanie-routera.png#big)

Jeśli nasz router został rozpoznany, oznacza to, że wszystko jest w należytym porządku i możemy
spróbować się na to urządzenie zalogować klikając na odpowiednią pozycję na liście. Jako, że w tym
przypadku jest tylko jedna pozycja, to nie mamy za dużego wyboru. Dane logowania to
`admin`/`admin` , czyli standardowy użytkownik i hasło, które są wykorzystywane w panelach
administracyjnych TP-LINK'a:

![tether-tp-link-smartfon-logowanie](/img/2016/11/003.tether-tp-link-smartfon-logowanie.png#medium)

Po zalogowaniu się na router przywita nas takie oto okienko:

![tether-tp-link-smartfon-aplikacja](/img/2016/11/004.tether-tp-link-smartfon-aplikacja.png#medium)

Mamy tutaj informacje na temat liczby aktualnie podłączonych urządzeń do routera oraz o stanie
połączenia z internetem. Klikając zaś w nazwę urządzenia, zostaną nam pokazane bardziej
szczegółowe dane dotyczące modelu, typu połączenia, wersji firmware oraz wersji sprzętowej samego
routera. Nazwę wyświetlaną zawsze można sobie dostosować klikając na nią:

![tether-tp-link-smartfon-nazwa](/img/2016/11/005.tether-tp-link-smartfon-nazwa.png#big)

Niżej w głównym oknie aplikacji Tether mamy listę podłączonych urządzeń w formie ikonek przypisanych
w oparciu o wykrytego/ustawionego klienta. Jak widzimy, aktualnie są podłączone dwa klienty, jeden
przedstawia się jako Android, drugi jako laptop. Klikając na każdej z tych pozycji, możemy uzyskać
nieco więcej informacji na temat połączenia danego klienta, min. adres IP oraz MAC:

![tether-tp-link-smartfon-klienci](/img/2016/11/006.tether-tp-link-smartfon-klienci.png#big)

Jak widać wyżej, jesteśmy też w stanie zablokować tego konkretnego klienta przyciskając przycisk
"Blokuj".

Na samym dole okna głównego aplikacji Tether mamy jeszcze pozycję z opcjami (to ten żółty przycisk).
Tutaj możemy skonfigurować szereg aspektów pracy routera:

![tether-tp-link-smartfon-ustawienia](/img/2016/11/007.tether-tp-link-smartfon-ustawienia.png#medium)

Możemy włączyć lub wyłączyć bezprzewodową sieć oraz ustawić jej zarówno nazwę jak i hasło logowania.
Jest też opcja konfiguracji zabezpieczeń sieci WiFi, z tym, że możemy albo wyłączyć te
zabezpieczenia kompletnie, albo włączyć WPA-PSK/WPA2-PSK:

![tether-tp-link-smartfon-wifi](/img/2016/11/008.tether-tp-link-smartfon-wifi.png#big)

Dalej jesteśmy w stanie skonfigurować połączenie z internetem oraz jest też widoczna dokładna
informacja na temat uzyskanej adresacji.

![tether-tp-link-smartfon-internet](/img/2016/11/009.tether-tp-link-smartfon-internet.png#big)

Mamy także możliwość konfiguracji sieci dla gości:

![tether-tp-link-smartfon-siec-goscinna](/img/2016/11/010.tether-tp-link-smartfon-siec-goscinna.png#big)

W przypadku, gdy blokowaliśmy jakichś klientów, to ci są dodawani na specjalną listę i do momentu
usunięcia określonych pozycji dana maszyna nie będzie w stanie się podłączyć bezprzewodowo do
naszego routera. Wszystkie pozycje z listy możemy usunąć klikając na nich:

![tether-tp-link-smartfon-lista-zablokowanych-klientow](/img/2016/11/011.tether-tp-link-smartfon-lista-zablokowanych-klientow.png#big)

W aplikacji Tether mamy też opcję dotyczącą kontroli rodzicielskiej. Po jej aktywacji będziemy w
stanie dodać do listy urządzenia, które mają podlegać kontroli (maksymalnie 32).

![tether-tp-link-smartfon-kontrola-rodzicielska](/img/2016/11/012.tether-tp-link-smartfon-kontrola-rodzicielska.png#big)

Po dodaniu stosownych klientów pojawi nam się harmonogram, w którym możemy określić czas
obowiązywania restrykcji.

![tether-tp-link-smartfon-harmonogram](/img/2016/11/013.tether-tp-link-smartfon-harmonogram.png#huge)

Ostatnimi funkcjami jakie oferuje nam aplikacja Tether są restart urządzenia, resetowanie jego
ustawień do fabrycznych oraz zmiana hasła do panelu admina/aplikacji:

![tether-tp-link-smartfon-opcje-system](/img/2016/11/014.tether-tp-link-smartfon-opcje-system.png#huge)

## Mobilna wersja panelu administracyjnego

Jeśli miałbym być szczery, to ta aplikacja Tether niezbyt mi przypadła do gustu. Jest ona póki co
dość uboga w opcje, no i nie umknął mi też fakt niezbyt precyzyjnego dopasowania interfejsu
aplikacji na moim smartfonie, co widać na kilku fotkach wyżej. Gdybym miał wybierać między aplikacją
Tether i mobilną wersją standardowego panelu administracyjnego, to jednak wolę zarządzać routerem z
poziomu Firefox'a:

![tether-tp-link-smartfon-panel-wersja-mobilna](/img/2016/11/015.tether-tp-link-smartfon-panel-wersja-mobilna.png#big)


[1]: http://www.tp-link.com.pl/products/details/cat-9_Archer-C9.html
[2]: http://www.tp-link.com/en/support/download
[3]: http://www.neffos.pl/product/details/C5
[4]: https://play.google.com/store/apps/details?id=com.tplink.tether
[5]: https://play.google.com/store/apps/details?id=com.tplink.tpmifi
[6]: https://play.google.com/store/apps/details?id=com.tplink.skylight
[7]: https://play.google.com/store/apps/details?id=com.tplink.kasa_android
