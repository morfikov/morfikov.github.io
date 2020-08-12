---
author: Morfik
categories:
- Linux
date: "2015-06-02T15:35:53Z"
date_gmt: 2015-06-02 13:35:53 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- ssh
- szyfrowanie
title: Uwierzytelniające klucze SSH
---

[Klucze SSH](https://wiki.archlinux.org/index.php/SSH_keys) mogą być wykorzystane jako sposób
identyfikacji danej osoby przy logowaniu się do zdalnego serwera SSH. Te klucze zawsze występują w
parach -- jeden prywatny, drugi publiczny. Pierwszy z nich jest znany tylko nam i powinien być
trzymany w sekrecie i pilnie strzeżony. Klucz publiczny z kolei zaś jest przesyłany na każdy serwer
SSH, z którym chcemy się połączyć. Gdy serwer jest w posiadaniu naszego klucza publicznego i widzi
przy tym, że próbujemy nawiązać połączenie, używa on tego klucza by wysłać do nas zapytanie
(challange) -- jest ono zakodowane i musi na nie zostać udzielona odpowiednia odpowiedź, a tej może
dokonać ktoś, kto jest w posiadaniu klucza prywatnego. Nie ma innej opcji by rozkodować wiadomość,
dlatego też nikt inny nie może udzielić na nią prawidłowej odpowiedzi. To rozwiązanie eliminuje
wrażliwość na różne formy podsłuchu -- ten kto nasłuchuje nie będzie w stanie przechwycić pakietów
zawierających hasło, bo ono nie jest nigdy transmitowane prze sieć. No i oczywiście jeśli chodzi o
samo hasło -- odpadają nam ataki typu Brute Force pod kątem jego złamania.

<!--more-->
## Agent GPG

Jako, że klucze SSH zawierają wrażliwe informacje, można je zaszyfrować, tak by każdorazowe ich
wykorzystanie wymagało podania hasła ale wtedy przy logowaniu się do serwera trzeba podać hasło do
klucza zamiast do konta, co niekoniecznie może być wygodniejsze w przypadku ciągłego wywoływania
`ssh` lub `scp` . Nic też nie zyskujemy przy posiadaniu wielu różnych kluczy -- każdy inny dla
każdego kolejnego serwera SSH. Niemniej jednak, mamy do dyspozycji narzędzia, np.
[gpg-agent](https://www.gnupg.org/documentation/manuals/gnupg/) , które to pozwalają na dodanie i
przechowywanie szeregu kluczy i udzielaniu do nich dostępu przy wykorzystaniu tylko jednego hasła --
mechanizm znany z choćby `gnome-keyring` . W taki sposób możemy zdefiniować wiele kluczy,
przechowywać je w bezpieczny sposób i mieć tylko jedno hasło by nimi wszystkimi zarządzać. W
przypadku gpg-agent , możemy także ustawić czas dostępu do klucza. Po jego upłynięciu, trzeba będzie
ponownie wprowadzić hasło by wydobyć klucz od agenta.

Jeśli nie mamy w swoim systemie zainstalowanego pakietu `gnupg-agent` , to czym prędzej go
dociągnijmy, bo to na nim będzie się opierać cały ten artykuł. Dzięki nemu, a konkretnie jednemu z
plików, które on dostarcza `/etc/X11/Xsession.d/90gpg-agent` , środowisko graficzne będzie w stanie
powołać do życia odpowiedni proces:

    # ps -eo user,group,pid,args | grep -i gpg
    morfik   morfik   100144 /usr/bin/gpg-agent --daemon --sh --write-env-file=/home/morfik/.gnupg/gpg-agent-info-morfikownia /usr/bin/dbus-launch --exit-with-session /usr/bin/openbox-session

By się upewnić, że wszystko działa jak należy, powinniśmy porównać zmienne zawarte w pliku
`~/.gnupg/gpg-agent-info-morfikownia` z tymi widocznymi w środowisku terminala:

    $ cat ~/.gnupg/gpg-agent-info-morfikownia
    GPG_AGENT_INFO=/tmp/gpg-7UsEbL/S.gpg-agent:100144:1
    SSH_AUTH_SOCK=/tmp/gpg-UcPwOy/S.gpg-agent.ssh
    SSH_AGENT_PID=100144

    $ env | grep -i gpg_agent
    GPG_AGENT_INFO=/tmp/gpg-7UsEbL/S.gpg-agent:100144:1

    $ env | grep -i ssh_auth
    SSH_AUTH_SOCK=/tmp/gpg-UcPwOy/S.gpg-agent.ssh

    $ env | grep -i ssh_agent
    SSH_AGENT_PID=100144

Jeśli wszystko się zgadza, oznacza to, że konfiguracja demona przebiegła pomyślnie. Dostosujmy
jeszcze trochę samą konfigurację agenta. Interesuje nas plik `~/.gnupg/gpg-agent.conf` . Domyślnie
jest w nim sprecyzowanych kilka opcji. My zaś jeszcze ustawimy dodatkowo te poniższe:

    default-cache-ttl 1800
    default-cache-ttl-ssh 1800

    pinentry-program /usr/bin/pinentry-gtk-2

Opcje `default-cache-ttl` oraz `default-cache-ttl-ssh` ustalają czas (w sekundach) ważności kluczy.

## Wybór i generowanie klucza SSH

Są różne typy kluczy SSH. Zwykle wykorzystuje się klucze RSA 1024/2048/4096 bitowe. Innym typem
klucza jest `ECDSA` , który to opiera się o krzywe eliptyczne zapewniając ten sam poziom
bezpieczeństwa co klucze RSA, z tym, że sam klucz w przypadku ECDSA jest o wiele krótszy. Istnieje
jeszcze inny typ klucza -- ED25519, który to jest zmodyfikowaną wersją ECDSA i powstał w odpowiedzi
na podejrzenia pod kątem NSA, czy prawdziwe, tego nie wiem. Na necie natomiast można się natknąć na
informację, że te krzywe wykorzystywane przy ECDSA [nie są chyba aż tak idealnie
krzywe](http://www.hyperelliptic.org/tanja/vortraege/20130531.pdf) czy coś i mogą stanowić
zagrożenie bezpieczeństwa. W każdym razie, poniżej [jest tabelka
porównująca](https://security.stackexchange.com/questions/5096/rsa-vs-dsa-for-ssh-authentication-keys)
długość poszczególnych kluczy:

    Symmetric  |  ECC2N  |  ECP  |  DH/DSA/RSA
           80  |   163   |  192  |     1024
          128  |   283   |  256  |     3072
          192  |   409   |  384  |     7680
          256  |   571   |  521  |    15360

Jak widzimy, zaledwie 521 bitów wystarczy by zapewnić poziom bezpieczeństwa, który można uzyskać
stosując klucze RSA 15360 bitów.

Część oprogramowania może być różnie skonfigurowana i nie wszystkie typy kluczy będą działać
prawidłowo. Ja, póki co, zaobserwowałem problemy przy kluczach RSA dłuższych niż 4096 bitów, oraz
są także problemy z obsługą kluczy typu ED25519 przez gpg-agenta -- same klucze działają bez
problemu.

By wygenerować klucze, wpisujemy poniższe linijki:

    $ ssh-keygen -t ecdsa -b 521 -C "$(whoami)@$(hostname)-$(date -I)"
    $ ssh-keygen -t ed25519 -C "$(whoami)@$(hostname)-$(date -I)"
    $ ssh-keygen -t rsa -b 4096 -C "$(whoami)@$(hostname)-$(date -I)"

Wszystkie utworzone klucze są przechowywane w katalogu `~/.ssh` . Każdy też może je podejrzeć,
dlatego warto zadbać o odpowiednie prawa dostępu do plików ale trzeba przy tym pamiętać o jednej
istotniej rzeczy -- użytkownik root może i tak obejść te ograniczenia dostępu i jeśli nie ufamy
adminowi, lepiej nie trzymać w jego systemie kluczy prywatnych. W każdym razie, jeśli to my jesteśmy
administratorem systemu i przy tym mamy w nim wielu użytkowników, możemy zabezpieczyć swoje klucze
przed nieuprawnionym dostępem:

    $ chmod 700 .ssh/
    $ chmod 600 ~/.ssh/*
    $ ls -al ~/.ssh | grep -i id
    -rw-------  1 morfik morfik  444 Oct  1 22:19 id_ecdsa
    -rw-------  1 morfik morfik  283 Oct  1 22:19 id_ecdsa.pub
    -rw-------  1 morfik morfik  464 Oct  1 22:21 id_ed25519
    -rw-------  1 morfik morfik  111 Oct  1 22:21 id_ed25519.pub
    -rw-------  1 morfik morfik 3.3K Sep 30 18:57 id_rsa
    -rw-------  1 morfik morfik  744 Sep 30 18:57 id_rsa.pub

## Wykorzystanie agenta GPG

Klucze przy dodawaniu do agenta są odzierane z ewentualnie założonego wcześniej szyfru -- nie
potrzebny on już, bo od tego momentu wszystkie klucze będą zaszyfrowane przez agenta. Dodajmy zatem
nasze klucze:

    $ ssh-add /home/morfik/.ssh/id_ecdsa
    Enter passphrase for /home/morfik/.ssh/id_ecdsa:
    Identity added: /home/morfik/.ssh/id_ecdsa (/home/morfik/.ssh/id_ecdsa)

    $ ssh-add /home/morfik/.ssh/id_rsa
    Enter passphrase for /home/morfik/.ssh/id_rsa:
    Identity added: /home/morfik/.ssh/id_rsa (/home/morfik/.ssh/id_rsa)

    $ ssh-add /home/morfik/.ssh/id_ed25519
    Enter passphrase for /home/morfik/.ssh/id_ed25519:
    SSH_AGENT_FAILURE
    Could not add identity: /home/morfik/.ssh/id_ed25519

Jak widzimy wyżej, klucz `id_ed25519` zwraca błąd. W takim przypadku by korzystać z tego klucza,
trzeba będzie wpisywać hasło do niego, no chyba, że zrezygnujemy z hasła zupełnie, co nie jest
zalecane. Ja generalnie bym wolał już używać kluczy RSA.

Od chwili dodania kluczy do agenta, te są przechowywane w katalogu `~/.gnupg/private-keys-v1.d/` pod
nazwą 40 znakowego ciągu hexalnego + końcówka `.key` . Ten ciąg to tzw. keygrip, czyli hash
wszystkich publicznych parametrów klucza. Jest on również wpisany do pliku `~/.gnupg/sshcontrol`
wraz z dodatkowymi informacjami:

    # Key added on: 2014-10-01 22:32:10
    # Fingerprint:  b0:b2:c9:08:6d:8c:41:33:15:d4:41:d9:93:d6:b8:77
    576C2BF129D68EF23D4729F53711D44FEFB49855 0
    # Key added on: 2014-10-01 22:32:29
    # Fingerprint:  cb:a2:a5:70:01:12:fb:04:99:3d:be:bd:76:9a:29:ef
    1B42644E0A0EEB24C8F0152458E7EA2EC87AF108 0

Jeśli mamy zaimportowane klucze w agencie, to te prywatne w katalogu `~/.ssh/` nie są nam już do
niczego potrzebne, przynajmniej jeśli nie zamierzamy korzystać z aplikacji, które nie potrafią
obsługiwać poprawnie agenta GPG, np. mysql-workbench. W przypadku takich programów trzeba będzie
trzymać klucze prywatne.

## Przesyłanie klucza publicznego na serwer SSH

Musimy teraz skopiować nasz klucz publiczny na serwer, do którego chcemy się podłączyć. Możemy to
zrobić na kilka sposobów. Pierwszym z nich jest posłużenie się skryptem `ssh-copy-id` (dostępny w
pakiecie `openssh-client` ). Tworzy on listę z odciskami palca kluczy i próbuje ją wykorzystać do
zalogowania się do systemu by sprawdzić czy jakiś klucz jest już zainstalowany na zdalnej maszynie.
Jeśli mamy kilka kluczy i każdy przy tym ma inne hasło oraz nie korzystamy z gpg-agenta, zostaniemy
poproszeni o podanie hasła do każdego z kluczy. Po tej operacji zostanie zebrana lista kluczy, przy
pomocy których udało się zalogować do systemu, po czym zostaną one dodane do pliku
`~/.ssh/authorized_keys` na zdalnym systemie, na koncie użytkownika, na które skrypt zdołał się
zalogować. Poniżej jest przedstawiony proces dodawania kluczy z wykorzystaniem w/w skryptu:

    $ ssh-copy-id 11.22.55.66
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 2 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    morfik@11.22.55.66's password:

    Number of key(s) added: 2

    Now try logging into the machine, with:   "ssh '11.22.55.66'"
    and check to make sure that only the key(s) you wanted were added.

Jeśli mamy wiele kluczy i chcemy posłużyć się tylko jakimś określonym z nich, trzeba skorzystać z
parametru `-i` i podać ścieżkę do pliku, przykładowo:

    $ ssh-copy-id -i ~/.ssh/id_ecdsa.pub 11.22.55.66

Powyższa metoda nie zawsze jednak jest skuteczna. Jeśli nie działa ona w naszym przypadku, możemy
skorzystać z opcji manualnego przeprowadzenia opisanych wyżej operacji:

    $ scp ~/.ssh/id_ecdsa.pub morfik@11.22.55.66:
    $ ssh morfik@11.22.55.66
    $ mkdir ~/.ssh
    $ cat ~/id_ecdsa.pub >> ~/.ssh/authorized_keys
    $ rm ~/id_ecdsa.pub

W pierwszej linijce, po adresie hosta jest dodany `:` -- określa on katalog domowy, nie trzeba
podawać ścieżki typu `/home/morfik/` zamiast niego.

Jeśli mamy zamiar łączyć się do wielu serwerów, dobrze jest upchnąć ich konfigurację w pliku
`~/.ssh/config`:

    Host 22.11.55.66
        User morfik
        IdentitiesOnly yes
        IdentityFile ~/.ssh/id_ecdsa
        CheckHostIP yes
        Port 53214

W powyższym pliku można zdefiniować wiele takich bloków, a wszystkie możliwe opcje [są opisane w
manie](http://manpages.ubuntu.com/manpages/trusty/man5/ssh_config.5.html).

## Dodatkowe zabezpieczenia

Na maszynie zdalnej, po tym jak już przeprowadzimy wszystkie czynności związane z instalacją klucza
publicznego, dobrze jest zabezpieczyć plik `authorized_keys` przed zmianami, tak by żaden nowy klucz
nie mógł być dodany do tego pliku. W przeciwnym wypadku, umożliwiłoby to zalogowanie się komuś
trzeciemu na nasze konto. Jeśli nie posiadamy dostępu do konta admina na zdalnym serwerze, zawsze
można poprosić administratora systemu by zabezpieczył nam ten plik:

    $ chmod 400 ~/.ssh/authorized_keys
    # chattr +i ~/.ssh/authorized_keys

By dodatkowo zwiększyć bezpieczeństwo serwera, przydałoby się wyłączyć możliwość logowania przy
pomocy hasła. W przypadku gdy ktoś trzeci będzie się chciał podłączyć do serwera i nie będzie przy
tym posiadał klucza prywatnego, będzie on mógł próbować odgadnąć hasło do naszego konta. By
zrezygnować z uwierzytelniania opartego o login i hasło, w pliku `/etc/ssh/sshd_config` zmieniamy
poniższe linijki:

    PasswordAuthentication no
    ChallengeResponseAuthentication no

Po wyłączeniu logowania z użyciem haseł, jeśli stracimy klucz prywatny, nie dostaniemy się wtedy do
serwera. Możemy zostawić opcję logowania via hasło, z tym, że samo hasło powinno zawierać dłuższy
losowy ciąg znaków. Taki hash można sobie zaszyfrować lokalnie, np. przy pomocy kluczy gpg (lub
zwyczajnie schować) i trzymać na wypadek zgubienia klucza prywatnego. Wszelkie próby złamania hasła
w takim przypadku są pozbawione sensu, także mamy pewność, że nikt się nie dostanie do systemu przez
ewentualne ataki Brute Force na hasło.
