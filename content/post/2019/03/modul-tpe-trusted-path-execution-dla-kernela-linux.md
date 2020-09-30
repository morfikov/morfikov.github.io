---
author: Morfik
categories:
- Linux
date: "2019-03-22T20:10:11Z"
published: true
status: publish
tags:
- debian
- kernel
- moduły-kernela
- tpe
title: Moduł TPE (Trusted Path Execution) dla kernela linux
---

Użytkownicy linux'a są zwykle chronieni przez mechanizmy bezpieczeństwa, które ten system jest w
stanie zaoferować. Oczywiście deweloperzy różnych dystrybucji, np. Debiana, dokładają wszelkich
możliwych starań, by system jako całość był wstępnie skonfigurowany tak, by końcowy użytkownik nie
musiał za wiele majstrować przy zabezpieczeniach i mógł się czuć i być (przynajmniej względnie)
bezpieczny. No i faktycznie złowrogie oprogramowanie ma czasem spore problemy dostać się do maszyny,
którą operuje linux. Niemniej jednak, gdy już taki syf się do systemu dostanie, to zwykle niewiele
dzieli go od przejęcia kontroli nad komputerem. Może i część zabezpieczeń linux'a zadziała i sprawi,
że taki wirus/trojan czy nawet zwykły skrypt będzie miał ograniczone pole manewru, to i tak będzie
on mógł przeprowadzić te akcje, które zwyczajny użytkownik systemu (nie root) jest zwykle w stanie
poczynić, np. odebrać dźwięk i video ze stosownych urządzeń i przesłać te dane przez sieć. My z
kolei możemy nawet tego faktu nie być świadomi. Jasne, że powinno się zwracać uwagę na to jakie
pliki się pobiera z internetu i nie uruchamiać wszystkiego lekkomyślnie ale też trzeba mieć na
uwadze fakt, że często jedna maszyna jest współdzielona, np. z członkami rodziny i oni już
niekoniecznie muszą władać zaawansowaną wiedza z zakresu IT, by przewidzieć wszystkie możliwe
zagrożenia czyhające na nich w sieci. Można za to postarać się, by uczynić naszą maszynę nieco
bardziej odporną na niezbyt przemyślane zachowania użytkowników jej systemu operacyjnego. Jednym z
kroków, które możemy podjąć, jest wdrożenie mechanizmu Trusted Path Execution (TPE), który póki co
jest dostępny jedynie w [formie patch'a](https://patchwork.kernel.org/patch/9773791/) na kernel
linux'a lub też jako jego [osobny moduł](https://github.com/cormander/tpe-lkm) oferujący sporo
więcej możliwości w stosunku do wspomnianej wcześniej łaty. W niniejszym artykule rzucimy sobie
okiem na ten cały mechanizm TPE i zobaczymy jak jest on w stanie uchronić nasz OS przed niezbyt
zaawansowaną ludzką inteligencją.

<!--more-->
## Czym jest Trusted Path Execution (TPE)

Trusted Path Execution (TPE) to mechanizm zabezpieczający przed próbą wykonania plików ze ścieżek
niezaufanych. Ścieżki zaufane to takie, w których root jest właścicielem katalogu zawierającego
pliki wykonywalne oraz pliki w tym katalogu mogą być zapisywane jedynie przez niego. Dla przykładu,
katalog `/bin/` , `/usr/bin/` czy też `/usr/local/bin/` są uważane za ścieżki zaufane, podczas gdy
katalogi zwykłego użytkownika, np. `/home/morfik/bin/` , jak i katalog `/tmp/` nie są już
traktowane jako zaufane, bo pliki w nich może zapisać albo użytkownik inny niż root, albo też może
to zrobić dowolny użytkownik systemu.

Jeśli dowolny użytkownik systemu jest w stanie w obrębie swojego katalogu domowego zapisać jakiś
plik, np. po pobraniu go z internetu czy skopiowaniu z pendrive, i jednocześnie ten sam użytkownik
ma możliwość uruchomić taki plik, to tego typu sytuacja  stwarza w zasadzie nieograniczone pole
manewru dla złośliwego oprogramowania. Pamiętajmy przy tym, że jeśli mamy do czynienia z sesją
graficzną (np. mamy uruchomione środowisko graficzne na bazie Xorg'a), to zwykły użytkownik może
mieć dostęp do klawiatury, myszy, obrazu (w tym webcam) i dźwięku (również mikrofon). Jeżeli zwykły
użytkownik ma dostęp do tych urządzeń, to złośliwy skrypt, który taki użytkownik uruchomi, też
będzie go miał, przez co będzie w stanie on nas podglądać z kamery, nagrywać dźwięk z mikrofonu
oraz rejestrować przyciśnięcia klawiszy.

By zaradzić tego typu problemom powstał właśnie Trusted Path Execution (TPE) , bo skutecznie
uniemożliwia on uruchamianie się różnych rzeczy w systemie, nawet jeśli dany użytkownik miałby na
to wielką ochotę. Po prostu wymagane jest, by administrator, który trzyma pieczę nad tą konkretną
linux'ową maszyną, rzucił okiem na dany plik wykonywalny i ocenił go pod kątem negatywnego wpływu
na system. Każda potajemna próba uruchomienia pliku bez świadomej zgody ze strony administratora
systemu zwyczajnie się nie powiedzie, a zdarzenie przy tym zostanie odnotowane w logu.

## Patch na kernel czy osobny moduł TPE

By wdrożyć u siebie mechanizm Trusted Path Execution (TPE), musimy się zdecydować czy zamierzymy
korzystać z patch'a na kernel linux'a, czy też wolimy osobny moduł ładowany wraz ze startem systemu
operacyjnego. Jeśli nasz kernel nie posiada modułów i całą swoją funkcjonalność ma wkompilowaną na
stałe, to być może lepszym wyjściem okaże się skorzystanie z łaty na kernel. Niemniej jednak, ten
patch jest dość ubogi jeśli chodzi o oferowane mechanizmy ochrony i w zasadzie daje on nam jedynie
podstawowe zabezpieczenie w postaci ochrony plików wykonywalnych. Osobny moduł TPE z kolei potrafi
o wiele więcej, np. w kwestii odmowy dostępu zwykłym użytkownikom systemowym do plików takich jak
`/proc/modules`  czy `/proc/kallsyms` albo też jest w stanie uniemożliwić podglądanie procesów
innych użytkowników czy nawet sprawdzenie wersji jądra operacyjnego poleceniem `uname -a` . To
oczywiście tylko wierzchołek góry możliwości modułu TPE.

Tak czy inaczej, Jeśli chcemy mieć jedynie TPE bez tych dodatkowych mechanizmów zbrojących, to
wystarczy nam w zasadzie sama łata na kernel linux'a. Niemniej jednak, aplikowanie łaty wymusza na
nas [posiadanie własnego kernela](/post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/),
który trzeba będzie sobie zbudować. Jeśli to zadanie nas przerasta, to opcją alternatywną jest
skorzystanie z
modułu, [choć go również trzeba zbudować](/post/jak-na-debianie-zrobic-pakiet-deb-zawierajacy-modul-kernela-linux-dkms/),
z tym, że za pomocą [mechanizmu DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support).
Oczywiście można pójść na skróty i pobrać repozytorium git na dysk i dać `make` i `make install` .
Sposób instalacji modułu nie jest istotny, za to liczy się efekt posiadania go w systemie.

### Problemy z modułem/patch'em TPE

W moim przypadku po zbudowaniu kernela z łatą TPE system działał w miarę normalnie, choć część
opcji, które można było ustawić za pomocą tego patch'a, nie działała. Niemniej jednak, ograniczenie
dostępu do plików wykonywalnych spełniało swoją rolę. Dodatkowo, pojawiły się pewne problemy w
przypadku Docker'a i odmontowania systemu plików `overlayfs` , co z kolei powodowało BUG'a w
kernelu. Można zatem przypuszczać, że ta łatka na kernel nie jest należycie dopracowana i jeśli
ktoś chciałby TPE wdrożyć u siebie, to jednak lepiej zainteresować się osobnym modułem, który
działa o wiele lepiej.

Niestety modułowi też się dostanie, bo z racji bardzo szybkich zmian, które przechodzi kernel
linux'a, pewne mechanizmy oferowane przez moduł TPE nie do końca są sprawne. Jednym z nich jest
ukrycie numeru wersji kernela (w `uname -a`), choć to akurat nie jest zbyt uporczywe. Ale, np.
niedziałająca opcja whitelist'owania aplikacji może już się dać we znaki. Pozostałe rzeczy zdają
się działać prawidłowo. Te problemy zgłosiłem już twórcy modułu i być może w niedługim czasie
zostaną one poprawione.

## Co oferuje moduł TPE

Poniżej znajduje się lista opcji modułu TPE, które administrator systemu może ustawić za pomocą
pliku `/etc/sysctl.conf` . Wszystkie parametry posiadają ustawienia domyślne (w nawiasach), przez
co nie trzeba ich wyraźnie definiować jeśli nam te ustawienia odpowiadają. Większość opcji można
włączyć lub wyłączyć stosując odpowiednio `1` lub `0` .

- `tpe.paranoid` -- mechanizm TPE będzie również aplikowany dla administratora systemu (wyłączone).
- `tpe.hardcoded_path` -- lista katalogów (oddzielonych dwukropkiem), które będą uznane przez
mechanizm TPE jako zaufane. W ten sposób żaden plik, za wyjątkiem tych znajdujących się w
katalogach określonych tutaj, nie będzie mógł zostać wykonany/zmapowany. Lepiej się zastanowić dwa
razy, zanim się tutaj jakąś ścieżkę określi (puste).
- `tpe.extras.lsmod` -- nie zezwala zwykłym użytkownikom na odczyt pliku `/proc/modules` (włączone).
- `tpe.extras.proc_kallsyms` -- nie zezwala zwykłym użytkownikom na odczyt pliku `/proc/kallsyms`
(włączone).
- `tpe.extras.harden_ptrace` -- nie zezwala zwykłym użytkownikom na wykonanie operacji `ptrace`
(włączone).
- `tpe.extras.hide_uname` -- nie zezwala zwykłym użytkownikom na odczyt wersji kernela, np. przy
użyciu polecenia `uname -a` (wyłączone).
- `tpe.extras.ps` -- nie zezwala zwykłym użytkownikom na podglądanie procesów innych użytkowników
(wyłączone).
- `tpe.extras.ps_gid` -- określa grupę, do której użytkownik musi zostać dodany, by być w stanie
podglądać procesy innych użytkowników (gdy `extras.ps` jest ustawiony na `1` ) (wyłączone).
- `tpe.extras.restrict_setuid` -- nie zezwala zwykłym użytkownikom niebędący w grupie określonej
przez `trusted_gid` używać wywołań `setuid()` (wyłączone).
- `tpe.trusted_gid` -- określa identyfikator zaufanej grupy, której członkowie będą wyjęci spod
działania mechanizmu TPE (wyłączone).
- `tpe.trusted_apps` -- pliki określone tutaj (oddzielone przecinkami) będą wyjęte spod działania
mechanizmu TPE (puste).
- `tpe.trusted_invert` -- gdy ustawiony na `1` , znaczenie parametru `trusted_gid` ulega zmianie. W
takim przypadku, tylko użytkownicy grupy określonej w `trusted_gid` będą podlegać mechanizmowi TPE
(wyłączone).
- `tpe.admin_gid` -- określa grupę, której pliki będą traktowane jak te, których właścicielem jest
root. W ten sposób mechanizm TPE nie będzie brany pod uwagę w ich przypadku (wyłączone) .
- `tpe.dmz_gid` -- określa grupę, której użytkownicy nie mogą w ogóle wykonywać żadnych plików
(wyłączone).
- `tpe.softmode` -- określa tryb łagodny, w którym logowane są jedynie komunikaty bez faktycznego
blokowania dostępu (wyłączone).
- `tpe.extras.ignore_softmode` -- opcje `tpe.extras.*` będą aktywne nawet w trybie łagodnym
(wyłączone).
- `tpe.log` oraz `tpe.extras.log` -- gdy parametr `tpe.log` zostanie ustawiony na `1` , to odmowy
dostępu przy wykonywaniu plików będą logowane w dzienniku systemowym. Można także zalogować
dodatkowych rzeczy, np. odmowę dostępu przy odczycie pliku `/proc/modules` , przez ustawienie
`tpe.extras.log` na `1` (włączone dla obu).
- `tpe.log_verbose` -- włącza tryb rozmowy przy logowaniu, który loguje również informacje mogące
posłużyć do odblokowania zablokowanego procesu (włączone).
- `tpe.log_max` -- maksymalna ilość procesów nadrzędnych, która może zostać zalogowana w
pojedynczym wpisie logu (50).
- `tpe.log_floodburst` oraz `tpe.log_floodtim` -- włączają ratelimit dla komunikatów logowania.
Parametr `log_floodburst` określa ilość komunikatów, które system jest w stanie zalogować zanim
logowanie zostanie wstrzymane. Natomiast `log_floodtime` określa czas w sekundach jaki musi upłynąć
do ponownego włączenia logowania (5 dla obu).
- `tpe.kill` -- zabija proces wraz z procesem nadrzędnym, gdy dojdzie do odmowy dostępu ze strony
TPE przy wykonaniu pliku, chyba, że wywoła go root (wyłączone).
- `tpe.strict` -- szereg opcji mechanizmu TPE (jakich?? FIXME) będzie aplikowany również dla
zaufanych użytkowników (włączone).
- `tpe.group_writable` -- sprawdza czy plik/katalog może zostać zapisany również przez grupę
(włączone).
- `tpe.check_file` -- sprawdza właściciela i prawa dostępu również w przypadku katalogów (włączone).
- `tpe.lock` -- uniemożliwia dalsze zmiany ustawień za sprawą `sysctl` (wyłączone).
- `tpe.xattr_soften` -- sprawdza [rozszerzone atrybuty systemu plików](https://nfsec.pl/root/6030)
w poszukiwaniu flagi `soften` . W ten sposób można zezwolić na wykonanie poszczególnych plików,
bez posługiwania się parametrem `tpe.trusted_apps` (włączone).

## Różnice modułu TPE w stosunku do łatki kernela

Gdybyśmy zdecydowali się jedynie na założenie łatki, to opcji, które moglibyśmy wtedy ustawić, było
by sporo mniej. W zasadzie to byłyby tylko te poniższe:

- `kernel.tpe.enabled` -- włącza mechanizm TPE, przez co niezaufani użytkownicy nie będą mogli
wykonać plików w przypadku, gdy plik nie znajduje się w katalogu, którego właścicielem jest root
lub plik może zostać zapisany przez użytkownika innego niż root.
- `kernel.tpe.strict` -- aplikuje inny zestaw restrykcji dla wszystkich użytkowników za wyjątkiem
użytkownika root, nawet jeśli są oni zaufani, przez co wykonanie pliku może nastąpić w dwóch
przypadkach. Pierwszy: gdy plik znajduje się w katalogu, którego właścicielem jest dany użytkownik
i ten plik może być zapisywany tylko przez tego użytkownika, oraz drugi: gdy plik jest w katalogu,
którego właścicielem jest root, a sam plik może być zapisywany tylko przez niego.
- `kernel.tpe.restrict_root` -- aplikuje wszystkie restrykcje TPE dla użytkownika root, przez co
root nie będzie w stanie odpalać aplikacji/skryptów innych użytkowników.
- `kernel.tpe.gid` -- określa grupę, która będzie uważana za zaufaną. Jeśli użytkownik nie znajdzie
się w tej grupie, to będzie podlegał sprawdzaniu przez mechanizm TPE. W przypadku ustawienia tutaj
`0` , zaufana grupa zostanie wyłączona, czyniąc tym samym wszystkich użytkowników za wyjątkiem root
niezaufanymi.
- `kernel.tpe.invert_gid` -- sprawia, że grupa określona w `kernel.tpe.gid` staje się grupą
niezaufaną, przez co wszystkie inne grupy stają się zaufane. Ta opcja przydaje się w sytuacji, gdy
tylko wąskie grono użytkowników ma podlegać restrykcjom TPE.

## Problemy z uruchamianiem aplikacji

Jak widzieliśmy wyżej, moduł TPE jest dość rozbudowany jeśli chodzi o ilość opcji, co przekłada się
dość znacznie na poprawę bezpieczeństwa samego linux'a. Niestety szereg aplikacji działających w
tym systemie instaluje swoje pliki wykonywalne w katalogu `/home/` . Może i takie zachowanie jest
marginalną sprawą w przypadku dystrybucji Debian, ale nie zmienia to faktu, że po włączeniu
mechanizmu Trusted Path Execution, część programów może nam przestać działać. Do nich zaliczają się
np. Dropbox oraz kontenery Docker'a, czy też skrypt odpalający `conky` . W takiej sytuacji możemy
dodać wyjątki w parametrze `tpe.trusted_apps` (lub przez rozszerzone atrybuty pliku) lub też
przenieść pewne rzeczy pod `/usr/local/bin/` , gdzie właścicielem plików będzie root i tylko on
będzie miał prawa ich zapisu. Oczywiście to drugie rozwiązanie wymagać będzie zapewne dostosowania
konfiguracji systemu i nie zawsze będzie możliwe. Niemniej jednak, warto to uczynić w każdym z
pozostałych przypadków, bo przyniesie nam to sporo korzyści w przyszłości i wyrobi odpowiednie
nawyki przy korzystaniu z systemu.

## Testowanie Trusted Path Execution

Zakładając, że moduł działa na ustawieniach domyślnych, spróbujmy zatem uruchomić Dropbox'a. Ta
czynność się nie powiedzie, a w logu ujrzymy jedynie poniższy komunikat:

    kernel: tpe: Denied untrusted exec of /home/morfik/.dropbox-dist/dropboxd (uid:1000) by
    /usr/bin/python3.7 (uid:1000), parents: /usr/bin/python3.7 (uid:1000), /lib/systemd/systemd
    (uid:0). Deny reason: directory uid not trusted

No i jak widać mechanizm TPE uniemożliwił uruchomienie tej aplikacji, bo UID katalogu, w których
przechowywana jest binarka `dropboxd` nie jest zaufany.

Rozmowy tryb logowania sprawi, że w logu pojawi się również poniższy komunikat z informacją, która
pomoże nam dodać wyjątek dla uruchamianego pliku:

    kernel: tpe: If this exec was legitimate and you cannot correct the behavior, an exception can
    be made to allow this by running; setfattr -n security.tpe -v "soften_exec:soften_mmap"
    /home/morfik/.dropbox-dist/dropboxd. To silence this message, run; sysctl tpe.log_verbose = 0

Czyli trzeba by wykonać poniższe polecenie jako root:

    # setfattr -n security.tpe -v "soften_exec:soften_mmap" /home/morfik/.dropbox-dist/dropboxd

Co spowoduje, że zostanie dodany rozszerzony atrybut `security.tpe` o wartości
`soften_exec:soften_mmap` :

    # getfattr -d -m "-" /home/morfik/.dropbox-dist/dropboxd

    # file: home/morfik/.dropbox-dist/dropboxd
    security.tpe="soften_exec:soften_mmap"

Jeśli z jakichś przyczyn chcemy usunąć ten atrybut, robimy to w poniższy sposób:

    # setfattr -x security.tpe /home/morfik/.dropbox-dist/dropboxd

    # getfattr -d -m "-" /home/morfik/.dropbox-dist/dropboxd

Jeśli `getfattr` nie zwróci nam nic, to znaczy, że atrybut został usunięty i zwykły użytkownik
ponownie nie będzie w stanie uruchomić tego pliku.

Innym testem czy mechanizm TPE działa poprawnie jest próba podejrzenia pliku `/proc/modules` jako
zwykły użytkownik:

    $ cat  /proc/modules
    cat: /proc/modules: Permission denied

To samo polecenie wykonane jako root zwróci już:

    # cat /proc/modules
    tpe 32768 - - Live 0xffffffffc02d8000 (O)

No w tym przypadku jest dostępny tylko jeden moduł ale jak widać zwykły użytkownik nie jest w
stanie sprawdzić jakie modły są aktualnie załadowane w systemie.

## Dodatkowe zabezpieczenia

Gdy zdecydujemy się na używanie mechanizmu Trusted Path Execution, to wypadałoby także dodać do
pliku `/etc/sysctl.conf` te dwa poniższe parametry:

    kernel.modules_disabled = 1
    kernel.dmesg_restrict = 1

Parametr `kernel.modules_disabled` steruje możliwością dynamicznego ładowania modułów. Gdy ustawi
się w nim wartość `1` , nie będzie można już załadować nowych modułów. Stan taki będzie trwał do
momentu ponownego uruchomienia systemu, gdyż wartości tej opcji nie da rady przełączyć na `0` .

Z kolei parametr `kernel.dmesg_restrict` umożliwia wgląd w logi kernela za pomocą `dmesg` . Jeśli
ten parametr zostanie ustawiony na `1` ,
to [tylko użytkownicy posiadający CAP_SYS_ADMIN](http://lwn.net/Articles/414813/) będą w stanie
odczytać te logi. Domyślnie takie uprawnienia ma tylko root.
