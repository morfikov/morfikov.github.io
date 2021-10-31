---
author: Morfik
categories:
- Android
date:    2016-10-27 18:52:19 +0200
lastmod: 2016-10-27 18:52:19 +0200
published: true
status: publish
tags:
- dns
- resolver
- smartfon
- aplikacje
- dnscrypt
- root
- twrp
- szyfrowanie
GHissueID: 451
title: Jak zaszyfrować zapytania DNS na smartfonie (dnscrypt-proxy)
---

Smartfony to takie małe komputery, z których praktycznie każdy z nas korzysta na co dzień. Nie
różnią się one zbytnio od tych domowych PC czy laptopów, no może za wyjątkiem rozmiarów.
Wszystkie elementy tyczące się spraw sieciowych, np. korzystanie z internetu za pomocą przeglądarki,
są dokładnie taka same co w przypadku zwykłych komputerów. Na smartfonach domeny również trzeba
jakoś rozwiązać. Standardowo w Androidzie są wykorzystywane serwery od Google (8.8.8.8 i 8.8.4.4).
Jeśli nasza sieć WiFi oferuje inne DNS'y, to wtedy one mają pierwszeństwo. Niemniej jednak, nie
zawsze będziemy w stanie kontrolować środowisko sieciowe, do którego zostaniemy podłączeni. W takiej
sytuacji będziemy zdani na łaskę admina obcej sieci w kwestii poufności odwiedzanych przez nas stron
www czy jakichkolwiek innych domen w internecie. Z doświadczenia wiem, by nie składać swojej
prywatności w czyjeś ręce i dlatego też postanowiłem poszukać sposobu na zaszyfrowanie zapytań DNS
bezpośrednio na smartfonie. Długo nie musiałem szukać, bo okazuje się, że [dnscrypt-proxy jest
dostępny również na Androida][1].

<!--more-->
## Root smartfona

Jak to w życiu bywa, do tych bardziej zaawansowanych czynności przeprowadzanych na naszych
smartfonach potrzebny jest dostęp root. Podobnie jest w przypadku przekierowania zapytań DNS. Jeśli
nasz Android nie jest ukorzeniony i nie znamy sposobu by ten fakt zmienić, to niestety pozostaje nam
korzystanie z DNS'ów Google lub tych sieci, do których zostaniemy podłączeni.

Ja posiadam smartfon od TP-LINK, a konkretnie jest to [model Neffos C5][2]. Jakiś czas temu [udało
mi się na nim przeprowadzić proces root'owania][3], przez co mam pełny dostęp do systemu plików
tego telefonu, co z kolei otworzyło mi drogę do eksperymentowania z wymuszeniem szyfrowania zapytań
DNS.

## Jak pozyskać dnscrypt-proxy na Androida

Mając ukorzeniony system na smartfonie, możemy przejść do pozyskania `dnscrypt-proxy` i jego
instalacji w Androidzie. Na tym Neffos C5 siedzi Android w wersji 5.1 (Lollipop). W starszych
wersjach Androida mogą pojawić się błędy, przez co instalacja i prawidłowe funkcjonowanie tego
narzędzia nie zawsze będzie możliwe. Trzeba także wziąć pod uwagę fakt, że tej aplikacji nie ma w
sklepie Google Play ani w [repozytorium F-Droid][4] i trzeba posiłkować się zewnętrznym źródłem.
Odpowiedni plik [można pobrać stąd][5], konkretnie chodzi o
`dnscrypt-proxy-android-armv7-a-1.7.0.zip` . Musimy go zainstalować z poziomu trybu recovery mając
przy tym wgrany [TWRP][6] lub [CWM][7]. Ja będę korzystał z TWRP.

W pobranej paczce `.zip` znajduje się plik `system/etc/init.d/99dnscrypt` . W nim zaś jest zmienna
`RESOLVER_NAME` zawierająca nazwę serwera DNS, do którego będą wysyłane zapytania o domeny w formie
szyfrowanej. Jeśli nie odpowiada nam ta nazwa, to zawsze możemy wybrać inną. Pełna lista serwerów
DNS, z których możemy skorzystać znajduje się w pliku
`system/etc/dnscrypt-proxy/dnscrypt-resolvers.csv` . Tę zmienną możemy sobie dostosować przed
instalacją paczki `.zip` , jak i później już po jej wgraniu do systemu.

Paczkę `.zip` wrzucamy na kartę SD czy flash smartfona i uruchamiamy urządzenie w trybie recovery.
Na Neffos C5 trzeba wyłączyć telefon i włączyć go trzymając jednocześnie VolumeUp i Power. Będąc w
trybie recovery, przechodzimy do Instaluj i wskazujemy lokalizację paczki `.zip` . W procesie
instalacji odznaczamy weryfikację sygnatury pliku.

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-instalacja-twrp](/img/2016/10/001.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-instalacja-twrp.png#huge)

## Wsparcie dla skryptów init

Tak zainstalowany `dnscrypt-proxy` dostarcza skrypt init, który ma za zadanie uruchamiać tę
aplikację na starcie systemu. Problem w tym, że nie każdy Android zawiera natywne wsparcie dla
skryptów init. To, czy kernel naszego Androida ma takie wsparcie, możemy łatwo ustalić sprawdzając
czy na flash'u telefonu mamy katalog `/system/etc/init.d/` . Jeśli on jest, to wsparcie również
jest. Jeśli zaś nie mamy tego katalogu, to musimy postarać się o alternatywny menadżer uruchamiania
skryptów init, który będzie je nam inicjował na starcie systemu.

Ten Neffos C5 nie ma wsparcia dla skryptów init, zatem trzeba zainstalować [Universal Init.d][8].
Ta aplikacja będzie wymagać dostępu root ale [jest ona OpenSource][9], także bez obaw:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-instalacja-universal-init-d](/img/2016/10/002.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-instalacja-universal-init-d.png#huge)

Po instalacji, uruchamiamy aplikację. Powinno nam się pojawić poniższe okienko:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-init](/img/2016/10/003.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-init.png#medium)

Jak widzimy wyżej, skrypt init `99dnscnrypt` został wykryty. Jest też opcja przetestowania kernela
pod kątem wsparcia dla skryptów init. Na wypadek gdybyśmy nie byli pewni klikamy w przycisk Test.
Przy teście będzie wymagany restart urządzenia:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-test](/img/2016/10/004.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-test.png#huge)

Podczas testu został stworzony tymczasowy skrypt, który miał na celu wgranie na kartę SD małego
pliku. Po restarcie systemu, obecność tego pliku jest weryfikowana. Jeśli nie został stworzony, to
kernel nie ma wsparcia dla skryptów init.

Widoczny wyżej brak wsparcia nie stanowi żadnego problemu, bo właśnie ta aplikacja zajmie się
wywoływaniem skryptów przy starcie Androida. Ten test ma jedynie na celu zapewnienie nas, że
skrypty nie zostaną wywołane dwukrotnie, tj. raz przez sam kernel, a drugi przez aplikację Universal
Init.d.

W przypadku braku wsparcia musimy jeszcze zrobić jedną rzecz. Chodzi o przestawienie przełącznika z
OFF na ON. W przypadku, gdyby nasz kernel obsługiwał skrypty init natywnie, to przełącznika nie
ruszamy. Po przestawieniu pozycji przełącznika i wykonaniu testu, powinien nam zostać zwrócony
komunikat, że nasz kernel posiada już wsparcie dla skryptów init:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-init](/img/2016/10/005.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-kernel-init.png#medium)

Ostatnia rzeczą jest przetestowanie startu skryptu `99dnscnrypt` . Zaznaczamy go zatem i przy górnej
krawędzi ekranu powinno nam się pojawić menu. Klikamy na strzałkę w kółku po prawej stronie:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-skrypt-init-test](/img/2016/10/006.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-skrypt-init-test.png#medium)

Teraz możemy odpalić terminal, np. [termux][10] i sprawdzić, czy proces `dnscrypt-proxy` został
uruchomiony:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-skrypt-init-test](/img/2016/10/007.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-skrypt-init-test.png#huge)

Jak widzimy, działa on sobie tak jak powinien. Możemy naturalnie uruchomić ponownie smartfona, by
się przekonać czy autostart działa należycie.

## Problemy

Oczywiście nasłuchujący demon, to tylko połowa drogi. Trzeba jeszcze jakoś przekierować do niego
zapytania DNS. Tym zadaniem zajmują się reguły w `iptables` . Skrypt `99dnscnrypt` na starcie
pociąga za sobą tworzenie dwóch reguł w tablicy NAT w łańcuchu OUTPUT. Pakiety przesyłane na port
53 TCP/UDP są kierowane na adres 127.0.0.1:53 , a tutaj już nasłuchuje demon `dnscrypt-proxy` . W
zasadzie nie musimy nic robić, by resolver DNS w Androidzie nam działał.

Jest jednak jeden problem. Jako, że wykorzystujemy przekierowanie na poziomie firewall'a, to
aplikacje Androida nie są o tym fakcie powiadomione. W efekcie wartości dostępne w `getprop`
wskazują na adresy DNS uzyskane z lease DHCP sieci, do której zostaliśmy podłączeni, czy też w
przypadku ich braku na adresy DNS Google. Poniżej przykład:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-problemy](/img/2016/10/008.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-problemy.png#medium)

Za sprawą takiego stanu rzeczy, niektóre aplikacje mogą zwracać mylące adresy, choć wszystkie nazwy
DNS są wrzucane w szyfrowany kanał i przesyłane do serwerów DNS, które sobie wybraliśmy w
konfiguracji `dnscrypt-proxy` .

## Test szyfrowania DNS pod Androidem

W przypadku, gdy `dnscrypt-proxy` przesyła zapytania do serwerów DNS od OpenDNS ( `cisco` w
konfiguracji ), to bardzo prosto możemy sprawdzić czy zapytania są naprawdę szyfrowane i czy na
pewno są przesyłane do tego providera DNS:

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-test-resolver](/img/2016/10/009.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-test-resolver.png#huge)

Widzimy wyraźnie, że zapytanie jest szyfrowane. Możemy również przetestować stronę w przeglądarce.
Odpalmy zatem, np. Firefox'a i odwiedźmy adres `https://www.opendns.com/welcome/` :

![dnscrypt-proxy-android-smartfon-szyfrowanie-dns-firefox-opendns-test](/img/2016/10/010.dnscrypt-proxy-android-smartfon-szyfrowanie-dns-firefox-opendns-test.png#huge)


[1]: https://dnscrypt.org/#dnscrypt-android
[2]: /post/recenzja-smartfon-neffos-c5-od-tp-link/
[3]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[4]: /post/android-repozytorium-aplikacji-opensource-f-droid/
[5]: https://download.dnscrypt.org/dnscrypt-proxy/
[6]: https://twrp.me/
[7]: https://www.clockworkmod.com/
[8]: https://play.google.com/store/apps/details?id=com.androguide.universal.init.d
[9]: https://github.com/Androguide/Universal-init.d
[10]: https://play.google.com/store/apps/details?id=com.termux
