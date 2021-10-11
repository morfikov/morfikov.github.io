---
author: Morfik
categories:
- Android
date:    2021-10-08 23:47:00 +0200
lastmod: 2021-10-08 23:47:00 +0200
published: true
status: publish
tags:
- smartfon
- magisk
- webview
- root
- prywatność
- aosp
- xiaomi
- redmi-9
GHissueID: 334
title: Zmiana implementacji WebView z Google/AOSP na Bromite w Androidzie
---

Przeglądając ostatnio opcje deweloperskie w swoim telefonie z Androidem 11, wpadła mi w oczy pozycja
`WebView implementation` . Nie ukrywam, że trochę mnie ona zainteresowała i zacząłem się
zastanawiać czym tak naprawdę jest ten cały WebView. Mój smartfon działa aktualnie pod kontrolą
crDroid (ROM na bazie AOSP/LineageOS) i nie jest on sprzęgnięty z usługami od Google (brak
jakichkolwiek GAPPS'ów). Dlatego też w tym przypadku w implementacji WebView widnieje w zasadzie
tylko jedna opcja, tj. Android System WebView. W przypadku stock'owych ROM'ów producentów telefonów
będziemy mieli zaś do czynienia z Google System WebView. Jakby nie patrzeć, zarówno Android/AOSP
System WebView, jak i Google System WebView pochodzą od Google, który niezbyt troszczy się o naszą
prywatność. W mojej głowie pojawiło się zatem pytanie na temat tego czym te dwie implementacje się
od siebie różnią, no i naturalnie też czy są jakieś alternatywne implementacje WebView, z których
można by skorzystać zastępując te domyślnie preinstalowane?

<!--more-->
## Co to jest WebView

[WebView to taki mechanizm][5], z którego korzystają aplikacje Androida do generowania zawartości
stron WWW. Zapewne każdy z nas spotkał się z taki appkami, które domyślnie otwierają linki w swoim
obrębie, dając też opcję otwarcia takich stron w zewnętrznej przeglądarce internetowej. Za taką
funkcjonalność właśnie odpowiada WebView, który umożliwia zaimplementowanie prostej przeglądarki
internetowej wewnątrz danej aplikacji bez potrzeby pchania do niej całego kodu odpowiedzialnego za
realizowanie renderowania kodu HTML. W ten sposób deweloperzy aplikacji mogą zająć się rozwijaniem
samej aplikacji, a sprawy związane z renderowaniem stron WWW oddelegować do WebView. Każdy Android
ma preinstalowany taki WebView domyślnie, choć jego implementacje mogą się naturalnie różnić.

|   |   |
|---|---|
| ![](/img/2021/10/001.bromite-aosp-google-webview-developer-options.jpg#small) | ![](/img/2021/10/002.bromite-aosp-google-webview-developer-options.jpg#small) |

W przypadku integracji ROM'u z usługami od Google, za implementacje WebView będzie robił Google
System WebView (oparty na Chrome). W ten sposób każda strona WWW, która zostanie otworzona wewnątrz
jakiejś aplikacji użytkownika, będzie otwierana w środowisku Google, przez co Google ma pełny wgląd
w to, jakie witryny oglądamy i co na nich robimy. W zasadzie otworzenie takiej strony WWW niczym
nie różni się w stosunku do wrzucenia jej do pełnowymiarowej przeglądarki Google Chrome, gdzie
jesteśmy zalogowani, tj. mamy powiązane w niej konto Google.

W przypadku ROM'ów opartych na AOSP (np. LineageOS) bez integracji z usługami od Google, za WebView
będzie robił Android System WebView, który jest implementacją opartą o otwartoźródłowego Chromium.
Podobnie jak w przypadku Google System WebView, otworzenie strony w aplikacji, która korzysta z
Android System WebView, będzie równoznaczne z renderowaniem jej w przeglądarce Chromium,
przynajmniej w dużym uproszczeniu.

### Czym się różni WebView Android/AOSP od Google

Jako, że WebView robi za silnik renderujący kod HTML, to kluczowe z perspektywy bezpieczeństwa jest,
by ten [mechanizm był aktualizowany dość często][4]. W przypadku Google System WebView,
aktualizacje są wypuszczane regularnie (bo mamy tutaj do czynienia ze zwykłą aplikacją, którą można
pobrać przez sklep Google Play). Natomiast jeśli chodzi o Android/AOSP System WebView, to z tymi
update'ami różnie bywa, bo trzeba tutaj zaktualizować cały firmware/ROM. Warto zatem się zastanowić
w tym miejscu czy ten ROM, z którego aktualnie korzystamy, dostaje regularnie jakieś aktualizacje
(zwykle wydawane w interwale miesięcznym) i jeśli od dłuższego czasu nie było takiego update, to z
punktu widzenia bezpieczeństwa powinniśmy zainstalować ten WebView od Google.

Pomijając kwestię aktualizacji, wygląda na to, że te dwie implementacje WebView nie różnią się
zbytnio od siebie, choć nie do końca wiadomo czy Google nie dokłada czegoś własnościowego do
swojego WebView (różne są zdania na ten temat). Tak czy inaczej, rezygnacja z WebView AOSP/Google i
zastąpienie jej inną implementacją może sprawić, że niektóre aplikacje nie będą działać poprawnie.
Dla przykładu, przeglądarki wewnątrz aplikacji użytkownika mogą crash'ować na innych implementacjach
WebView i jeśli doświadczamy takiego zachowania przy korzystaniu z nich, to trzeba będzie
prawdopodobnie powrócić do Google/AOSP System WebView i raczej tej bariery nie przeskoczymy.

### Czym jest Bromite

Android System WebView może i jest OpenSource ale w dalszym ciągu pochodzi od Google, a ten za
bardzo nie troszczy się o prywatność swoich użytkowników. Sam WebView nie ma też żadnych opcji
konfiguracyjnych, dzięki którym moglibyśmy dostosować ten aspekt przeglądania stron WWW wewnątrz
konkretnych aplikacji. Trzeba zatem zdać się na łaskę Google w kwestii prywatności, tj. wszystkie
informacje na temat naszej aktywności w takich aplikacjach korzystających z WebView będą przez
Google zbierane. Takie zachowanie jest domyślnie włączone również w implementacji Android/AOSP
System WebView, chyba, że twórca danego ROM'u ten ficzer wyłączył. Niemniej jednak, Google w
dalszym ciągu może nas śledzić przez WebView za sprawą nadużywania nagłówka `X-Requested-With` i z
tym za bardzo nie da się nic zrobić. Istnieje jednak inna implementacja WebView, tj. [Bromite
System WebView][3], która jest ukierunkowana właśnie na dostarczenie swoim użytkownikom maksimum
prywatności.

Problem jednak w tym, że tego całego WebView nie da się od tak po prostu wymienić. Jest to bardzo
kluczowy mechanizm i wręcz krytyczny pod względem bezpieczeństwa. Dlatego ta aplikacja rezyduje
gdzieś w obrębie partycji `/super/` (zwykle `/system/` lub `/product/` ), która w nowszych
Androidach (10+) jest dostępna tylko i wyłącznie do odczytu. Z poziomu działającego systemu nie da
rady tej partycji zamontować w trybie do zapisu. Dlatego też by zmienić implementację WebView,
trzeba będzie mieć ukorzenionego Androida, tj. posiadać w nim prawa administratora systemu root.

## Jak zmienić WebView od Google/AOSP na Bromite

Zgodnie z informacjami zawartymi na stronie projektu Bromite, [jest kilka metod by zainstalować ich
WebView][1]. W moim przypadku żadna z tych instrukcji nie przyniosła jakiegoś wymiernego efektu.
Zwykła próba zainstalowania `arm64_SystemWebView.apk` kończyła się niepowodzeniem. Można o tym
problemie też poczytać na podlinkowanym wyżej wiki. Chodzi generalnie o to, że plik `webview.apk`
rezyduje w obrębie partycji `/system/` , no i też jest podpisany innym kluczem. Nie da rady zatem
tej appki od tak sobie zainstalować w systemie, bo mechanizmy bezpieczeństwa nam na to zwyczajnie
nie pozwolą. Próbowałem też wyłączyć stock'owy WebView przy pomocy tego poniższego polecenia:

    $ adb shell "pm list packages | grep webview"
    package:com.android.webview

    $ adb shell "pm uninstall -k --user 0 com.android.webview"
    Success

Oczywiście udało się tę aplikację wyłączyć ale w dalszym ciągu nie szło zainstalować WebView od
Bromite. Przywróciłem zatem tę odinstalowaną appkę w poniższy sposób:

    $ adb shell "cmd package install-existing com.android.webview"
    Package com.android.webview installed for user: 0

### Kompatybilność ROM'u z Bromite

Pomyślałem może, że mój Android nie jest jakoś specjalnie kompatybilny z Bromite, więc sprawdziłem,
zgodnie z zaleceniem, co zwróci analiza pliku `/system/framework/framework-res.apk` (sygnatury w
linijkach z `C:` zostały przycięte dla czytelności):

    $ adb pull /system/framework/framework-res.apk
    /system/framework/framework-res.apk: 1 file pulled. 14.7 MB/s (53618124 bytes in 3.490s)

    $ aapt d xmltree framework-res.apk res/xml/config_webview_packages.xml
    E: webviewproviders (line=18)
      E: webviewprovider (line=20)
        A: availableByDefault=(type 0x12)0xffffffff
        A: description="Google WebView" (Raw: "Google WebView")
        A: packageName="com.google.android.webview" (Raw: "com.google.android.webview")
        E: signature (line=21)
          C: "MIIDuzCC...g69H8l8M"
      E: webviewprovider (line=23)
        A: availableByDefault=(type 0x12)0xffffffff
        A: description="Google WebView Beta" (Raw: "Google WebView Beta")
        A: packageName="com.google.android.webview.beta" (Raw: "com.google.android.webview.beta")
        E: signature (line=24)
          C: "MIIFxzCC...UbGtPA=="
      E: webviewprovider (line=26)
        A: availableByDefault=(type 0x12)0xffffffff
        A: description="Google WebView Dev" (Raw: "Google WebView Dev")
        A: packageName="com.google.android.webview.dev" (Raw: "com.google.android.webview.dev")
        E: signature (line=27)
          C: "MIIFxzCC...9+p2OA=="
      E: webviewprovider (line=29)
        A: availableByDefault=(type 0x12)0xffffffff
        A: description="Google WebView Canary" (Raw: "Google WebView Canary")
        A: packageName="com.google.android.webview.canary" (Raw: "com.google.android.webview.canary")
        E: signature (line=30)
          C: "MIIFxzCC......hrFdCQ=="
      E: webviewprovider (line=34)
        A: availableByDefault=(type 0x12)0xffffffff
        A: description="AOSP WebView" (Raw: "AOSP WebView")
        A: packageName="com.android.webview" (Raw: "com.android.webview")

Zgodnie z wymaganiami, powinniśmy mieć tutaj `packageName="com.android.webview"` bez sygnatury i
faktycznie, jak można wyżej zauważyć, przy tej ostatniej pozycji nie ma żadnej sygnatury. Zatem
najwyraźniej nic nie stoi na przeszkodzie, by ten WebView wymienić.

### Próba skorzystania z aplikacji /system/app mover

Próbowałem więc dalej i kolejnym sposobem było skorzystanie z [appki /system/app mover][2]. Ten
sposób również okazał się bezowocny. Najwyraźniej appka `/system/app mover` jest przeznaczona dla
starszych Androidów i próba przeniesienia aplikacji w Andku 11 z partycji systemowych na partycję
`/data/` nie daje żadnego efektu, a jedynie kończy się błędem
`Could not remount /system/` .

### Brak katalogu /system/app/webview/

Niezniechęcony jednak brakiem wymiernych efektów, próbowałem dalej. Kolejnym sposobem było ręczne
usunięcie pliku `webview.apk` z katalogu `/system/app/webview/` . Niestety w przypadku ROM'u
crDroid wgranego na mojego Xiaomi Redmi 9, taki katalog w ogóle nie istnieje. Nie poddałem się
jednak tak łatwo i przy pomocy narzędzia `find` postanowiłem przeszukać cały flash telefonu w
poszukiwaniu tego pliku, bo przecie gdzieś on być musi. No i znalazłem:

    galahad:/ # find / -iname "*webview.apk"
    ...
    /product/app/webview/webview.apk
    ...

Czyli `webview.apk` znajduje się tutaj nie pod `/system/app/webview/` , a pod
`/product/app/webview/` , ot taka drobna różnica. Niemniej jednak, usunięcie tego pliku ze
wskazanej wyżej lokalizacji też nie wchodzi w grę, bo system plików jest domyślnie zamontowany w
trybie tylko do odczytu.

### System plików tylko do odczytu

W Androidzie 11 mamy partycję `/super/` , na którą składają się trzy inne partycje, tj. `/system/` ,
`/vendor/` oraz `/product/` . Poszukałem zatem, które urządzenie odpowiada za tę partycję
`/product/` w celu przemontowania jej systemu plików w tryb do zapisu:

    galahad:/ # mount | grep -i product
    /dev/block/dm-2 on /product type ext4 (ro,seclabel,relatime)

Niby teraz wystarczyłoby przemontować tę partycję podając w `mount` flagę `-o remount,rw` ale (jak
to zwykle bywa) taki zabieg się również nie powiódł:

    galahad:/ # mount -o remount,rw /product/
    mount: '/product/' not in /proc/mounts

    galahad:/ # mount -o remount,rw /dev/block/dm-2
    '/dev/block/dm-2' is read-only

### Zaciągnięcie do pracy TWRP/SHRP

Trochę już zacząłem tracić cierpliwość i ostatnia rzecz, która wpadła mi do głowy, to odpalenie
trybu recovery TWRP/SHRP. Tam były opcje montowania partycji systemowych w trybie do zapisu, w tym
też tego nieszczęsnego `/product/` . Problem natomiast był taki, że z menu SHRP nie szło tej
partycji zamontować w trybie do zapisu, przynajmniej bezpośrednio z GUI. Ale od czego jest
terminal? Sprawdziłem zatem czy jak ręcznie będę próbował przemontować partycję `/product/` z
poziomu recovery, to czy coś to da:

    $ adb shell
    lancelot:/ # mount | grep product
    /dev/block/dm-2 on /product type ext4 (ro,seclabel,relatime)

    lancelot:/ # mount -o remount,rw /product

No i proszę, nie ma żadnego błędu. Sprawdziłem jeszcze by się upewnić, że coś w tym katalogu w
ogóle się znajduje i czy jest tam ten plik, o który się nam rozchodzi:

    lancelot:/ # ls -al /product/app/webview/
    total 159380
    drwxr-xr-x  3 root root      4096 2009-01-01 00:00 .
    drwxr-xr-x 19 root root      4096 2021-10-06 21:51 ..
    drwxr-xr-x  4 root root      4096 2009-01-01 00:00 oat
    -rw-r--r--  1 root root 163726925 2009-01-01 00:00 webview.apk

#### Backup webview.apk

Zatem wszystko wygląda dobrze. Postanowiłem na wszelki wypadek przenieść zarówno katalog `oat/` jak
i sam plik `webview.apk` na kartę SD (w razie czego będzie co przywrócić jakby się coś popsuło):

    lancelot:/product/app/webview # mv webview.apk /external_sd/
    lancelot:/product/app/webview # mv oat /external_sd/

#### Podmiana webview.apk na ten od Bromite

Mając backup pliku `webview.apk` możemy kontynuować z instalacją Bromite. Ze strony
projektu [pobieramy plik .apk][3]. W tym przypadku jest to `arm64_SystemWebView.apk` . Wgrywamy go
do katalogu `/product/app/webview/` zmieniając mu naturalnie nazwę na `webview.apk` :

    $ adb push arm64_SystemWebView.apk /sdcard/Download
    arm64_SystemWebView.apk: 1 file pushed. 12.5 MB/s (86437563 bytes in 6.608s)

    lancelot:/product/app/webview # mv /sdcard/Download/arm64_SystemWebView.apk ./webview.apk
    lancelot:/product/app/webview # ls -al
    total 84420
    drwxr-xr-x  2 root     root     4096 2021-10-06 23:00 .
    drwxr-xr-x 19 root     root     4096 2021-10-06 21:51 ..
    -rw-r--r--  1 root     root 86437563 2021-10-06 21:45 webview.apk

Sprawdźmy też uprawnienia do tak wgranego pliku `webview.apk` , bo czasami mogą one nie być
odpowiednie. Właściciel oraz grupa powinna wskazywać na `root:root` oraz prawa do pliku trzeba
określić na `644` . Jeśli po wgraniu `webview.apk` któreś z tych powyższych rzeczy się różni, to
trzeba te uprawnienia przepisać via `chown`/`chmod` .

    lancelot:/product/app/webview # chown root:root webview.apk
    lancelot:/product/app/webview # chmod 644 webview.apk

Na koniec montujemy partycję `/product/` z powrotem w tryb do odczytu:

    lancelot:/ # mount -o remount,ro /product

Z menu TWRP/SHRP czyścimy także cały cache, po czym restartujemy telefon:

    lancelot:/ # reboot

### Ponowna instalacja pliku .apk

Smartfon powinien uruchomić się bez problemu ale to nie wszystko co trzeba zrobić, by WebView od
Bromite zainstalować w systemie. Trzeba jeszcze raz wgrać paczkę `arm64_SystemWebView.apk` , tym
razem w zwyczajny sposób, tj. przez instalator Androida. Proces instalacji tej aplikacji powinien
zakończyć się powodzeniem.

## Sprawdzenie aktualnej implementacji WebView

Po procesie instalacyjnym wypadałoby jeszcze zweryfikować, czy aby na pewno przebiegł on tak jak
należy. Wchodzimy zatem w ustawienia deweloperskie i sprawdzamy czy pod `WebView implementation`
widnieje `Bromite System WebView` , tak jak na poniższym obrazku:

|   |   |
|---|---|
| ![](/img/2021/10/003.bromite-aosp-google-webview-developer-options.jpg#small) | ![](/img/2021/10/004.bromite-aosp-google-webview-developer-options.jpg#small) |

Jeśli mamy podobny wynik, to znaczy, że z powodzeniem udało nam się zastąpić implementację WebView
od Google/AOSP tą od Bromite.

### Aplikacja WebView Test

Warto też w tym miejscu dodać, że w sklepie Google Play jest dostępna aplikacja, która jest w stanie
zweryfikować, czy aktualna implementacja WebView działa prawidłowo. Chodzi o [appkę WebView
Test][7]. Wystarczy ją uruchomić i sprawdzić czy przykładowa strona WWW jest w stanie się w tej
aplikacji bez problemu wyrenderować.

![](/img/2021/10/005.bromite-aosp-google-webview-app-check.jpg#small)

Jeśli tak, to znaczy, że wszystko jest w porządku.

W aplikacji WebView Test jest też możliwość zweryfikowania aktualnie wykorzystywanej wersji silnika
WebView -- wystarczy z menu wybrać `WebView Info` :

![](/img/2021/10/006.bromite-aosp-google-webview-info.jpg#small)

Jak możemy zauważyć na powyższej fotce, w mamy tam sporo fraz z `Chrome` . Jest to [intencjonalne
zachowanie Bromite][8], który wykorzystuje szereg technik mających przeciwdziałać profilowaniu
przeglądarki w oparciu o jej fingerprint. Dlatego też te informacje nie do końca pokrywają się ze
stanem faktycznym i nie ma co do nich przykładać większej wagi. Kluczowa rzecz przy ustaleniu czy
posiadamy WebView od Bromite, to obecność stosownej implementacji w ustawieniach deweloperskich.

Możemy też zrobić test oferowany przez Bromite w tej appce WebView Test. Powinniśmy uzyskać wynik
5/9:

|   |   |
|---|---|
| ![](/img/2021/10/012.bromite-aosp-google-webview-test.jpg#small) | ![](/img/2021/10/013.bromite-aosp-google-webview-test.jpg#small) |

## Repozytorium F-Droid dla Bromite

Mając już wgrany Bromite System WebView, trzeba zatroszczyć się o jego regularną aktualizację.
Manualne odwiedzanie strony projektu, pobieranie nowszej wersji pliku `.apk` i ręczna aktualizacja
tej aplikacji nie jest zbyt wygodnym rozwiązaniem. Dlatego też [Bromite oferuje swoje własne
repozytorium][6], które możemy dodać do F-Droid. W ten sposób aktualizacja WebView będzie
przebiegać o wiele sprawniej i szybciej.

|   |   |
|---|---|
| ![](/img/2021/10/007.bromite-aosp-google-webview-f-droid-repo.jpg#small) | ![](/img/2021/10/008.bromite-aosp-google-webview-f-droid-repo.jpg#small) |

## Magisk i moduł WebView Switcher

Gdy już w zasadzie udało mi się ten WebView wymienić, to przez przypadek zauważyłem na liście
modułów Magisk'a, że widnieje tam pozycja `WebView Switcher` . Z opisu wygląda na to, że ten moduł
jest w stanie zrobić wszystkie te kroki poczynione wyżej ale bez wprowadzania żadnych zmian w
partycji `/system/` czy `/product/` .

|   |   |
|---|---|
| ![](/img/2021/10/009.bromite-aosp-google-webview-magisk-module.jpg#small) | ![](/img/2021/10/010.bromite-aosp-google-webview-magisk-module.jpg#small) |

Postanowiłem zatem przywrócić wszystko na swoje miejsce i sprawdzić jak WebView Switcher sobie
poradzi ze zmianą WebView w Androidzie mojego smartfona.

![](/img/2021/10/011.bromite-aosp-google-webview-magisk-module-install.jpg#small)

Po zainstalowaniu tego dodatku i uruchomieniu telefonu ponownie, wszystko zdaje się działać
dokładnie w ten sam sposób, co został opisany powyżej, choć patrząc za `webview` w `mount`
znalazłem jedynie te poniższe wpisy:

    galahad:/ # mount | grep -i webview
    /dev/HNTLL8/.magisk/block/data on /product/overlay/WebviewOverlay.apk type ext4 (rw,seclabel,relatime,nobarrier,noauto_da_alloc,inlinecrypt,resuid=10010,resgid=1065,errors=panic,data=ordered)
    /dev/HNTLL8/.magisk/block/data on /system/app/BromiteWebview type ext4 (rw,seclabel,relatime,nobarrier,noauto_da_alloc,inlinecrypt,resuid=10010,resgid=1065,errors=panic,data=ordered)
    /dev/HNTLL8/.magisk/block/system_root on /system/etc/permissions/android.software.webview.xml type ext4 (ro,seclabel,relatime)

Zatem wygląda na to, że `BromiteWebview` został wgrany do `/system/app/` , a ten problematyczny
plik `webview.apk` nie był w żaden sposób ruszony.

## Podsumowanie

Osoby, którym zależy na prywatności w sieci, zwłaszcza w kontekście przeglądania stron WWW z
poziomu swojego telefonu, powinny zainteresować się implementacją WebView, którą oferuje Bromite.
Niestety nie wszędzie wymiana tego kluczowego komponentu Androida będzie wchodzić w grę, bo do tego
celu potrzebne nam będą prawa root, a te z kolei nie są standardowo dostępne, gdy korzystamy z
fabrycznego oprogramowania producenta smartfona. Trzeba też mieć na uwadze ewentualne problemy,
które może ze sobą nieść wymiana silnika renderującego kod HTML, bo niektóre aplikacje mogą na
innych implementacjach WebView nie działać prawidłowo. Póki co jednak, nie zdarzyło mi się, by
appki, z których korzystam na co dzień (a jest ich dość sporo), zachowywały się w jakiś bliżej
nieprzewidywalny sposób. Można zatem przyjąć, że w przypadku tych otwartoźródłowych aplikacji,
problemów być raczej nie powinno.


[1]: https://github.com/bromite/bromite/wiki/Installing-SystemWebView
[2]: https://f-droid.org/en/packages/de.j4velin.systemappmover/
[3]: https://www.bromite.org/system_web_view
[4]: https://forum.f-droid.org/t/webkit-aosp-google-webview-gecko/486/7
[5]: https://developer.android.com/reference/android/webkit/WebView.html
[6]: https://www.bromite.org/fdroid
[7]: https://play.google.com/store/apps/details?id=com.snc.test.webview2
[8]: https://github.com/bromite/bromite/issues/138
