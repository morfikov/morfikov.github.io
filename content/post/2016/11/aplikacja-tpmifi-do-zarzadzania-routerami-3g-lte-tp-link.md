---
author: Morfik
categories:
- Android
date: "2016-11-25T20:57:39Z"
date_gmt: 2016-11-25 19:57:39 +0100
published: true
status: publish
tags:
- router
- tp-link
- smartfon
- aplikacje
title: Aplikacja tpMiFi do zarządzania routerami 3G/LTE od TP-LINK
---

Jakiś czas temu opisywałem jeden z mobilnych routerów WiFi, który był w stanie realizować połączenie
LTE i udostępniać je w obrębie swojej sieci. Konkretnie był to [hotspot
M7310](/post/recenzja-przenosny-router-lte3g-mifi-m7310-od-tp-link/). W recenzji
tego urządzenia wspomniałem o tym, że dysponując smartfonem jesteśmy w stanie przy jego pomocy
zarządzać tym routerem. Oczywiście potrzebna jest do tego celu specjalna aplikacja tpMiFi
wypuszczona również przez TP-LINK, którą można pobrać bez większego problemu z Google Play. Jako, że
w tamtym wpisie potraktowałem temat tej aplikacji jedynie powierzchownie, to postanowiłem nieco
bardziej się jej przyjrzeć i dokładnie opisać jej właściwości.

<!--more-->
## Instalacja aplikacji tpMiFi w smartfonie

Jak już nadmieniłem we wstępie, [aplikację
tpMiFi](https://play.google.com/store/apps/details?id=com.tplink.tpmifi) można pobrać ze sklepu
Google Play. Nie waży ona za dużo, bo nieco ponad 2 MiB, no i też nie wymaga zbyt wielu
uprawnień.

![](/img/2016/11/001.tpmifi-tp-link-android-smartfon-instalacja-aplikacji.png#huge)

Ta aplikacja jest przeznaczona jedynie do współpracy z urządzeniami TP-LINK i wspiera praktycznie
dowolne urządzenie MiFi tego producenta. Nie powinniśmy zatem napotkać problemów przy jej
użytkowaniu. Niemniej jednak, z aplikacji można korzystać jedynie w przypadku podłączenia do
urządzeń MiFi. W przeciwnym wypadku wszystkie opcje dostępne w tpMiFi są nieaktywne i każda próba
wejścia w interakcje z nimi powoduje pojawienie się poniższego monitu:

![](/img/2016/11/002.tpmifi-tp-link-android-smartfon-brak-polaczenia.png#medium)

## Podłączanie się do routera

By zarządzać routerem z poziomu smartfona musimy podłączyć się do jego sieci bezprzewodowej. Każde
takie urządzenie ma na obudowie informacje, które mogą nam zapewnić dostęp do jego sieci WiFi.
Odczytujemy ESSID oraz hasło i konfigurujemy taką sieć w smartfonie pod Ustawienia => WLAN:

![](/img/2016/11/003.tpmifi-tp-link-android-smartfon-podlaczanie-hotspot.png#big)

Routery WiFi zwykle mają dostępną opcję WPS. Smartfony również takim wynalazkiem dysponują, przez co
możemy przycisnąć wirtualne przyciski na obu tych urządzeniach, tak by konfiguracja połączenia
bezprzewodowego dokonała się bez naszej ingerencji. W tym celu z menu WLAN wybieramy pozycję
"Zaawanasowane" i "Przycisk WPS":

![](/img/2016/11/004.tpmifi-tp-link-android-smartfon-parowanie-wps.png#huge)

## Sterownie routerem przy pomocy tpMiFi

Będąc podłączonym do sieci WiFi routera możemy odpalić aplikację tpMiFi. Teraz na smartfonie
powinniśmy być w stanie zobaczyć statystyki transferu, stan baterii, ilość zalogowanych
użytkowników oraz ilość nieprzeczytanych wiadomości SMS.

W górnej części ekranu znajdziemy menu (to te trzy poziome kreseczki), nazwę modelu urządzenia, oraz
pozycję Login umożliwiającą zalogowanie się na router.

![](/img/2016/11/005.tpmifi-tp-link-android-smartfon-menu-aplikacji.png#huge)

Praktycznie wszystkie opcje widoczne na ekranie powyżej wymagają zalogowania się. Hasło logowania
jest tym, które wykorzystywane jest przy dostępie do urządzenia z poziomu panelu webowego.
Standardowo jest to `admin` . Po zalogowaniu się możemy operować na routerze.

### Odczytywanie wiadomości SMS

Jak mogliśmy dostrzec na jednej z powyższych fotek, na skrzynce karty SIM, która została wsadzona do
tego routera, zalega jedna wiadomość. By ją odczytać wystarczy tapnąć w ekran na pozycji SMS.
Otworzy się nam lista wiadomości, które możemy sobie przeglądać, a nawet odpowiadać na nie
bezpośrednio z aplikacji tpMiFi. Wiadomości można też naturalnie usuwać.

![](/img/2016/11/006.tpmifi-tp-link-android-smartfon-sms.png#huge)

Jedyny problem jaki tutaj dostrzegam, to brak powiadamiania dźwiękowego o nadchodzących SMS. Widać
co prawda żółtą kropkę ale skąd mam wiedzieć, że jakaś wiadomość dotarła na router, skoro nie mam
żadnej notyfikacji, gdy aplikacja jest zamknięta i nie widać jej interfejsu?

### Stan baterii

Przez aplikację tpMiFi jesteśmy też w stanie sprawdzić status baterii. Są również opcje dostosowania
trybu oszczędzania energii. Możemy dostosować sobie moc nadajnika naszego routera, jak i czas
bezczynności WiFi, po którym radio ma zostać wyłączone.

![](/img/2016/11/007.tpmifi-tp-link-android-smartfon-bateria.png#medium)

### Zalogowani użytkownicy sieci WiFi

Jest też co prawda możliwość podejrzenia aktualnie zalogowanych użytkowników w sieci WiFi. Widoczna
jest ich nazwa jak i adres MAC. Nie mamy jednak opcji zablokowania konkretnego użytkownika, czy
ograniczenia mu dostępnego pasma sieciowego:

![](/img/2016/11/008.tpmifi-tp-link-android-smartfon-klienci-wifi.png#medium)

### Udostępnianie zawartości karty SD

Jeśli nasz router dysponuje slotem na kartę SD, to pewnie chcielibyśmy udostępnić jej zawartość w
sieci. Aplikacja tpMiFi daje nam możliwość przełączenia trybu udostępniania karty SD z USB na WiFi:

![](/img/2016/11/009.tpmifi-tp-link-android-smartfon-karta-sd.png#big)

W trybie WiFi jesteśmy w stanie wgrać nawet pliki przez aplikację tpMiFi na tą kartę SD:

![](/img/2016/11/010.tpmifi-tp-link-android-smartfon-karta-sd-pliki.png#big)

### Statystyki i limity transferu danych

Dzięki aplikacji tpMiFi jesteśmy w stanie śledzić statystyki wykorzystania łącza w czasie
rzeczywistym. Nie tylko aktualna prędkość pobierania i wysyłana jest uwzględniona ale również i
ilość przetransferowanych danych. Poza samymi statystykami, tpMiFi umożliwia nam nałożenie
ograniczeń transferu:

![](/img/2016/11/011.tpmifi-tp-link-android-smartfon-transfer-statystyki.png#big)

Zabrakło jednak opcji, które umożliwiłyby dostosowanie godzin, w których transfer nie jest
naliczany. Taka pozycja widnieje w standardowym panelu admina.

### Konfiguracja połączenia WiFi i 3G/LTE

Przez tpMiFi możemy także dostosować sobie nazwę i hasło sieci WiFi. Nie ma jednak możliwości wyboru
zabezpieczeń. Jest też opcja ukrycia sieci jak i wyboru pasma. Jeśli zaś chodzi o połączenie 3G i
LTE, to mamy oczywiście możliwość wymuszenia 3G lub LTE oraz stan preferowanego LTE. Zabrakło jednak
opcji wyboru/wymuszenia częstotliwości/kanału na jakim pracuje modem, a to przecie bardzo użyteczna
funkcja.

![](/img/2016/11/012.tpmifi-tp-link-android-smartfon-siec-wifi-lte-3g.png#big)

Jak widać wyżej, tpMiFi oferuje także możliwość konfiguracji kodu PIN, na wypadek, gdyby karta SIM
była zabezpieczona. Możemy również sobie skonfigurować niestandardowy APN przy połączeniu 3G/LTE.

![](/img/2016/11/013.tpmifi-tp-link-android-smartfon-pin-apn.png#big)

### Pozostałe ustawienia

Przez tpMiFi jesteśmy także w stanie zmienić hasło wykorzystywane do logowania zarówno w panelu
administracyjnym jak i samej aplikacji na smartfonie.

![](/img/2016/11/014.tpmifi-tp-link-android-smartfon-haslo.png#big)

Ta aplikacja potrafi także zwrócić informacje o urządzeniu, do którego jesteśmy aktualnie
zalogowani:

![](/img/2016/11/015.tpmifi-tp-link-android-smartfon-informacje-router.png#medium)

No i na koniec standardowo są też opcje umożliwiające restart oaz wyłączenie hotspotu/routera, jak i
przywrócenie jego ustawień do fabrycznych:

![](/img/2016/11/016.tpmifi-tp-link-android-smartfon-reboot-poweroff.png#medium)

## Problemy z tpMiFi i mobilna wersja panelu administracyjnego

Warto też wiedzieć, że te nowsze urządzenia mają wersję mobilną panelu administracyjnego, który
możemy załadować sobie w dowolnej przeglądarce na smartfonie przechodząc na adres IP routera.

![](/img/2016/11/017.tpmifi-tp-link-android-smartfon-panel-admina-mobilny.png#huge)

To o tyle ważna rzecz, że przy pewnych niestandardowych konfiguracjach telefonu, np. [szyfrowany DNS
za sprawą
dnscrypt-proxy](/post/jak-zaszyfrowac-zapytania-dns-na-smartfonie-dnscrypt-proxy/)
czy VPN, aplikacja tpMiFi może się bardzo dziwnie zachowywać [uniemożliwiając zalogowanie na
konkretny
sprzęt](http://tplink-forum.pl/index.php?/topic/5485-popsu%C5%82em-aplikacj%C4%99-tpmifi/#comment-46863).
Bierze się to z faktu rozwiązywania przez aplikację domeny `tplinkmifi.net` , która ma wskazywać na
adres prywatny (np. 192.168.0.1), a takiego rozwiązania nie otrzymamy z OpenDNS czy innego serwera
DNS. Zatem jeśli w jakiś sposób dokonujemy przekierowania ruchu DNS, to raczej postawmy na mobilną
wersję panelu administracyjnego zamiast na aplikację tpMiFi, bo zaoszczędzi nam to wielu problemów.
