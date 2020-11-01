---
author: Morfik
categories:
- Android
date:    2020-11-01 09:01:00 +0100
lastmod: 2020-11-01 09:01:00 +0100
published: true
status: publish
tags:
- smartfon
- lollipop
- marshmallow
- nougat
- youtube
- microg
- push
- reklamy
- aplikacje
title: Blokowanie reklam w YouTube na smartfonie z Androidem bez root
---

Użytkownicy Androida do przeglądania serwisu YouTube używają z reguły tej dedykowanej aplikacji od
Google. Problem z tą appką jest taki, że serwuje ona całą masę reklam, których to nie można wykroić
stosując popularne rozwiązania na bazie [Blokada][1] czy [AdAway][2]. Niektórzy starają się
korzystać z innych aplikacji pokroju [NewPipe][3] czy [SkyTube][4] ale one mają swoje ograniczenia,
np. nie można pisać komentarzy czy też nie działają powiadomienia push. Kiedyś by rozwiązać ten
problem reklam w appce YouTube [korzystałem z Magisk'a][5] i jego modułu YouTube Vanced ale to
rozwiązanie od jakiegoś czasu nie jest już wspierane, choć w dalszym ciągu można z niego korzystać.
Jako, że od paru miechów nie zaglądałem na [stronę YouTube Vanced][6], to postanowiłem sprawdzić
czy coś w tej kwestii się zmieniło. Wygląda na to, że jednak coś drgnęło, bo teraz dostępny jest
Vanced Manager, który to jest w stanie tak skonfigurować nasz telefon, by aplikacja YT Vanced
działała bez problemu nawet na nieukorzenionym Androidzie (nie trzeba mieć root'a). Możemy zatem
zachować całą funkcjonalność serwisu YouTube pozbywając się przy tym reklam oraz segmentów
sponsorowanych, no i też nie musimy nic kombinować z telefonem, tj. odblokowywać bootloader'a czy
wgrywać TWRP. Problematyczne może być jednak zainstalowanie YouTube Vanced, bo czasami powiadomienia
(notyfikacje push) mogą nam nie działać poprawnie. Właśnie dlatego postanowiłem napisać parę słów
na temat instalacji tej aplikacji z wykorzystaniem Vanced Manager w Androidach bez root, by uniknąć
tego jak i innych problemów.

<!--more-->
## Włączenie opcji "Nieznane źródła"

Jak zapewne się domyślacie, aplikacji YouTube Vanced czy Vanced Manager raczej w sklepie Google
Play nie znajdziemy. Dlatego trzeba też posiłkować się plikiem `.apk` pobranym bezpośrednio ze
strony twórcy aplikacji. Wymagane jest zatem włączenie w opcjach telefonu pozycji `Nieznane źródła`
(Unknown sources), co umożliwi nam instalowanie aplikacji spoza sklepu Google Play. Ta opcja może
być w różnych miejscach i kryć się pod nieco zmienioną nazwą w innych wersjach Androida. U mnie na
Andku 7.0 można ją znaleźć pod Ustawienia -> Blokowanie ekranu, odcisk palca i zabezpieczenia:

![](/img/2020/11/001-android-youtube-yt-vanced-microg-manager-block-commercials-unknown-sources.png#small)

## Włączenie debugowania USB

Włączenie debugowania USB raczej nie jest konieczne. Niemniej jednak, gdy korzystałem z modułu YT
Vanced dla Magisk'a, to ta aplikacja zastępowała tę stock'ową appkę YouTube. W przypadku YouTube
Vanced  instalowanego przez Vanced Manager wygląda na to, że system traktuje te dwie aplikacje, tak
jakby to były zupełnie różne appki. Dlatego też postanowiłem wyłączyć tę aplikację od Google przy
pomocy `adb` . By to jednak zrobić, musimy włączyć w smartfonie debugowanie USB. W moim systemie
trzeba wejść w Ustawienia -> Opcje programistyczne i zaznaczyć Debugowanie USB:

|   |   |
|---|---|
| ![](/img/2020/11/002-android-youtube-yt-vanced-microg-manager-block-commercials-menu.png#small) | ![](/img/2020/11/003-android-youtube-yt-vanced-microg-manager-block-commercials-developer-options.png#small) |

Jeśli nie mamy w ustawieniach pozycji `Opcje programistyczne` , to trzeba ją pierw odkryć stukając
parokrotnie w numer kompilacji, który zwykle jest widoczny w menu informacji o
telefonie/oprogramowaniu.

Mając włączony tryb debugowania USB, podłączamy nasz telefon do portu USB komputera. Przełączamy się
teraz na naszego linux'a i w terminalu wpisujemy `adb shell` :

    $ adb shell

Na ekranie telefonu powinien nas przywitać poniższy komunikat:

![](/img/2020/11/004-android-youtube-yt-vanced-microg-manager-block-commercials-adb-connect.png#small)

Naturalnie zezwalamy na połączenie.

Jeśli jednak system zwraca błąd polecenia, to prawdopodobnie nie mamy w nim wgranych stosownych
narzędzi. Na Debianie trzeba doinstalować pakiet `adb` (wcześniejsze wersje Debiana korzystały z
`android-tools-adb` ). Czasami też mogą pojawić się problemy z uprawnieniami, zatem lepiej jest to
powyższe polecenie wydawać jako root. Być może też trzeba będzie ubić wcześniej nasłuchujący serwer
ADB, by system w końcu był w stanie się połączyć ze smartfonem:

    $ adb kill-server
    $ adb shell

### Wyłączenie stock'owej appki YouTube

Po wydaniu na komputerze polecenia `adb shell` powinniśmy być w stanie uzyskać dostęp do shell'a
Androida. Przy jego pomocy możemy wyłączyć stock'ową appkę YouTube i nie potrzebne nam są do tego
prawa administracyjne (root). Wystarczy wpisać w terminalu polecenie
`pm uninstall -k --user 0 com.google.android.youtube` :

![](/img/2020/11/005-android-youtube-yt-vanced-microg-manager-block-commercials-adb-shell.png#big)

Jeśli teraz przejdziemy do listy aplikacji w telefonie, to powinniśmy zauważyć, że aplikacja YouTube
została wyłączona dla standardowego użytkownika:

![](/img/2020/11/006-android-youtube-yt-vanced-microg-manager-block-commercials-app-list.png#small)

## Vanced Manager, YouTube Vanced i MicroG

Możemy teraz przejść na [stronę YouTube Vanced][6] i pobrać z niej Vanced Manager. Po zakończonym
procesie pobierania pliku, instalujemy Vanced Manager w systemie i uruchamiamy go:

|   |   |
|---|---|
| ![](/img/2020/11/007-android-youtube-yt-vanced-microg-manager-block-commercials-website.png#small) | ![](/img/2020/11/008-android-youtube-yt-vanced-microg-manager-block-commercials-install-process.png#small) |
| ![](/img/2020/11/009-android-youtube-yt-vanced-microg-manager-block-commercials-install-process.png#small) | ![](/img/2020/11/010-android-youtube-yt-vanced-microg-manager-block-commercials-app.png#small) |

### Instalacja MicroG

Jak widać na powyższych fotkach, przy pomocy Vanced Manager możemy zainstalować trzy aplikacje:
YouTube Vanced, Music oraz MicroG. Nas w zasadzie interesuje jedynie pierwsza i ostatnia pozycja z
tym, że MicroG trzeba zainstalować jako pierwsze:

|   |   |
|---|---|
| ![](/img/2020/11/011-android-youtube-yt-vanced-microg-manager-block-commercials-microg-install.png#small) | ![](/img/2020/11/012-android-youtube-yt-vanced-microg-manager-block-commercials-microg-install.png#small) |
| ![](/img/2020/11/013-android-youtube-yt-vanced-microg-manager-block-commercials-microg-install.png#small) | ![](/img/2020/11/014-android-youtube-yt-vanced-microg-manager-block-commercials-microg-installed.png#small) |

#### Dodanie konta

Po instalacji MicroG, odpalamy jego ustawienia klikając w to kółko zębate widoczne wyżej. Następnie
musimy dodać swoje konto Google do MicroG:

|   |   |   |
|---|---|---|
| ![](/img/2020/11/015-android-youtube-yt-vanced-microg-manager-block-commercials-microg-options.png#small) | ![](/img/2020/11/016-android-youtube-yt-vanced-microg-manager-block-commercials-microg-account.png#small) | ![](/img/2020/11/017-android-youtube-yt-vanced-microg-manager-block-commercials-microg-account.png#small) |

|   |   |
|---|---|
| ![](/img/2020/11/018-android-youtube-yt-vanced-microg-manager-block-commercials-microg-account.png#small) | ![](/img/2020/11/019-android-youtube-yt-vanced-microg-manager-block-commercials-microg-account.png#small) |

To dodane konto powinno być widoczne na liście kont systemowych:

![](/img/2020/11/020-android-youtube-yt-vanced-microg-manager-block-commercials-accounts.png#small)

#### Wyłączenie optymalizacji baterii

Kolejnym krokiem jest wyłączenie optymalizacji baterii dla MicroG. W zależności od wersji
posiadanego Androida trzeba nieco inaczej to zrobić. Dla przykładu, w jednym z moich telefonów z
Androidem 5 nie trzeba było nic dodatkowo ustawiać ale już w Androidzie 7 dodano sporo nowych
mechanizmów i trzeba było wejść w Ustawienia -> Bateria -> Szczegóły -> Optymalizacja Baterii i
tam odznaczyć pozycję z Vanced MicroG:

|   |   |
|---|---|
| ![](/img/2020/11/021-android-youtube-yt-vanced-microg-manager-block-commercials-battery.png#small) | ![](/img/2020/11/022-android-youtube-yt-vanced-microg-manager-block-commercials-battery.png#small) |
| ![](/img/2020/11/023-android-youtube-yt-vanced-microg-manager-block-commercials-battery-ptimization.png#small) | ![](/img/2020/11/024-android-youtube-yt-vanced-microg-manager-block-commercials-battery-optimization.png#small) |

#### Włączenie autostartu

Następna rzecz to autostart MicroG. Trzeba się upewnić, że ta aplikacja będzie startować z systemem
oraz, że ten jej nie ubije i zezwoli jej na ciągłą pracę w tle. Podobnie jak w przypadku
optymalizacji baterii, ten krok może wyglądać inaczej w zależności od posiadanej wersji Androida.
Na Andku 7 trzeba wejść w listę zainstalowanych aplikacji i wybrać Vanced MicroG. Tam z kolei trzeba
wybrać pozycję "Bateria" i zaznaczyć "Uruchamianie automatyczne" oraz odznaczyć "Czyszczenie po
wyłączeniu ekranu", tak jak to widać na poniższych fotkach:

|   |   |
|---|---|
| ![](/img/2020/11/025-android-youtube-yt-vanced-microg-manager-block-commercials-app.png#small) | ![](/img/2020/11/026-android-youtube-yt-vanced-microg-manager-block-commercials-app-list.png#small) |
| ![](/img/2020/11/027-android-youtube-yt-vanced-microg-manager-block-commercials-app-list-microg.png#small) | ![](/img/2020/11/028-android-youtube-yt-vanced-microg-manager-block-commercials-app-list-microg-autostart.png#small) |

#### Restart telefonu

W tym miejscu trzeba zrestartować telefon. Jeśli się tego nie zrobi, to jest spora szansa na to, że
coś się nam pochrzani w późniejszym czasie, np. powiadomienia push nie będą do nas docierać i w ten
sposób nie będziemy otrzymywać informacji o nowych filmach dodanych na kanałach YouTube.

### Instalacja YouTube Vanced

Po ponownym uruchomieniu telefonu wchodzimy w Vanced Manager. Tym razem instalujemy YouTube Vanced:

|   |   |   |
|---|---|---|
| ![](/img/2020/11/029-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-install.png#small) | ![](/img/2020/11/030-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-installed.png#small) | ![](/img/2020/11/031-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced.png#small) |

#### Wyłączenie optymalizacji baterii

Podobnie jak w przypadku MicroG, dla aplikacji YouTube Vanced również wyłączamy optymalizację
baterii:

![](/img/2020/11/032-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-battery-optimization.png#small)

#### Włączenie autostartu

Również i w przypadku YouTube Vanced włączamy autostart:

|   |   |   |
|---|---|---|
| ![](/img/2020/11/033-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-app-list.png#small) | ![](/img/2020/11/034-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-app-list.png#small) | ![](/img/2020/11/035-android-youtube-yt-vanced-microg-manager-block-commercials-yt-vanced-app-list-autostart.png#small) |

#### Restart telefonu

W tym przypadku restart raczej nie jest wymagany. Ja jednak na wszelki wypadek zrestartowałem swój
telefon by sprawdzić czy wszystko uruchomi się tak jak należy i czy nie będzie ewentualnych
problemów w działaniu czy to MicroG, czy YouTube Vanced.

## Rejestracja urządzenia

By być w stanie odbierać notyfikacje push, musimy zarejestrować swoje urządzenie w strukturach
Google. W tym celu trzeba odpalić Vanced Manager i przejść w nim do ustawień MicroG:

|   |   |
|---|---|
| ![](/img/2020/11/036-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings.png#small) | ![](/img/2020/11/037-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings.png#small) |

Tam z kolei mamy pozycję `Google Device Registration` . Naturalnie klikamy w nią:

![](/img/2020/11/038-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings-device-register.png#small)

Powinniśmy mieć wygenerowany zarówno identyfikator, jak i informację o fakcie zarejestrowania nas w
strukturach Google. Jeśli takie informacje posiadamy, to odpalamy aplikację YouTube Vanced:

|   |   |
|---|---|
| ![](/img/2020/11/039-android-youtube-yt-vanced-microg-manager-block-commercials-test.png#small) | ![](/img/2020/11/040-android-youtube-yt-vanced-microg-manager-block-commercials-test-account.png#small) |

Powinniśmy zostać automatycznie zalogowani na nasze konto YouTube, co możemy zweryfikować po
awatarze w prawym górnym rogu okna aplikacji lub też bezpośrednio w menu aplikacji. Jeśli zostaliśmy
poprawnie zalogowani, to znaczy, że wszystko póki co działa jak należy.

## Notyfikacje push

Ostatnią rzeczą, którą musimy zweryfikować, to czy notyfikacje push są do nas dostarczane. Ten fakt
możemy potwierdzić jedynie w przypadku dodania na przykładowy kanał YouTube jakieś treści (filmu czy
komentarza). Zapewne będzie trzeba trochę poczekać zanim jakieś powiadomienie do nas dotrze. Jeśli
nie jesteśmy pewni czy te powiadomienia push w ogóle działają, to możemy spróbować to sprawdzić. W
tym celu przechodzimy do ustawień MicroG i wchodzimy w pozycję GCM (Google Cloud Messaging):

|   |   |   |
|---|---|---|
| ![](/img/2020/11/041-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings.png#small) | ![](/img/2020/11/042-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings-gcm.png#small) | ![](/img/2020/11/043-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings-gcm.png#small) |

Jak widać, jesteśmy połączeni z usługą GCM oraz aplikacja YouTube Vanced chce korzystać z
notyfikacji push. Powinna być zatem ona w stanie odbierać powiadomienia z serwera Google. Gdy ten
moment nastąpi, to w ustawieniach MicroG powinniśmy ten stan rzeczy zauważyć:

![](/img/2020/11/044-android-youtube-yt-vanced-microg-manager-block-commercials-microg-settings-gcm.png#small)

I tu już widzimy, że aplikacja YouTube Vanced odebrała dwa powiadomienia, co również można zauważyć
na ekranie telefonu:

![](/img/2020/11/045-android-youtube-yt-vanced-microg-manager-block-commercials-phone-notifs.png#small)

## Wyłączenie USB debug i nieznanych źródeł

Po pomyślnym wgraniu YouTube Vanced, możemy wyłączyć zarówno opcję instalowania aplikacji z
nieznanych źródeł jak i debugowanie USB. To tak na wypadek, gdyby aplikacje bankowe/finansowe miały
problem i nie chciały się uruchomić lub zgłaszały inne problemy, gdy któraś z tych opcji jest
włączona. Być może przy aktualizacji YT Vanced/MicroG/Vanced Manager trzeba będzie włączyć na czas
tego procesu instalowanie appek z nieznanych źródeł ale to raczej niewielka niedogodność, z którą
trzeba będzie się zmierzyć w stosunku do korzyści jakie YouTube Vanced nam jest w stanie zaoferować.

## Podsumowanie

Trzeba przyznać, że nieco zawiły jest ten cały proces instalowania YouTube Vanced na smartfonie z
Androidem. Niemniej jednak, mając tę aplikację w systemie, możemy w końcu pozbyć się reklam z
serwisu YouTube. Niewątpliwą zaletą jest też fakt, że nie trzeba już posiadać ukorzenionego Androida
(root), by YT Vanced można było w nim zainstalować. Także miłego oglądania filmików na YT bez
reklam.


[1]: https://blokada.org/
[2]: https://adaway.org/
[3]: https://newpipe.schabi.org/
[4]: https://skytube-app.com/
[5]: https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445
[6]: https://vancedapp.com/
