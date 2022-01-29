---
author: Morfik
categories:
- OpenWRT
date:    2022-01-29 17:12:00 +0100
lastmod: 2022-01-29 17:12:00 +0100
published: true
status: publish
tags:
- wifi
- prywatność
- lokalizacja
- gps
- router
- aplikacje
- f-droid
GHissueID: 585
title: Jak ukryć nazwę sieci WiFi (ESSID) przed Wigle.net/Google/Mozilla
---

Parę dni temu napisał do mnie megabajt z forum DUG z wiadomością, że nazwa mojego WiFi figuruje w
bazie danych serwisu [Wigle.net][1] oraz, że lepiej nie zdradzać nazw swoich WiFi osobom trzecim,
bo ktoś w oparciu o te dane może poznać nasze fizyczne położenie. Jak najbardziej istnieje taka
możliwość, a sam serwis Wigle.net nie jest jedynym, który zbiera informacje o położeniach
geograficznych punktów dostępowych WiFi, bo od lat robi to też Google i Mozilla. Niemniej jednak,
podchodziłbym z pewną dozą ostrożności do danych zgromadzonych w serwisie Wigle.net (i tych
pozostałych dwóch też), bo nie zawsze muszą one być akuratne. W zasadzie to nie zdziwiłem się
zbytnio, że nazwa mojej sieci WiFi figuruje w bazie danych Wigle.net, bo sam ją tam umieściłem
świadomie jakiś czas temu w ramach testów wykorzystując do tego celu aplikację [WiGLE WiFi
Wardriving][2], którą można wgrać przez F-Droid na każdy telefon z Androidem. Jako, że nadarzyła się
okazja, to postanowiłem napisać parę słów na temat unikania bycia zmapowanym, tak by jakiś nieznany
nam bliżej osobnik przez przypadek nie wrzucił lokalizacji naszych domowych/firmowych AP do bazy
danych Google/Mozilla czy też wspomnianego wyżej serwisu Wigle.net, o ile oczywiście sobie tego nie
życzymy.

<!--more-->
## Nazwy punktów dostępowych, a prywatność

Sporo osób zapewne zdaje sobie sprawę, że by poznać czyjąś lokalizację, zwykle wystarczy odczyt
danych z odbiornika GPS. Jeśli teraz pochwalimy się tymi wskazaniami GPS, np. za sprawą [metadanych
EXIF obecnych w zrobionych przez nas fotkach][5] (zakładając, że nie [pozorujemy lokalizacji][6]),
to każdy, kto wejdzie w posiadanie tych informacji, będzie w stanie ustalić gdzie dana osoba się
znajdowała w momencie zrobienia takiego zdjęcia. Jeśli zaś te fotki zostaną wykonane w domu, to
wtedy bez problemu będzie można również ustalić gdzie ten osobnik mieszka.

Z AP w sieciach WiFi jest bardzo podobnie. Dla ułatwienia sobie życia każdy z nas praktycznie
codziennie korzysta ze swojego domowego WiFi zapominając czasem, że sygnał rozgłaszany przez router
WiFi wychodzi również poza ściany naszego domu. Jeśli teraz ktoś przejdzie się po okolicy ze
smartfonem lub innym urządzeniem będącym w stanie wyłapać fale radiowe z zakresu 2,4 GHz i 5 GHz,
to bez większego problemu zmapuje on okolicę i wykryje w niej wszystkie takie nadajniki wraz z
nazwami sieci (ESSID) i ich adresami MAC (BSSID). Jeśli dodatkowo będzie miał ten osobnik włączony
GPS, to lokalizacja takiego AP zostanie z dość dużą dokładnością określona.

Punkty dostępu mają tę wadę, że praktycznie wcale nie zmieniają swojego położenia w czasie. Raz
odpalony i skonfigurowany router WiFi będzie zwykle pracował w tym konkretnym miejscu, aż któregoś
dnia się popsuje i już się więcej nie uruchomi (albo ze starości zostanie wymieniony na nowy).
Jeśli dajmy na to teraz dana osoba napisze posta na forum i będzie szukać pomocy przy konfiguracji
sieci WiFi i wrzuci log systemowy, w którym ESSID i/lub BSSID będą widoczne, to ci bardziej sprytni
użytkownicy będą w stanie poznać lokalizację tej osoby wykorzystując do tego celu publiczne bazy
danych AP (np. Wigle.net). Wciąż trzeba pamiętać o tym, że w tych bardziej zagęszczonych miastach
(zwłaszcza na blokowiskach), informacja o lokalizacji punktu dostępowego WiFi na niewiele nam się
zda. Niemniej jednak, w przypadku chaty ulokowanej gdzieś pod lasem, ta sama informacja już
jednoznacznie może określić, gdzie ktoś przebywa czy mieszka.

Problem z bazami danych typu Wigle.net jest taki, że dane zawarte w tych bazach nie zawsze oddają
stan faktyczny. Dla pewności zapytałem zespół Wigle.net, o czas jaki dany wpis może przebywać w
bazie danych. Odpowiedź jest poniżej:

> Any new-to-the-project network is checked, and if the ssid is set to _nomap or _optout than it
> is made not available. We periodically check for ssid's that may have switched and set those as
> well. There isn't an expiration date, but most searches include date ranges.

Zatem wpisy dodawane do bazy danych nie mają żadnego terminu ważności i nikt nie usuwa z tej bazy
punktów, które zostały odłączone już lata (albo i dekady) temu. W ten sposób lokalizacja sporej
części AP może być przestarzała i nie do końca oddawać stan aktualny.

## Jak uniknąć mapowania WiFi

W tej opisanej wyżej sytuacji, dane o AP trafiły do bazy danych w sposób świadomy i dobrowolny. Nie
zawsze jednak tak być musi. Jeśli w pobliżu by się znalazł inny użytkownik, który by korzystał z
aplikacji WiGLE WiFi Wardriving, to on też będzie w stanie wyłapać naszą sieć WiFi. Jeśli prześle
wyniki do bazy danych, to stosowny wpis będzie w niej figurował, a każdy chętny będzie w stanie
poznać nasze przybliżone położenie. Przynajmniej takie wnioski się nasuwają, choć nie do końca są
one prawdą.

### Sufiks _nomap lub _optout w nazwie sieci WiFi (ESSID)

By wpis do bazy danych Wigle.net trafił, musi być spełniony jeden warunek. Nazwa sieci (ESSID) nie
może zawierać sufiksu `_nomap` lub `_optout` . Jeśli nasza domowa sieć WiFi posiada w nazwie
`_nomap` lub `_optout` , to taki wpis zostanie odfiltrowany i nie będzie uwzględniony na mapie.
Zatem jest bez znaczenia kto skanowanie za punktami dostępowymi będzie przeprowadzał i jak często,
bo i tak nasz AP nigdy w bazie się nie pojawi. Warto tutaj też zaznaczyć, że nie tylko serwis
Wigle.net filtruje AP mające w nazwie `_nomap` . Punkty dostępowe WiFi mające `_nomap` nie są także
dodawane do bazy danych [Google][3] i [Mozilla][4].

### Ukrycie sieci WiFi

Jeśli zmiana nazwy sieci WiFi nie jest dla nas rozwiązaniem, to istnieje także opcja z [ukryciem
nazwy sieci][8], co powinno sprawić, że serwisy mapujące AP nie powinny dodawać takiego ukrytego
punktu dostępowego do swoich baz danych. Trzeba tutaj jednak wyraźnie zaznaczyć, że "nie powinny"
nie oznacza, że "nie mogą".

Standardowo w konfiguracji sieci WiFi na routerze mamy opcję ukrycia sieci (Hide ESSID). W OpenWRT
wystarczy w pliku `/etc/config/wireless` dopisać parametr `hidden '1'` w zwrotkach mających
`config wifi-iface` , przykładowo:

    config wifi-iface 'default_radio1'
        ...
        option hidden '1'
        ...

## Co robić, gdy nasz AP figuruje w bazie Wigle.net/Google/Mozilla

Odwiedzając serwis Wigle.net i podając w nim ESSID i/lub BSSID, możemy zweryfikować czy AP naszej
domowej sieci WiFi został zmapowany. Jeśli tak, i chcielibyśmy usunąć te dane, to wystarczy zmienić
nazwę, tak by zawierała ona sufiks `_nomap` lub `_optout` . Przy następnym skanowaniu (i wysłaniu
tych danych do bazy Wigle.net), informacje o naszym AP powinny zniknąć z tej baz danych. Jeśli
zależy nam na tym, by taka sytuacja się w przyszłości nie powtórzyła, tj. by nasz AP nie trafił
ponownie do bazy danych, to ten sufiks `_nomap` lub `_optout` musi w nazwie sieci pozostać na stałe.

Jeśli zaś chodzi o bazy danych Google/Mozilla, to nie wiem czy w oparciu o nie zwykły użytkownik da
radę ustalić położenie jakiegoś konkretnego AP, tak jak to ma miejsce w przypadku serwisu
Wigle.net. Wygląda na to, że usługi lokalizacji Google/Mozilla zbierają co prawda informacje o
lokalizacji poszczególnych punktów dostępowych WiFi i jak najbardziej przechowują te dane w swoich
bazach ale to położenie, które później jest zwracane (np. w przypadku braku GPS), [nie jest
obliczane na podstawie tylko jednego konkretnego AP][7]. Potrzebne są co najmniej dwa punkty
dostępowe, a najlepiej to trzy i więcej, by pomiar był możliwie dokładny. W przypadku, gdy
zdecydujemy się dopisać sufiks `_nomap` do nazwy sieci WiFi, to taki AP przez te usługi
lokalizacyjne nie będzie brany pod uwagę i nie powinien też trafić do tych baz danych, a jeśli tam
się znalazł, to zostanie usunięty.

## Podsumowanie

W pewnych sytuacjach faktycznie lepiej jest nie ujawniać na publicznych forach informacji o
ESSID/BSSID punktów dostępowych WiFi, z których korzystamy na co dzień, zwłaszcza gdy nie mieszkamy
w zatłoczonych miastach. Takie z pozoru niegroźne informacje mogą bez większego problemu przyczynić
się do poznania dokładnej lokalizacji naszego domu, co w dzisiejszych czasach zwykle nie wróży nic
dobrego. Nie ma też co popadać w rozpacz, bo nawet jeśli nasz AP zostanie pomyłkowo (albo i celowo)
zmapowany, to bez większego problemu możemy go z tych baz danych usunąć. Niemniej jednak trzeba też
liczyć się z firmami komercyjnymi, które niekoniecznie mogą respektować sufiks `_nomap`/`_optout`
czy nawet ukrywanie sieci i będą mapować punkty dostępu pomimo naszego wyraźnego sprzeciwu.


[1]: https://wigle.net/
[2]: https://f-droid.org/en/packages/net.wigle.wigleandroid/
[3]: https://support.google.com/maps/answer/1725632
[4]: https://location.services.mozilla.com/optout
[5]: /post/metadane-plikow-graficznych-exif/
[6]: /post/pozorowanie-lokalizacji-gps-w-androidzie-mock-location/
[7]: https://developers.google.com/maps/documentation/geolocation/overview
[8]: https://en.wikipedia.org/wiki/Network_cloaking
