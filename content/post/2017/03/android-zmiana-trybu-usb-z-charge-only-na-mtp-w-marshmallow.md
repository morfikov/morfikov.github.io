---
author: Morfik
categories:
- Android
date: "2017-03-05T18:25:13Z"
date_gmt: 2017-03-05 17:25:13 +0100
published: true
status: publish
tags:
- smartfon
- mtp
- marshmallow
title: 'Android: Zmiana trybu USB z Charge-Only na MTP w Marshmallow'
---

System Android w większej lub mniejszej części zmienia się z wydania na wydanie. Te nowsze wersje
zwykle zawierają całą masę nowych mechanizmów i rozbudowują te już istniejące, tak by ten OS w
lepszym stopniu zaspokajał zachcianki użytkowników smartfonów. Problem w tym, że niektóre kroki
deweloperów Androida potrafią wprawić w zastanowienie niejednego logicznie myślącego osobnika.
Przykładem może być przestawienie domyślnego trybu USB w Marshmallow z MTP na Charge-Only (tylko
ładowanie). Jedni mówią, że takie posunięcie jest podyktowane względami bezpieczeństwa, a inni, że
chodzi o performance przy ładowaniu baterii, gdzie moduł USB nie działa w tym drugim trybie i nie
konsumuje energii, przez co ładowanie ma przebiegać szybciej. Ile w tym prawdy, tego nie wiem ale ja
za bardzo nie widzę żadnych wymiernych korzyści z przestawienia tego trybu na Charge-Only. Natomiast
widzę bardzo wyraźnie utrudnienia przy interakcji telefonu z komputerem za sprawą tej zmiany.
Poszukałem trochę informacji na ten temat i znalazłem rozwiązanie w postaci aplikacji MTP enabler.

<!--more-->
## Domyślny tryb USB w Marshmallow i Lollipop

W Androidzie 5.1 Lollipop nie było żadnego problemu. Podłączało się smartfon do komputera i można
było wymieniać dane na linii tych dwóch urządzeń. Natomiast w Androidzie 6.0 Marshmallow trzeba
dodatkowo [ręcznie przestawić tryb
USB](https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-usb)
ilekroć podłączamy telefon do portu USB. Ta czynność jest mało wygodna, gdy się często podłącza
telefon do komputera i takie ciągłe przestawianie tego trybu może nawet spokojnego człowieka
wyprowadzić z równowagi:

![]({{< baseurl >}}/img/2017/03/001.usb-charge-only-mtp-tryb-android-marshmallow-domyślne.png#big)

## Konfiguracja trybu USB w opcjach programistycznych

Gdzieniegdzie można się spotkać z opinią, że domyślny tryb USB w Marshmallow można trwale przestawić
w opcjach programistycznych. No i faktycznie stosowna opcja jest tam obecna:

![]({{< baseurl >}}/img/2017/03/002.usb-charge-only-mtp-tryb-android-marshmallow-opcje-dev.png#huge)

Jeśli teraz byśmy podłączyli smartfon do komputera, to naturalnie zostanie ustawiony tryb przesyłu
plików, a nie ładowania. To czego ludzie zapominają zrobić, to przetestowanie trwałości tej
konfiguracji przez proste odłączenie urządzenia od komputera i ponowne jego podłączenie do portu
USB. W takim przypadku z niewiadomych przyczyn system wraca do trybu ładowania i musimy ręcznie go
przestawić na przesył plików. Nie wiem, która opcja jest wygodniejsza.

## Root Androida i MTP enabler

Niestety próżno szukać w ustawieniach Androida opcji, która mogłyby nam umożliwić wybranie
domyślnego trybu USB. Dlaczego? O to już trzeba zapytać deweloperów tego systemu. W efekcie mamy do
wyboru jeden z dwóch scenariuszy. Opcja pierwsza to zaakceptowanie domyślnego trybu, który został
ustawiony w Marshmallow. Drugim rozwiązaniem jest naturalnie próba obejścia tego całego mechanizmu i
zdefiniowanie sobie takiego trybu USB, który najlepiej nam odpowiada.

Stosowna aplikacja, która umożliwi nam swobodne operowanie na porcie USB w naszym telefonie,
niestety wymaga praw administratora root, czyli musimy posiadać ukorzenionego Androida. Innej opcji
o zgrozo nie ma. Aplikacja, o której mowa, nazywa się [MTP enabler, a link do jej darmowej wersji
widnieje na forum
XDA](https://forum.xda-developers.com/android/apps-games/app-mtp-enbaler-t3263467). Instalacja tej
aplikacji raczej nie powinna sprawić żadnych problemów. Po zainstalowaniu tego programiku,
uruchamiamy go. Tak prezentują się opcje tej aplikacji:

![]({{< baseurl >}}/img/2017/03/003.usb-charge-only-mtp-tryb-android-marshmallow-aplikacja.png#huge)

Jak widać jest ich dość sporo. Ta najważniejsza opcja wyboru domyślnego trybu jest naturalnie
dostępna. Wybranie tutaj MTP zamiast Charging zaowocuje przestawieniem trybu domyślnego i ilekroć
tylko będziemy podłączać smartfon do portu USB, to ten tryb będzie automatycznie aplikowany. Jeśli
chcemy, by przy każdym podłączeniu smartfona do komputera pojawiała nam się notyfikacja z opcjami
wyboru konkretnego trybu (tak jak to widać na pierwszej fotce wyżej), to również mamy taką
możliwość.

Tryb przesyłu plików (MTP) można także sobie zabezpieczyć w oparciu o znany ESSID sieci
bezprzewodowej WiFi. W przypadku podłączenia smartfona do komputera przy braku WiFi lub też będąc
podłączonym do innej sieci, aplikacja MTP enabler załączy nam jedynie tryb ładowania. Nie mogło
także zabraknąć notyfikacji dźwiękowych.

## Czy przestawienie domyślnego trybu USB jest bezpieczne

Naprawdę ciężko mi jest zrozumieć dlaczego przestawienie domyślnego trybu USB z MTP na Charge-Only
ma miejsce w Androidzie Marshmallow. Wątpię, by ktoś odczuł jakąś różnicę przy ładowaniu smartfona
przez port USB komputera za sprawą tej zmiany. Natomiast jeśli chodzi o kwestię bezpieczeństwa, to i
tak przecież hasło blokady ekranu uniemożliwia zamontowanie zasobów smartfona w systemie operacyjnym
komputera, przez co mając domyślny tryb MTP i tak trzeba wpisać hasło, by móc cokolwiek ze
smartfonem robić. Może ktoś dysponuje argumentami, które przemawiają za przestawieniem trybu USB na
Charge-Only? Bardzo chętnie bym się z nimi zapoznał.
