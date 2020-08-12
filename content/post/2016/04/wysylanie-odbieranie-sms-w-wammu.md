---
author: Morfik
categories:
- Linux
date: "2016-04-11T20:19:26Z"
date_gmt: 2016-04-11 18:19:26 +0200
published: true
status: publish
tags:
- debian
- sms
- modem
- huawei
- e3372
title: Wysyłanie i odbieranie SMS w wammu
---

[Wammu](https://wammu.eu/) to aplikacja, przy pomocy której jesteśmy w stanie zarządzać swoim
telefonem komórkowym. Można ją także wykorzystać do zarządzania modemami USB, tymi samymi, które są
w stanie nam dostarczyć połączenie LTE. Przy pomocy `wammu` nie damy rady jednak nawiązać połączenia
internetowego ale jest kilka rzeczy, do których ten soft może nam się przydać. Karta SIM obecna w
takim modemie może mieć zapisane kontakty, które możemy edytować, usuwać i ewentualnie dodawać nowe.
Ważniejszym ficzerem, który oferuje `wammu` , jest możliwość wysyłania i odbierania wiadomości SMS.
Wcześniej opisywałem [wysyłanie i odbieranie SMS za pomocą
gammu-smsd]({{< baseurl >}}/post/gammu-smsd-czyli-wysylanie-odbieranie-sms/), niemniej jednak, w
przypadku `wammu` nie będziemy uruchamiać żadnej usługi systemowej. Same wiadomości SMS odbiera i
wysyła się na podobnej zasadzie co w telefonie komórkowym. Przyjrzymy się zatem bliżej temu
kawałkowi oprogramowania.

<!--more-->
## Instalacja i wstępna konfiguracja wammu

W repozytoriach debiana mamy dostępny pakiet `wammu` . Instalujemy go zatem i podłączamy do portu
USB modem lub telefon. W tym przypadku będzie to modem LTE Huawei E3372 w wersji NON-HiLink ale może
to być dowolne urządzenie, które udostępnia interfejsy komunikacyjne w katalogu `/dev/` . Jeśli mamy
kilka urządzeń, które rejestrują swoje interfejsy pod `/dev/ttyUSB*` , [to przydałoby się ogarnąć
pierw te nazwy]({{< baseurl >}}/post/zmiana-nazwy-interfejsu-modemu-ttyusb0/). Trzeba także wziąć
pod uwagę, że `wammu` zajmuje jeden z tych interfejsów dla siebie i jeśli system wykrył tylko jeden
interfejs, to prawdopodobnie inne usługi czy demony nie będą w stanie korzystać z urządza,
przynajmniej do czasu zamknięcia aplikacji.

Odpalmy zatem `wammu` i dodajmy przykładowe połączenie z modemem czy telefonem. By to zrobić
uruchamiamy konfigurator połączenia (Wammu -\> Phone wizard). Powinno nam się wyświetlić to poniższe
okienko:

![]({{< baseurl >}}/img/2016/04/1.wammu-kreator-polaczenia.png)

Z godnie z instrukcjami na ekranie, upewniamy się, że mamy podłączony modem lub telefon. Aplikacja
`wammu` potrafi nawiązać połączenie przez bluetooth, IrDA (Infrared Data Association) i zwykły kabel
USB. Przechodzimy dalej i wybieramy w jaki sposób chcemy skonfigurować to urządzenie:

![]({{< baseurl >}}/img/2016/04/2.wammu-kreator-polaczenia.png)

Mamy tam trzy opcje do wyboru: przewodnik, automat i tryb manualny. Automat powinien wykryć
praktycznie większość sprzętu i jest zalecany dla początkujących użytkowników. Przewodnik daje nam
szersze pole manewru, to tak na wypadek jeśli chcemy skonfigurować kilka rzeczy ręcznie. Tryb ręczny
jest dla osób, które znają się na rzeczy. My oczywiście skorzystamy z tej ostatniej opcji.

Na sam początek wskazujemy interfejs oraz typ połączenia. Urządzenie, które powinniśmy tutaj wskazać
zwykle będzie miało postać `/dev/ttyUSB0` . W tym przypadku jest inaczej ale to z przyczyn opisanych
powyżej. Natomiast typ połączenia zależy głównie od urządzenia. Jako, że tutaj mamy modem LTE, to
ten obsługuje polecenia AT. Liczba wskazuje na prędkość transmisji danych podczas komunikacji z
modemem. Jeśli nie jesteśmy pewni co do samej prędkości, to wybierzmy po prostu `AT` :

![]({{< baseurl >}}/img/2016/04/3.wammu-kreator-polaczenia.png)

Po chwili zostanie przeprowadzony test połączenia z modemem:

![]({{< baseurl >}}/img/2016/04/4.wammu-kreator-polaczenia.png)

Jak widzimy, urządzenie zostało rozpoznane poprawnie. Możemy zatem przejść do ostatniego kroku, tj.
zapisania konfiguracji:

![]({{< baseurl >}}/img/2016/04/5.wammu-kreator-polaczenia.png)

Parametry konfiguracji zostały zapisane w pliku `~/.gammurc` . To nie jest jednak cała konfiguracja.
W menu `wammu` mamy jeszcze pozycję z ustawieniami (Settings) i na nią też powinniśmy rzucić okiem:

![]({{< baseurl >}}/img/2016/04/5.wammu-ustawienia.png)

Tutaj możemy skonfigurować min. zachowanie samej aplikacji. Te ustawienia są z kolei przechowywane w
pliku `~/.Wammu` .

## Nawiązywanie połączenia z modemem przez wammu

Mając skonfigurowane połączenie, nawiążmy je (Phone -\> Connect). Po chwili połączenie powinno
zostać ustanowione. Sprawdźmy z jakim urządzeniem mamy do czynienia (Retrieve -\> Info):

![]({{< baseurl >}}/img/2016/04/6.wammu-info-modem.png)

Być może przy pobieraniu informacji o urządzeniu napotkamy jakiś błąd, w efekcie którego nie
wszystkie informacje o urządzeniu zostaną nam wyświetlone. Ten błąd nie wpływa jednak na samą
interakcję z urządzeniem.

## Kontakty w wammu

Jako, że w tym przypadku mamy do czynienia z modemem LTE, to część funkcji `wammu` nie działa.
Możemy jednak pobrać kontakty z karty SIM (Retrieve -\> Contacts (SIM)):

![]({{< baseurl >}}/img/2016/04/7.wammu-kontakty-sim.png)

Te powyższe są domyślne dla tego konkretnego startera. Nic jednak nie stoi na przeszkodzie by dodać
nowe kontakty. Limitowani jesteśmy jednak przez pojemność karty SIM. Tak czy inaczej. Mając
zdefiniowany numer telefonu jakiejś osoby, możemy kliknąć na tym kontakcie prawym przyciskiem myszy
i wysłać jej SMS.

## Wysyłanie i odbieranie SMS w wammu

Po zaznaczeniu kontaktu i wybraniu opcji "Send Message" powinno nam pojawić się to poniższe okienko:

![]({{< baseurl >}}/img/2016/04/8.wammu-wiadomosc-sms.png)

Teraz pozostaje nam już tylko napisać kilka słów i wysłać wiadomość. Tego SMS'a możemy także zapisać
sobie. Z kolei by sprawdzić odebrane wiadomości SMS, wybieramy z menu Retrieve -\> Messages. Po
chwili wszystkie wiadomości powinny zostać nam zaprezentowane:

![]({{< baseurl >}}/img/2016/04/9.wammu-odebrane-wiadomosci-sms.png)

Wiadomości SMS są przez `wammu` stosownie oznaczane i umieszczane w odpowiednich katalogach. Na
każdą z nich możemy w łatwy i szybki sposób odpowiedzieć. Możemy też bez problemu hurtowo skasować
zalegające wiadomości. No i chyba najważniejsze, nie musimy przekładać karty SIM do telefonu, by
sprawdzić czy ktoś nam czasem nie wysłał jakiegoś SMS'a. Niemniej jednak, `wammu` nie monitoruje
statusu nadsyłanych wiadomości i trzeba telefon odświeżać ręcznie.
