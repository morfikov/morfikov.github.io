---
author: Morfik
categories:
- Android
date:    2021-09-27 05:50:00 +0200
lastmod: 2021-09-27 05:50:00 +0200
published: true
status: publish
tags:
- smartfon
- prywatność
- xiaomi
- redmi-9
- root
- aplikacje
- f-droid
GHissueID: 328
title: Czym jest tryb lockdown w smartfonie z Androidem
---

Jeśli mamy wgranego w miarę nowego Androida (9+) na naszego smartfona i korzystamy aktywnie z tego
urządzenia, to zapewne zdążyliśmy się już przyzwyczaić, że co kilka dni musimy wpisywać ręcznie
hasło do blokady ekranu i to nawet pomimo aktywnej biometrii w ustawieniach systemu. Co ciekawe,
w stock'owych ROM'ach producentów telefonów (nawet w moim Xiaomi Redmi 9), nie ma żadnej opcji,
która byłaby w stanie skonfigurować czas, po którym taki monit z hasłem ma wyskakiwać. Przyznam, że
trochę mnie ten ficzer denerwował ale nie mogłem z nim w zasadzie nic zrobić i trzeba było nauczyć
się go tolerować. Do niedawna nawet nie wiedziałem, że takie pytanie użytkownika o hasło bierze się
z faktu przejścia telefonu w jeden ze jego trybów pracy, tj. [tryb lockdown][2], w który telefon
jest przełączany automatycznie zwykle co 72 godziny (3 dni). Ludzie mówią, że ten tryb ma chronić
użytkownika smartfona na wypadek utraty urządzenia. Z racji, że aktualnie wgrałem sobie custom ROM
na bazie AOSP/LineageOS, to postanowiłem sprawdzić czy jakieś opcje konfiguracji tego trybu są w
nim dostępne. Przy okazji chciałem nieco bardziej zapoznać się z tym mechanizmem lockdown'u i
zweryfikować na ile może on być użyteczny.

<!--more-->
## Szyfrowane smartfony z Androidem

Już od dość dawna, każdy smartfon z Androidem ma szyfrowaną partycję `/data/` , na której są
przechowywane wszystkie ustawienia i dane użytkownika. We wcześniejszych wersjach Androida (<7) był
wykorzystywany blokowy system szyfrowania, którym szyfrowana była cała partycja `/data/` , czyli
zwykły FDE ([Full Disk Encryption][3]). Zdaniem dev'ów Androida, to szyfrowanie miało szereg wad,
więc przeszli na FBE ([File Based Encryption][4]), gdzie szyfrowane są już nie całe partycje, a
pojedyncze pliki w jej obrębie.

Szukając informacji na temat dokładnej zasady działania szyfrowania FBE w kontekście smartfonów z
Androidem, znalazłem [bardzo ciekawy artykuł na ten temat][7] (o wiele bardziej przystępnie
napisany niż te dwa powyżej). Postanowiłem zatem tutaj zawrzeć szereg informacji z tego tekstu,
oczywiście po ich uprzednim przetłumaczeniu, bo ta wiedza bardzo przyda się do zrozumienia czym
dokładnie jest FDE i FBE.

Klucz główny, którym dane są szyfrowane w przypadku FDE, jest generowany losowo w momencie, gdy
użytkownik uruchomia telefon po raz pierwszy (i ewentualnie po factory reset). Po wygenerowaniu
klucza, jest on szyfrowany i dodawany do magazynu kluczy przez Androidowego Keymaster'a opartego na
[TEE][1]. Jeśli nie ustawimy blokady ekranu, to Keymaster korzysta z domyślnego hasła i szyfruje
nim ten wygenerowany losowo klucz szyfrujący. W ten sposób dane wciąż będą szyfrowane ale dostęp do
nich będzie w stanie uzyskać każdy, kto wejdzie w posiadanie naszego smartfona. W przypadku
ustawienia hasła/PIN'u/wzoru, klucz główny również zostanie zaszyfrowany przez Keymaster ale przy
użyciu właśnie tych danych uwierzytelniających, które będziemy musieli podawać ilekroć tylko
będziemy chcieli odblokować ekran w telefonie. Przy ponownym uruchomieniu systemu, dostęp do klucza
głównego jest zapewniany czy to za sprawą domyślnego hasła (brak ustawionej blokady ekranu), czy
też danych uwierzytelniających użytkownika, co umożliwia zamontowanie partycji `/data/` i
szyfrowanie/deszyfrowanie danych w locie. Warto tutaj zaznaczyć, że zmiana hasła do blokady ekranu
nie sprawia, że dane na partycji `/data/` zostaną przeszyfrowane. Jedynie klucz główny zostanie
zaszyfrowany nowymi danymi uwierzytelniającymi użytkownika. Sam klucz wykorzystywany w procesie
szyfrowania/deszyfrowania pozostaje niezmieniony do momentu zresetowania telefonu do ustawień
fabrycznych, bo wtedy ten klucz naturalnie zostanie wygenerowany na nowo.

W przypadku FBE, każdy plik na partycji `/data/` może być szyfrowany niezależnie przez zaprzęgnięcie
do tego celu unikalnych kluczy szyfrujących pliki (File Encryption Key). Te klucze są uzyskiwane w
oparciu o klucz główny, który jest chroniony przez TEE/Keymaster. Klucz główny jest zabezpieczony
hasłem użytkownika, tj. podobnie jak w przypadku mechanizmu FDE. Zastosowanie FBE otwiera drogę do
wdrożenia [mechanizmu Direct Boot][5], który umożliwia rozruch systemu telefonu praktycznie do
momentu, w którym zostaniemy poproszeni o wpisanie hasła do blokady ekranu, co znacząco różni się w
stosunku do FDE, gdzie tuż po włączeniu telefonu trzeba było wpisać hasło, by odszyfrować całą
partycję `/data/` , przez co praktycznie cała funkcjonalność telefonu była zablokowana. Dlatego
właśnie wprowadzono FBE.

By aplikacje mogły wykorzystać pełny potencjał trybu Direct Boot, muszą być one świadome, że taki
mechanizm w ogóle istnieje. Dlatego też trzeba odpowiednio pisać aplikacje na Androida. Jeśli
aplikacja nie jest świadoma, że Direct Boot w naszym urządzeniu jest możliwy, to wszystkie dane
takiej aplikacji zostaną zaszyfrowane domyślnie. W przypadku tych appek, które są świadome, tylko
część plików może zostać zaszyfrowana dając takim aplikacjom możliwość działania w trybie Direct
Boot, choć z ograniczoną funkcjonalnością. Taki model bezpieczeństwa realizowany jest przez
wprowadzenie dwóch różnych lokalizacji w obrębie pamięci masowej telefonu, w których dane mogą być
przechowywane: DE (Device Encrypted) oraz CE (Credential Encrypted). CE jest dostępne tylko i
wyłącznie wtedy, gdy użytkownik poda swoje dane uwierzytelniające i odblokuje urządzenie. Natomiast
DE jest dostępne w trybie Direct Boot oraz również po odblokowaniu urządzenia. Dane
prywatne/wrażliwe nigdy nie powinny zostać umieszczone w DE przez dewelopera konkretnej aplikacji.

Ta zmiana w szyfrowaniu telefonu zapewnia bardziej elastyczny schemat ochrony danych. Różne obszary
systemu plików z danymi użytkownika są chronione za pomocą swoich kluczy głównych, które pochodzą
od różnych danych uwierzytelniających. Osobne klucze główne są generowane dla pamięci masowej CE i
DE, przy czym klucze główne CE wykorzystują zarówno klucz sprzętowy unikalny dla danego urządzenia,
jak i dane uwierzytelniające użytkownika, a klucze DE są chronione wyłącznie za pomocą klucza
sprzętowego unikalnego dla danego urządzenia.

Ponieważ obszar pamięci masowej DE nie jest powiązany z danymi uwierzytelniającymi użytkownika,
pamięć ta jest udostępniana po ponownym uruchomieniu urządzenia, co pozwala aplikacjom obsługującym
technologię Direct Boot na działanie przed odblokowaniem urządzenia przez użytkownika. Pozwala to
na przykład na odbieranie połączeń telefonicznych natychmiast po uruchomieniu telefonu (gdy jest on
jeszcze zablokowany). Ta elastyczność pozwala również na ochronę profili roboczych za pomocą
zestawu kluczy głównych oddzielonych od osobistych magazynów danych urządzenia.

## Wady lockdown'u

Zwykle, gdy przychodzi do odblokowania ekranu w naszych smartfonach z Androidem, to raczej już chyba
większość z nas korzysta dla wygody z jakiegoś czynnika biometrii, np. przykłada palec do czytnika
linii papilarnych. Raz, że o wiele szybciej się ten telefon odblokowuje, a dwa nie ma obawy o
zdradzenie komuś naszego hasła, co przydaje się w sytuacji, gdy dookoła nas jest cała masa ludzi.

Biometria ma jednak jedną bardzo poważną wadę jeśli chodzi o bezpieczeństwo systemu. Wystarczy
przechwycić nasz odcisk palca, czy siłowo go przyłożyć do czytnika i telefon zostaje odblokowany
bez naszej wyraźnej zgody, co daje pełny dostęp do plików użytkownika osobom do tego nie
uprawnionym. I tutaj właśnie do gry wchodzi tryb lockdown, który ma na celu zapobiec takiemu
obrotowi spraw. Po aktywowaniu tego mechanizmu dezaktywowana jest biometria oraz Android Smart Lock,
który odpowiada za odblokowanie telefonu przy pomocy naszego pyszczka, zaufanych urządzeń Bluetooth
czy też zaufanych miejsc w oparciu o aktualną lokalizację telefonu. Na ekranie blokady zostaną też
ukryte wszystkie powiadomienia. W ten sposób tryb lockdown zmusza tę osobę, która jest w posiadaniu
naszego smartfona, do wpisania hasła, by jakiekolwiek dane z niego pozyskać, nawet te pozornie
niegroźne jak wspomniane wyżej powiadomienia.

Tryb lockdown nie jest idealny i raczej mu daleko do uzyskania takiego statusu. Chodzi generalnie o
to, że aktywuje się on, raz, że on samoczynnie, a dwa, to co 72 godziny od ostatniego wpisania
poprawnego hasła. Standardowo przez ten okres trzech dni nie mamy możliwości ręcznie zainicjować
tego trybu. Najwyraźniej zdaniem deweloperów Androida i producentów telefonów, średnio tylko dwa
razy w tygodniu ktoś będzie próbował siłowo uzyskać dostęp do naszych danych w telefonie. Dlatego
też ten tryb lockdown w stock'owych ROM'ach z jednej strony niesamowicie potrafi człowieka wnerwić,
bo trzeba wpisywać hasło w najmniej oczekiwanych momentach, a z drugiej strony nie daje nam żadnej
ochrony, gdyby przyszło co do czego -- wpiszemy hasło i przez okres trzech dni lockdown nie
zostanie aktywowany samoczynnie. Najgorsze jest to, że nic nie możemy w tej sprawie zrobić. No może
prawie nic, bo jeśli dysponujemy smartfonem, na który został wypuszczony alternatywny ROM na bazie
AOSP/LineageOS, to możemy pokusić się o odblokowanie bootloader'a i wgranie takiego ROM'u na
telefon, co otwiera nam drogę do okiełznania tego całego trybu lockdown.

## Możliwość aktywacji trybu lockdown z poziomu blokady ekranu

Od kilku już tygodni jestem w fazie testowania szeregu ROM'ów i pewnie za jakiś czas na któryś z
nich się zdecyduje, by w końcu na stałe wgrać go na swój smartfon. Niemniej jednak, z moich
obserwacji wynika, że praktycznie każdy ROM, który do tej pory testowałem, posiadał opcję aktywacji
trybu lockdown na żądanie. Także wygląda na to, że jest to raczej standardowa opcja w przypadku
alternatywnych ROM'ów. Tak czy inaczej, jeśli włączymy sobie możliwość aktywacji trybu lockdown na
żądanie, to będziemy mogli to uczynić z poziomu blokady ekranu. Trzeba tylko dodać do niego
stosowną opcję, która figuruje pod Settings > Display > Lock Screen, a konkretnie chodzi o `Show
lockdown option` :

|   |   |   |
|---|---|---|
| ![](/img/2021/09/001.android-lockdown-mode-settings-lock-screen.jpg#small) | ![](/img/2021/09/002.android-lockdown-mode-settings-lock-screen.jpg#small) | ![](/img/2021/09/003.android-lockdown-mode-settings-lock-screen.jpg#small) |

Do momentu aktywacji trybu lockdown, telefon będzie pracował sobie tak, jak dotychczas, tj.
będziemy w stanie go odblokować przy pomocy biometrii czy Android Smart Lock'a i nie trzeba będzie
co kilka dni wpisywać hasła, czyli smartfon nie będzie z automatu przechodził w ten tryb,
przynajmniej nie powinien jeśli z niego aktywnie korzystamy. Nie mam pojęcia czy jak odłożymy
telefon na kilka dni i z niego korzystać nie będziemy, to czy ten tryb lockdown się aktywuje
automatycznie.

## Ręczna aktywacja trybu lockdown

Po włączeniu stosownej opcji w ustawieniach Androida, tryb lockdown można aktywować ręcznie w
dowolnym momencie z poziomu blokady ekranu. Wystarczy podświetlić ekran i przytrzymać przycisk
zasilania. Oprócz standardowych opcji od wykonywania połączeń alarmowych, czy
wyłączenia/zresetowania telefonu, będzie jedna dodatkowa pozycja właśnie z trybem lockdown, co
wygląda mniej więcej tak:

![](/img/2021/09/004.android-lockdown-mode-lock-screen.jpg#small)

Tapnięcie w ten środkowy kafelek nic wielkiego, przynajmniej z pozoru, nie uczyni. Na pierwszy rzut
oka wszystko wygląda normalnie i na pozór nic się nie zmieniło.

![](/img/2021/09/005.android-lockdown-mode-lock-screen.jpg#small)

Jeśli teraz byśmy spróbowali odblokować smartfon przy pomocy biometrii, np. skanera linii
papilarnych, to nam się to już nie uda. System zażąda od nas hasła do blokady ekranu:

![](/img/2021/09/006.android-lockdown-mode-lock-screen-password.jpg#small)

Po wpisaniu hasła i pomyślnym odblokowaniu ekranu, tryb lockdown zostaje zdjęty i znów można z
telefonu korzystać w normalny sposób.

## Kwestia zaszyfrowanych danych

Początkowo myślałem, że lockdown będzie w stanie usunąć klucze szyfrujące z pamięci operacyjnej
telefonu ale najwyraźniej taki zabieg nie ma miejsca. Można to wywnioskować podglądając zawartość
pewnych katalogów tuż po uruchomieniu się systemu ale przed odblokowaniem telefonu oraz po aktywacji
trybu lockdown. Po uruchomieniu się systemu, zawartość w tych poniższych katalogach prezentuje się
następująco:

 - `/data/app/` (zaszyfrowane)
 - `/data/data/` (zaszyfrowane)
 - `/data/media/0/` (zaszyfrowane)
 - `/data/misc/` , `/data/misc_ce/0/` (zaszyfrowane) oraz `/data/misc_de/0/`
 - `/data/user/` (zaszyfrowane) oraz `/data/user_de/`
 - `/data/system/` , `/data/system_ce/0/` (zaszyfrowane) oraz `/data/system_de/0/`
 - `/data/vendor/` , `/data/vendor_ce/0/` (zaszyfrowane) oraz `/data/vendor_de/0/`
 - `/storage/emulated/0/` (puste)

Katalog `/storage/emulated/0/` jest pusty, choć tutaj trzymane są pliki użytkownika, takie jak fotki
czy pobrane rzeczy z internetu. Zatem stosowny zasób najwyraźniej nie został w to miejsce
podmontowany. W katalogach mających w nazwie `_ce` oraz pozostałych pozycjach z tagiem
"zaszyfrowane" są dostępne pliki ale ich nazwy są nieczytelne, tj. mają postać typu
`nInt0ruNvVZZq,qASvHjpC` . Można co prawda przemieszczać się po drzewie katalogów ale próba
odczytania pliku kończy się poniższym błędem:

    galahad:/data/user/0/zNqNZIAL11b1C2YI,hzg4jxqViF/iNChdh,jOBFRIfmujb5VaD # cat nInt0ruNvVZZq,qASvHjpC
    cat: nInt0ruNvVZZq,qASvHjpC: Required key not available

No jak widać brakuje stosownego klucza, by ten plik odszyfrować.

Natomiast po pierwszym odblokowaniu telefonu i włączeniu trybu lockdown, wszystkie te katalogi mają
odszyfrowaną zawartość. Zatem wygląda na to, ze poza tymi wyszczególnionymi wyżej w artykule
właściwościami mechanizmu lockdown (blokada powiadomień, wyłączenie biometrii i dezaktywacja Smart
Lock), ten tryb nic w zasadzie innego nie robi. W dalszym ciągu dostęp do danych użytkownika jest
możliwy, przez co aplikacje mogą bez problemu sobie działać, przynajmniej na tyle na ile pozwala im
system Androida przy zablokowanym ekranie.

### Wdrożenie Wrong PIN Shutdown

Biorąc pod uwagę powyższe informacje na temat obecności kluczy/haseł w pamięci operacyjnej, trzeba
pomyśleć nad innym rozwiązaniem, które pomogłoby nam te klucze z pamięci telefonu usunąć. Wiedząc,
że potencjalny złoczyńca nie będzie miał możliwości uzyskać dostępu do telefonu przez mechanizm
biometrii, co wymusza na nim siłowe łamanie hasła do blokady ekranu, możemy wdrożyć rozwiązanie na
bazie [aplikacji Wrong PIN Shutdown][6], która standardowo jest dostępna w repozytorium F-Droid.

Appka Wrong PIN Shutdown zlicza jedynie (albo i aż) błędne próby odblokowania ekranu z
wykorzystaniem PIN'u czy też zwykłego hasła alfanumerycznego. Po osiągnięciu ustawionego przez nas
limitu, inicjowane jest wykonanie jakiegoś polecenia. Standardowo jest to `reboot -p` , który
efektywnie wyłącza telefon. By jednak do takiej sytuacji doszło, Wrong PIN Shutdown musi działać na
prawach root'a, zatem ukorzeniony Android jest do tego celu niezbędny.

Poniżej znajdują się fotki ustawień appki Wrong PIN Shutdown:

|   |   |
|---|---|
| ![](/img/2021/09/007.android-lockdown-mode-wrong-pin-shutdown-settings.jpg#small) |![](/img/2021/09/008.android-lockdown-mode-wrong-pin-shutdown-settings.jpg#small) |

Ustawiając ilość błędnych prób logowania na względnie małą wartość, np. 2, sprawi, że po drugiej
nieudanej próbie odblokowania ekranu telefon nam się wyłączy. Nawet jeśli ten niedobry człowiek
będzie próbował włączyć telefon w późniejszym czasie, to wszelkie dane prywatne użytkownika będą
zaszyfrowane tym dość skomplikowanym hasłem, którego nie chce nam się wpisywać co 3 dni, w sytuacji,
gdy wszyscy dookoła się na nas gapią.

By Wrong PIN Shutdown mógł realizować swoje zadanie, trzeba dla niego też dodać wyjątek w
Settings > Security > Device admin apps:

|   |   |
|---|---|
| ![](/img/2021/09/009.android-lockdown-mode-wrong-pin-shutdown-device-admin.jpg#small) | ![](/img/2021/09/010.android-lockdown-mode-wrong-pin-shutdown-device-admin.jpg#small) |

## Podsumowanie

Nie ukrywam, że po mechanizmie, który nosi nazwę lockdown spodziewałem się jednak czegoś bardziej
wyrafinowanego w kwestii bezpieczeństwa i prywatności danych użytkownika. Niemniej jednak,
wyłączenie na żądanie biometrii czy Android Smart Lock'a jest dość pożyteczną funkcją, która może
zminimalizować albo chociaż odwlec w czasie wyciek poufnych informacji z telefonu. Pozostaje mieć
nadzieję, że może w przyszłych wersjach Androida, ten mechanizm lockdown'u zostanie nieco bardziej
rozbudowany i faktycznie umożliwi pełny lockdown telefonu. Niemniej jednak, do czasu aż to nastąpi,
przydałoby się wdrożyć aplikację Wrong PIN Shutdown, która będzie stać na straży dostępu do ekranu
blokady, gdy tryb lockdown zostanie aktywowany i w razie czego wyłączy nam telefon, tak by nasze
dane pozostały w formie zaszyfrowanej i nikt bez naszej świadomej i dobrowolnej zgody nie uzyskał
do nich dostępu.


[1]: https://en.wikipedia.org/wiki/Trusted_execution_environment
[2]: https://android-developers.googleblog.com/2020/09/lockscreen-and-authentication.html
[3]: https://source.android.com/security/encryption/full-disk
[4]: https://source.android.com/security/encryption/file-based
[5]: https://developer.android.com/training/articles/direct-boot
[6]: https://f-droid.org/en/packages/org.nuntius35.wrongpinshutdown/
[7]: https://docs.samsungknox.com/admin/knox-platform-for-enterprise/kbas/kba-360039577713.htm
