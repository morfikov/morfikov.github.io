---
author: Morfik
categories:
- Android
date:    2021-09-29 17:09:00 +0200
lastmod: 2021-09-29 17:09:00 +0200
published: true
status: publish
tags:
- smartfon
- xiaomi
- redmi-9
- root
- aosp
- crdroid
- microg
- safetynet
- twrp
- shrp
GHissueID: 323
title: Jak wgrać crDroid Android ROM na Xiaomi Redmi 9 (lancelot/galahad)
---

Przyszła pora pozbyć się w końcu tego stock'owego oprogramowania, które zostało wgrane na mojego
smartfona Xiaomi Redmi 9 (lancelot/galahad) przez producenta tego urządzenia. Przez ostatnich parę
tygodni testowałem różne wersje ROM'ów na bazie AOSP/LineageOS, z których bardziej użyteczne
okazały się [PixelPlusUI][1], [Pixel Extended][2] oraz [crDroid Anadroid][3]. Niestety nikt jeszcze
nie opracował LineageOS na ten telefon, więc pozostaje w zasadzie wgranie jednej z tych trzech
powyższych pozycji, jako że Xiaomi Redmi 9 jest oficjalnie przez te ROM'y wspierany. Obecnie Pixel
Extended ma jednak problemy z hostowaniem swoich plików i od paru miesięcy nie miał praktycznie
żadnej aktualizacji, przez co wybór został ograniczony do dwóch pozostałych ROM'ów. Powodem, dla
którego zdecydowałem się wgrać crDroid Android, jest fakt, że nie ma on zintegrowanych aplikacji od
Google (GAPPS). Oczywiście można po instalacji samego ROM'u wgrać także Open GAPPS ale naturalnie
nie jest to wymagane, przez co można sobie skonfigurować cały telefon według własnego uznania
wykorzystując jako bazę początkową microG. W tym artykule zostanie przedstawiony sposób wgrania
ROM'u crDroid Android na smartfon, tak by czasem nie uszkodzić tego urządzenia.

<!--more-->
## Odblokowany bootloader

Jak zapewne już większość użytkowników telefonów z Androidem się orientuje, by być w stanie wgrać
alternatywne ROM'y (nie tylko te na bazie AOSP/LineageOS), trzeba w takim urządzeniu posiadać
odblokowany bootloader. Stosowny [artykuł na temat odblokowania bootloader'a w Xiaomi Redmi 9
(lancelot/galahad)][4] znajduje się tutaj. Jeśli nasz smartfon nie ma odblokowanego bootloader'a,
to zapraszam do zapoznania się z tym podlinkowanym artykułem.

## TWRP/SHRP recovery

Mając odblokowany bootloader, możemy wgrać na telefon ROM crDroid Anadroid. Możemy to jednak zrobić
z poziomu trybu recovery ale nie tego standardowego, który jest obecny w ROM'ach od Xiaomi.
Potrzebny nam jest do tego celu albo TWRP albo SHRP. [Proces wgrywania TWRP na Xiaomi Redmi 9
(lancelot/galahad)][5] został opisany tutaj, na wypadek, gdyby ktoś chciał sobie zbudować własny
obraz TWRP recovery.

Zalecanym jednak trybem recovery jest [SHRP][6]. Co ciekawe smartfon [Xiaomi Redmi 9 jest oficjalnie
wspierany przez SHRP][7], zatem możemy bez większego problemu pobrać oficjalny obraz SHRP recovery
i wrzucić go na smartfon przy pomocy `fastboot` :

    $ fastboot flash recovery SHRP_v3.1_stable-Official_lancelot-1628943678.img

Po wgraniu SHRP, uruchamiamy smartfon w trybie recovery.

## Kopia zapasowa całego ROM'u

Zanim cokolwiek zrobimy w kwestii instalacji nowego ROM'u na smartfonie, zróbmy sobie [kopię
zapasową całego flash'a telefonu][20]. W ten sposób w przypadku, gdyby poszło coś nie tak, to
będziemy w stanie odtworzyć stan urządzenia sprzed dokonywania w nim z mian. Jeśli nie sporządzimy
tego backup'u i uszkodzą/wyczyszczą nam się z jakiegoś powodu numery IMEI, adresy MAC WiFi/BT,
dane kalibracyjne dla modemu GSM, czy też inne wrażliwe informacje, to nasz smartfon nie będzie
już działać poprawnie i w zasadzie będzie do wyrzucenia, przynajmniej jeśli zależy nam na takiej
funkcjonalności jak połączenie z internetem czy możliwość dzwonienia. Mając backup w sytuacji, gdy
coś pójdzie nie tak, bez większego problemu będziemy mogli przy pomocy `fastboot` i obrazów
poszczególnych partycji przywrócić telefon do stanu, który był tuż po odblokowaniu jego
bootloader'a i tym samym cofnąć wszystkie zmiany, które przyczyniły się do jego uszkodzenia.

## Wyczyszczenie danych użytkownika (Factory Reset)

Pierwszą rzeczą, którą trzeba uczynić przy przejściu z jednego ROM'u (zwykle producenta) na jakiś
inny (zwykle AOSP/LineageOS), to wyczyszczenie wszystkich danych użytkownika. Musimy się tych
danych pozbyć, bo szereg aplikacji systemowych (i ich ustawień) może zwyczajnie nie być
kompatybilnych z nowym ROM'em, przez co telefon może nam się już nie uruchomić albo będzie się
zapętlał przy starcie systemu. Dlatego za każdym razem jak zmieniamy ROM na inny, tutaj stock
Android10/MIUI12 na crDroid Anadroid11, to trzeba całą partycję `/data/` wyczyścić.

Oczywiście nic nie stoi na przeszkodzie, by dokonać [backup'u pojedynczych aplikacji (i ich
ustawień), np. przy pomocy OAndBackupX i Syncthing][8]. Trzeba jednak pamiętać, że nie wszystkie
aplikacje da radę w ten sposób odtworzyć, np. Signal, który robi użytek z Androidowego KeyStore.
Takie aplikacje trzeba będzie na nowo sobie skonfigurować. Niemniej jednak, ogromna większość
aplikacji użytkownika może zostać w ten sposób przeniesiona ze starego ROM'u na nowy, oszczędzając
nam przy tym czas potrzebny na przygotowanie naszego telefonu do pracy po wgraniu nowego
oprogramowania.

By wyczyścić dane użytkownika, w SHRP recovery przechodzimy do `Wipe` > `Advanced Wipe` i tam
najpierw czyścimy `Cache` oraz `Dalvik`/`ART Cache` :

|   |   |   |
|---|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-shrp](/img/2021/09/001.crdroid-android-rom-xiaomi-redmi-9-shrp.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-wipe](/img/2021/09/002.crdroid-android-rom-xiaomi-redmi-9-shrp-wipe.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-wipe-cache](/img/2021/09/003.crdroid-android-rom-xiaomi-redmi-9-shrp-wipe-cache.png#small) |

Następnie z poziomu `Wipe` wybieramy `Format Data` i potwierdzamy przez wpisanie `yes` :

|   |   |
|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-shrp-wipe](/img/2021/09/004.crdroid-android-rom-xiaomi-redmi-9-shrp-wipe.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-wipe-format-data](/img/2021/09/005.crdroid-android-rom-xiaomi-redmi-9-shrp-wipe-format-data.png#small) |

Ten krok efektywnie usunie wszystkie dane z partycji `/data/` . Zostaną także usunięte klucze
szyfrujące. Później podczas konfigurowania nowego ROM'u zostaną wygenerowane nowe klucze, przez co
dane na partycji `/data/` zostaną zaszyfrowana ponownie. Usunięcie starych kluczy uniemożliwia
odzyskanie danych użytkownika. Upewnijmy się zatem, czy aby na pewno mamy zrobiony backup
wszystkich istotnych dla nas rzeczy.

## Aktualizacja firmware Xiaomi Redmi 9 (lancelot/galahad)

W tym przypadku dokonujemy przesiadki na ROM, który bazuje na Androidzie 11. Niemniej jednak,
Xiaomi na mojego Redmi 9 wypuścił ROM jedynie z Androidem 10 (przynajmniej póki co). Nowsze ROM'y,
tj. te z nowszym Androidem, mają zwykle nowszy firmware. Dlatego też ROM z Androidem 11 może nam
nie chcieć działać poprawnie z firmware z Andka 10 (podobnie w drugą stronę).

Jakiś czas temu próbowałem zaktualizować firmware swojego Redmi 9 do wersji z Androida 11 ale
ROM (to był Pixel Extended, który bazował na Andrku 11) nie potrafił obsłużyć nowego
firmware, [czego efektem było uszkodzenie IMEI][9] (system nie był go w stanie odczytać). W
zasadzie nic większego się nie stało, bo po wgraniu starszego firmware, wszystko wróciło do normy.
By uniknąć takich właśnie niezbyt przyjemnych sytuacji, trzeba zadbać o to, by przed wgraniem
nowego ROM'u, na urządzeniu znalazł się odpowiedni firmware. Zwykle w instrukcjach instalacji
danego ROM'u jest wzmianka na temat firmware, którego ROM potrzebuje i podobnie jest w
przypadku [crDroid Android][19].

Jeśli nasz Xiaomi Redmi 9 miał wgrany fabryczny ROM z Androidem 11, to krok aktualizacji firmware
możemy pominąć. Jeśli jednak chcemy przejść na crDroid Android z Androida10/MIUI12, to trzeba
będzie pierw [pobrać stosowny firmware][10] i wrzucić go na telefon przy pomocy trybu recovery. Nas
interesuje pozycja `stable V12.5.1.0.RJCEUXM 11.0 Europe` (z dnia 2021-09-13). W zaleceniach co do
wersji firmware jest informacja, by wgrać firmware dla regionu `Global` , a nie tak jak wyżej
`Europe` . Przetestowałem obie wersje firmware i w zasadzie nie zauważyłem żadnej różnicy w ich
działaniu. Biorac jednak pod uwagę fakt, że ten europejski firmware jest nowszy, to postanowiłem go
wgrać na stałe. Po pobraniu tego pliku, instalujemy go w standardowy sposób:

|   |   |   |
|---|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-shrp](/img/2021/09/006.crdroid-android-rom-xiaomi-redmi-9-shrp.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-flash-firmware](/img/2021/09/007.crdroid-android-rom-xiaomi-redmi-9-shrp-flash-firmware.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-flash-firmware-success](/img/2021/09/008.crdroid-android-rom-xiaomi-redmi-9-shrp-flash-firmware-success.png#small) |

## Wgranie ROM'u crDroid Anadroid na Redmi 9

Mając zaktualizowany firmware, możemy w końcu przejść do instalacji [crDroid Android][11]. W
zasadzie instalacja tego ROM'u niczym się nie różni od wgrywania samego firmware. Wystarczy pobrać
paczkę `.zip` pilnując przy tym by była ona przeznaczona dla naszego urządzenia. Na tej
podlinkowanej wyżej stronie próżno jednak szukać za frazami `Redmi 9` czy `lancelot` albo
`galahad` . Wszystko dlatego, że sporo różnych urządzeń może wykorzystywać bardzo zbliżony hardware,
przez co nazwy kodowe wielu urządzeń nie mają w świecie alternatywnych ROM'ów już takiego znaczenia.
Wygląda na to, że nie tylko `lancelot` i `galahad` jest tym samym urządzeniem (przynajmniej z punktu
widzenia alternatywnych ROM'ów) ale również `shiva` i `lava` . Zatem ROM na którekolwiek z tych
urządzeń powinien pasować na każde z pozostałych. Na stronie crDroid mamy wzmiankę na temat
urządzenia `lava` (Redmi G80 Series) i to właśnie ta pozycja we wspieranych oficjalnie urządzeniach
nas interesuje dla naszego Redmi 9.

![crdroid-android-rom-xiaomi-redmi-9-download-crdroid](/img/2021/09/009.crdroid-android-rom-xiaomi-redmi-9-download-crdroid.png#huge)

Klikamy naturalnie w ten zielony przycisk z napisem `Download latest version` i po chwili powinno
rozpocząć się pobieranie ROM'u crDroid Android. Jak możemy zauważyć, ten ROM waży jedynie 873.9 MiB.
Jeśli zamierzamy korzystać z GAPPS'ów, to naturalnie trzeba dociągnąć stosowną paczkę.

Po pobraniu ROM'u, wrzucamy go na telefon i instalujemy z poziomu SHRP recovery:

|   |   |
|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-shrp-flash-crdroid-rom](/img/2021/09/010.crdroid-android-rom-xiaomi-redmi-9-shrp-flash-crdroid-rom.png#small) | ![crdroid-android-rom-xiaomi-redmi-9-shrp-flash-crdroid-rom](/img/2021/09/011.crdroid-android-rom-xiaomi-redmi-9-shrp-flash-crdroid-rom.png#small) |

### GAPPS (opcjonalne)

Póki co jeszcze nie testowałem wariantu crDroid Android z wgranymi GAPPS'ami, bez znaczenia czy
tymi rekomendowanymi czy zwykłymi [OpenGAPPS w wersji pico][12], także póki co nie mogę nic więcej
na temat integracji GAPPS w crDroid Android powiedzieć. Być może w przyszłości coś tutaj dopiszę,
choć jest to dość wątpliwe, bo microG zdaje się na tym ROM'ie działać wyśmienicie. Jeśli jednak
zdecydowalibyśmy się na wgranie GAPPS, to stosowną paczkę ZIP również trzeba będzie pobrać i
umieścić na telefonie i podobnie, jak we wcześniejszych przypadkach, zainstalować z poziomu SHRP
recovery.

## Ponownie uruchomienie

Reasumując: obecnie mamy wyczyszczony telefon z danych użytkownika, poleciał też cały cache,
firmware smartfona mamy aktualny, no i oczywiście wgraliśmy crDroid Android (i opcjonalnie też
GAPPS). Po przeprowadzeniu tych wszystkich czynności z poziomu SHRP trzeba zrestartować telefon.
Pierwsze uruchomienie telefonu zajmie dłuższą chwilę ale nie ma powodów do obaw. Jeśli tylko widzimy
na ekranie logo crDroid Android, to znaczy, że system się uruchamia i po paru minutach powinien się
w pełni zainicjować.

![crdroid-android-rom-xiaomi-redmi-9-crdroid-status](/img/2021/09/012.crdroid-android-rom-xiaomi-redmi-9-crdroid-status.jpg#small)

## Root przy pomocy Magisk

Po wykonaniu pierwszego uruchomienia i przejściu przez przewodnika konfiguracji, możemy ponownie
przełączyć się w tryb SHRP recovery w celu wgrania [Magisk'a][13]. Pobieramy najnowszą dostępną
wersję Magisk'a (obecnie jest to `Magisk-v23.0.apk` ). Trzeba ten plik APK wgrać przez SHRP
recovery, dokładnie w taki sam sposób jak instaluje się zwykłe pliki `.zip` . Po wgraniu pliku APK,
uruchamiamy telefon ponownie w zwykłym trybie, by dokończyć instalację Magisk'a z poziomu Androida.

## Instalacja i konfiguracja microG

Magisk nie tylko przyda nam się do uzyskania praw root administratora systemu ale również przy jego
pomocy zainstalujemy [microG][14]. Sam microG składa się z kilku mniejszych pakietów i jakiś czas
temu w artykule [Czy smartfon z Androidem bez Google Apps/Services ma sens][15] opisywałem jak
manualnie wszystkie te składowe microG pogodzić ze sobą. Od czasu napisania tamtego artykułu, świat
nieco poszedł do przodu i nie trzeba już ręcznie konfigurować microG, tak jak to zostało tam
przedstawione. Chodzi generalnie o [moduł Magisk'a zwany microG Installer Revived][17].

|   |   |
|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-magisk-microg](/img/2021/09/013.crdroid-android-rom-xiaomi-redmi-9-magisk-microg.jpg#small) | ![crdroid-android-rom-xiaomi-redmi-9-magisk-microg-info](/img/2021/09/014.crdroid-android-rom-xiaomi-redmi-9-magisk-microg-info.jpg#small) |

Przy pomocy tego modułu, wszystkie potrzebne pliki zostaną umieszczone w swoich lokalizacjach i
odpowiednio skonfigurowane do pracy. Praktycznie nic od nas nie jest wymagane, by microG zaczął
poprawnie funkcjonować. Co ciekawe ROM crDroid Android wspiera [Signature Spoofing][16], zatem
odpada nam też potrzeba wgrywania i konfigurowania Xposed.

|   |   |
|---|---|
| ![crdroid-android-rom-xiaomi-redmi-9-microg-settings](/img/2021/09/015.crdroid-android-rom-xiaomi-redmi-9-microg-settings.jpg#small) | ![crdroid-android-rom-xiaomi-redmi-9-microg-self-check](/img/2021/09/016.crdroid-android-rom-xiaomi-redmi-9-microg-self-check.jpg#small) |

### A co z SafetyNet

Zatem wystarczy zainstalować microG przez Magisk'a i mamy praktycznie skonfigurowany do pracy ROM i
do tego pozbawiony całego własnościowego oprogramowania od Google, w tym tego całego Google Play
Services. Jedyny minus tego rozwiązania jest taki, że nasz telefon nigdy (najwyraźniej) nie
przejdzie testów SafetyNet.

Szukając informacji na temat problemów z SafetyNet na ROM'ach pozbawionych aplikacji od Google,
natrafiłem na szereg wzmianek, że od jakiegoś czasu, [Google sukcesywnie pracuje nad
rozwiązaniami][18], by ROM'y, w których nie ma nawet minimalnego GAPPS'a, nie mogły przejść testów
SafetyNet. Zatem wygląda na to, że jeśli będziemy chcieli przejść te testy SafetyNet, to trzeba
będzie jakiś pakiet GAPPS'ów niestety zainstalować.

## Podsumowanie

Nie ukrywam, że bardzo się cieszę z faktu przejścia na otwartoźródłowy ROM na bazie AOSP/LineageOS,
choć w dalszym ciągu firmware tego telefonu pozostaje zamknięty. Wyrzucenie z telefonu aplikacji
Google oraz innego zbędnego oprogramowania na pewno przyczyni się do poprawy naszej prywatności ale
najwyraźniej Google się zapiera wszystkimi możliwymi łapkami, by do takiego stanu rzeczy nie
dopuścić.

Uniemożliwienie użytkownikom telefonów z Androidem przejścia testów SafetyNet tylko dlatego, że nie
mają oni wgranych aplikacji od Google nie jest zbyt miłym zagraniem ze strony Google. Brak przejścia
testów SafetyNet skutkować będzie niemożliwością uruchamiania się na takim urządzeniu różnych
aplikacji, które od SafetyNet zależą, np. aplikacje bankowe, choć Revolut zdaje się działać bez
większego problemu. Jeśli jednak potrzebujemy korzystać z takich appek, to trzeba rozważyć
zainstalowanie nawet tego minimalnego pakietu GAPPS, przynajmniej na razie. Być może w przyszłości
uda się jakoś temu problemowi zaradzić.


[1]: https://forum.xda-developers.com/t/rom-11-official-pixelplusui-for-redmi-9-poco-m2-lancelot-shiva.4254891/
[2]: https://forum.xda-developers.com/t/rom-11-pixel-extended-official-redmi-9-poco-m2-lava.4296941/
[3]: https://forum.xda-developers.com/t/rom-official-crdroidandroid-for-redmi-9-poco-m2-lancelot-galahad-shiva.4333181/
[4]: /post/jak-odblokowac-bootloader-w-xiaomi-redmi-9-galahad-lancelot/
[5]: /post/jak-wgrac-twrp-recovery-i-magisk-w-xiaomi-redmi-9-galahad-lancelot/
[6]: https://shrp.github.io/
[7]: https://sourceforge.net/projects/shrp/files/lancelot/
[8]: /post/kopia-zapasowa-danych-smartfona-z-androidem-oandbackupx-syncthing/
[9]: https://forum.xda-developers.com/t/all-devices-xiaomi-firmware-updater-v5-auto-updated-daily.3741446/page-37#post-85610467
[10]: https://xiaomifirmwareupdater.com/firmware/lancelot/
[11]: https://crdroid.net/
[12]: https://github.com/opengapps/opengapps/wiki/Pico-Package
[13]: https://github.com/topjohnwu/Magisk
[14]: https://microg.org/
[15]: /post/czy-smartfon-z-androidem-bez-google-apps-services-ma-sens/
[16]: https://github.com/microg/GmsCore/wiki/Signature-Spoofing
[17]: https://github.com/nift4/microg_installer_revived
[18]: https://github.com/microg/RemoteDroidGuard/issues/24
[19]: https://telegra.ph/Firmware-Download-Links-09-12
[20]: /post/jak-wgrac-twrp-recovery-i-magisk-w-xiaomi-redmi-9-galahad-lancelot/#utworzenie-kopi-zapasowej-flasha-telefonu
