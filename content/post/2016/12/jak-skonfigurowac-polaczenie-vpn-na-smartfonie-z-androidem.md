---
author: Morfik
categories:
- Android
date:    2016-12-07 18:54:37 +0100
lastmod: 2016-12-07 18:54:37 +0100
published: true
status: publish
tags:
- vpn
- smartfon
- aplikacje
- openvpn
- prywatność
GHissueID: 453
title: Jak skonfigurować połączenie VPN na smartfonie z Androidem
---

[W artykule o postawieniu serwera VPN][1] poruszyłem jedynie kwestię konfiguracji klienta mającego
system operacyjny z rodziny linux, a konkretnie była to dystrybucja Debian. Niemniej jednak, mając
działający serwer VPN gdzieś tam za granicą, możemy również do niego podłączyć się za pomocą
smartfona z Androidem i to praktycznie z dowolnego miejsca na ziemi. W ten sposób możemy
zabezpieczyć nasze połączenie przed cenzurą internetu, która obecnie jest przeprowadzana na naszych
oczach. Jako, że smartfony są popularniejsze od komputerów czy laptopów i zwykle przesyłamy z nich
tak samo ważne (albo i ważniejsze) dane, to wypadałoby zaszyfrować cały ruch z takiego telefonu.
Niniejszy wpis będzie właśnie dotyczył tego tematu, który zostanie opisany w oparciu [smartfon
Neffos C5][2] od TP-LINK mający na pokładzie Androida w wersji 5.1 (Lollipop).

<!--more-->
## Aplikacje VPN na Androida

Generalnie rzecz biorąc, Android ma wbudowany mechanizm VPN. Można go skonfigurować przechodząc pod
Ustawienia => Więcej => VPN. By być w stanie skonfigurować takie bezpieczne połączenie, musimy
również aktywować mechanizm blokady ekranu.

![](/img/2016/12/001.vpn-openvpn-smartfon-android-standardowy-klient.png#huge)

Niemniej jednak, ku mojemu zdziwieniu, to domyślne oprogramowanie nie wspiera OpenVPN (albo ja nie
potrafię skonfigurować tego połączenia):

![](/img/2016/12/002.vpn-openvpn-smartfon-android-standardowy-klient.png#huge)

Potrzebne jest zatem alternatywne oprogramowanie. Z tego co znalazłem w sklepie Google Play i w
[repozytorium F-Droid][3], to w zasadzie są dwie aplikacje, które się nadają do wykorzystania przy
OpenVPN. Jedna z nich to [OpenVPN Connect][4], a druga to [OpenVPN for Android][5]. [Z tego co
znalazłem][6], to źródła OpenVPN Connect nie są do końca wolne, no i też ta aplikacja wyświetla
reklamy. Natomiast jeśli chodzi o OpenVPN for Android, to on nie ma reklam i jest to projekt w
pełni otwartoźródłowy.

Inna różnica między tymi aplikacjami tkwi w możliwości konfiguracji połączenia z serwerem. W
zasadzie OpenVPN Connect nie oferuje nam zbytnio żadnych opcji. Możemy jedynie zaimportować
konfigurację klienta VPN, którą napisaliśmy sobie na komputerze. Natomiast OpenVPN for Android ma
naprawdę sporo opcji i praktycznie każdy aspekt połączenia jesteśmy w stanie dostosować po
zaimportowaniu pliku konfiguracyjnego.

Obie te aplikacje nie wymagają ukorzenionego Androida (root) ale biorąc pod uwagę, że OpenVPN for
Android jest i OpenSource i nie serwuje reklam, to niżej opiszę tylko to narzędzie.

Jeśli zaś chodzi o Androidy, które przeszły proces root, to możemy pokusić się o instalację binarki
OpenVPN, mniej więcej takiej jaka jest wykorzystywana w standardowym linux'ie. To rozwiązanie wymaga
zainstalowania na smartfonie aplikacji [OpenVPN Settings][7] oraz [OpenVPN Installer][8] .
Generalnie rzecz biorąc, ta druga aplikacja dostarczy demona `openvpn` , a ta pierwsza zaimportuje
dla niego konfigurację. Niemniej jednak, te aplikacje są datowane na rok 2012, czyli ponad 4 lata
nie były aktualizowane. Dlatego też nie będę opisywał procesu konfiguracji połączenia w oparciu o to
rozwiązanie.

## Certyfikaty klienckie dla smartfonów

Są z grubsza dwie metody uwierzytelniania się klientów na serwerze VPN. Jedna z nich zakłada
wykorzystanie hasła i loginu, druga certyfikatów klienckich. Ja będę korzystał z certyfikatów, bo
zapewniają większe bezpieczeństwo. Nie będę jednak tutaj opisywał całego procesu generowania takiego
certyfikatu, bo to zostało zrobione już we [wpisie poświęconemu generowaniu certyfikatów z
wykorzystaniem easy-rsa][9]. Jeśli nie mamy takich certyfikatów, to oczywiście musimy pierw zajrzeć
do tego podlinkowanego wyżej artykułu i sobie je stworzyć. Unikajmy jednak wykorzystywania tego
samego certyfikatu na każdym kliencie. Może OpenVPN daje nam taką możliwość ale jest to wielce
niezalecane.

## Konfiguracja połączenia VPN z OpenVPN for Android

Zakładam tutaj, że mamy już postawiony działający serwer VPN oraz, że wygenerowaliśmy sobie stosowne
certyfikaty. Możemy teraz przejść na smartfon i zainstalować tam aplikację OpenVPN for Android:

![](/img/2016/12/003.vpn-openvpn-smartfon-android-aplikacja.png#huge)

Po uruchomieniu aplikacji przywita nas informacja o braku połączeń VPN. Musimy zatem sobie stworzyć
jakąś konfigurację ale nie musimy tego robić ręcznie. Jeśli dysponujemy plikiem konfiguracyjnym
OpenVPN dla standardowego linux'a, to możemy go zaimportować:

![](/img/2016/12/004.vpn-openvpn-smartfon-android-import-konfiguracja.png#huge)

Pamiętajmy jednak, by certyfikaty/klucze zawrzeć w konfiguracji OpenVPN (znaczniki `ca` , `cert` ,
`key` oraz `tls-auth` ) zamiast dłączać je w osobnych plikach. Poniżej jest przykładowa konfiguracja
dla mojego smartfona (nazwa pliku jest dowolna ale musi się kończyć `.ovpn` ):

    client
    dev tun
    proto udp
    remote 11.22.33.44 1194
    resolv-retry infinite
    nobind
    ;user nobody
    ;group nogroup
    persist-key
    persist-tun
    cipher AES-256-CBC
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
    auth SHA512
    keysize 256
    tls-version-min 1.2
    comp-lzo
    verb 3
    ;mute 20
    script-security 2
    up /etc/openvpn/update-resolv-conf.sh
    down /etc/openvpn/update-resolv-conf.sh
    auth-nocache

    ;auth-user-pass /etc/openvpn/auth/morfitronik.auth

    remote-cert-tls server
    verify-x509-name "morfitronik-server-vpn" name
    ;ca /etc/openvpn/certs/morfitronik-ca.crt
    ;cert /etc/openvpn/certs/morfitronik-client-vpn-morfik.crt
    ;key /etc/openvpn/certs/morfitronik-client-vpn-morfik.key
    ;tls-auth /etc/openvpn/certs/morfitronik-ta.key 1
    key-direction 1

    <ca></ca>
    <cert></cert>
    <key></key>
    <tls-auth></tls-auth>

Po zaimportowaniu takiego pliku zostanie wyrzucone krótkie podsumowanie. W tym przypadku kilka opcji
zostało zignorowanych ale nie ma to większego wpływu na połączenie. Konfiguracja zostanie również
zapisana w katalogu aplikacji OpenVPN for Android, a sam plik, który przenieśliśmy na pamięć/kartę
SD telefonu możemy usunąć.

![](/img/2016/12/005.vpn-openvpn-smartfon-android-import-konfiguracja.png#huge)

By się teraz połączyć z serwerem VPN i przesłać cały ruch ze smartfona szyfrowanym kanałem,
wystarczy tapnąć w widoczną wyżej pozycję na liście:

![](/img/2016/12/006.vpn-openvpn-smartfon-android-polaczenie-server.png#huge)

To w zasadzie cała istota podłączenia do VPN naszego smartfona. Rzućmy sobie jeszcze okiem na samą
konfigurację połączenia.

## Dostosowanie konfiguracji połączenia w OpenVPN for Android

Aplikacja OpenVPN for Android importując plik konfiguracyjny OpenVPN automatycznie dostosowała
wszystkie opcje za nas. Jeśli z jakiegoś powodu konfiguracja serwera VPN ulegnie zmianie, to
naturalnie możemy ponownie zaimportować plik konfiguracyjny ale też możemy ręcznie przestawić te
opcje, które zmieniły się.

W zasadzie jesteśmy w stanie wskazać pliki certyfikatów/kluczy jeśli te nie są dodane bezpośrednio w
pliku konfiguracyjnym.

![](/img/2016/12/007.vpn-openvpn-smartfon-android-opcje-polaczenia.png#huge)

Możemy także określić adres IP, port i protokół serwera VPN, jak i również dodać kilka serwerów:

![](/img/2016/12/008.vpn-openvpn-smartfon-android-opcje-polaczenia.png#medium)

Są tez opcje przepisania konfiguracji DNS:

![](/img/2016/12/009.vpn-openvpn-smartfon-android-opcje-polaczenia.png#huge)

Oraz parametry konfigurujące tablicę routingu:

![](/img/2016/12/010.vpn-openvpn-smartfon-android-opcje-polaczenia.png#huge)

Konfiguracja szyfrów wykorzystywanych w kanale kontrolnym i kanale danych też może zostać
dostosowana:

![](/img/2016/12/011.vpn-openvpn-smartfon-android-opcje-polaczenia.png#huge)

W opcjach zaawansowanych możemy nieco dostosować sobie zachowanie klienta VPN oraz dodać
niestandardowe parametry:

![](/img/2016/12/012.vpn-openvpn-smartfon-android-opcje-polaczenia.png#huge)

Cały ruch ze smartfona będzie przesyłany do serwera VPN. Jeśli chcemy jakieś aplikacje wyłączyć spod
działania tego mechanizmu, to możemy to jak najbardziej uczynić:

![](/img/2016/12/013.vpn-openvpn-smartfon-android-opcje-polaczenia.png#medium)

Wszelkie zmiany jakie wprowadzimy w konfiguracji połączenia VPN zostaną zapisane, co wygeneruje nowy
plik konfiguracyjny. Ten plik możemy wyeksportować w celu zaimportowania go na innym urządzeniu.
Możemy także powielić go i użyć w celu skonfigurowania kolejnego serwera VPN zmieniając tylko
określone parametry:

![](/img/2016/12/014.vpn-openvpn-smartfon-android-eksport-ustawienia.png#huge)

Po podłączeniu się do serwera VPN, na belce systemowej powinniśmy zauważyć notyfikację z informacją
o ilości przesłanych danych i aktualnej prędkości transferu:

![](/img/2016/12/015.vpn-openvpn-smartfon-android-test-polaczenia.png#medium)

## Połączenie VPN i oszczędzanie baterii

Zastosowanie klienta VPN w smartfonach dość znacznie utylizować baterię. Jakby nie patrzeć,
połączenie VPN standardowo jest utrzymywane, by transfer danych mógł być realizowany. To z kolei
wymaga aktywnego WiFi czy LTE, a wiemy, że te moduły dają się mocno we znaki baterii. Możemy jednak
tak skonfigurować aplikację OpenVPN for Android, by przy niewielkim obciążeniu łącza i zablokowanym
ekranie rozłączała nam połączenie VPN.

![](/img/2016/12/016.vpn-openvpn-smartfon-android-opcje-bateria.png#huge)

Jak widzimy wyżej, nie tylko możemy rozłączyć połączenie VPN, gdy smartfon jest nieużywany ale tez
możemy automatycznie zestawiać takie połączenie po uruchomieniu/odblokowaniu urządzenia.

Pamiętajmy jednak, że przy restarcie systemu część danych zawsze może się przedostać nieszyfrowanym
tunelem zanim klient VPN zostanie uruchomiony. Tutaj raczej nic więcej nie da się zrobić, no może za
wyjątkiem ręcznego dezaktywowania WiFi przed wyłączeniem telefonu. Po starcie systemu, WiFi będzie w
dalszym ciągu wyłączone i wystarczy poczekać, aż klient VPN zostanie uruchomiony. Może i nie będzie
połączenia jako takiego ale aplikacja OpenVPN for Android będzie oczekiwać na sieć:

![](/img/2016/12/017.vpn-openvpn-smartfon-android-brak-sieci.png#medium)

W takim wypadku już wszystko jest przygotowane do połączenia i wystarczy włączyć WiFi.


[1]: /post/jak-skonfigurowac-serwer-vpn-na-debianie-openvpn/
[2]: http://www.neffos.pl/product/details/C5
[3]: /post/android-repozytorium-aplikacji-opensource-f-droid/
[4]: https://play.google.com/store/apps/details?id=net.openvpn.openvpn
[5]: https://play.google.com/store/apps/details?id=de.blinkt.openvpn
[6]: http://ics-openvpn.blinkt.de/FAQ.html
[7]: https://play.google.com/store/apps/details?id=de.schaeuffelhut.android.openvpn
[8]: https://play.google.com/store/apps/details?id=de.schaeuffelhut.android.openvpn.installer
[9]: /post/generowanie-certyfikatow-przy-pomocy-easy-rsa/
