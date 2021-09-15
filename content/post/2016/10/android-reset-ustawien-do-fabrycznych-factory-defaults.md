---
author: Morfik
categories:
- Android
date:    2016-10-04 20:39:20 +0200
lastmod: 2016-10-04 20:39:20 +0200
published: true
status: publish
tags:
- smartfon
- factory-reset
title: Android: Reset ustawień do fabrycznych (factory defaults)
---

Android zadomowił się w większości smartfonów. Jest to mniej więcej taki sam system operacyjny, co w
przypadku stacjonarnych komputerów czy laptopów. W obu przypadkach instalujemy i konfigurujemy różne
rzeczy, tak by ten system oraz jego aplikacje były dostosowane do naszych potrzeb w sposób
optymalny. Niemniej jednak, każdy system wymaga odpowiedniego traktowania i mam na myśli tutaj
regularne czyszczenie go ze zbędnych aplikacji czy plików, które nie są nam już do niczego
potrzebne. W przypadku desktopowych systemów zwykliśmy wgrywać je na nowo za pomocą płytki CD/DVD
czy też pendrive live. Ten proces z reguły jest szybki i w miarę bezproblemowy. Później trzeba tylko
trochę czasu poświęcić, by ten świeży system skonfigurować. A jak sprawa wygląda w przypadku
smartfona z Androidem? Nie mamy jak za bardzo wgrać świeżego systemu, no i na dobrą sprawę nie
musimy. Zamiast tego, możemy przywrócić ustawienia do fabrycznych (factory reset). W tym artykule
postaramy się obadać proces resetu ustawień telefonu na przykładzie [smartfona Neffos C5][1] od
TP-LINK z Androidem 5.1 (Lollipop).

<!--more-->
## Co zostanie wyczyszczone podczas resetu

Musimy sobie zdawać sprawę, że wszystkie dane, które mamy na flash'u smartfona są tak naprawdę
podzielone na kilka sekcji. Z grubsza rzecz ujmując, taki flash ma szereg partycji: `/system/`,
`/boot/` , `/data/` , `/cache/` oraz czasami też `/recovery/` . Na partycjach `/system/` i `/boot/`
siedzi sobie system Android. Te dwie partycje są tylko do odczytu, za wyjątkiem aktualizacji systemu
oficjalną drogą oferowaną przez producenta sprzętu. Dane na partycjach `/data/` oraz `/cache/` mogą
być zaś zmieniane przez użytkownika.

Wszystko co wgrywamy na telefon, wędruje na partycję `/data/`. Każdy program generuje także jakieś
dane na partycji `/cache/` . To te dwie partycje zostaną wyczyszczone podczas resetowania telefonu.
Dane na karcie SD/SIM pozostają nietknięte. Musimy zatem zgrać sobie fotki z kamery, zapisać
kontakty, czy też zsynchronizować konto Google/YouTube, tak by po restarcie ustawień smartfona, te
dane nam nie przepadły.

Przy resecie ustawień do fabrycznych zostaniemy powiadomieni jakie dane przepadają:

![](/img/2016/10/1.android-factory-reset-defaults-backup.png#huge)

## Zaszyfrowanie zawartości telefonu

Proces resetowania Androida do ustawień fabrycznych nie koniecznie musimy rozpocząć w momencie, gdy
mamy już straszny syf w systemie. Innym przypadkiem są względy bezpieczeństwa. Być może
kontaktowaliśmy się kiedyś ze Snowden'em i teraz ktoś przy pomocy naszego telefonu będzie chciał
wydobyć szczegóły tamtej rozmowy czy rozmów. Jeszcze inną sytuacją może być próba
odsprzedania/odstąpienia telefonu komuś trzeciemu. Raczej nie chcielibyśmy, by taka osoba była w
stanie odzyskać jakieś dane z naszego telefonu, prawda?

Bez uprzedniego zaszyfrowania smartfona może i wyczyścilibyśmy wszelkie dane na nim się znajdujące
ale nie znikną one od razu w sposób automagiczny z fizycznego flash'a telefonu. Takie dane można
odzyskać mimo, że system nie ma informacji co do ich dokładnego położenia w systemie plików. Dlatego
też przed factory reset przydałoby się pierw zaszyfrować telefon. Mając zaszyfrowany telefon, trzeba
flash wypełnić danymi, tak by nie został wolny ani jeden bit. Dopiero wtedy możemy zresetować
telefon do ustawień fabrycznych wiedząc, że na flash'u pozostaną jedynie skrawki zaszyfrowanych
bitów, z których nikt nic nigdy nie wyciągnie.

Szyfrowanie możemy przeprowadzić przechodząc w menu Ustawienia => Zabezpieczenia.

![](/img/2016/10/2.android-factory-reset-defaults-szyfrowanie-danych.png#huge)

Potwierdzamy proces zaszyfrowania danych i po chwili nasz telefon powinien się uruchomić ponownie.
Pojawi się też ekran z zielonym robocikiem oraz informacją o szacowanym czasie jaki pozostał do
zaszyfrowania telefonu. W moim przypadku czas na wyświetlaczu pokazał około 2 minut. Nie mam za
wiele danych na telefonie zatem u mnie ten proces trwał krótko. Po zaszyfrowaniu, smartfon powinien
się automatycznie uruchomić ponownie.

Teraz możemy zacząć zapisywać flash tak, by ilość wolnego miejsca na nim skurczyła się do 0:

![](/img/2016/10/3.android-factory-reset-defaults-szyfrowanie-danych.png#medium)

## Resetowanie ustawień

Teraz możemy przejść do czyszczenia danych i ustawień. Ponownie przechodzimy w miejsce, gdzie była
informacja na temat backupu danych, tj. w Ustawienia => Kopia i kasowanie danych => Ustawienia
fabryczne. Tam z kolei klikamy już tylko przycisk `Resetuj telefon` . Zostaniemy jeszcze poproszeni
o potwierdzenie tego kroku:

![](/img/2016/10/5.android-factory-reset-defaults.png#medium)

Potwierdzamy i telefon powinien się uruchomić ponownie, gdzie przez chwilę będziemy mogli obejrzeć
zielonego robocika. Po paru sekundach smartfon się znów zrestartuje, tym razem już ze świeżym
systemem i z domyślnymi ustawieniami. Proces startu może dłuższą chwilę potrwać, po czym załaduje
się nam ekran z wyborem języka, czyli dokładnie ten sam co tuż po wyjęciu telefonu z pudełka.

## Factory reset przez tryb recovery

Android posiada także tryb recovery, który może się nam przydać w momencie utraty kontroli nad
systemem. Krótko mówiąc, gdy nie możemy uruchomić z jakiegoś powodu systemu. W trybie recovery mamy
kilka użytecznych opcji, min. [resetowanie Androida do ustawień fabrycznych][2]. Przez ten tryb
recovery nie damy rady zaszyfrować telefonu ale w przypadku braku możliwości jego uruchomienia, to
raczej by było nasze najmniejsze zmartwienie.

W tryb revocery na Androidzie wchodzimy w następujący sposób. Musimy pierw wyłączyć telefon.
Następnie wciskamy i przytrzymujemy przycisk Volume Up oraz Power do momentu aż telefon się włączy.
Po chwili na ekranie powinny pojawić się nam trzy opcje: recovery mode, fastboot mode i normal boot.
Przyciskiem Volume Up przełączamy się między tymi trybami, a Volume Down aktywuje wybraną pozycję.
Musimy zaznaczy i wybrać `recovery mode` .

Naszym oczom powinien ukazać się przewrócony zielony robot z czerwonym znakiem `!` oraz podpisem
`brak polecenia` . W tym momencie przyciskamy i trzymamy przycisk Power i przyciskamy także przycisk
Volume Up i puszczamy. I tak weszliśmy w tryb recovery. Tutaj z kolei przy pomocy Volume Down
zaznaczy pozycję `wipe data/factory reset` i potwierdzamy przyciskiem Power. Po tym kroku zacznie
się czyszczenie danych, po którym prawdopodobnie odzyskamy kontrolę nad Androidem.


[1]: http://www.neffos.pl/product/details/C5
[2]: https://support.google.com/android-one/answer/6088915?hl=en
