---
author: Morfik
categories:
- Android
date:    2016-11-10 15:44:05 +0100
lastmod: 2016-11-10 15:44:05 +0100
published: true
status: publish
tags:
- smartfon
- aplikacje
- xposed
- root
- xda
title: 'XDA Labs: Repozytorium aplikacji i modułów Xposed'
---

Przeglądając sobie forum XDA w poszukiwaniu pewnych informacji natrafiłem na [XDA Labs][1]. Niby
jest to aplikacja mająca na celu poprawę doznań przy przeglądaniu tego forum na urządzeniach
mobilnych ale posiada ona też kilka użytecznych funkcji niekoniecznie związanych bezpośrednio z
interakcją ze stroną xda-developers. Przede wszystkim, mamy tutaj dostęp do repozytorium aplikacji
na Androida, mniej więcej coś jak [F-Droid][2]. Przy pomocy XDA Labs jesteśmy też w stanie w prosty
sposób instalować moduły Xposed. Te powyższe rzeczy sprawiły, że postanowiłem się nieco bliżej
przyjrzeć aplikacji XDA Labs.

<!--more-->
## Instalacja XDA Labs

We wstępie zdążyłem już nadmienić, że XDA Labs umożliwia instalację dodatkowych aplikacji z
pominięciem Google Play. [Zgodnie z regulaminem][3], taki programik nie trafi nigdy do sklepu
Google. W F-Droid również nie znajdziemy XDA Labs. Sama aplikacja jest jeszcze w fazie beta i jej
źródła nie są póki co udostępnione. Z tego co można wyczytać na forum XDA, w planach jest
udostępnienie źródeł na github'ie ale jeszcze nie wiadomo kiedy ten event będzie miał miejsce.
Dlatego też jedynym sposobem na zainstalowanie XDA Labs jest skorzystanie z tego linku:
[https://labs.xda-developers.com/latest][4]. ([wątek na forum XDA][5]).

Po pobraniu pliku `.apk` , wrzucamy go na smartfona. Jako, że jest to aplikacja spoza sklepu Google
Play, to musimy w ustawieniach Androida włączyć możliwość instalowania takich aplikacji:

![](/img/2016/11/001.xda-labs-android-nieznane-xrodla-.png#big)

Następnie przechodzimy do katalogu, gdzie znajduje się plik `.apk` i instalujemy go w standardowy
sposób:

![](/img/2016/11/002.xda-labs-aplikacja-instalacja.png#huge)

Może się wydawać, że instalacja XDA Labs w taki sposób, zwłaszcza wersji beta, jest ździebko
problematyczna pod kątem aktualizacji tego oprogramowania. Jakby nie patrzeć tej aplikacji nie ma w
żadnym repozytorium, a raczej nie chciałoby nam się ciągle pilnować czy deweloperzy wypuścili
nowszą wersję. Na szczęście w XDA Labs jest zaszyty mechanizm aktualizacji OTA, przez co aplikacja
jest zdolna zaktualizować samą siebie. Gdy tylko pojawi się nowsza wersja, zostaniemy o tym fakcie
powiadomieni.

## Wygląd i opcje XDA Labs

XDA Labs powinien już być zainstalowany na naszym smartfonie. Rzućmy zatem okiem na wygląd
interfejsu tej aplikacji i sprawdźmy, jakie opcje jest on nam w stanie udostępnić. Po uruchomieniu
programu pojawi nam się poniższa informacja:

![](/img/2016/11/003.xda-labs-aplikacja-platnosci.png#medium)

Warto tutaj wspomnieć, że w przeciwieństwie do Google Play, na aplikacje udostępniane w repozytorium
XDA nie są nakładane żadne prowizje w przypadku, gdy jakiś deweloper ma ochotę je sprzedawać. Sam
Google pobiera 30%, a do tego dochodzą również jeszcze inne podatki jak VAT, przez co sprzedając
aplikację w Google Play, tylko niewielka część kasy trafia do twórcy samego programu. W przypadku
XDA sprawa finansowania twórców zdaje się wyglądać nieco lepiej, no i są obsługiwane płatności przez
PayPal i Bitcoin.

Interfejs XDA Labs jest prosty i przejrzysty. Pod lewą krawędzią ekranu jest ukryte menu, które
umożliwia nam min. odwiedzenie forum XDA:

![](/img/2016/11/004.xda-labs-aplikacja-interfejs.png#big)

Jeśli zaś chodzi o opcje, to mamy do dyspozycji te poniższe:

![](/img/2016/11/005.xda-labs-aplikacja-opcje.png#huge)

### Forum XDA

Moduł do przeglądania forum XDA jest podobny do aplikacji TapTalk, z tym, że przeglądając forum
przez XDA Labs nie będą nam serwowane żadne reklamy, bo aplikacja ich zwyczajnie nie zawiera:

![](/img/2016/11/006.xda-labs-aplikacja-forum.png#huge)

### Repozytorium aplikacji XDA

W przypadku F-Droid'a, w repozytorium były dostępne jedynie aplikacje Free i OpenSource. Natomiast
jeśli chodzi o repozytorium XDA, to aplikacje tutaj niekoniecznie muszą być darmowe i o otwartym
kodzie źródłowym. Poza tym, za sprawą tego repozytorium mamy dostęp do wersji alfa i beta aplikacji.
Niekoniecznie musimy instalować te najświeższe wydania ale dla tych którzy mają taką ochotę, to
możliwość zawsze jest. To co niewątpliwie odróżnia repozytorium XDA od F-Droid, to możliwość
wystawiania opinii i ocen używanym przez nas aplikacjom.

![](/img/2016/11/007.xda-labs-aplikacja-repozytorium.png#huge)

Wszystkie aplikacje wymagające opłat są stosownie oznaczane przy pomocy znaczka `$` w prawym górnym
rogu na kafelku. Zabrakło jednak wyraźnego oznaczania aplikacji otwartoźródłowych. Wszystkie
aplikacje, które mają swój wątek na forum XDA, mają również bezpośredni link do takiego wątku:

![](/img/2016/11/008.xda-labs-aplikacja-repozytorium-forum-link.png#medium)

Jeśli z jakiegoś powodu nie chcemy, aby XDA Labs próbował aktualizować określone aplikacje, to
zawsze możemy je wyłączyć spod działania tego mechanizmu. Wystarczy w menu aplikacji przycisnąć to
przekreślone kółko, co spowoduje pojawienie się listy zainstalowanych aplikacji, które są widziane
przez XDA Labs.

![](/img/2016/11/009.xda-labs-aplikacja-aktualizacje.png#medium)

### Moduły Xposed

Moduły Xposed są w stanie modyfikować ustawienia systemowe smartfona bez ingerowania w jego ROM.
Niestety, by móc korzystać z tych modułów, [potrzebny nam jest odpowiedni framework][6], a ten z
kolei już wymaga ukorzenionego Androida (root). Gdy nasz telefon ma root, to możliwość łatwego
wgrania w systemie modułów Xposed bardzo nam się przyda.

![](/img/2016/11/010.xda-labs-aplikacja-repozytorium-xposed.png#big)

Warto tutaj zaznaczyć, że moduły Xposed mogą być przeglądane, pobierane i instalowane bez root ale
nie damy rady ich włączyć i z nich korzystać na Androidzie, który nie ma root'a.

### Tapety

Ostatnią rzeczą, którą umożliwia nam XDA Labs, to ustawienie tapety.

![](/img/2016/11/011.xda-labs-aplikacja-tapety.png#medium)


[1]: https://forum.xda-developers.com/android/apps-games/labs-t3241866
[2]: /post/android-repozytorium-aplikacji-opensource-f-droid/
[3]: https://play.google.com/about/developer-distribution-agreement.html
[4]: https://labs.xda-developers.com/latest/
[5]: https://www.xda-developers.com/xda-labs/
[6]: http://repo.xposed.info/module/de.robv.android.xposed.installer
