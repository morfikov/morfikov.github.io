---
author: Morfik
categories:
- Linux
date: "2015-08-08T23:33:59Z"
date_gmt: 2015-08-08 21:33:59 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apparmor
GHissueID: 281
title: AppArmor i profilowanie aplikacji
---

Po ostatnich doniesieniach na temat [błędu jaki został znaleziony w
Firefox'ie](https://blog.mozilla.org/security/2015/08/06/firefox-exploit-found-in-the-wild/) ,
doszedłem do wniosku, że najwyższy czas nauczyć się obsługi narzędzia `AppArmor` . Ma ono pomóc w
kontrolowaniu praw dostępu do zasobów systemu operacyjnego, np. plików, katalogów czy określonych
urządzeń. Jeśli weźmiemy przytoczony wyżej błąd, to przeglądarka bez takiego profilu AppArmor'a była
w stanie przeszukać lokalne pliki i wysłać je gdzieś na net, co powodowałoby udostępnienie poufnych
danych, np. historia poleceń shell'owych, czy klucze prywatne.

<!--more-->
## Aktywacja AppArmor'a

Jako, że AppArmor jest częścią kernela, to jego włączenie odbywa się przez dopisanie [odpowiednich
parametrów w bootloaderze](https://wiki.debian.org/AppArmor/HowToUse#Enable_AppArmor) :

    APPEND ... apparmor=1 security=apparmor ...

Po dopisaniu tych dwóch opcji, resetujemy maszynę. Po tym jak się ona podniesie, AppArmor powinien
być aktywny.

### Narzędzia do obsługi AppArmor'a

By wszystko nam ładnie działało, to musimy jeszcze sobie doinstalować kilka pakietów. Na dobrą
sprawę wystarczy wgrać pakiet `apparmor` ale w debianie mamy jeszcze dwa pakiety, w których zawarte
są predefiniowane profile dla określonych aplikacji. Oczywiście, możemy sobie te profile stworzyć
samemu ale skoro już ktoś się napracował, to czemu nie wykorzystać jego pracy? Wpisujemy zatem w
terminal to poniższe polecenie:

    # aptitude install apparmor apparmor-profiles apparmor-profiles-extra

## Profile dla AppArmor'a

Wszystkie profile AppArmor'a dla aplikacji są trzymane w katalogu `/etc/apparmor.d/` . To zwykłe
pliki tekstowe, które są później kompilowane do postaci binarnej w celu załadowania do kernela,
zwykle w fazie startu systemu. Naturalnie, każdy z profili możemy sobie osobno załadować/wyładować w
trakcie pracy systemu operacyjnego jeśli zajdzie taka potrzeba.

Profile możemy ładować w kilku trybach, min. `enforce` , `complain` i `audit` . W przypadku trybu
`enforce` , wszelkie zdarzenia niepasujące do zdefiniowanych reguł będą blokowane, a informacja o
całym zajściu zostanie zanotowana w logu. Tryb `complain` różni się od `enforce` jedynie tym, że
akcja zablokowania dostępu do zasobu nie zostanie podjęta. Z kolei tryb `audit` ma na celu
zalogowanie wszelkich zdarzeń bez względu na to czy są one zaakceptowane czy też blokowane przez
AppArmor. Zwykle jednak będziemy korzystać z tych dwóch pierwszych.

### Przygotowywanie profilu

Profile możemy pisać manualnie z wykorzystaniem zwykłego edytora tekstu lub też możemy skorzystać z
narzędzi obecnych w dodatkowym pakiecie `apparmor-utils` . Te narzędzia ułatwiają pracę z
AppArmor'em ale też mogą pojawić się błędy przy ich wykorzystywaniu. Jeśli chcemy skorzystać z
narzędzi, to cały proces tworzenia profilu jest praktycznie automatyczny i do tego wielce
interaktywny.

Zaczynamy od określenia położenia pliku wykonywalnego aplikacji, którą chcemy sprofilować, np.
`/opt/firefox/firefox` . Następnie wywołujemy `aa-autodep` podając w argumencie ścieżkę do tego
pliku:

    # aa-autodep /opt/firefox/firefox
    Writing updated profile for /opt/firefox/firefox.

Zaowocuje to stworzeniem pliku profilu w katalogu `/etc/apparmor.d/` o nazwie `opt.firefox.firefox`
mającego poniższą treść:

    # Last Modified: Fri Aug 8 11:50:38 2015
    #include <tunables/global>

    /opt/firefox/firefox flags=(complain) {
      #include <abstractions/base>

      /opt/firefox/firefox mr,

    }

Teraz odpalamy `aa-genprof` , w argumencie którego również podajemy ścieżkę do pliku:

    # aa-genprof /opt/firefox/firefox
    ...
    Profiling: /opt/firefox/firefox

    [(S)can system log for AppArmor events] / (F)inish

W tej chwili profil dla tej powyższej aplikacji jest ustawiony w tryb `complain` , czyli nie będą
blokowane żadne akcje przy dostępie do zasobów systemowych ale będą zwracane komunikaty zawierające
takie żądania. Dlatego też, odpalamy w tym momencie profilowany program i staramy się go używać tak
jak dotąd, tj. wchodzimy w menu, przestawiamy różne rzeczy, włazimy na net, itp. Chodzi o to, by
ustalić z jakich plików/katalogów/urządzeń dana aplikacja chce korzystać. To wszystko zostanie
zalogowane i na podstawie tych komunikatów zostanie stworzony dokładny profil.

W przypadku posiadania systemd, logowanie może nie działać jak należy. Dzieje się tak ze względu na
journal, za sprawą którego nie mamy już pliku `/var/log/syslog` , a to właśnie ten plik AppArmor
bierze pod uwagę przy skanowaniu. Dobrze jest zatem przekierować wyjście dziennika do tego pliku
przy pomocy:

    # journalctl -n 0 -f > /var/log/syslog

Gdy skończymy się bawić, zamykamy program i w terminalu wciskamy S . Spowoduje to przeskanowanie
logu i zwrócenie szeregu zapytań odnośnie dostępu do zasobów, przykładowo:

    ...
    Profile:  /opt/firefox/firefox
    Path:     /dev/dri/card0
    Mode:     rw
    Severity: unknown

      1 - #include <abstractions/X>
      2 - #include <abstractions/evince>
      3 - #include <abstractions/gnome>
      4 - #include <abstractions/kde>
      5 - #include <abstractions/lightdm>
      6 - #include <abstractions/totem>
      7 - #include <abstractions/ubuntu-browsers.d/chromium-browser>
      8 - #include <abstractions/ubuntu-browsers.d/kde>
      9 - #include <abstractions/ubuntu-browsers.d/mailto>
      10 - #include <abstractions/ubuntu-browsers.d/multimedia>
      11 - #include <abstractions/ubuntu-gnome-terminal>
      12 - #include <abstractions/ubuntu-konsole>
      13 - #include <abstractions/ubuntu-unity7-base>
     [14 - /dev/dri/card0]
    [(A)llow] / (D)eny / (I)gnore / (G)lob / Glob with (E)xtension / (N)ew / Abo(r)t / (F)inish / (M)ore
    ...

W zależności od aplikacji, takich zapytań może być sporo. Generalnie rzecz biorąc, interesują nas
głównie ścieżki `Path: /dev/dri/card0` oraz uprawnienia `Mode: rw` . W tym przypadku proces
Firefox'a prosi o zapis oraz odczyt urządzenia karty graficznej. Mamy do wyboru 14 pozycji, z czego
numery od 1-13 włącznie zawierają predefiniowane profile abstrakcyjne, które możemy wykorzystać
budując nowy profil. Te podpowiedzi pojawiają się ilekroć dana ścieżka występuje w którymś z
przytoczonych profili.

Jeśli chcemy korzystać z plików zlokalizowanych w katalogu `/etc/apparmor.d/abstractions/` , to
przydałoby się zajrzeć w każdy profil tam umieszczony i ustalić, czy faktycznie potrzebujemy całego
profilu, czy jedynie pojedynczych pozycji. Jeśli w takim profilu, który chcemy załączyć, będzie
sporo wpisów i tylko jeden czy dwa, które nam nie będą pasować, to zawsze możemy załączyć cały
profil i dodać dyrektywę `deny` .

W przypadku posiadania dwóch wpisów w profilu, dyrektywa `deny` zawsze jest brana pod uwagę w
pierwszej kolejności. Przykładowo, jeśli zezwolimy na dostęp do urządzenia `/dev/dri/card0`
dopisując odpowiednią linijkę w pliku profilu, to można przypuszczać, że dostęp zostanie przyznany.
Natomiast jeśli w tym samym pliku lub też w którymś załączonym profilu abstrakcyjnym będzie istnieć
inny wpis z tą powyższą ścieżką, z tym, że poprzedzony dyrektywą `deny` , to dostęp do tego
urządzenia zostanie odmówiony i to pomimo faktu, że manualnie zezwoliliśmy na jego odczyt/zapis.

Aplikacje zwykle korzystają z plików w określonych katalogach, np. ich własny, i zamiast zezwalać na
dostęp do każdego z plików w takim folderze, lepiej jest zezwolić na dostęp do całego katalogu.
Jeśli na liście nie widać odpowiedniej pozycji, to zawsze możemy określić nowy zasób wciskając N i
podać mniej akuratną pozycję przez zastosowanie masek, np. `*` lub `**` . Różnią się one jedynie
poziomem katalogów. `*` odpowiada za określony katalog, natomiast `**` uwzględnia również
podkatalogi.

Zatem w powyższym przypadku, jeśli chcielibyśmy zezwolić na dostęp do wszystkich plików i katalogów
w folderze `/dev/` , wciskamy N i wpisujemy `/dev/**` :

    Enter new path: /dev/**
    ...
      14 - /dev/dri/card0
     [15 - /dev/**]
    [(A)llow] / (D)eny / (I)gnore / (G)lob / Glob with (E)xtension / (N)ew / Abo(r)t / (F)inish / (M)ore

Jak widać, pojawiła się nowa pozycja nr. 15. Teraz już tylko pozostaje zezwolić (lub zabronić) na
dostęp wciskając klawisz A . Cały proces się będzie powtarzał aż do wyczerpania wszystkich żądań o
zasoby.

Nie musimy także się bać o duplikaty reguł, bo te automatycznie zostaną usunięte z pliku ilekroć
tylko znajdzie się inna reguła, która będzie pokrywać te wcześniej zdefiniowane wpisy.

Tak powstały profil włączamy przy pomocy `aa-enforce` :

    # aa-enforce /etc/apparmor.d/opt.firefox.firefox
    Setting /etc/apparmor.d/opt.firefox.firefox to enforce mode.

### Test profilu

Odpalamy teraz profilowaną aplikację i weryfikujemy czy AppArmor ogranicza jej dostęp do zasobów
przy pomocy `aa-status` :

    # aa-status
    ...
    3 processes have profiles defined.
    3 processes are in enforce mode.
       /opt/firefox/firefox (68279)
    ...

Możemy także sprawdzić czy proces jest ograniczony przez AppArmor przy pomocy narzędzia `ps` :

    # ps auxZ | grep -v '^unconfined'
    LABEL                           USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    /opt/firefox/firefox (enforce)  morfik    68279 10.1 25.3 2734420 484552 ?      Sl   17:25  10:28 /opt/firefox/firefox -new-instance -ProfileManager

### Usuwanie profilu

Jako, że te profile są kompilowane do postaci binarnej, to w przypadku gdy chcemy się ich pozbyć,
nie wystarczy samo usunięcie pliku profilu. Musimy go usunąć również z kernela. Dlatego też zanim
usuniemy sam plik, musimy skorzystać z `apparmor_parser` z przełącznikiem `-R` , przykładowo:

    # apparmor_parser -R /etc/apparmor.d/opt.firefox.firefox

To powyższe polecenie nie usuwa samego pliku profilu, a jedynie wyładowuje go z kernela. Przy czym,
trzeba pamiętać, że profil nie jest określany przez samą nazwę pliku, tylko przez ten poniższy blok:

    /opt/firefox/firefox {
    ...
    }

Jeśli byśmy usunęli ten plik, to wyładowanie profilu z kenrela nie będzie możliwe.

### Przeładowanie profilu

Zmieniając wpisy w profilu aplikacji AppArmor'a, nie dokonujemy zmian w polityce bezpieczeństwa
bezpośrednio. Zmieniony profil trzeba przeładować i do tego celu również wykorzystujemy
`apparmor_parser` , z tym, że teraz z przełącznikiem `-r` :

    # apparmor_parser -r /etc/apparmor.d/opt.firefox.firefox

### Włączanie i wyłączanie profilu

Wszystkie pliki obecne w katalogu `/etc/apparmor.d/` mające poprawną składnię profilu będą
automatycznie ładowane ze startem systemu. Jeśli z jakichś powodów nie chcemy ładować któregoś z
nich, nie musimy usuwać samego pliku, wystarczy stworzyć dowiązanie symboliczne do katalogu
`/etc/apparmor.d/disable/` . Oczywiście, nie musimy tego robić ręcznie -- możemy posłużyć się
`aa-disable` :

    # aa-disable opt.firefox.firefox
    Disabling /etc/apparmor.d/opt.firefox.firefox.

    # ls -al /etc/apparmor.d/disable/
    ...
    lrwxrwxrwx  1 root root   35 2015-08-08 10:29:56 opt.firefox.firefox -> /etc/apparmor.d/opt.firefox.firefox

Profil włączamy przez usunięcie samego dowiązania, z tym, że możemy to zrobić przy po mocy jednego z
dwóch narzędzi: `aa-enforce` oraz `aa-complain` . Różnice między tymi dwoma wyjaśniliśmy sobie
wyżej. Warto jednak dodać, że po włączeniu profilu via `aa-complain` , do zwrotki profili w pliku
zostanie dopisana automatycznie flaga:

    /opt/firefox/firefox flags=(complain) {
    ...
    }

I to właśnie tylko ona odróżnia profil enforce od complain.

### Zakodowana nazwa pliku

Podczas ręcznego profilowania możemy natknąć się na log, który nie zwróci nam jako takiej nazwy
pliku, do którego aplikacja żąda dostępu. Zamiast niego, w logu zostanie nam zwrócony długi ciąg
znaków, podobny do tego
    poniżej:

    Aug 8 07:58:21 morfikownia audit[34909]: AVC apparmor="DENIED" operation="open" profile="/opt/firefox/firefox" name=2F686F6D652F6D6F7266696B2F2E6D6F7A696C6C612F66697265666F782F4372617368205265706F7274732F496E7374616C6C54696D653230313530383037303835303435 pid=34909 comm="firefox" requested_mask="r" denied_mask="r" fsuid=1000 ouid=1000

Jak zatem ustalić o jaki plik chodzi? Do tego celu wykorzystujemy narzędzie `aa-decode` , w
argumencie to którego podajemy wartość parametru `name` widoczną wyżej w
    logu:

    # aa-decode 2F686F6D652F6D6F7266696B2F2E6D6F7A696C6C612F66697265666F782F4372617368205265706F7274732F496E7374616C6C54696D653230313530383037303835303435
    Decoded: /home/morfik/.mozilla/firefox/Crash Reports/InstallTime20150807085045

Zatem zakodowany plik to `/home/morfik/.mozilla/firefox/Crash Reports/InstallTime20150807085045` .
Jak widzimy, w katalogu `Crash Reports` jest spacja i [to właśnie ona
sprawia](https://unix.stackexchange.com/questions/56540/fix-this-apparmor-rule), że AppArmor nie
może nam podać prostej ścieżki i musi ją zwrócić w takiej formie jak widać wyżej.
