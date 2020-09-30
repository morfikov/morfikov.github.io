---
author: Morfik
categories:
- Linux
date: "2015-10-15T19:32:48Z"
date_gmt: 2015-10-15 17:32:48 +0200
published: true
status: publish
tags:
- wifi
- debian
- sieć
title: Konfiguracja połączenia WiFi pod debianem
---

Sieci bezprzewodowe w obecnych czasach to standard i nie ma chyba miejsca na ziemi gdzie nie dałoby
rady ulokować routera WiFi, do którego można by podłączyć szereg urządzeń. Każdy kto próbował
konfigurować sieć bezprzewodową na debianie, wie, że może to być bardzo upierdliwe, zwłaszcza jeśli
mamy dostęp do wielu AP, które posiadają różne konfiguracje. Wynaleziono, co prawda, automaty, które
mają pomagać w ogarnięciu tego całego bezprzewodowego zamieszania, np. `network-manager` czy `wicd`
ale w przypadku lekkich stacji roboczych, które nie mają wgranego pełnego środowiska graficznego, a
jedynie jakiś menadżer okien, np. Openbox, to instalacja tych powyższych narzędzi może zwyczajnie
nie wchodzić w grę.

<!--more-->
## Przestarzałe narzędzia

W linux'ie istnieje obecnie podział na dwie grupy narzędzi, które służą do konfiguracji interfejsów
sieciowych. Jedna z nich zawiera narzędzia, które mają już swoje lata i powoli się od nich odchodzi,
choć nadal wiele osób z nich korzysta. Drugi zestaw to właśnie te narzędzia, na które ludzie
migrują. Wobec czego mamy pakiet `wireless-tools` , który został wyparty przez `iw` . Podobnie
zresztą też narzędzia zawarte w pakiecie `net-tools` , takie jak `ifconfig` , zostały zastąpione
przez narzędzia z pakietu `iproute2` .

W przypadku przechodzenia z `wireless-tools` na `iw` , można skorzystać ze ściągawki dostępnej [pod
tym adresem][1]. Jeśli chodzi zaś o `net-tools` i `iproute2` , to można skorzystać z rozpiski
znajdującej się [tutaj][2]. Więcej na temat aktualnego oprogramowania i nowszych standardów
sieciowych można poczytać i [tutaj][3].

## Narzędzia do konfiguracji sieci WiFi

W tym artykule nie będziemy operować na graficznych aplikacjach, które umożliwiają w miarę szybkie
skonfigurowania połączenia WiFi. Będziemy za to korzystać z oprogramowania jakiego dostarcza nam
pakiet `wpasupplicant` . Jeśli nie mamy tego pakietu w systemie, to czym prędzej go doinstalujmy.

Będziemy operować na dwóch plikach. Pierwszym jest `/etc/network/interfaces` , drugim zaś
`/etc/wpa_supplicant/wpa_supplicant.conf` . W przypadku połączeń przewodowych, chyba każdy potrafi
skonfigurować sobie takie połączenie przez plik `/etc/network/interfaces` . Jeśli chodzi o
połączenia WiFi, to można postąpić podobnie, tylko trzeba dopisać trochę więcej parametrów.

Generalnie istnieją dwie metody konfiguracji: jedna zakłada wpisywanie opcji bezpośrednio do pliku
`/etc/network/interfaces` , druga zaś ma na celu przeniesienie konfiguracji wszystkich sieci
bezprzewodowych do osobnego pliku, tj. do `/etc/wpa_supplicant/wpa_supplicant.conf` , który trzeba
stworzyć. Nie musimy go pisać od początku, bo dostępny jest szkielet w
`/usr/share/doc/wpasupplicant/examples/wpa_supplicant.conf.gz` , który wystarczy przekopiować do
katalogu `/etc/wpa_supplicant/` i odpowiednio przerobić. Ta druga metoda ma tę przewagę, że można
sobie zarządzać tym gdzie zostaniemy podłączeni w oparciu o określone priorytety danych sieci WiFi,
a gdy te są takie same, decydować będzie siła sygnału czy jakość zabezpieczeń.

### Poufność danych logowania

Z racji faktu, że w obu powyższych plikach, w zależności od konfiguracji, mogą znajdować się hasła
do sieci bezprzewodowych, dobrze jest zmienić domyślne uprawnienia tych plików na `600`:

    # chmod 0600 /etc/network/interfaces
    # chmod 0600 /etc/wpa_supplicant/wpa_supplicant.conf

## Plik /etc/network/interfaces

Przechodzimy do edycji pliku `/etc/network/interfaces` . W moim przypadku on wygląda tak:

    auto lo
    iface lo inet loopback

    allow-hotplug eth1
    #auto eth1
    iface eth1 inet dhcp

    #allow-hotplug wlan1
    auto wlan1
    iface wlan1 inet dhcp
          wpa-driver nl80211
          wpa-debug-level -1
          wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Powyższy blok kodu konfiguruje trzy interfejsy. Pierwszy z nich zwykle jest taki sam na każdej
maszynie, bo odpowiada on za pętlę zwrotną. Pozostałe będą się różnić w zależności od konfiguracji.
W każdym razie, w tym przypadku mamy po jednym interfejsie przewodowym i bezprzewodowym. Poniżej
jest krótkie wyjaśnienie wykorzystanych opcji:

  - `auto` -- odpowiada za automatyczną konfigurację interfejsu via skrypt `/etc/init.d/networking`
    . Jeśli jakiś interfejs posiada opcję `auto`, będzie tym samym konfigurowany na starcie systemu.
  - `allow-hotplug` -- mniej więcej to samo co `auto` , z tym, że kernel będzie próbował podnieść
    interfejs za każdym razem gdy zostanie wykryte zdarzenie hotplug przez udev.
  - `iface` -- definiuje parametry konfiguracyjne dla danego interfejsu. To tu określamy, czy
    konfiguracja ma być przeprowadzana statycznie ( `static` ), przy czym trzeba ręcznie określić
    address, network, netmask, broadcast oraz gateway, czy też dynamicznie, tj. przy pomocy
    protokołu DHCP ( `dhcp` ).
  - `wpa-` -- oznacza, że wykorzystywane jest oprogramowanie `wpasupplicant` i te opcje odnoszą się
    do niego.

W przypadku interfejsu `wlan1` profile sieci są definiowane w osobnym pliku. Konfiguracja WiFi
wymaga określenia różnych protokołów i kilku innych parametrów, których nie ma w przypadku
połączenia przewodowego. Plik konfiguracyjny definiujemy przez `wpa-conf` . Mamy tam także opcję
zdefiniowania sterownika dla karty. Nowsze modele będą bezproblemowo działać z `nl80211` , starsze
zaś mogą mieć z nim problemy i w takim przypadku trzeba tam ustawić `wext` . Jeśli nie określi się
sterownika, zostanie także użyty `wext` . Domyślnie też poziom logowania narzędzia `wpa_supplicant`
jest trochę zbyt niski i log systemowy może zbierać sporo komunikatów podobnych do tego poniżej:

    wpa_supplicant[112756]: wlan0: WPA: Group rekeying completed with f3:92:d5:21:31:14 [GTK=CCMP]

W zależności od częstości zmian kluczy, może tych wiadomości być więcej lub mniej. Jako, że jest to
tylko informacja, którą człowiek nie powinien sobie głowy zawracać, można ją zignorować i oczyścić
nieco log. Do tego celu służy parametr `wpa-debug-level` .

## Plik /etc/wpa_supplicant/wpa_supplicant.conf

Jako, że część konfiguracji WiFi znajduje się w pliku `/etc/wpa_supplicant/wpa_supplicant.conf` ,
rzućmy na niego okiem:

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    eapol_version=2
    ap_scan=1
    country=PL
    bss_max_count=200
    filter_ssids=0

    network={
            priority=10
            ssid="Winter Is Coming"
            bssid=11:22:33:44:55:66
            #psk="Moje haslo"
            psk=811339fbba9c799a6216ec3390bd1ae84680242fc0fcc918b0636a3dbe0fd310
            proto=RSN
            key_mgmt=WPA-PSK
            pairwise=CCMP
            group=CCMP
            auth_alg=OPEN
            scan_ssid=0
            disabled=0}

Powyższy plik składa się z dwóch części: globalnej i sieciowej. Parametry globalne, czyli te
nieujęte w `network={}` są aplikowane do każdej sieci wpisanej poniżej. Z kolei wszystko to co się
znajdzie w danym `network={}` dotyczy tylko i wyłącznie konkretnej sieci. Parametry w obu sekcjach
mogą się różnić, w zależności od tego jaki sprzęt posiadamy i gdzie przebywamy, oraz ma też wpływ
sama konfiguracja sieci na routerze. Powyższy config powinien działać w większości przypadków.

### Sekcja globalna

Sekcja globalna może zawierać `ctrl_interface` , który to składa się z dwóch opcji: `DIR` oraz
`GROUP`, z których pierwsza określa miejsce gdzie będzie tworzone gniazdo nasłuchujące zapytań od
zewnętrznych programów, np w celu konfiguracji sieci. Druga opcja odpowiada za grupę, której
członkowie będą mogli zarządzać konfiguracją WiFi, przydatne w korzystaniu z `wpa_cli` , czy też z
graficznych narzędzi, np. `wpa_gui` .

Możemy także określić wersję protokołu EAPOL przy pomocy `eapol_version` . Starsze urządzenia mogą
mieć problemy z obsługą wersji `2` . W takim przypadku trzeba ustawić tutaj `1` .

Narzędzie `wpa_supplicant` może dokonywać skanowania pasma i wybrać najlepszą sieć, do której nas
podłączy. Kwestię skanowania można zostawić sterownikowi karty albo też odpowiednio ustawić opcję
`ap_scan` .

Wykorzystywanie pewnych konfiguracji WiFi na terenie pewnych państw może być zakazane i by uniknąć
nieporozumień, można zdefiniować profil w oparciu o miejsce pobytu przy pomocy parametru `country` .
Zwykle jest to dwuznakowy kod państwa, np. dla Polski to będzie `PL` .

Jeśli w naszej okolicy jest dużo sieci bezprzewodowych, możemy ograniczyć ilość wpisów w cache za
pomocą `bss_max_count` . Wpisy są dodawane do cache przy skanowaniu pasma radiowego, po tym jak
zostanie napotkana jakaś sieć. Im jest ich więcej, tym też więcej zajmują one pamięci.

Istnieje również filtr sieci, który ma na celu zwrócenie jedynie tych wyników, do których posiadamy
konfiguracje. Jeśli chcemy skorzystać z tego ficzera ustawiamy `filter_ssids` .

Warto mieć jeszcze na uwadze parametr `update_config=1`, po zdefiniowaniu którego będzie możliwa
zmiana parametrów ustawionych w pliku `/etc/wpa_supplicant/wpa_supplicant.conf` przez zewnętrzne
narzędzia konfiguracyjne. Jednak w takim przypadku zostaną usunięte wszystkie komentarze.

### Sekcja sieciowa

Jeśli chodzi o część sieciową, nie trzeba definiować wszystkich wypisanych wyżej parametrów, być
może wystarczy samo `ssid` oraz `psk`. Jeśli jednak chce się mieć kontrolę nad konfiguracją i nie
ufa się zbytnio automatom, które resztę parametrów powinny sobie dobrać same, można doprecyzować
poszczególne opcje, tak jak to zostało zrobione powyżej.

Każda sieć musi posiadać unikalny identyfikator sieci ( `ssid` ). By się połączyć z siecią o danym
SSID, trzeba znać hasło ( `psk` ), oczywiście tylko w przypadku gdy sieć jest zabezpieczona. Jako,
że jest to hasło do naszej sieci, powinno być ono możliwe długie i skomplikowane, bo od niego, w
głównej mierze, zależeć będzie bezpieczeństwo sieci. Mamy do wyboru dwa rodzaje haseł. Jednym z
nich jest hasło w postaci czystego tekstu ([7 bitowy kod ASCII][4]), przy czym musi ono zostać
ujęte w `" "` i posiadać 8-63 znaków. Drugim typem hasła jest ciąg hexalny o długości 64 znaków,
wpisywany bez `" "` . Istnieje także możliwość sprecyzowania `bssid` , z którym chcemy się połączyć.
Jest to nic innego jak tylko adres fizyczny danego urządzenia (MAC), podobny do tego w kartach
sieciowych. W konkretnej sieci o określonym SSID może znajdować się kilka punków dostępowych. Każdy
z tych routerów ma osobny BSSID i jeśli z jakichś powodów chce się korzystać z konkretnego z nich,
to trzeba określić jego adres za pomocą tego parametru.

Szyfr grupowy ( `group cipher` ) jest używany w przypadku transmisji ramek multicast, włączając w to
broadcast, i musi być zrozumiały (taki sam) dla wszystkich powiązanych stacji roboczych. Szyfr
parowy (`pairwise cipher`) jest wykorzystywany przez ramki unicastowe, np. połączenia http, i musi
być zrozumiały dla routera i oddzielnie dla każdej ze stacji roboczych -- różne stacje mogą używać
różnych szyfrów parowych. W WPA2 jest możliwość skonfigurowania routera by wspierał wiele
unicastowych szyfrów parowych. Jednak tylko jeden szyfr grupowy jest wykorzystywany do kodowania
ramek broadcast/multicast, tak by wszystkie stacje powiązane z daną siecią mogły być w stanie
odebrać tego typu pakiety. Będzie to najsłabszy z dozwolonych szyfrów parowych. To dlatego można
skorzystać, albo z TKIP, albo CCMP jeśli chodzi o szyfry parowe ale tylko TKIP może być wykorzystany
jako szyfr grupowy. CCMP może być użyty jako szyfr grupowy ale tylko w przypadku gdy zdefiniowany
jest on jako jedyny w dozwolonych szyfrach parowych.

Mamy także do dyspozycji kilka różnych protokołów zabezpieczających sieć przed nieautoryzowanym
dostępem, np. WEP, WPA, czy WPA2. Choć na dobrą sprawę WEP już się do niczego nie nadaje. By
poprawić bezpieczeństwo naszej sieci, możemy sprecyzować to, z którego protokołu zamierzamy
korzystać. Możemy to zrobić za pomocą opcji `proto` . RSN odpowiada za WPA2. Nie wszystkie
urządzenia, szczególnie te starsze, potrafią operować na tym protokole. Warto to wziąć pod uwagę w
przypadku ewentualnych problemów z siecią. Ponad to, trzeba także mieć na względzie obciążenie sieci
przez mocniejsze szyfrowanie w przypadku WPA2 -- może ono obniżyć maksymalną przepustowość łącza.

By dodatkowo zwiększyć poufność połączeń wewnątrz sieci, wprowadzono protokół zarządzania kluczami
uwierzytelniającymi (authenticated key management protocol). Definiowany jest on przez opcję
`key_mgmt` ). Zarządzanie kluczami może odbywać się w dwóch trybach. Pierwszym z nich jest WPA-PSK
(Personal albo Pre-shared), gdzie wszystkie stacje robocze korzystają z tego samego klucza, co daje
możliwość podsłuchiwania hostów w sieci. Niczym się taka sieć nie różni zbytnio od zwykłego LAN.
Drugi tryb to WPA-EAP (Enterprise), który wykorzystuje serwer RADIUS do przydzielania kluczy każdemu
użytkownikowi z osobna, szyfrując ruch wewnątrz sieci na podobnej zasadzie co w przypadku VPN.

Z kolei `auth_alg` określa rodzaj algorytmu uwierzytelniającego. Open system authentication (OPEN)
składa się z dwóch komunikatów. Pierwszy z nich jest to prośba o uwierzytelnienie wysłana przez
klienta do routera. Taka wiadomość zawiera ID stacji roboczej, zwykle jest to adres MAC. Drugi
komunikat to odpowiedź routera na to żądanie i może ona zawierać wiadomość o powodzeniu lub porażce,
np. gdy adres MAC klienta jest zbanowany w konfiguracji routera. W przypadku Shared Key
Authentication (SHARED) obie strony połączenia dzielą między sobą ten sam klucz lub hasło, które
jest ręcznie ustawiane zarówno na stacji klienckiej jak i na routerze. OPEN jest wymagany w
przypadku WPA/WPA2, zaś SHARED dla WEP .

Sieci w dostępnym paśmie radiowym mogą być jawne albo ukryte. Różnią się one między sobą
rozgłaszaniem informacji o swoim istnieniu w okolicy. Część osób uważa, że lepiej jest wyłączyć
tego typu broadcast na routerze i schować w ten sposób sieć przed innymi osobami, które mogą
spróbować ją spenetrować. W przypadku ukrytych sieci, każde urządzenie, które chce się podłączyć
musi wysyłać próbki żądań z określonym SSID i jeśli taka sieć się znajduje w okolicy, odpowie na
nie. Jeśli posiadamy ukrytą sieć, to by ją odnaleźć trzeba ustawić `scan_ssid` . Ukrywanie sieci
jednak mija się z celem, gdyż SSID jest rozgłaszane w pakietach sieciowych. Ponadto, w przypadku gdy
sieć jest ukryta, inne routery nie mogą dobrać optymalnego kanału dla połączenia, co może skutkować
sporymi zakłóceniami, np. gdy na jednym kanale nie widać żadnej sieci (a jest tam 10 ukrytych), a na
pozostałych są 1-2 sieci, które widać. W takiej sytuacji router spróbuje wleźć na ten pozornie wolny
kanał, co dodatkowo pogorszy komunikacje w obrębie wszystkich znajdujących się tam sieci, a
normalnie, router by wszedł na którychś z mniej zapchanych kanałów.

Do sieci możemy łączyć się z automatu albo ręcznie. Jeśli mamy zaufaną sieć, możemy się do niej
łączyć automatycznie. Natomiast jeśli w grę wchodzi jakiś publiczny Access Point, który nie jest w
żaden sposób zabezpieczony, dobrze jest wyłączyć ten ficzer przez ustawienie `disabled=1` .

Jeśli chodzi o kwestię automatycznej konfiguracji sieci bezprzewodowych i dynamicznego przełączania
się między nimi, trzeba dodatkowo posłużyć się parametrem `priority` . Przyjmuje on wartości od 0 do
1000, im wyższy numer, tym większy priorytet. W przypadku dostępności kilku zaufanych sieci,
`wpa_supplicant` wybierze sieć o najwyższym priorytecie. Jeśli będzie kilka sieci z takim samym
priorytetem, wtedy będzie decydować siła sygnału i zabezpieczenia danej sieci.

#### Narzędzie wpa_passphrase

Nikomu za bardzo nie chce się wymyślać długich i skomplikowanych haseł ale istnieje narzędzie, które
może wygenerować hasło szesnastkowe w oparciu o nazwę sesji (SSID) oraz o jakąś frazę, dajmy na to
faktyczne hasło. Mowa oczywiście o `wpa_passphrase` . Poniżej przykład jego wykorzystania:

    # wpa_passphrase "Moja siec" "Moje haslo"
    network={
            ssid="Moja siec"
            #psk="Moje haslo"
            psk=811339fbba9c799a6216ec3390bd1ae84680242fc0fcc918b0636a3dbe0fd310
    }

I w ten sposób, mając słabe hasło, można dość solidnie zabezpieczyć swoją sieć. Przy czym trzeba
pamiętać, że ten ciąg zawsze będzie taki sam o ile SSID i hasło nie zostaną zmienione i każdy kto
zna te dwie wartości, może wygenerować sobie ten 64 znakowy ciąg.

## Interfejs bond

W ten oto sposób można łatwo pozbyć się pakietów `network-manager` lub/i `wicd` . Jeśli ktoś
naprawdę chce korzystać z GUI, to może sobie doinstalować pakiet `wpagui` . Jednak cała konfiguracja
może zostać przeprowadzona z poziomu konsoli via `wpa_cli` , który to potrafi działać w trybie
interaktywnym i posiada opcję autouzupełniania przy pomocy klawisza TAB.

Problem jednak pojawia się w przypadku gdy interesowałaby nas konfiguracja taka, w której
chcielibyśmy się przełączać dynamicznie między interfejsem przewodowym i tym bezprzewodowym, na
takiej zasadzie, że gdy podpinamy przewód, to sygnał leci przez niego, natomiast w przypadku jego
odłączenia, komunikacja odbywa się przez sieć WiFi. Jest to możliwe do zrealizowania ale w takim
przypadku trzeba by skorzystać z [interfejsów bond][5] ale to jest zagadnienie na zupełnie [inny
artykuł][6].


[1]: https://wiki.archlinux.org/index.php/Wireless_network_configuration#Manual_setup
[2]: https://www.tty1.net/blog/2010/ifconfig-ip-comparison_en.html
[3]: https://wireless.wiki.kernel.org/
[4]: https://pl.wikipedia.org/wiki/ASCII
[5]: https://www.kernel.org/doc/Documentation/networking/bonding.txt
[6]: /post/konfiguracja-interfejsow-bond-bonding/
