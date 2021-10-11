---
author: Morfik
categories:
- Linux
date: "2015-10-29T16:50:34Z"
date_gmt: 2015-10-29 14:50:34 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- firefox
- prywatność
GHissueID: 174
title: Więcej niż jeden profil w Firefox'ie
---

Ogromna większość ludzi korzysta z jednego profilu swojej przeglądarki internetowej. Niesie to ze
sobą spore zagrożenie bezpieczeństwa jak i może godzić w naszą prywatność. Jeśli dzielimy z kimś
komputer, to raczej wszyscy domownicy posiadają osobne konta w systemie, a co z tym się wiąże, inny
profil przeglądarki. I na tym zwykle podział się kończy ale przecie to nie wszystko. Profil, jak
sama nazwa wskazuje, jest w stanie dostosować opcje przeglądarki, np. pod kątem pewnych aktywności.
W tym wpisie postaramy się utworzyć kilka profili w Firefox'ie i sprawdzimy korzystanie z nich
będzie odczuwalne w jakiś sposób dla przeciętnego użytkownika internetu.

<!--more-->
## Po co mi kilka profili

Gdy przeglądamy internet za pomocą tylko jednego profilu w Firefox'ie, cała nasza aktywność jest w
nim rejestrowana. Zapisywane są takie dane jak historia przeglądanych stron, ciasteczka dla
konkretnych domen, hasła do serwisów czy też wpisywane frazy w formularzach na stronach www. W
Firefox'ie mamy możliwość dostosowania szeregu opcji, w tym też tych wszystkich wyżej wymienionych,
i nie powinno to sprawić większego problemu, bo wystarczy wejść w Preferences -> Privacy i rzucić
okiem na te poniższe opcje:

![](/img/2015/10/1.opcje-firefox-profil.png#huge)

Mamy możliwość skonfigurowania przeglądarki tak by nie akceptowała ciasteczek, czy też nie
zapisywała historii i całej masy innych rzeczy ale to powoduje, że w większości przypadków tego
typu ustawienia są kompletnie bez sensu. Oczywistym jest, że możemy nie chcieć by pewne strony www
pojawiały się w historii, np. te pornograficzne, ale nie musimy zaraz usuwać całej historii, bo ta
może się nam przydać w późniejszym czasie, gdy będziemy potrzebować odnaleźć adres jakiegoś serwisu,
który odwiedzaliśmy, np. wczoraj. W przypadku wielu profili, te wszystkie ustawienia możemy bez
problemu sobie dostosować i tak w jednym profilu możemy włączyć historię odwiedzanych stron www, a w
drugim ją wyłączyć.

Poza opcjami prywatności wymienionymi powyżej, możemy także niezależnie konfigurować wtyczki, np.
[flash](/post/konfiguracja-wtyczki-flash-na-linuxie-mms-cfg/), czy też wszelkie
inne ustawienia udostępniane przez
[about:config](/post/usuwanie-wpisow-z-aboutconfig-w-firefoxie/) . Wobec czego
możemy posiadać w pełni funkcjonalny profil pod sieć TOR z uwzględnieniem konfiguracji PROXY i
wyłączeniem wszystkich wtyczek, a jednocześnie możemy mieć osobny profil na różne konta email,
serwisy społecznościowe czy też porno. Gdy do tego dorzucimy jeszcze [profil
apparmor'a](/post/apparmor-profilowanie-aplikacji/) lub/i [kontener TrueCrypt czy
też LUKS](/post/przejscie-z-truecrypt-na-luks/), możemy kompletnie odseparować od
siebie pewne aspekty naszego życia internetowego. W ten sposób możemy uchronić się przed ciągłym
zmienianiem i dostosowywaniem opcji, które mogą przyczynić się do przecieków informacyjnych.

## Jak utworzyć nowy profil w Firefox'ie

Pliki profili Firefox'a na linux'ie są zlokalizowane w katalogu `~/.mozilla/firefox/` . W zależności
od tego ile mamy profili, tyle będzie katalogów -- jeden na każdy profil. Wygląda to mniej więcej
tak:

![](/img/2015/10/2.katalog-profil-firefox.png#small)

Poza katalogami o dziwacznych nazwach, widzimy także plik `profiles.ini` . To w nim są zawarte
[informacje identyfikujące konkretny profil](http://kb.mozillazine.org/Profiles.ini_file). Poniżej
zawartość tego pliku:

    [General]
    StartWithLastProfile=0

    [Profile0]
    Name=default
    IsRelative=1
    Path=l6ayurcg.Default

    [Profile1]
    Name=mail
    IsRelative=0
    Path=/media/firefox/oqv05gdi.mail
    Default=1

Plik z grubsza zawiera dwie sekcje: `[General]` oraz `[ProfileX]` , gdzie `X` to kolejny numer.
Niżej zaś są wyjaśnione wykorzystane w nim opcje.:

  - `StartWithLastProfile` , gdy ustawiony na `0` sprawi, że przy otwieraniu Firefox'a pojawi się
    menadżer profili.
  - `Name` określa nazwę, pod którą dany profil będzie występował w menadżerze profili.
  - `IsRelative` odpowiada za rodzaj ścieżki do katalogu z profilem.
  - `Path` to ścieżka do profilu. Możliwe są do określenia względne i bezwzględne ścieżki. Względne
    odnoszą się do katalogu `~/.mozilla/firefox/` .
  - `Default` może pojawić się tylko raz i określa, który profil ma być domyślny.

### Tworzenie profilu przy pomocy menadżera

Tworzenie profili jest bardzo proste i sprowadza się do odpalenia w terminalu Firefox'a z parametrem
`--ProfileManager` . Spowoduje to pojawienie się tego poniższego okienka:

![](/img/2015/10/3.tworzenie-profil-firefox.png#medium)

Klikamy na przycisk `Create Profile` i dodajemy tyle profili ile potrzebujemy. W każdym z nich
możemy określić nazwę oraz katalog:

![](/img/2015/10/5.wybor-profil-firefox.png#medium)

Po skończonej robocie, powinniśmy mieć na liście kilka profili:

![](/img/2015/10/4.zmiana-nazwy-profil-firefox.png#big)

Pamiętajmy też by odznaczyć pozycję `Use the selected profile without asking at startup` . W
przeciwnym wypadku, ostatnio użyty profil będzie wykorzystywany jako domyślny.

### Skrót do profilu Firefox'a

Jeśli nie chcemy za każdym razem wywoływać menadżera profili i ręcznie wybierać tego jaki profil
powinien zostać załadowany, możemy stworzyć sobie kilka plików `.desktop` i umieścić je sobie np. na
[panelu tint2](https://gitlab.com/o9000/tint2) . Oczywiście jeśli korzystamy z jakiegoś środowiska
graficznego, to trzeba w nim dodać odpowiedni skrót. Ja ze środowisk graficznych nie korzystam,
także nie wiem gdzie te skróty w menu się dodaje. Tak czy inaczej, w przypadku paneli tint2, musimy
utworzyć plik `.desktop` o poniższej zawartości:

    [Desktop Entry]
    Type=Application
    Version=1.0
    Name=Firefox
    Comment=Web Browser
    Comment[pl]=Przeglądarka www
    TryExec=/opt/firefox/firefox --new-instance --profile /media/firefox/oqv05gdi.mail/
    Exec=/opt/firefox/firefox --new-instance --profile /media/firefox/oqv05gdi.mail/
    Icon=/home/morfik/.config/launchers/icons/firefox-32x32.png
    Terminal=false
    Keywords=browser;www;http;mozilla;

Kluczowe jest ustawienie tutaj odpowiedniej ścieżki do profilu w parametrze `--profile` . Natomiast
`--new-instance` odpowiada za możliwość otwierania kilku profili jednocześnie.
