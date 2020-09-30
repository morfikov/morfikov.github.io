---
author: Morfik
categories:
- Android
date: "2017-03-04T21:56:54Z"
date_gmt: 2017-03-04 20:56:54 +0100
published: true
status: publish
tags:
- smartfon
- lollipop
- xposed
- marshmallow
title: 'Android: Framework Xposed i moduły do YouTube'
---

Stock'owe Androidy w smartfonach mają ten problem, że zawierają całą masę preinstalowanych aplikacji
od Google. Nie to by jakoś mnie to bolało, no może za wyjątkiem braku możliwości ich wywalenia czy
wyłączenia. To co mnie trochę irytuje, to fakt obecności reklam w aplikacji YouTube. Nie da rady się
ich pozbyć praktycznie w żaden sposób. Zdaję sobie sprawę, że serwis YT można przeglądać w
Firefox'ie i jeśli mamy [zainstalowanego w telefonie adblock'a, np.
AdAway](/post/android-blokowanie-reklam-z-adaway-na-smartfonie/), czy też [wdrożony
podobny filtr na domowym routerze WiFi z
LEDE/OpenWRT](/post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/), to te
reklamy mogą zostać z powodzeniem odfiltrowane, przynajmniej w Firefox'ie. Jestem też świadom
istnienia [aplikacji
NewPipe](/post/android-youtube-bez-reklam-na-smartfonie-newpipe/) , która jest
zubożonym klientem YouTube. Niemniej jednak, te opisane wyżej sposoby mają jedną podstawową wadę.
Mianowicie tracimy lwią część funkcjonalności serwisu YouTube. Przykładem mogą być powiadamiania w
przypadku, gdy na jeden z subskrybowanych kanałów zostanie wrzucony jaki materiał video. Taką opcję
ma ta aplikacja od Google ale klikając w powiadomienie jest niemal pewne, że włączy nam się jakaś
wredna reklama o wiele głośniejsza niż sam filmik, który zamierzamy obejrzeć. Innym problemem w
przypadku tej góglowskiej aplikacji jest brak możliwości odtwarzania video w tle czy też przy
zgaszonym wyświetlaczu. Postanowiłem w końcu wziąć się za ogarnięcie tej góglowskiej aplikacji
YouTube i wyeliminować te drażniące mnie problemy [instalując w smartfonie framework
Xposed](http://repo.xposed.info/module/de.robv.android.xposed.installer) wraz z odpowiednimi
modułami: [YouTube Background
Playback](http://repo.xposed.info/module/com.pyler.youtubebackgroundplayback) oraz [YouTube
AdAway](http://repo.xposed.info/module/ma.wanam.youtubeadaway). Jako, że nie jest to proces łatwy,
to postanowiłem go opisać krok po kroku.

<!--more-->
## Ukorzeniony Android (root) oraz backup flash'a smartfona

Może i Instalator Xposed można zainstalować na Androidzie, który nie przeszedł procesu root, ale
pełna funkcjonalność framework'a Xposed zostanie aktywowana dopiero w momencie, gdy ukorzenimy
sobie system. Naturalnie ta pełna funkcjonalność obejmuje instalowanie dodatkowych modułów. Zatem
jak widzimy, bez root sam instalator jest w zasadzie bezużyteczny.

Ten artykuł jest pisany w oparciu o smartfony Neffos, a konkretnie o modele C5 MAX oraz Y5, jako, że
dysponują one różnymi platformami sprzętowymi (Mediatek i Qualcomm) oraz różnymi wersjami Androida
(Lollipop oraz Marshmallow). Jeśli dysponujemy smartfonami Neffos C5 lub Y5L, to naturalnie poniżej
opisane kroki można zastosować również i do tych modeli. Niemniej jednak, by cokolwiek zacząć robić
potrzebny jest ukorzeniony system. Stosowne howto można znaleźć w osobnych wątkach: [Neffos C5
MAX](/post/android-root-smartfona-neffos-c5-max-od-tp-link/), [Neffos
C5](/post/android-root-smartfona-neffos-c5-od-tp-link/), [Neffos
Y5L](/post/android-root-smartfona-neffos-y5l-tp-link/) i [Neffos
Y5](/post/android-root-smartfona-neffos-y5-od-tp-link/).

Druga sprawa, to backup danych zgromadzonych w telefonie. Niby nic naszemu smartfonowi nie powinno
się stać z racji instalowania w nim framework'a Xposed ale lepiej dmuchać na zimne i zrobić sobie
kopię zapasową partycji `/system/` oraz `/data/` , a w ogóle to najlepiej zrobić pełny backup
flash'a i w razie problemów przywrócić konkretne partycje. Informacje na temat tworzenia kopi
zapasowej flash'a w smartfonach Neffos można znaleźć w wyżej podlinkowanych artykułach dotyczących
przeprowadzania procesu root.

Poniższa część artykułu zakłada, że mamy już ukorzeniony system i w razie ewentualnych problemów
wiemy jak cofnąć w nim wprowadzone zmiany.

## Instalacja instalatora Xposed

W zasadzie to musimy zainstalować w systemie dwie rzeczy: Instalator Xposed do zarządzania modułami
oraz właściwy framework Xposed. Oba te pliki są dostępne [w oficjalnym wątku na
XDA](https://forum.xda-developers.com/showthread.php?t=3034811).

Sam instalator instalujemy z poziomu działającego Androida. Jeśli mamy dodatkowo zainstalowany
F-Droid lub XDA Labs, to z ich repozytoriów również możemy ten instalator sobie wgrać na smartfona.
Natomiast framework można zainstalować przez instalator Xposed lub też z poziomu TWRP recovery
(ręcznie lub inicjując proces za pomocą instalatora Xposed).

## Instalacja framework'a Xposed

Plik instalatora Xposed jest wspólny dla każdej wersji Androida (5.0, 5.1 i 6.0). Natomiast plik z
framework'iem Xposed trzeba dobrać w zależności od wersji Androida i architektury systemowej. W tym
przypadku mamy Androida 6.0 Marshmallow oraz Androida 5.1 Lollipop. Architektury z kolei to ARM64 i
ARM32.

![](/img/2017/03/001.xposed-android-lollipop-marshmalow-tp-link-smartfon-platforma-arch.png#big)

### Framework Xposed i Android 6.0 (Marshmallow)

Niezależnie od wybranego sposobu instalacji instalatora Xposed (ręczne wgranie pliku `.apk` ,
F-Droid czy XDA Labs), musimy jeszcze sobie zainstalować sam framework. Odpalamy zatem instalator
Xposed i klikamy na numerku wersji, by zainstalować framework:

![](/img/2017/03/002.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-y5-instalacja.png#medium)

Jak widzimy, mamy do wyboru dwie opcje. Ja wybrałem sobie "Install" ale można też bez problemu
zainstalować przez TWRP recovery. Po chwili proces instalacji powinien się zakończyć powodzeniem, a
smartfon uruchomi się ponownie:

![](/img/2017/03/003.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-y5-instalacja.png#big)

Urządzenie się będzie dość długo uruchamiać ponownie i nie powinniśmy się tym faktem przejmować.
Pojawi się także informacja o optymalizowaniu wszystkich aplikacji w smartfonie, co potrwa dłuższą
chwilę. Warto tutaj zaznaczyć, że to optymalizowanie aplikacji będzie miało miejsce zawsze po
instalowaniu jak i usuwaniu framework'a Xposed.

Mając zainstalowany framework możemy przejść do instalacji interesujących nas modułów.

![](/img/2017/03/004.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-y5-instalacja.png#medium)

### Framework Xposed i Android 5.1 (Lollipop)

Instalacja framework'a Xposed na smartfonach Neffos C5 MAX oraz C5 sprawia nieco więcej problemów,
niż w przypadku Neffos'ów Y5 i Y5L. Prawdopodobnie winna jest tutaj starsza wersja Androida, która
nie jest zbytnio kompatybilna z samym framework'iem. Niemniej jednak, w dalszym ciągu Xposed może
zostać zainstalowany i uruchomiony z powodzeniem o ile zastosujemy się do poniższych wskazówek.

Przede wszystkim, nie możemy zainstalować najnowszej wersji frameworka (obecnie jest to v87). Ta
wersja powoduje jakieś bliżej nieznane problemy objawiające się następującym komunikatem w oknie
instalatora: Wersja 87 Xposed framework jest zainstalowana ale nie jest aktywna. Proszę sprawdzić
logi, aby poznać szczegóły.

![](/img/2017/03/005.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-c5-max-bledy.png#big)

Po zajrzeniu w logi, mamy taką informację:

    -----------------
    Starting Xposed version 87, compiled for SDK 22
    Device: Neffos C5 Max (TP-LINK), Android version 5.1 (SDK 22)
    ROM: H10S103D01B20160812R1035
    Build fingerprint: Neffos/TP702A/C5_Max:5.1/LMY47D/H10S103D01B20160812R1035.160811:user/release-keys
    Platform: arm64-v8a, 64-bit binary, system server: yes
    SELinux enabled: yes, enforcing: yes
    : -----------------
    : Added Xposed (/system/framework/XposedBridge.jar) to CLASSPATH
    : Detected ART runtime
    : Found Xposed class 'de/robv/android/xposed/XposedBridge', now initializing
    : Error while loading XResources class 'android/content/res/XResources':
    : java.lang.IncompatibleClassChangeError: xposed.dummy.XResourcesSuperClass
    :   at java.lang.Class dalvik.system.DexFile.defineClassNative(java.lang.String, java.lang.ClassLoader, long) (DexFile.java:-2)
    :   at java.lang.Class dalvik.system.DexFile.defineClass(java.lang.String, java.lang.ClassLoader, long, java.util.List) (DexFile.java:226)
    :   at java.lang.Class dalvik.system.DexFile.loadClassBinaryName(java.lang.String, java.lang.ClassLoader, java.util.List) (DexFile.java:219)
    :   at java.lang.Class dalvik.system.DexPathList.findClass(java.lang.String, java.util.List) (DexPathList.java:321)
    :   at java.lang.Class dalvik.system.BaseDexClassLoader.findClass(java.lang.String) (BaseDexClassLoader.java:54)
    :   at java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String, boolean) (ClassLoader.java:511)
    :   at java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String, boolean) (ClassLoader.java:504)
    :   at java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String) (ClassLoader.java:469)
    :   at java.lang.Class dalvik.system.DexFile.defineClassNative(java.lang.String, java.lang.ClassLoader, long) (DexFile.java:-2)
    :   at java.lang.Class dalvik.system.DexFile.defineClass(java.lang.String, java.lang.ClassLoader, long, java.util.List) (DexFile.java:226)
    :   at java.lang.Class dalvik.system.DexFile.loadClassBinaryName(java.lang.String, java.lang.ClassLoader, java.util.List) (DexFile.java:219)
    :   at java.lang.Class dalvik.system.DexPathList.findClass(java.lang.String, java.util.List) (DexPathList.java:321)
    :   at java.lang.Class dalvik.system.BaseDexClassLoader.findClass(java.lang.String) (BaseDexClassLoader.java:54)
    :   at java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String, boolean) (ClassLoader.java:511)
    :   at java.lang.Class java.lang.ClassLoader.loadClass(java.lang.String) (ClassLoader.java:469)
    :   at boolean de.robv.android.xposed.XposedBridge.initXResourcesNative() (XposedBridge.java:-2)
    :   at void de.robv.android.xposed.XposedInit.hookResources() (XposedInit.java:217)
    :   at void de.robv.android.xposed.XposedBridge.main(java.lang.String[]) (XposedBridge.java:87)
    : Cannot hook resources
    : Cannot load any modules because /data/data/de.robv.android.xposed.installer/conf/modules.list was not found
    : -----------------

Wersja, która zdaje się działać, to v85 i to ją musimy sobie wgrać na smartfona. Wystarczy, że w
oknie instalatora Xposed, z prawego górnego rogu rozwiniemy menu i wybierzemy "Show outdated
versions", a pojawi nam się lista wersji, które będziemy w stanie zainstalować. Jest tam też min.
v85. Wybieramy i instalujemy ją:

![](/img/2017/03/006.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-c5-max-instalacja.png#big)

Naturalnie smartfon się będzie dłużej uruchamiał i w tym przypadku również doświadczymy
optymalizowania aplikacji, co potrwa kilka minut. Po uruchomieniu instalatora Xposed, zobaczymy ten
sam komunikat ostrzeżenia, który był widoczny wyżej. By go zlikwidować, trzeba wejść w Ustawienia i
zaznaczyć opcję "Wyłącz zasoby konfliktów". Po zresetowaniu systemu, framework powinien już się
załadować bez problemu:

![](/img/2017/03/007.xposed-android-lollipop-marshmalow-tp-link-smartfon-neffos-c5-max-opcje.png#big)

Ta opcja "Wyłącz zasoby konfliktów" powoduje, że część modułów Xposed nie będzie nam funkcjonować
prawidłowo (min. YouTube AdAway). Niemniej jednak, dobrze chociaż, że jakiekolwiek moduły w ogóle
będą w stanie działać.

## Instalacja modułów Xposed

Moduły do Xposed możemy instalować bezpośrednio z instalatora. Możemy też do tego celu zaprzęgnąć
[aplikację XDA Labs](/post/xda-labs-repozytorium-aplikacji-modulow-xposed/), która
również ma wbudowane repozytorium modułów Xposed.

![](/img/2017/03/008.xposed-android-lollipop-marshmalow-tp-link-smartfon-xda-labs-repo.png#medium)

W tym przypadku interesują nas dwa moduły: YouTube Background Playback oraz YouTube AdAway. Ich
instalacja sprowadza się do odszukania konkretnego modułu i pobrania go:

![](/img/2017/03/009.xposed-android-lollipop-marshmalow-tp-link-smartfon-xda-labs-moduly.png#huge)

Po zainstalowaniu modułów Xposed, te nie są z automatu aktywne, o czym jesteśmy informowani w
notyfikacjach systemowych. Wszystkie moduły mają swój własny autostart i możemy je dowolnie włączać
lub wyłączać:

![](/img/2017/03/010.xposed-android-lollipop-marshmalow-tp-link-smartfon-aktywacja-moduly.png#big)

Wszystkie te wyżej zaznaczone moduły będą startowane wraz z systemem. Jeśli jakieś moduły chcemy
czasowo wyłączyć, to nie musimy ich wywalać z systemu, tylko wystarczy odptakować stosowną pozycję
na liście modułów. Każda zmiana konfiguracji modułów musi być załadowana ponownie, a do tego celu
trzeba uruchomić ponownie system w telefonie.

## Odinstalowanie framework'a Xposed i jego modułów

Cały framework Xposed wraz z jego modułami można naturalnie odinstalować. W tym celu wystarczy
odpalić instalator Xposed i odinstalować framework:

![](/img/2017/03/011.xposed-android-lollipop-marshmalow-tp-link-smartfon-usuwanie.png#medium)

Tutaj również mamy opcje usuwania bezpośrednio z systemu jak i przez tryb TWRP recovery. Oba
działają bez zarzutu. Jeśli mieliśmy wgrane jakieś moduły Xposed, to one nie zostaną usunięte przy
deinstalacji framework'a. Każdy moduł trzeba ręcznie usunąć, np. przez XDA Labs lub instalator
Xposed. Po usunięciu modułów, instalator można usunąć w XDA Labs lub F-Droid.

![](/img/2017/03/012.xposed-android-lollipop-marshmalow-tp-link-smartfon-usuwanie-instalator-moduly.png#big)
