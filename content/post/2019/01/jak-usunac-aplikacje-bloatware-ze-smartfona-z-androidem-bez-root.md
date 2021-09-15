---
author: Morfik
categories:
- Android
date:    2019-01-27 06:01:02 +0100
lastmod: 2020-12-01 16:30:00 +0100
published: true
status: publish
tags:
- smartfon
- aplikacje
- bloatware
- spam
- reklamy
- adb
title: Jak usunąć aplikacje bloatware ze smartfona z Androidem bez root
---

Jeśli mamy smartfon z Androidem na pokładzie, to zapewne każdy za nas zadawał sobie pytanie, czy da
radę z takiego telefonu pozbyć się szeregu aplikacji, z których praktycznie nie korzystamy na co
dzień. Część z tych programów można wyłączyć w ustawieniach systemowych ale są też i takie
aplikacje (głównie producenta telefonu, czy też operatora GSM albo te od Google), których
standardowo nie da się wyłączyć z poziomu działającego Androida. Nawet jeśli wymusimy zatrzymanie
stosownych usług, to za chwilę (lub po restarcie urządzenia) one i tak nam automatycznie wystartują.
Im więcej zbędnych aplikacji działa w tle, tym częstsze wybudzanie telefonu, a więc i szybsze
wyczerpywanie się baterii. Dlatego też jeśli nie korzystamy z wbudowanego w ROM [bloatware][1], to
przydałoby się go usunąć lub chociaż trwale wyłączyć. Co ciekawe, tego typu proces nie musi odbywać
się za sprawą administratora systemu (root), bo w zasadzie każda aplikacja w Androidzie może zostać
zainstalowana/odinstalowana dla konkretnego użytkownika w systemie. Nie potrzebujemy mieć zatem
nawet ukorzenionego Androida, by pozbyć się tego całego syfu z systemu, który naszemu urządzeniu
spędza sen z powiek i nie daje mu się przy tym porządnie wyspać.

<!--more-->
## Wyłączenie aplikacji z poziomu ustawień Androida

Niektóre bardziej cywilizowane ROM'y (np. LineageOS) nie narzucają nam aplikacji, z których musimy
korzystać. Niemniej jednak, nawet w przypadku takich ROM'ów kilka tych najbardziej podstawowych
programów jest już preinstalowana, np. aplikacja galerii czy klient email. Jeśli wejdziemy w
ustawienia aplikacji Androida, to mamy możliwość wyłączenia szeregu z tych preinstalowanych rzeczy.
Poniżej fotki:

|     |    |
| --- | ---|
|![](/img/2019/01/001-android-disable-apps.png#small) | ![](/img/2019/01/002-android-disable-apps.png#small)|

Każda wyłączona w ten sposób aplikacja już nam się automatycznie nie uruchomi. Co prawda, w dalszym
ciągu taka aplikacja jest obecna w systemie ale z racji, że jest ona nieaktywna, to możemy o niej
zapomnieć. W tych alternatywnych ROM'ach są też i takie programy, których nie damy rady się pozbyć.
Można tu się posłużyć przykładem aplikacji od SMS'ów:

![](/img/2019/01/003-android-force-stop-apps.png#small)

Mamy możliwość siłowego zatrzymania tej aplikacji ale nie ma żadnej pewności co do tego czy nie
wystartuje nam ona w przyszłości sama z siebie.

Natomiast jeśli chodzi o stock'owe ROM'y różnych producentów telefonów, to tutaj zwykle nie mamy
opcji wyłączenia żadnych z tych preinstalowanych aplikacji. Dla przykładu, mam telefon, który w
ROM ma zawarte już dwie aplikacje do oglądania materiałów video. Jedna aplikacja od Google, a druga
od producenta telefonu:

|     |    |     |
| --- | ---| --- |
|![](/img/2019/01/004-android-force-stop-apps-stock.png#small) | ![](/img/2019/01/005-android-force-stop-apps-stock.png#small)|

Zamiast tych aplikacji używam otwartoźródłowego VLC. Niemniej jednak, mimo, że nie korzystam z
żadnego z tych dwóch preinstalowanych odtwarzaczy video, to i tak nie mam możliwości ich trwałego
wyłączenia w systemie. Podobnie sprawa wygląda w przypadku aplikacji od obrazków, muzyki, czy
przeglądarki internetowej, itp.

## Usuwanie aplikacji przez pm disable , pm hide i pm uninstall

Jeśli chodzi o usuwanie aplikacji z telefonu, to w zasadzie na necie można spotkać się z trzema
poleceniami: `pm disable` , `pm hide` oraz `pm uninstall` . Każda z tych komend ma nieco inne
zastosowanie ale to co je łączy to dodawanie odpowiednich atrybutów do pliku
`/data/system/users/0/package-restrictions.xml` . Po obserwacji można stwierdzić, że `pm hide`
dodaje `hidden="true"` , jeśli zaś chodzi o `pm disable` , to dodaje on `enabled="2"` (lub
przepisuje z `enabled="1"`). Z kolei `pm uninstall` dodaje `inst="false" stopped="true" nl="true"` .

Zarówno `pm hide` jak i `pm disable` wymagają odpowiednich uprawnień przy ich wydawaniu.
Standardowo próba wykonania tych dwóch poleceń jako zwykły użytkownik, nawet precyzując `--user 0`
(domyślny użytkownik systemu), za pomocą `adb shell` będzie skutkować tymi poniższymi błędami:

    $ pm hide --user 0 com.google.android.apps.translate
    Error: java.lang.SecurityException: Neither user 2000 nor current process has android.permission.MANAGE_USERS.

    $ pm disable --user 0 com.google.android.apps.translate
    Error: java.lang.SecurityException: Shell cannot change component state for com.google.android.apps.translate/null to 2

W zasadzie to różnica między `pm hide` a `pm disable` sprowadza się do faktu, że `pm hide` jest w
stanie ukryć i za razem wyłączyć całą aplikację i jej wszystkie usługi, podczas gdy `pm disable`
umożliwia wyłączenie także poszczególnych usług konkretnej aplikacji bez potrzeby jej całkowitej
dezaktywacji. Dodatkowo, aplikacja potraktowana `pm hide` jest w dalszym ciągu widoczna w
ustawieniach Androida w sekcji z aplikacjami, choć szereg opcji dostępnych tam jest wyszarzonych,
np. uprawnienia. Jeśli zaś chodzi o `pm uninstall` , jak nazwa wskazuje, to polecenie jest w stanie
całkowicie usunąć daną aplikację z systemu. O `pm hide` można też myśleć w kategoriach takiego
tymczasowego `pm uninstall` . Warto tutaj zaznaczyć, że `pm uninstall` można wywołać jako zwykły
użytkownik, o czym będzie za moment.

Na
necie jeszcze [znalazłem dokładną rozpiskę][2] na temat różnic pomiędzy `pm hide` oraz
`pm disable` . Wrzucam ją niżej, by nie zginęła:

![](/img/2019/01/006-difference-pm-hide-disable-android.png#huge)

Tam na tej fotce jest zawarta informacja, że `pm hide` może być używany przez zwykłego użytkownika.
Nie jest to prawdą, gdyż do czasu Android 6 (2016-08-01) były pewne problemy z uprawnieniami
MANAGE_USERS i CREATE_USERS, co umożliwiało obejście restrykcji dostępu ([CVE-2016-3833][3]).
Dlatego też w kolejnych wersjach Androida zwykły user już nie może korzystać z `pm hide` .

### Jak używać pm uninstall bez root

By usuwanie aplikacji było możliwe bez zaprzęgania do tego celu administratora systemu, trzeba
skorzystać z `pm uninstall` oraz określić w nim dwa dodatkowe parametry: `-k` oraz `--user 0` .
Dzięki opcji  `-k` cache oraz dane aplikacji nie zostaną usunięte z ROM'u (partycji `/system/` ),
co naturalnie wymaga uprawnień administratora systemu. Z kolei opcja `--user 0` precyzuje ID
użytkownika, którego dotyczyć będzie ta akcja. Bez tej opcji usunięcie aplikacji dotyczyłoby
wszystkich użytkowników w systemie, a to już wymaga uprawnień root. Całe polecenie wyglądać
będzie zatem następująco:

    $ pm uninstall -k --user 0 com.google.android.videos

W tym przypadku zostanie usunięta aplikacja `com.google.android.videos` ale możemy określić każdą
inną aplikację, która widnieje na liście menadżera pakietów `pm` . Listę zainstalowanych aplikacji
możemy uzyskać w poniższy sposób:

    $ pm list packages
    package:com.android.fmradio
    package:com.box.android
    package:org.omnirom.omnijaws
    package:com.android.cts.priv.ctsshim
    package:com.google.android.youtube
    ...

Czasami nazwy aplikacji nijak mają się do tego co widnieje w Google Play czy F-Droid, dlatego też
można posłużyć się narzędziem `grep` i poszukać interesującej nas frazy:

    $ pm list packages | grep key
    package:com.android.keychain

Nazwy zwrócone przez `pm list packages` są poprzedzone frazą `package:` i trzeba pamiętać o tym, by
tę frazę skasować, gdy będziemy podawać nazwę aplikacji do usunięcia w `pm uninstall` .

## Usunięte aplikacje i aktualizacje OTA

Cały ten proces pozbywania się bloatware nie wpływa w żaden sposób na aktualizacje OTA. Innymi
słowy, po usunięciu dowolnej aplikacji przez `pm uninstall -k --user 0` bez problemu będziemy mogli
zaktualizować telefon za pomocą mechanizmu OTA Update (no chyba, że usuniemy sobie aplikację od
OTA). Całość działa OOTB i użytkownik nie musi przy tym przeprowadzać żadnych dodatkowych
czynności, np. przywracać pierw usuniętych programów. Te aplikacje, które usunęliśmy, nie powrócą
nam też po przeprowadzeniu procesu aktualizacji OTA.

## Jak przywrócić usunięte aplikacje

W zasadzie, to przy usuwaniu aplikacji przez `pm uninstall -k --user 0` nie dokonujemy żadnych
zmian na partycji `/system/` . Dlatego też aktualizacje OTA będą działać bez problemu. Jeśli
jednak przez przypadek usunęliśmy sobie nie tą aplikację, którą chcieliśmy, to w pewnych przypadkach
możemy mieć spore problemy z jej przywróceniem. Może i fizycznie pliki programu są dostępne w ROM
na partycji `/system/` ale ustawienia, które każą Androidowi ten fakt zignorować, w części
smartfonów może zmienić jedynie administrator systemu. Do końca nie wiem czy to zależy od samego
Androida (jego wersji), czy też od producenta urządzenia ale trzeba ten fakt mieć na uwadze. W
przypadku ewentualnej pomyłki może okazać się, że trzeba będzie ratować się przywróceniem stanu
urządzenia do ustawień fabrycznych (factory reset), by ta aplikacja na nowo się w systemie pojawiła.

### Polecenie cmd

W przypadku jednego z moich urządzeń z Androidem 10 jest możliwość skorzystania z polecenia `cmd` .
Przy jego pomocy możemy przywrócić wcześniej odinstalowane aplikacje i nie trzeba do tego celu
pozyskiwać praw administratora systemu. Trzeba tutaj jednak wyraźnie zaznaczyć, że nie we wszystkich
Androidach ten `cmd` jest obecny. W zasadzie, to tylko w jednym z moich telefonów spotkałem się z
możliwością skorzystania z tego polecenia. Jeśli Android w naszym smartfonie oferuje polecenie
`cmd` , to wpisując to poniższe polecenie możemy przywrócić odinstalowane appki:

    $ adb shell
    galahad:/ $ cmd package install-existing com.miui.calculator
    Package com.miui.calculator installed for user: 0

### Plik /data/system/users/0/package-restrictions.xml

W przypadku gdy mamy zapewniony dostęp root, nawet niekoniecznie w działającym systemie, np. przez
TWRP recovery, to możemy wszystkie wprowadzone przez nas zmiany podczas usuwania aplikacji cofnąć
ręcznie. Musimy tylko odnaleźć plik `/data/system/users/0/package-restrictions.xml` , bo to w nim są
przechowywane informacje na temat restrykcji dostępu do tych faktycznie zainstalowanych w systemie
programów.

Sam plik składa się z wpisów określających nazwę aplikacji oraz paru dodatkowych atrybutów
mówiących m.in. czy dany program jest zainstalowany dla tego konkretnego użytkownika w systemie. W
przypadku, gdy usuwamy program, to plik `package-restrictions.xml` jest uzupełniany o odpowiednie
atrybuty. Przykładowo, poniżej znajdują się dwie aplikacje, które zostały usunięte z systemu:

    <pkg name="com.tplink.video" inst="false" stopped="true" nl="true" />
    <pkg name="com.google.android.videos" inst="false" stopped="true" nl="true" />

Mamy tutaj atrybut `inst="false"` , który mówi, że dla tego użytkownika w systemie ten pakiet jest
niezainstalowany. Jest też `stopped="true"` oznajmiający, że aplikacja aktualnie nie jest odpalona
i może zostać uruchomiona jedynie na wyraźne polecenie użytkownika. Na końcu mamy zaś `nl="true"`
(skrót nl pochodzi od "not launched"), który zostaje ustawiony, gdy system nigdy tej aplikacji
jeszcze nie uruchomił.

Każda aplikacja, którą potraktujemy przy pomocy `pm uninstall -k --user 0` będzie miała dodane te
trzy widoczne wyżej atrybuty. Zatem, by ponownie te aplikacje były widoczne dla domyślnego
użytkownika (tego z id `0` ), to trzeba wyedytować plik `package-restrictions.xml` i usunąć z niego
`inst="false" stopped="true" nl="true"` , tak by te powyższe wpisy wyglądałyby następująco:

    <pkg name="com.tplink.video" />
    <pkg name="com.google.android.videos" />

## Jak sprawdzić, które aplikacje zostały odinstalowane

Generalnie, to w celu sprawdzenia, które aplikacje usunęliśmy z systemu, można wykorzystać
wspomniany już plik `package-restrictions.xml` ale do tego celu potrzebne są uprawnienia root.
Niemniej jednak, każda usunięta aplikacja za sprawą `pm uninstall -k --user 0` będzie widoczna w
ustawieniach Androida, w menu aplikacji. Zwykle trzeba będzie wybrać filtr wszystkie/systemowe, bo
bez niego tych usuniętych aplikacji nie zobaczymy:

![](/img/2019/01/007-android-removed-uninstalled-apps-menu.png#small)

Widać wyraźnie, których aplikacji nie ma już w systemie, a raczej, które z nich zostały usunięte
dla tego konkretnego użytkownika, co oznajmia nam wyraźnie komunikat przy pozycji stosownej
aplikacji.

## Preinstalowane aplikacje, które można bezpiecznie usunąć

Podczas ostatniego zaorania swojego smartfona, postanowiłem zrobić listę aplikacji, które można
bezpiecznie z systemu usunąć i być pewnym, że nie niesie to (chyba) żadnych negatywnych skutków
ubocznych, przynajmniej jeśli chodzi o zachowanie samego Androida. Trzeba jednak pamiętać, że każda
aplikacja, która jest preinstalowana w Androidzie może być wykorzystywana przez inne aplikacje
zarówno te stock'owe jak i również te, które sami zainstalujemy w późniejszym czasie użytkowania
telefonu.

Nazwy pakietów zainstalowanych w systemie raczej identyfikują konkretne aplikacje, poza nielicznymi
wyjątkami. Niżej jest lista pakietów z krótką informacją po prawej prawej stronie, gdzie znajduje
się fraza, którą trzeba wpisać w sklepie Google Play, by tę konkretną aplikację odszukać:

    com.google.android.music            # Muzyka Google Play
    com.google.android.videos           # Filmy Google Play
    com.google.android.apps.photos      # Zdjęcia Google
    com.google.android.apps.maps        # Mapy Google
    com.google.android.calendar         # Kalendarz Google
    com.google.android.apps.docs        # Dysk Google
    com.google.android.gm               # Gmail
    com.google.android.tts              # Zamiana tekstu na mowę Google (Text-to-Speech)
    com.google.android.feedback         #
    com.google.android.marvin.talkback  # Android Accessibility Suite (TalkBack)
    com.google.android.apps.tachyon     # Google Duo
    com.google.android.googlequicksearchbox # Google

    com.android.messaging               # Wiadomości
    com.baidu.map.location              #

    # Tapety
    com.android.wallpaper.holospiral    #
    com.android.wallpaper.livepicker    #
    com.android.phasebeam               #
    com.android.galaxy4                 #

    # Aplikacje TP-link
    com.tplink.calculator               #
    com.tplink.gallery                  #
    com.tplink.tpnotes                  #
    com.tplink.weather                  # Pogoda Neffos
    com.tplink.tpsoundrecorder          #
    com.tplink.tpwlan                   #
    com.tplink.video                    #
    com.tplink.tpclock                  #
    com.tplink.camera                   #
    com.tplink.flashlight               #
    com.tplink.tpdemo                   #

Niżej są jeszcze opcjonalne pozycje i generalnie, to usuwanie tych poniższych pakietów nie powinno
mieć miejsca, no chyba, że [mamy zainstalowanego szpiega][4] w aplikacji OTA, który zbiera o nas
całą masę informacji i wysyła je w świat. Mój smartfon raczej tego szpiega nie posiada ale z racji,
że lubi sam sobie instalować aktualizacje nawet, gdy mu się wyraźnie mówi by tego nie robił, to
można odinstalować cały ten mechanizm OTA. Oczywiście w ten sposób nie zainstalujemy już żadnej
aktualizacji do momentu przywrócenia tych pakietów, ani nawet nie będziemy świadomi faktu, że
producent telefonu/operator GSM wypuścił jakiś nowy update, choć o to akurat bym się nie martwił...

    com.tplink.fota
    com.tplink.fotaui
    com.tplink.update

Jeśli nie korzystamy z bluetooth, to można usunąć także te poniższe pakiety, zwłaszcza w sytuacji,
gdy mamy dziurawe BT, bo producent telefonu czy też i operator GSM nie załatał [podatności
BlueBorne][5].

    com.android.bluetooth
    com.android.bluetoothmidiservice

Wydawać by się mogło, że te dwie poniższe aplikacje także nie są nikomu do niczego potrzebne, gdy
mamy na pokładzie zainstalowane ich odpowiedniki. Niemniej jednak, usunięcie tych dwóch pozycji
skutkuje crash'em podczas importowania listy kontaktów w stock'owej aplikacji dialer'a i lepiej ich
nie usuwać.

    com.tplink.filemanager
    com.android.chrome


[1]: https://en.wikipedia.org/wiki/Software_bloat
[2]: https://android.stackexchange.com/questions/128949/pm-hide-vs-pm-disable-the-identity-crisis
[3]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3833
[4]: https://zaufanatrzeciastrona.pl/post/popularne-chinskie-telefony-przylapane-na-wysylaniu-smsow-i-kontaktow-do-chin/
[5]: https://armis.com/blueborne/
