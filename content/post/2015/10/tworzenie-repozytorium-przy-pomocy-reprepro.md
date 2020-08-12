---
author: Morfik
categories:
- Linux
date: "2015-10-07T17:37:20Z"
date_gmt: 2015-10-07 15:37:20 +0200
published: true
status: publish
tags:
- apache2
- debian
title: Tworzenie repozytorium przy pomocy reprepro
---

Ten kto tworzył kiedyś paczki `.deb` wie, że cały proces może w końcu człowieka nieco przytłoczyć.
Paczka, jak to paczka, budowana jest ze źródeł i konfigurowana przez jej opiekuna. Z reguły ludzie
instalują kompilowane programy via `make install` . Niektórzy idą o krok dalej i używają do tego
celu narzędzi typu `checkinstall` . I wszystko jest w miarę w porządku, przynajmniej jeśli chodzi o
utrzymywanie jednej paczki. Przeprowadzamy kompilację tylko raz, po czym instalujemy dany pakiet i
zapominamy o nim. Niemniej jednak, tego typu postępowanie może doprowadzić nasz system na skraj
niestabilności. W tym poście nie będziemy zajmować się zbytnio sposobem w jaki powinno się tworzyć
paczki `.deb` , a jedynie tym jak je przechowywać. Do tego celu potrzebne jest nam repozytorium,
które zbudujemy w oparciu o oprogramowanie `reprepro` .

<!--more-->
## Przygotowywanie repozytorium

Mając wiele pakietów, które przerabiamy lokalnie, musimy w końcu zacząć myśleć o jakimś sposobie,
który by nam ułatwił zarządzanie tymi pakietami. Oczywiście nie muszą to być tylko rekompilowane
pakiety z nałożonymi przez nas zmianami. Jeśli opiekun danej paczki w debianie opieprza się, możemy
zwyczajnie zaktualizować taki pakiet w oparciu o już wcześniej przygotowany przez tego kogoś katalog
`debian/`. Podobnie sprawa ma się z oprogramowaniem, którego nie ma, lub, które nigdy nie zostanie
zaakceptowane przez debiana z powodu jego dość ostrej polityki. W takim przypadku nic nie stoi na
przeszkodzie, by taki pakiet dodać sobie do własnego repozytorium.

[Jest kilka różnych sposobów na tworzenie
repozytoriów](https://wiki.debian.org/HowToSetupADebianRepository) i tutaj zostanie przedstawiony
ten z wykorzystaniem
[oprogramowania](https://wiki.debian.org/SettingUpSignedAptRepositoryWithReprepro)
[reprepro](https://anonscm.debian.org/cgit/mirrorer/reprepro.git/plain/docs/short-howto). Po wgraniu
wspomnianego pakietu musimy określić na dysku katalog, w którym będą przechowywane pakiety, np.
`/repo/debian/` . Taki folder może być również w posiadaniu zwykłego użytkownika, dzięki czemu przy
zarządzaniu nie będziemy potrzebować uprawnień administratora. Po tym jak już ustalimy katalog,
przechodzimy do niego:

    # cd /repo/debian/

Będąc w katalogu, tworzymy podkatalog `conf/` , a w nim plik `distributions` o poniższej treści:

    Origin: apt.morfikownia.lh
    Label: debian
    Suite: testing
    Codename: jessie
    Version: testing
    Architectures: i386 amd64 source
    Components: main non-free contrib
    UDebComponents: main contrib non-free
    Description: morfik's repository for debian testing
    SignWith: 771B6520

    Origin: apt.morfikownia.lh
    Label: debian
    Suite: unstable
    Codename: sid
    Version: unstable
    Architectures: i386 amd64 source
    Components: main non-free contrib
    UDebComponents: main contrib non-free
    Description: morfik's repository for debian unstable
    SignWith: 771B6520

    Origin: apt.morfikownia.lh
    Label: debian
    Suite: experimental
    Codename: experimental
    Version: experimental
    Architectures: i386 amd64 source
    Components: main non-free contrib
    UDebComponents: main contrib non-free
    Description: morfik's repository for debian experimental
    SignWith: 771B6520
    NotAutomatic: yes
    ButAutomaticUpgrades: yes

Mamy tutaj trzy gałęzie: `testing` , `sid` oraz `experimental` . Można sobie to oczywiście
dostosować i jeśli potrzebujemy jedynie sid'a, to kasujemy dwa pozostałe bloki. Struktura tego
pliku powinna być raczej zrozumiała.

W oparciu o te powyższe informacje, narzędzia takie jak `apt` lub `aptitude` będą identyfikować
nasze repozytorium. W powyższym config'u jest także określony klucz GPG (parametr `SignWith` ),
którym to będzie podpisywany plik `InRelease` zawierający min. sumy kontrolne plików z listami
pakietów. Ten plik jest generowany za każdym razem, gdy będziemy dodawać/usuwać pakiety.
Podpisywanie pakietów jest opcjonalne ale ze względów bezpieczeństwa dobrze jest [utworzyć sobie
klucz GPG]({{< baseurl >}}/post/bezpieczny-klucz-gpg/).

Dodatkowo, w katalogu `conf/` możemy stworzyć plik `options` , który to będzie zawierał opcje dla
programu `reprepro`, te które zwykle się dopisuje przy jego wywoływaniu. Wtedy zamiast pisać rządek
parametrów, możemy sprecyzować plik konfiguracyjny przy pomocy opcji `--confdir` . W moim pliku nie
ma póki co zbyt wielu opcji, jedynie te poniższe:

    basedir /repo/debian
    verbose

### Dostęp do repozytorium z poziomu www

Repozytorium jest już po części przygotowane i można do niego zacząć wrzucać pakiety. Niemniej
jednak, będzie ono działać jedynie na lokalnej maszynie. Jeśli chcemy udostępnić zawartość takiego
repozytorium w sieci, możemy to zrobić przy pomocy serwera www `apache2`. W tym celu będziemy
musieli przeprowadzić kilka poniższych czynności. Zakładam, że mamy już zainstalowane potrzebne
oprogramowanie, tj. serwer `apache2` .

Tworzymy zatem wirtualnego hosta w pliku `/etc/apache2/sites-enabled/000-default.conf` przez dodanie
poniższego kodu:

    <VirtualHost *:80>
          ServerName deb.morfikownia.lh
          ServerAdmin morfik@localhost
          DocumentRoot /repo/
          ErrorLog ${APACHE_LOG_DIR}/error_repo.log
          CustomLog ${APACHE_LOG_DIR}/access_repo.log combined
    </VirtualHost>

Musimy także stworzyć dedykowaną konfigurację dla katalogu `/repo/` , który będzie udostępniany
przez serwer apache. Tworzymy zatem plik `repo.conf` w katalogu `/etc/apache2/conf-available/` i
dopisujemy w nim poniższą treść:

    <Directory /repo/ >
          Options Indexes FollowSymLinks Multiviews
          AllowOverride None
          Require all granted
    </Directory>

    <Directory "/repo/debian/db/">
          Require all denied
    </Directory>

    <Directory "/repo/debian/conf/">
          Require all denied
    </Directory>

    <Directory "/repo/debian/incoming*/">
          Require all denied
    </Directory>

Włączamy konfigurację przy pomocy `a2enconf` lub też linkujemy powyższy plik do katalogu
`/etc/apache2/conf-enabled/` i sprawdzamy poprawność konfiguracji:

    # apache2ctl configtest
    Syntax OK

Po zresetowaniu/przeładowaniu usługi `apache2` , repozytorium powinno być już dostępne pod adresem
jaki określiliśmy w konfiguracji wirtualnego hosta. W tym przypadku jest to `deb.morfikownia.lh` .

## Importowanie pierwszego pakietu do repozytorium

By przetestować czy repozytorium działa jak należy, dodajmy do niego przykładową paczkę. Przy czym,
można importować gołe paczki, tj. te bez źródeł, wskazując plik `.deb` . Można też zaimportować
pakiety posługując się plikiem `.changes` , przykładowo:

    # reprepro --confdir /repo/debian/conf/ includedeb sid ./amarok_2.8.0-2.1_amd64.deb

    # reprepro --confdir /repo/debian/conf/ include sid ./amarok_2.8.0-2.1_amd64.changes

W katalogu repozytorium, możemy zaobserwować, że zostało utworzonych szereg innych podkatalogów
oraz, że wygenerowanych zostało również kilka dodatkowych plików. Jeśli precyzowaliśmy klucz do
repozytorium, powinniśmy także zostać poproszeni o hasło do niego. Dodana zaś paczka znajduje się w
`/repo/debian/pool/main/a/amarok/` .

## Wpisy dla apt/aptitude

By narzędzia takie jak `apt` lub `aptitude` mogły zacząć korzystać z naszego repozytorium, dodajemy
poniższe wpisy do pliku `/etc/apt/sources.list` :

    #     deb         file:/media/Server/repo/debian/ sid main contrib non-free
    #     deb-src     file:/media/Server/repo/debian/ sid main contrib non-free

          deb         http://deb.morfikownia.lh/debian/ sid main contrib non-free
          deb-src     http://deb.morfikownia.lh/debian/ sid main contrib non-free

W zależności od tego czy chcemy korzystać lokalnie czy zdalnie, wybieramy odpowiednie linijki i
aktualizujemy listę pakietów przy pomocy poniższego polecenia:

    # aptitude update

Być może będzie również potrzeba dodania klucza od repozytorium do keyring'a apt, jeśli tego
wcześniej nie zrobiliśmy. Klucz można albo importować z pliku, albo z serwera kluczy, np. za pomocą
poniższych dwóch linijek:

    # gpg --keyserver keyserver.ubuntu.com --recv-keys ID_KLUCZA
    # gpg --armor --export ID_KLUCZA | apt-key add -

## Pinning'i dla repozytoriów

Jeśli chcielibyśmy, by budowane przez nas paczki miały wyższy priorytet niż wszelkie pozostałe
pakiety w repozytorium debiana, musimy [ustawić pinning'i](https://wiki.debian.org/AptPreferences) w
pliku `/etc/apt/preferences` :

    Package: *
    Pin: release o=Debian,a=testing
    Pin-Priority: 990

    Package: *
    Pin: origin ""
    Pin-Priority: 995

    Package: *
    Pin: origin deb.morfikownia.lh
    Pin-Priority: 995

By sprawdzić czy `apt` widzi dodaną przez nas paczkę i czy ma ona odpowiedni priorytet, wydajemy
poniższe polecenie:

    $ apt-cache policy amarok
    amarok:
      Installed: 2.8.0-2.1
      Candidate: 2.8.0-2.1
      Version table:
         2.8.0-2.1+b1 0
            990 http://ftp.pl.debian.org/debian/ testing/main amd64 Packages
            500 http://ftp.pl.debian.org/debian/ sid/main amd64 Packages
     *** 2.8.0-2.1 0
            995 file:/repo/debian/ sid/main amd64 Packages
            100 /var/lib/dpkg/status

Z powyższego listingu wychodzi na to, że mamy do wyboru dwie wersje amarok'a. Jedna z nich jest
dostępna w tesingu/sid. Druga zaś w naszym lokalnym repozytorium. Również priorytety się zgadzają.
Domyślne 500 dla sid'a, 990 dla testing'a oraz 995 dla lokalnego repozytorium. W przypadku
instalacji tego pakietu, zostanie on pobrany właśnie z naszego repo.

## Operowanie na repozytorium

Poniżej jest kilka linijek, których dobrze jest się nauczyć, bo to przy ich pomocy będziemy operować
na repozytorium.

Dodawanie pakietów do repozytorium:

    # reprepro --confdir /repo/debian/conf/ includedeb sid ./amarok_2.8.0-2.1_amd64.deb
    # reprepro --confdir /repo/debian/conf/ include sid ./amarok_2.8.0-2.1_amd64.changes

Można również korzystać z opcji: "-C component", "-A architecture", "-S section" oraz "-P priority"
w celu dokładniejszego sprecyzowania, gdzie dana paczka powinna trafić.

Usuwanie wszystkich pakietów powiązanych ze źródłami:

    # reprepro --basedir /repo/debian/ removesrc sid amarok

Listowanie paczek w repozytorium dla sida:

    # reprepro --basedir /repo/debian/ list sid
    # reprepro --basedir /repo/debian/ list sid amarok

Jeśli chcemy przy pomocy `aptitude` sprawdzić co znajduje się w naszym repozytorium, wpisujemy:

    # aptitude search '~O apt.morfikownia.lh'

I taka mała uwaga jeszcze. Przy dodawaniu paczek trzeba zwracać uwagę na plik `.changes` , bo jest
tam zawarta informacja na temat gałęzi, do której pakiet powinien trafić, przykładowo:

    ...
    Distribution: unstable
    ...

W przypadku, gdy chcielibyśmy taką paczkę dodać do innej gałęzi, dostaniemy błęda:

    .changes put in a distribution not listed within it!
    To ignore use --ignore=wrongdistribution.
    There have been errors!

I to w zasadzie tyle. Przy czym, przy aktualizacji paczki w repozytorium, nie trzeba usuwać
uprzednio jej starszej wersji. `reprepro` zrobi to automatycznie przy dodawaniu nowszej. W ten
sposób możemy trzymać własne pakiety w jednym miejscu, zarządzać nimi i mieć do nich dostęp przy
pomocy natywnych narzędzi debiana.
