---
author: Morfik
categories:
- Linux
date: "2016-03-02T15:14:33Z"
date_gmt: 2016-03-02 14:14:33 +0100
published: true
status: publish
tags:
- firefox
- anonimowość
title: Systemowy klient TOR w TorBrowser
---

[TorBrowser](https://www.torproject.org/projects/torbrowser.html.en) to projekt, który ma na celu
zabezpieczenie użytkownika przed przeciekiem informacji. Jest to połączenie klienta sieci TOR oraz
przeglądarki Firefox (plus kilka dodatków). Ten mechanizm jest tak skonfigurowany, by możliwie jak w
największym stopniu dbał o naszą prywatność podczas przeglądania stron internetowych. Kilka lat
wstecz, użytkownicy Firefox'a mogli się zaopatrzyć w addon TorButton. Niemniej jednak, obecnie [ten
dodatek nie jest już rozwijany](https://www.torproject.org/docs/torbutton/index.html.en),
przynajmniej nie jako osobny projekt. Cały ten TorButton został zintegrowany z TorBrowser i nie ma
obecnie sposobu na to, by przeznaczyć jeden profil Firefox'a pod bezpieczne przeglądanie internetu.
Jeśli chcemy mieć taką możliwość, to musimy korzystać z TorBrowser. Nie stanowi to oczywiście
problemu ale jako, że ma on w sobie wbudowanego klienta TOR'a, to uruchamia też pewne procesy, które
mogą okazać się zbędne, zwłaszcza, gdy na swoim linux'ie mamy już systemową instancję TOR'a. W takim
przypadku, przydałoby się wyłączyć tego klienta TOR w TorBrowser, a ruch z przeglądarki przekierować
do systemowego TOR'a i przez ten proces postaramy się przebrnąć w tym wpisie.

<!--more-->
## Systemowa instancja TOR'a i TorBrowser

Zanim zaczniemy cokolwiek robić, trzeba zaznaczyć, że TorBrowser działa praktycznie OOTB i nie
musimy sobie zawracać głowy instalowaniem w debianie pakietu `tor` . Jeśli jednak korzystamy z
pewnych udziwnień, np. [komunikacja z serwerem kluczy GPG odbywa się przez sieć
TOR]({{< baseurl >}}/post/serwer-kluczy-gpg-i-kwestia-prywatnosci/), to ten pakiet mamy już
zainstalowany. Sama usługa TOR'a została też prawdopodobnie już skonfigurowana i działa sobie w tle.
Jeśli zależy nam na oszczędności zasobów systemowych, to musimy wyłączyć tego klienta sieci TOR,
który siedzi w TorBrowser. Na dobrą sprawę wszystkie potrzebne nam instrukcje na temat tego jak to
zrobić są dostępne w paczce z TorBrowser. Pobierzmy ją zatem i wypakujmy. W środku mamy skrypt
startowy `start-tor-browser.desktop` oraz właściwy katalog przeglądarki `Browser/` . W tym katalogu
z kolei jest plik `start-tor-browser` , to tam są te instrukcje.

## Wyłączenie klienta sieci TOR w TorBrowser

Samo wyłączenie klienta sieci TOR w przypadku TorBrowser sprowadza się do edycji kilku opcji w
ustawieniach Firefox'a. Odpalamy zatem tę przeglądarkę i wpisujemy w pasku adresu `about:config` .
Musimy wyszukać dwie frazy: `extensions.torbutton.` oraz `extensions.torlauncher.` . Następnie
zmieniamy wartości poszczególnych opcji:

    # SETTING NAME                          VALUE
    extensions.torbutton.banned_ports       [...],<SocksPort>,<ControlPort>
    extensions.torbutton.block_disk         false
    extensions.torbutton.custom.socks_host  127.0.0.1
    extensions.torbutton.custom.socks_port  <SocksPort>
    extensions.torbutton.inserted_button    true
    extensions.torbutton.launch_warning     false
    extensions.torbutton.loglevel           2
    extensions.torbutton.logmethod          0
    extensions.torbutton.settings_method    custom
    extensions.torbutton.socks_port         <SocksPort>
    extensions.torbutton.use_privoxy        false

    extensions.torlauncher.control_port      <ControlPort>
    extensions.torlauncher.loglevel          2
    extensions.torlauncher.logmethod         0
    extensions.torlauncher.prompt_at_startup false
    extensions.torlauncher.start_tor         false

Wyżej widzimy, że w kilku parametrach został wykorzystany `SocksPort` i `ControlPort` . W ich
miejscu musimy podać porty, na których nasłuchuje systemowa usługa TOR. Domyślne porty to
odpowiednio 9050 i 9051.

Drugą bardzo istotną rzeczą jest skonfigurowanie hasła, które umożliwi przeglądarce podłączenie się
do systemowej usługi TOR. Robimy to w pliku `Browser/start-tor-browser` . Odnajdujemy w nim poniższą
pozycję i ustawiamy hasło dla gniazda kontrolnego (tak, to hasło musi być ujęte w `'" "'` ):

    setControlPortPasswd ${TOR_CONTROL_PASSWD:='"jakies-haslo"'}

Edytujemy teraz konfigurację systemowego TOR'a, która zwykle znajduje się w pliku `/etc/tor/torrc` i
dopisujemy w nim tę poniższą linijkę:

    HashedControlPassword 16:ciag-hex

Nie możemy tutaj jednak wpisać tego samego hasła, które wpisaliśmy do pliku
`Browser/start-tor-browser` . Hasło w pliku `/etc/tor/torrc` musi być przechowywane w postaci
szyfrowanej. Jak zatem uzyskać ten ciąg HEX? Trzeba go wygenerować przy pomocy `tor --hash-password`
, przykładowo:

    $ tor --hash-password jakies-haslo
    16:D948AD0EB6A8C33F6035D7B72A767F617B8A17489AF80A24EC0BD622CF

Uzupełniamy konfigurację i restartujemy usługę TOR'a.

## Test połączenia w TorBrowser

Jeśli wszystkie kroki przeprowadziliśmy jak należy, TorBrowser powinien być w stanie podłączyć się
do lokalnej instancji TOR'a. Sprawdźmy zatem czy tak się w istocie dzieje. Odpalamy przeglądarkę i
przechodzimy na testową stronę `check.torproject.org` . Powinna nam się pojawić zielona cebula
sugerująca nie tylko fakt korzystania z sieci TOR ale także wykorzystywanie przeglądarki TorBrowser:

![]({{< baseurl >}}/img/2016/03/1.torbrowser-konfiguracja-test-tor.png)

W przypadku, gdy korzystamy z [graficznej nakładki
Vidalia](https://pl.wikipedia.org/wiki/Vidalia_%28program%29), to podczas przeglądania stron w
TorBrowser powinniśmy zarejestrować ruch na jego grafie:

![]({{< baseurl >}}/img/2016/03/2.torbrowser-test-vidalia-graf.png)
