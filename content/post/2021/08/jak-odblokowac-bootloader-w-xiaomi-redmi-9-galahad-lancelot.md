---
author: Morfik
categories:
- Android
date:    2021-08-27 22:26:00 +0200
lastmod: 2021-08-27 22:26:00 +0200
published: true
status: publish
tags:
- xiaomi
- redmi-9
- xiaomitool
- debian
- kvm
- qemu
- bootloader
- smartfon
GHissueID: 340
title: Jak odblokować bootloader w Xiaomi Redmi 9 (galahad/lancelot)
---

Jakiś czas temu wpadł w moje łapki smartfon Xiaomi Redmi 9 (galahad albo lancelot, bo w różnych
częściach systemu jest to inaczej określone), który miał preinstalowanego Androida 10 oraz MIUI 11.
Przez parę miesięcy używania telefonu, dostał on dwa albo trzy większe update całego ROM'u,
wliczając w to aktualizację MIUI do 12.0.1 ze stanem zabezpieczeń na dzień 2021-01-05. Zatem
ostatnia aktualizacja zabezpieczeń tego telefonu miała miejsce zaraz na początku Stycznia. Od tego
czasu cisza. Niby w przypadku tego modelu telefonu aktualizacje miały być wydawane [co trzy
miesiące do roku 2023][10] ale najwyraźniej coś jest nie tak i urządzenie od ponad pół roku nie
dostało żadnych aktualizacji. Niby pod tym linkiem można wyczytać informację, że planowana jest
aktualizacja do Androida 11 ale prawdę mówiąc jestem nieco zawiedziony opieszałością Xiaomi. Tak
się złożyło, że przez przypadek trafiłem w [to miejsce na forum XDA][11], gdzie z kolei znalazłem
m.in. [ten wątek][12]. Zatem alternatywne ROM'y na mój smartfon istnieją i tego faktu nie byłem
świadomy, bo w zeszłym roku jeszcze nic nie szło znaleźć. Postanowiłem zatem odblokować bootloader
w swoim Xiaomi Redmi 9 i spróbować wgrać na niego TWRP i jeden (a może nawet kilka) przykładowy ROM
na bazie AOSP/LineageOS. Proces odblokowania bootloader'a w urządzeniach Xiaomi nie wymaga zbytnio
wysiłku i da się go przeprowadzić w całości pod linux korzystając czy to z XiaoMiTool, czy też przy
pomocy [maszyn wirtualnych na bazie QEMU/KVM][6]. Ten proces nie do końca jest dla każdego taki
oczywisty, dlatego postanowiłem go dokładnie opisać.

<!--more-->
## Zablokowany bootloader i konto Mi

Proces odblokowania bootloader'a w Xiaomi Redmi 9 (czy też ogólnie w urządzeniach Xiaomi) trochę
odbiega od procesu odblokowania bootloader'a w smartfonach innych producentów. W przypadku
praktycznie wszystkich urządzeń wyprodukowanych przez Xiaomi po 2016 roku, ich bootloader jest
standardowo zablokowany, a my jako końcowi użytkownicy telefonów nie mamy możliwości sami tego
bootloader'a odblokować. Na szczęście Xiaomi nie stoi na przeszkodzie użytkownikom, by ci mogli
sobie odblokować telefony, gdy tylko będą chcieli to zrobić. Dlaczego zatem Xiaomi blokuje
bootloader? Są dwa oficjalne powody, którymi Xiaomi się posiłkuje.

Pierwszym powodem, dla którego Xiaomi blokuje bootloader'y w swoich urządzeniach, to problem z
nabyciem ich urządzeń w krajach, w których ta marka nie weszła jeszcze na rynek. Użytkownicy w
takiej sytuacji mogą próbować nabyć taki smartfon z nieoficjalnego źródła, co nie zawsze kończy się
dla nich dobrze, np. smartfon może być napchany różnym syfem już na poziomie ROM'u. Mój fon był z
oficjalnego źródła, a i tak miał pełno syfu, który musiałem powyłączać, choć i tak wszystkiego się
nie dało wyłączyć bez psucia urządzenia.

Poniżej znajduje się lista rzeczy, które do tej pory udało mi się wyłączyć:

    $ adb shell

    galahad:/ $ pm uninstall -k --user 0 com.miui.analytics
    galahad:/ $ pm uninstall -k --user 0 com.miui.msa.global
    galahad:/ $ pm uninstall -k --user 0 com.miui.daemon
    galahad:/ $ pm uninstall -k --user 0 com.miui.hybrid
    galahad:/ $ pm uninstall -k --user 0 com.miui.hybrid.accessory
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.joyose
    galahad:/ $ pm uninstall -k --user 0 com.miui.wmsvc

    galahad:/ $ pm uninstall -k --user 0 com.facebook.services
    galahad:/ $ pm uninstall -k --user 0 com.facebook.system
    galahad:/ $ pm uninstall -k --user 0 com.facebook.appmanager
    galahad:/ $ pm uninstall -k --user 0 com.amazon.appmanager
    galahad:/ $ pm uninstall -k --user 0 com.miui.weather2
    galahad:/ $ pm uninstall -k --user 0 com.miui.gallery
    galahad:/ $ pm uninstall -k --user 0 com.miui.cleanmaster
    galahad:/ $ pm uninstall -k --user 0 com.miui.mishare.connectivity
    galahad:/ $ pm uninstall -k --user 0 com.miui.notes
    galahad:/ $ pm uninstall -k --user 0 com.miui.videoplayer
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.scanner
    galahad:/ $ pm uninstall -k --user 0 com.miui.yellowpage
    galahad:/ $ pm uninstall -k --user 0 com.miui.calculator
    galahad:/ $ pm uninstall -k --user 0 com.miui.player
    galahad:/ $ pm uninstall -k --user 0 com.mi.globalbrowser
    galahad:/ $ pm uninstall -k --user 0 com.mi.android.globalminusscreen
    galahad:/ $ pm uninstall -k --user 0 com.miui.compass
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.midrop
    galahad:/ $ pm uninstall -k --user 0 com.google.android.apps.subscriptions.red
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.glgm
    galahad:/ $ pm uninstall -k --user 0 com.google.android.tts
    galahad:/ $ pm uninstall -k --user 0 com.google.ar.lens
    galahad:/ $ pm uninstall -k --user 0 com.google.android.projection.gearhead
    galahad:/ $ pm uninstall -k --user 0 com.google.android.apps.googleassistant
    galahad:/ $ pm uninstall -k --user 0 com.android.providers.partnerbookmarks
    galahad:/ $ pm uninstall -k --user 0 com.android.egg
    galahad:/ $ pm uninstall -k --user 0 com.milink.service
    galahad:/ $ pm uninstall -k --user 0 com.miui.backup
    galahad:/ $ pm uninstall -k --user 0 com.miui.cloudbackup
    galahad:/ $ pm uninstall -k --user 0 com.miui.cloudservice
    galahad:/ $ pm uninstall -k --user 0 com.miui.micloudsync
    galahad:/ $ pm uninstall -k --user 0 com.netflix.partner.activation
    galahad:/ $ pm uninstall -k --user 0 com.android.wallpaper.livepicker
    galahad:/ $ pm uninstall -k --user 0 com.android.wallpaperbackup
    galahad:/ $ pm uninstall -k --user 0 com.miui.miwallpaper
    galahad:/ $ pm uninstall -k --user 0 com.tencent.soter.soterserver
    galahad:/ $ pm uninstall -k --user 0 com.android.soundrecorder
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.mipicks
    galahad:/ $ pm uninstall -k --user 0 com.google.android.feedback
    galahad:/ $ pm uninstall -k --user 0 com.miui.bugreport
    galahad:/ $ pm uninstall -k --user 0 com.google.android.apps.wellbeing
    galahad:/ $ pm uninstall -k --user 0 com.google.android.googlequicksearchbox
    galahad:/ $ pm uninstall -k --user 0 com.google.android.marvin.talkback
    galahad:/ $ pm uninstall -k --user 0 com.google.android.apps.docs
    galahad:/ $ pm uninstall -k --user 0 com.google.android.calendar
    galahad:/ $ pm uninstall -k --user 0 com.micredit.in
    galahad:/ $ pm uninstall -k --user 0 com.ebay.carrier
    galahad:/ $ pm uninstall -k --user 0 cn.wps.xiaomi.abroad.lite
    galahad:/ $ pm uninstall -k --user 0 de.telekom.tsc
    galahad:/ $ pm uninstall -k --user 0 com.huaqin.diaglogger
    galahad:/ $ pm uninstall -k --user 0 com.huaqin.btlogger
    galahad:/ $ pm uninstall -k --user 0 com.huaqin.receivercontroller
    galahad:/ $ pm uninstall -k --user 0 com.huaqin.sarcontroller
    galahad:/ $ pm uninstall -k --user 0 com.mipay.wallet.in
    galahad:/ $ pm uninstall -k --user 0 com.miui.global.packageinstaller
    galahad:/ $ pm uninstall -k --user 0 com.miui.phrase
    galahad:/ $ pm uninstall -k --user 0 com.miui.touchassistant
    galahad:/ $ pm uninstall -k --user 0 com.xiaomi.payment
    galahad:/ $ pm uninstall -k --user 0 com.android.providers.downloads.ui

Także ilość syfu w tym Xiaomi Redmi 9 jest iście imponująca i raczej ten argument, że telefon z
oficjalnego źródła nie będzie go miał można spokojnie odrzucić. Oczywiście Xiaomi będzie się
upierał, że to co widzimy powyżej to nie syf. Wszystkie te powyższe aplikacje zostały wyłączone na
poziomie użytkownika bez potrzeby zaciągania do tego celu praw administratora telefonu (root) i w
dowolnym czasie każdą z tych aplikacji można doinstalować/włączyć via `cmd package install-existing
nazwa_appki` również bez potrzeby zaciągania do tego root'a.

Drugi powód zablokowania bootloader'a jest nieco bardziej przyziemny, a konkretnie chodzi o
możliwość utraty takiego urządzenia. Jeśli ktoś nam ukradnie telefon, to taka osoba będzie miała
niesamowity problem z jego użytkowaniem w sytuacji, gdy mamy zablokowany bootloader przez Xiaomi.
Gdyby bootloader był odblokowany, to nic by nie stało na przeszkodzie, by dowolnie przerobić
programowo urządzenie celem odsprzedania go w późniejszym czasie. Generalnie to każde
zabezpieczenie da radę złamać ale zawsze potrzebny jest na to czas. Trzeciorzędny złodziej nie
kradnie telefonu przy nadarzającej się okazji (przez naszą nieuwagę) po to, by użerać się z jego
zabezpieczeniami. Chce on jak najszybciej pozbyć się takiego kradzionego sprzętu i dostać za niego
hajs. Więc jest spora szansa, że sobie daruje dalsze czynności, gdy połapie się, że będzie miał
spory problem taki telefon odsprzedać, choć w takich sytuacjach bardziej liczy się świadomość
kupujących, a ta zbyt wielka znowu nie jest.

Biorąc pod uwagę powyższe tłumaczenie się Xiaomi, można do pewnego stopnia zrozumieć, że ten
producent smartfonów postanowił zablokować bootloader w swoich urządzeniach. Niemniej jednak,
nasuwa się pytanie, co w przypadku, gdy usługa, od której zależy odblokowanie bootloader'a
(wymagane jest połączenie internetowe) nie będzie z jakiegoś powodu dostępna? Wygląda na to, że
taki telefon zostanie zablokowany dożywotnio.

Warto tutaj zaznaczyć, że jedno konto Mi jest w stanie odblokować bootloader tylko w jednym
urządzeniu w okresie 30 dni, tj. jeśli mamy więcej telefonów marki Xiaomi i chcielibyśmy tuż po
zakupie ściągnąć im blokadę bootloader'a, to w przypadku każdego kolejnego urządzenia będziemy
musieli czekać miesiąc. Przynajmniej takie są [oficjalne informacje][5]. Nie wiem jak wygląda
sprawa z tworzeniem wielu osobnych kont dla każdego urządzenia.

### Powiązanie Xiaomi Redmi 9 z kontem Mi

By odblokować smartfon Xiaomi Redmi 9, musi on działać poprawnie, tj. nie być w żaden sposób
uwalony sprzętowo (hard-bricked) czy programowo (soft-bricked), ani też zapętlać się przy starcie
Androida. W ustawieniach telefonu trzeba dodać konto Mi. Sam smartfon powinien mieć
zainstalowany najnowszy dostępny oficjalny ROM, zatem przydałoby się zaktualizować telefon przed
przystąpieniem do procedury odblokowania jego bootloader'a. Jeśli zaś chodzi o sam komputer, z
którego proces odblokowania będziemy przeprowadzać, to musi on posiadać niezbędne sterowniki i w
przypadku przeciętnej dystrybucji linux'a wszystko powinno być na swoim miejscu.

Na sam początek musimy powiązać nasz smartfon z kontem Mi. By to uczynić, musimy [stworzyć konto
Mi][2]. Jeśli nie chcemy z jakiegoś powodu tworzyć tego konta, to możemy zapomnieć o dalszych
krokach. To konto jest niezbędne jeśli chcemy się bawić w odblokowanie telefonu. Póki co nie są mi
znane żadne obejścia oficjalnej procedury odblokowania bootloader'a i trzeba postępować według
niej. Dlatego też przed przeprowadzeniem dalszych kroków, upewnijmy się, że to konto posiadamy.

Samo posiadanie konta Mi nam nic nie daje. Musimy także w ustawieniach telefonu dodać to konto Mi:

|   |   |
|---|---|
| ![](/img/2021/08/031-xiaomi-redmi-9-linux-aosp-bootloader-mi-account-phone.jpg#small) | ![](/img/2021/08/032-xiaomi-redmi-9-linux-aosp-bootloader-mi-account-phone.jpg#small) |

Poniżej znajdują się wszystkie dane, które można ze smartfona wyciągnąć odnośnie jego specyfikacji,
tak by uniknąć ewentualnych pytań czy to urządzenie, które ja mam, jest faktycznie tym samym, które
ktoś inny może posiadać:

|   |   |   |
|---|---|---|
| ![](/img/2021/08/001-xiaomi-redmi-9-linux-aosp-bootloader-phone-info.jpg#small) | ![](/img/2021/08/002-xiaomi-redmi-9-linux-aosp-bootloader-phone-specs.jpg#small) | ![](/img/2021/08/003-xiaomi-redmi-9-linux-aosp-bootloader-model-hardware.jpg#small) |

Dodanie konta Mi w ustawieniach telefonu otwiera nam drogę do powiązania takiego urządzenia z tym
kontem. W celu powiązania naszego smartfona z kontem Mi, musimy skorzystać z opcji deweloperskich.
Przechodzimy zatem w telefonie do `Settings` > `About Phone` i klikamy parokrotnie w `MIUI version`
(widocznym wyżej), aż opcje deweloperskie staną się aktywne. Następnie przechodzimy do `Settings` >
`Additional Settings` i odszukujemy na liście `Developer options` . To właśnie w tym miejscu
przypisuje się smartfon do konta Mi:

|   |   |   |
|---|---|---|
| ![](/img/2021/08/004-xiaomi-redmi-9-linux-aosp-bootloader-settings.jpg#small) | ![](/img/2021/08/005-xiaomi-redmi-9-linux-aosp-bootloader-settings.jpg#small) | ![](/img/2021/08/006-xiaomi-redmi-9-linux-aosp-bootloader-additional-settings.jpg#small) |

Na liście ustawień deweloperskich trzeba odszukać pozycję `Mi Unlock status` . Urządzenie
przypisuje się do konta Mi w zasadzie tylko raz ale są pewne wymagania, by takie powiązanie
utworzyć. Przede wszystkim, trzeba wyłączyć sieć WiFi. W telefonie musi znajdować się aktywna karta
SIM z włączonymi danymi pakietowymi (3G/4G/5G). Gdy oba te warunki mamy spełnione, klikamy na
przycisk `Add account and device` u dołu ekranu:

|   |   |   |   |
|---|---|---|---|
| ![](/img/2021/08/007-xiaomi-redmi-9-linux-aosp-bootloader-developer-options.jpg#small) | ![](/img/2021/08/008-xiaomi-redmi-9-linux-aosp-bootloader-mi-unlock-status.jpg#small) | ![](/img/2021/08/009-xiaomi-redmi-9-linux-aosp-bootloader-mi-unlock-status.jpg#small) | ![](/img/2021/08/010-xiaomi-redmi-9-linux-aosp-bootloader-mi-unlock-status.jpg#small) |

Po chwili, powiązanie powinno zakończyć się powodzeniem, o czym poinformuje nas odpowiedni
komunikat:

![](/img/2021/08/011-xiaomi-redmi-9-linux-aosp-bootloader-mi-unlock-status.jpg#small)

Po udanej próbie powiązania telefonu z kontem Mi, przechodzimy jeszcze dla pewności do
[serwisu Xiaomi][2], w celu weryfikacji czy faktycznie na liście urządzeń pojawił się smartfon,
który przed chwilą powiązaliśmy. W moim przypadku tak się stało:

![](/img/2021/08/012-xiaomi-redmi-9-linux-aosp-bootloader-mi-account.png#huge)

Mając powiązane urządzenie z kontem Mi, możemy rozpocząć procedurę odblokowania bootloader'a.

### Mi Unlock status i OEM unlocking

Wszystko jedno, którą z poniższych metod odblokowania wybierzemy, tj. czy to przy pomocy XiaoMiTool
na linux, czy też via Mi Unlock na maszynie wirtualnej z windows10, to po odczekaniu kilku dni nasz
telefon ostatecznie dostanie zielone światło na ściągnięcie blokady, którą na bootloader założył
Xiaomi. Trzeba tutaj wyraźnie zaznaczyć, że blokada bootloader'a oferowana przez Androida i blokada
nałożona na bootloader przez Xiaomi, to dwie osobne rzeczy.

Technicznie rzecz biorąc blokadę bootloader'a możemy bez większego problemu ściągnąć sami (w
opcjach deweloperskich). Podczas procesu odblokowania bootloader'a następuje przywrócenie
urządzenia do ustawień fabrycznych. Taki zabieg ma na celu ochronę naszej prywatności, bo w
zasadzie wszystkie dane w telefonie, które umieścił jego użytkownik, zostaną usunięte.

Natomiast blokada, którą nałożył Xiaomi, [zakłada wykorzystanie przez fastboot 24-znakowego
tokena][13], który jest szyfrowany kluczem przechowywanym w `/unlock_key` w obrębie flash'a
telefonu (i też tymczasowo także w katalogu `MiFlashUnlock` w pliku `<<serial>>_sig.data` ). Te
zaszyfrowane dane są przesyłane na serwery Xiaomi wraz z nazwą kodową urządzenia i jej późniejszymi
wersjami, po czym w odpowiedzi zwracana jest podpisana wersja tych przesłanych danych. W ten sposób
uzyskuje się 256 znakowy szyfrogram, który podawany jest w `fastboot oem unlock "<<plik>>"` .
Sygnatura złożona pod danymi jest weryfikowana i jeśli jest prawidłowa, to blokada zostaje zdjęta,
a jeśli nie, to operacja zostaje odrzucona. Niezależnie od pomyślnego zweryfikowania sygnatury,
telefon zostaje zresetowany. Ten restart telefonu (podobnie jak inne polecenia fastboot) powoduje
zmianę tokenu, co ma na celu przeciwdziałać możliwości zapisania danych, które można by wykorzystać
w późniejszym czasie do odblokowania telefonu. Aktualny status tej blokady, jak i status
bootloader'a, można odczytać z opcji deweloperskich w ustawieniach Androida.

## Xiaomi Redmi 9 i XiaoMiTool

Jeśli chodzi o urządzenia Xiaomi, to w ich przypadku mamy możliwość odblokowania bootloader'a z
poziomu praktycznie każdej dystrybucji linux'a i w zasadzie maszyna z windows nie będzie nam tutaj
do niczego potrzebna. Jedyne narzędzie, które musimy pozyskać, to [XiaoMiToolV2][1]. Przechodzimy
zatem pod wskazany link i pobieramy plik `XMT2_Linux_*.run` . Nie jest to co prawda paczka `.deb`
ale nie będzie nam ona do niczego potrzebna. Ten plik po uruchomieniu wypakuje (do katalogu
`/tmp/` ) wszystkie niezbędne pliki i odpala graficzne narzędzie XiaoMiTool. Jeśli chcielibyśmy
ręcznie uruchomić XiaoMiTool to wystarczy podać te dwa poniższe parametry do tego powyższego
skryptu:

    $ chmod +x ./XMT2_Linux_*.run
    $ ./XMT2_Linux_*.run --noexec --keep

W katalogu roboczym zostanie utworzony katalog `XiaoMiTool-V2/` z dokładnie tą samą zawartością, co
w przypadku automatycznego rozpakowania do katalogu `/tmp/` .

Po odpaleniu skryptu, ostatecznie uruchamiany jest plik `XiaoMiTool.jar` . Mamy zatem do czynienia
z aplikacją Java, ale nie musimy instalować w swoim systemie pakietów `default-jdk`/`default-jre` .
Wszystkie niezbędne rzeczy znajdują się w pobranej paczce. W tej paczce obecne są także narzędzia
`fastboot`/`adb` , przez co odpada nam potrzeba ręcznego ich instalowania i konfigurowania w
naszym linux'ie. Wszystko powinno nam zatem działać OOTB po uruchomieniu pliku `XMT2_Linux_*.run`
bez żadnych dodatkowych parametrów.

### Xiaomi procedure failed: [getServiceToken] Missing serviceToken cookie

Od paru tygodni, użytkownicy narzędzia XiaoMiTool zgłaszają [problemy z jego poprawnym
działaniem][3]. Sprawa dotyczy wszystkich wersji, wliczając ostatnią, która została wydana, tj.
`20.7.28 (beta)` . Zmianie uległ URI logowania ale problem chyba dotyczy jedynie kont, które mają
włączone w opcjach konta Mi uwierzytelnianie dwuskładnikowe (2FA). Tak czy inaczej, u mnie ten
problem wystąpił, a jeśli się nam on przytrafi, to nie będziemy mieli możliwości przy pomocy
XiaoMiTool odblokować bootloader. Przy próbie odblokowania bootloader'a zostanie nam jedynie
zwrócony błąd `Xiaomi procedure failed: [getServiceToken] Missing serviceToken cookie` , który
prezentuje się w aplikacji następująco:

![](/img/2021/08/013-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-error-unlock.png#huge)

Zgodnie z sugestią w podlinkowanym wyżej wątku, możemy zmienić sobie ręcznie ten URI ale trzeba to
zrobić w źródłach XiaoMiTool.

#### Jak ręcznie poprawić XiaoMiTool

Do końca nie wiadomo kiedy zostanie wypuszczona łatka mająca wyeliminować problem, którego objawy
widać na powyższej fotce. Dobre wieści są takie, że stosowny pull request został już utworzony ale
kiedy zostanie zaaplikowany, tego nie wiadomo. Nie znana jest też data wydania nowej wersji
XiaoMiTool. Niemniej jednak, znając zmiany w plikach (podglądając wspomniany pull request), możemy
sami poprawić sobie źródła XiaoMiTool przywracając tym samym jego sprawność. Poniżej znajduje się
krótka instrukcja dotycząca tego zagadnienia.

Przede wszystkim, musimy sklonować źródła XiaoMiTool:

    $ git clone https://github.com/francescotescari/XiaoMiToolV2
    $ cd XiaoMiToolV2/

Problematyczny plik to `src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java` , w
którym rezyduje `LOGIN_URL` i to tu trzeba poczynić zmiany.

Poniżej znajduje się patch, który można zaaplikować przy pomocy `git apply` :

    $ cat uri.patch
    diff --git a/src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java b/src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java
    index c90bb86..d41bcd7 100644
    --- a/src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java
    +++ b/src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java
    @@ -36,7 +36,7 @@ import java.net.URI;
     import java.util.Locale;

     public class LoginController extends DefaultController {
    -    private static final String LOGIN_URL = "https://account.xiaomi.com/pass/serviceLogin?sid=passport&json=false&passive=true&hidden=false&_snsDefault=facebook&_locale=" + Locale.getDefault().getLanguage().toLowerCase();
    +    private static final String LOGIN_URL = "https://account.xiaomi.com/pass/serviceLogin?sid=unlockApi&json=false&passive=true&hidden=false&_snsDefault=facebook&checkSafePhone=true&_locale=" + Locale.getDefault().getLanguage().toLowerCase();
         private static boolean loggedIn = false;
         private static Thread loginThread = null;
         @FXML

By ten patch nałożyć na źródła, wystarczy zapisać powyższą zawartość w pliku `uri.patch` i
skorzystać z `git apply` podając w jego argumencie ścieżkę do pliku:

    $ git apply uri.patch

    $ git status
    On branch master
    Your branch is up to date with 'origin/master'.

    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
            modified:   src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java
    ...

#### Instalacja Gradle i OpenJDK w Debianie

Mając odpowiednio przygotowane źródła XiaoMiTool, musimy zainstalować kilka niezbędnych rzeczy w
swoim linux'ie (tutaj Debian). Technicznie rzecz biorąc, wystarczyło by doinstalować dwa pakiety
`gradle` oraz `default-jdk` :

    # apt-get install gradle default-jdk

Problem w tym, że wersja Gradle (4.4.1) w Debianie jest dość leciwa i nie będzie nam współpracować
z wersją Javy, która jest dostarczana z Debianem (11.0.12). Dlatego też trzeba będzie zaktualizować
Gradle ręcznie do najnowszej dostępnej wersji.

Nie spotkałem się z żadnym oficjalnym repozytorium Gradle dla APT, dlatego też trzeba będzie [pobrać
ze strony projektu][9] paczkę `.zip` . Do wyboru są dwa warianty: `binary-only` oraz `complete` . Ja
pobrałem `v7.2 complete` . Po pobraniu paczki `.zip` , wypakowujemy ją i zawartość umieszczamy, np.
w katalogu `/opt/` :

    # mv gradle-*.zip /opt/
    # cd /opt/
    # patool extract gradle-*.zip

Powinniśmy mieć już w katalogu `/opt/` podfolder `gradle-7.2/` . Ścieżkę do tego katalogu musimy
teraz uwzględnić w zmiennej `$PATH` :

    $ export PATH=/opt/gradle-7.2/bin:$PATH

Weryfikujemy czy ścieżka do `gradle` wskazuje na katalog `/opt/gradle-7.2/` :

    $ which gradle
    /opt/gradle-7.2/bin/gradle

Gradle powinien być zatem już w najnowszej wersji:

    $ gradle -v

    Welcome to Gradle 7.2!

    Here are the highlights of this release:
     - Toolchain support for Scala
     - More cache hits when Java source files have platform-specific line endings
     - More resilient remote HTTP build cache behavior

    For more details see https://docs.gradle.org/7.2/release-notes.html


    ------------------------------------------------------------
    Gradle 7.2
    ------------------------------------------------------------

    Build time:   2021-08-17 09:59:03 UTC
    Revision:     a773786b58bb28710e3dc96c4d1a7063628952ad

    Kotlin:       1.5.21
    Groovy:       3.0.8
    Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
    JVM:          11.0.12 (Debian 11.0.12+7-post-Debian-2)
    OS:           Linux 5.13.11-amd64 amd64

Przechodzimy do źródeł XiaoMiTool i aktualizujemy Gradle Wrapper:

    $ cd XiaoMiTool/
    $ ./gradlew wrapper --gradle-version=7.2 --distribution-type=bin
    Downloading https://services.gradle.org/distributions/gradle-7.2-bin.zip
    ...10%....20%....30%....40%....50%....60%....70%....80%....90%...100%
    Starting a Gradle Daemon (subsequent builds will be faster)

    BUILD SUCCESSFUL in 47s
    1 actionable task: 1 executed

Ostatnim krokiem jest uruchomienie XiaoMiTool:

    $ ./gradlew build
    $ ./gradlew run
    Starting a Gradle Daemon, 1 busy Daemon could not be reused, use --status for details

    > Configure project :
    Project : => no module-info.java found

    > Task :compileJava
    Note: Some input files use unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.

    > Task :run
    [04:08:48][INFO  ][492c4021] Current jar path: /media/debuilder/git-xiaomimitoolv2/XiaoMiToolV2/build/classes/java/main
    [04:08:48][DEBUG ][211bf547] javafx.scene.image.Image@95c5f65
    [04:08:48][INFO  ][492c4021] Current jar dir path: /media/debuilder/git-xiaomimitoolv2/XiaoMiToolV2/build/classes/java
    [04:08:48][DEBUG ][60e6bd7] 300.0
    [04:08:48][WARN  ][211bf547] Path not exists: /media/debuilder/git-xiaomimitoolv2/XiaoMiToolV2/build/classes/java/XiaoMiTool.jar
    [04:08:48][WARN  ][211bf547] Path not valid working dir:/media/debuilder/git-xiaomimitoolv2/XiaoMiToolV2/build/classes/java
    [04:08:48][WARN  ][211bf547] Path not exists: ./XiaoMiTool.jar
    [04:08:48][WARN  ][211bf547] Path not valid working dir:.
    [04:08:48][WARN  ][211bf547] Path failed check working dir:null
    [04:08:48][INFO  ][211bf547] Temporary dir used: ./res/tmp
    [04:08:48][ERROR ][211bf547] Failed to identify old logs: ./res/tmp/logs
    [04:08:50][INFO  ][211bf547] Downloading lang file: en_US
    [04:08:51][INFO  ][211bf547] Loading language from file: ./res/lang/en_US.xml
    [04:08:51][INFO  ][211bf547] Starting XiaoMiTool V2 99.9.9 : OS info: Linux - amd64 - 5.13.11-amd64 ||| Locale info: en_US - UTF-8
    [04:08:51][INFO  ][60e6bd7] Launching main window
    ...

#### Cannot execute fastboot command "fastboot devices"

Tak uruchomiony XiaoMiTool ma jeden problem. Mianowicie nie jest w stanie wykryć naszego telefonu,
przynajmniej pod linux. Przeszkodę stanowi brak narzędzi `fastboot`/`adb` , które w źródłach na
GitHub są jedynie dostarczane w postaci plików `.exe` , co skutkować będzie poniższymi komunikatami
w logu aplikacji:

    ...
    [04:12:37][PSTA  ][717ba1ac] Start process (1): "./res/tools/adb" "kill-server"
    [04:12:37][PSTA  ][717ba1ac] Start process (2): "./res/tools/adb" "start-server"
    [04:12:37][INFO  ][717ba1ac] Starting autoscan threads
    [04:12:37][INFO  ][717ba1ac] Searching at least one device
    [04:12:37][PSTA  ][11994e2] Start process (3): "./res/tools/adb" "track-devices"
    [04:12:37][PSTA  ][717ba1ac] Start process (4): "./res/tools/adb" "devices"
    [04:12:37][WARN  ][11994e2] Cannot register device scanner thread via track-devices
    [04:12:37][PSTA  ][717ba1ac] Start process (5): "./res/tools/fastboot" "devices"
    [04:12:37][ERROR ][717ba1ac] Cannot execute fastboot command "fastboot devices", reason: Cannot run program "./res/tools/fastboot": error=2, No such file or directory
    [04:12:38][INFO  ][717ba1ac] Total connected device found: 0
    [04:12:38][INFO  ][717ba1ac] Showing no devices visual
    ...

W repozytorium jest gałąź `linux` i stosowny commit został już poczyniony, by nie trzeba było
przeprowadzać żadnych dodatkowych kroków po sklonowaniu repozytorium. Jedyne co, to przy pomocy
poniższego polecenia musimy zmienić gałąź z `master` na `linux`

    $ git checkout linux
    M       src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java
    Branch 'linux' set up to track remote branch 'linux' from 'origin'.
    Switched to a new branch 'linux'

    $ git status
    On branch linux
    Your branch is up to date with 'origin/linux'.

    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
            modified:   src/main/java/com/xiaomitool/v2/gui/controller/LoginController.java

    no changes added to commit (use "git add" and/or "git commit -a")

Można by też pewnie podlinkować systemowe binarki do katalogu `res/tools/adb` i `res/tools/fastboot`
w źródłach XiaoMiTool ale nie wiem jak sprawa będzie wyglądać z ich kompatybilnością. Dlatego też
lepiej skorzystać z tych dostarczanych z projektem.

### Odblokowanie bootloader'a via XiaoMiTool

Powinniśmy być w stanie bez większego problemu uruchomić XiaoMiTool na dowolnej dystrybucji linux'a,
jako że wszystkie niezbędne nam rzeczy zostały dostarczone w pobranej paczce `.run` . Odpalamy
zatem XiaoMiTool:

    $ ./XMT2_Linux_*.run

Po chwili naszym oczom powinno ukazać się poniższe okienko:

![](/img/2021/08/014-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-region.png#huge)

Logujemy się na swoje konto Mi:

![](/img/2021/08/015-xiaomi-redmi-9-linux-aosp-bootloader-account-login.png#huge)

Jeśli mamy włączoną weryfikację dwuetapową w ustawieniach konta Mi, to trzeba będzie zweryfikować
swoją tożsamość:

![](/img/2021/08/016-xiaomi-redmi-9-linux-aosp-bootloader-account-login-2fa.png#huge)

Po przejściu procesu weryfikacji tożsamości, powinniśmy zostać zalogowani na swoje konto, co
poznamy po numerku umieszczonym w prawym górnym rogu okna aplikacji:

![](/img/2021/08/017-xiaomi-redmi-9-linux-aosp-bootloader-account-logged-in.png#huge)

Wybieramy region europejski. Po jego wskazaniu zostanie nam zaprezentowane okno z wyborem dwóch
opcji:

![](/img/2021/08/018-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Opcja po lewej dotyczy sytuacji, gdy urządzenie pracuje bez problemu, zaś opcja po prawej, gdy coś
mu dolega. Jako, że my chcemy jedynie odblokować bootloader, to wybieramy pozycję z lewej strony
( `My device works normaly. I want to mod it.` ).

Po chwili XiaoMiTool powinien rozpoznać nasz telefon, oczywiście jeśli go uprzednio podłączyliśmy
do komputera via port USB:

![](/img/2021/08/019-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Klikamy przycisk `Select` , co rozpocznie procedurę pozyskiwania pewnych informacji z telefonu,
m.in. status bootloader'a:

![](/img/2021/08/020-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Jak widać na powyższej fotce, status bootloader'a wskazuje na zablokowany ( `Bootloader Status:
Locked` ).

Po chwili zostanie nam zaprezentowana poniższa lista wyboru opcji:

![](/img/2021/08/021-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Wybieramy naturalnie `Unlock, lock bootloader and other` .

W kolejnym okienku wybieramy `Unlock bootloader` :

![](/img/2021/08/022-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Potwierdzamy chęć odblokowania bootloader'a:

![](/img/2021/08/023-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure.png#huge)

Po chwili nasz telefon powinien się automatycznie zresetować i zwrócić pewne informacje:

![](/img/2021/08/024-xiaomi-redmi-9-linux-aosp-bootloader-xiaomitool-unlock-procedure-timer.png#huge)

W tym przypadku nie udało się odblokować bootloader'a ze względu na niespełnienie wymagań co do
czasu oczekiwania po powiązaniu urządzenia do konta Mi. Jeśli taka informacja widnieje w
podsumowaniu, to trzeba będzie jeszcze poczekać, aż wskazany w nim czas upłynie. Dopiero wtedy
będzie można przejść dalej z procesem odblokowania bootloader'a.

Gdy już odczekaliśmy żądaną ilość czasu, ponawiamy procedurę odblokowania bootloader'a (wszystkie te
powyższe kroki). Tym razem efekt będzie nieco inny:

![](/img/2021/08/032-xiaomi-redmi-9-linux-aosp-bootloader-unlock-fastboot.png#huge)

A po chwili operacja powinna zakończyć się powodzeniem:

![](/img/2021/08/033-xiaomi-redmi-9-linux-aosp-bootloader-unlock-fastboot.png#huge)

Smartfon zostanie przy tym przywrócony do ustawień fabrycznych i wszystkie dane użytkownika zostaną
trwale usunięte.

Po ponownym uruchomieniu się urządzenia trzeba będzie je na nowo skonfigurować. Po przejściu tego
procesu włączamy ponownie ustawienia deweloperskie i sprawdzamy status bootloader'a:

![](/img/2021/08/034-xiaomi-redmi-9-linux-aosp-bootloader-unlock-status.jpg#small)

I jak widać wyżej, opcja `OEM unlocking` jest nieaktywna ze statusem `Bootloader is already
unlocked` . Podobnie mamy w przypadku opcji `Mi Unlock status` , która wskazuje `Unlocked` . Te
dwie informacje jednoznacznie mówią, że bootloader w smartfonie został z powodzeniem odblokowany
i można teraz wgrać TWRP i/lub jakiś alternatywny ROM na bazie AOSP/LineageOS.

## Maszyna wirtualna QEMU/KVM

Jeśli XiaoMiTool z jakiegoś powodu nie działa na naszym linux'ie, to możemy posiłkować się
windows'em i natywnym narzędziem do odblokowania bootloader'a w urządzeniach Xiaomi, tj. [Mi
Unlock][4]. Na moim laptopie nie da się zainstalować windows'a, więc jedynym rozwiązaniem jakie mi
pozostaje w takich sytuacjach, to uruchomienie maszyny wirtualnej QEMU/KVM z windows'em na
pokładzie. Cały [proces ogarniania maszyn wirtualnych QEMU/KVM pod linux][6] został opisany w
osobnym wątku i nie będę tego zagadnienia tutaj poruszał.

### Problematyczny windows7

Wygląda na to, że na windows7, to narzędzie Mi Unlock zdaje się nie działać poprawnie. Może i
Mi Unlock się uruchamia ale system nie był w stanie wykryć mojego smartfona w trybie fastboot. Gdy
podłączyłem telefon w normalnym trybie, to system go wykrywał bez problemu. Takie zachowanie
mogłoby wskazywać na brak jakichś sterowników w windows ale nie do końca wiedziałem jakich.
Próbowałem postępować według [instrukcji zawartych w tym wątku na XDA][7] ale nie przyniosło to
żadnych wymiernych efektów. Dlatego też dałem sobie spokój z windows7, a zamiast niego [pobrałem z
WinClub obraz z windows10][8] i to na jego bazie utworzyłem maszynę wirtualną QEMU/KVM. Po
doinstalowaniu jedynie Mi Unlock i udostępnieniu urządzenia USB maszynie wirtualnej, system wykrył
fona w trybie fastboot bez większego problemu. Zatem jeśli mamy zamiar korzystać z maszyn
wirtualnych, to lepiej jest postawić maszynę z windows10 na pokładzie.

### Udostępnienie urządzenia USB hosta maszynie wirtualnej

Poniżej jest przedstawiony proces udostępnienia urządzenia USB hosta maszynie wirtualnej. Ważnym
jest, by przed dodaniem urządzenia USB przełączyć telefon w tryb fastboot. W przypadku mojego
Xiaomi Redmi 9 trzeba przy wyłączonym fonie przytrzymać przycisk VolumeDown+Power. Dla
potwierdzenia, w systemie hosta powinniśmy w logu systemowym zobaczyć nowe urządzenie:

    kernel: usb 1-1.2: new high-speed USB device number 7 using ehci-pci
    kernel: usb 1-1.2: New USB device found, idVendor=18d1, idProduct=d00d, bcdDevice= 1.00
    kernel: usb 1-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 1-1.2: Product: Android
    kernel: usb 1-1.2: Manufacturer: MediaTek
    kernel: usb 1-1.2: SerialNumber: 2----------5
    kernel: usb 1-1.2: Device is not authorized for usage
    kernel: usb 1-1.2: authorized to connect

Mając już podłączony telefon w trybie fastboot do komputera przez port USB, wchodzimy w
konfigurację utworzonej już maszyny wirtualnej (tutaj wykorzystany był `virt-manager` ) i dodajemy
nowe urządzenie. Z listy urządzeń wybieramy `USB Host Device` i wskazujemy odpowiednią pozycję:

![](/img/2021/08/025-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-usb-fastboot.png#huge)

![](/img/2021/08/026-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-usb-fastboot.png#huge)

Zwróćmy uwagę, że to urządzenie na liście ma nazwę `Google Inc.Xiaomi Mi/Redmi 2 (fastboot)` .
Trochę nie pasuje ona do mojego modelu telefonu ale najważniejszą rzeczą jest fraza `fastboot` ,
która oznacza, że telefon został podłączony do komputera w odpowiednim trybie.

### Odblokowanie bootloader'a via Mi Unlock

Po dodaniu urządzenia, uruchamiamy maszynę wirtualną. Pobieramy ze strony Xiaomi [Mi Unlock][4].
Uruchamiamy pobrany plik i instalujemy narzędzie w systemie. Po zainstalowaniu Mi Unlock,
uruchamiamy go za pomocą pliku `batch_unlock` . Naszym oczom powinno ukazać się poniższe okienko:

![](/img/2021/08/027-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-mi-unlock-win10.png#huge)

Jak widać, telefon został poprawnie wykryty, no może za wyjątkiem tego, że to jest `galahad` , a
nie `lancelot` , choć szereg wpisów w konfiguracji telefonu również wskazuje na `lancelot` . Status
bootloader'a został wskazany na zablokowany, tj. `locked`  widoczny wyżej. By tę blokadę usunąć,
musimy zalogować się w tej aplikacji na konto Mi (w lewej dolnej części okna jest przycisk):

![](/img/2021/08/028-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-mi-unlock-win10.png#huge)

Podajemy dane do konta i powinniśmy zostać zalogowani z powodzeniem:

![](/img/2021/08/029-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-mi-unlock-win10.png#huge)

Będąc zalogowanym, klikamy w przycisk `Unlock` . Jeśli pierwszy raz przechodzimy przez procedurę
odblokowania telefonu (zakładając, że wcześniej powiązaliśmy telefon z kontem Mi), to nie zostanie
on odblokowany. Po przyciśnięciu przycisku `Unlock` , zostanie nam wyświetlony jedynie poniższy
komunikat:

![](/img/2021/08/030-xiaomi-redmi-9-linux-aosp-bootloader-qemu-kvm-mi-unlock-win10-timer.png#huge)

Mamy tam informację, że `Please unlock 165 hours later. And do not add your account in MIUI again,
otherwise you will wait from scratch` .  Musimy zatem poczekać około jednego tygodnia, by ten
telefon odblokować. Do końca też nie wiem gdzie mam nie dodawać ponownie konta w MIUI. Może chodzi
tutaj o ustawienia deweloperskie (przypisanie urządzenia do konta Mi) ale prawdę mówiąc nie mam
pojęcia. [Zgodnie z informacjami zawartymi na stronie Xiaomi][5], z telefonu można aktywnie
korzystać przez ten okres czasu czekając aż proces odblokowania się zakończy.

Po upłynięciu czasu oczekiwania, możemy rozpocząć procedurę odblokowania bootloader'a (dokładnie w
taki sam sposób jak wyżej), która efektywnie przywróci urządzenie do ustawień fabrycznych,
czyszcząc przy tym wszystkie dane użytkownika telefonu. Dlatego przed przystąpieniem do procedury
odblokowania telefonu wykonajmy backup wszystkich ważnych danym (SMS, kontakty, fotki, etc.).

## Podsumowanie

Proces odblokowania bootloader'a w przypadku smartfona Xiaomi Redmi 9 (galahad/lancelot), czy też
ogólnie urządzeń marki Xiaomi, nie jest niczym trudnym, choć wymagane do tego celu jest posiadania
konta Mi. Cały proces jest w zasadzie automatyczny ale niekoniecznie jest przy tym natychmiastowy,
bo telefon musi być przypisany do konta Mi przez pewien okres czasu. Jeśli jednak powiążemy
urządzenie z kontem Mi odpowiednio wcześniej, np. tuż po zakupie telefonu i w późniejszym czasie
będziemy chcieli odblokować bootloader, to nie będziemy musieli już czekać na zezwolenie.

Xiaomi dostarcza oficjalne narzędzie do odblokowania bootloader'a w swoich urządzeniach ale jedynie
dla windows. Użytkownicy linux'a muszą posiłkować się XiaoMiTool, który ostatnio ma problemy z
przejściem procesu uwierzytelniania użytkownika podczas dodawania konta do aplikacji. Na szczęście
istnieje prosty fix, który umożliwia przywrócenie XiaoMiTool pełnej sprawności, co niweluje
potrzebę korzystania z osobnej maszyny z zainstalowanym systemem windows. Ostatecznie też możemy
bez większego problemu odpalić na linux'ie maszynę wirtualną na bazie QEMU/KVM z windows10 i to z
jej poziomu bez większych przeszkód odblokować telefon.


[1]: https://www.xiaomitool.com/V2/download
[2]: https://account.xiaomi.com/
[3]: https://github.com/francescotescari/XiaoMiToolV2/issues/23
[4]: https://en.miui.com/unlock/download_en.html
[5]: https://c.mi.com/thread-2262302-1-0.html
[6]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
[7]: https://forum.xda-developers.com/t/official-tool-windows-adb-fastboot-and-drivers-15-seconds-adb-installer-v1-4-3.2588979/
[8]: https://winclub.pl/topic/40069-win-10%C2%A0ltsc-1809-x64-2k21/
[9]: https://gradle.org/releases/
[10]: https://www.mi.com/global/service/support/security-update-1.html
[11]: https://forum.xda-developers.com/f/redmi-9-poco-m2-roms-kernels-recoveries-dev.11175/
[12]: https://forum.xda-developers.com/t/rom-unofficial-11-0-pixel-plus-ui-for-redmi-9-lancelot-galahad.4243813/
[13]: https://github.com/mc-17/xiaomi-bootloader/blob/master/README.md
