---
author: Morfik
categories:
- Linux
date: "2015-11-21T19:52:58Z"
date_gmt: 2015-11-21 18:52:58 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- użytkownicy
- grupy
title: Czy zmiana nazwy użytkownika root ma sens?
---

Wielu ludzi potrafi się rozpisywać na temat bezpieczeństwa systemu linux jednocześnie nie zauważając
jednego bardzo poważnego problemu. Wszyscy wiemy, że linux'y są bezpieczne min. przez fakt
rozgraniczenia konta administratora od konta zwykłego użytkownika i nadaniu im różnych praw dostępu
do poszczególnych części systemu operacyjnego. O ile konta użytkowników mają różne nazwy, o tyle
administrator w praktycznie każdym linux'ie kryje się pod nazwą `root` . Znając nazwę konta, można
próbować złamać hasło. To trochę dziwne, że nie można sobie arbitralnie ustalić nazwy dla tego konta
tak by uniknąć wszelkich ataków, które związane są z logowaniem się na określonego użytkownika w
systemie. W tym wpisie postaramy się prześledzić całą procedurę zmiany nazwy konta `root` na jakąś
dowolną i zobaczymy czy wpłynie to w jakimś stopniu na pracę naszego systemu.

<!--more-->
## Zmiana nazwy administratora

Możemy, co prawda, stworzyć sobie nowe konto przy pomocy `adduser` i przypisać mu taką konfigurację
jaką ma użytkownik `root` ale ten krok nie jest wymagany. Każde konto w linux'ie jest identyfikowane
przez kilka plików, których wpisy są ze sobą ściśle powiązane. Możemy zatem zwyczajnie edytować te
pliki i odpowiednio zmienić w nich poszczególne linijki. Chodzi generalnie o zmianę nazwy
użytkownika `root` na, np. `roocica` . Możemy także dostosować sobie ścieżkę katalogu domowego.
Poniżej znajdują się te pliki, które musimy zmienić:

  - `/etc/passwd` przechowuje informacje o kontach w systemie.
  - `/etc/shadow` tutaj znajdują się zahashowane hasła.
  - `/etc/group` zawiera konfigurację grup, do których można dodawać poszczególnych użytkowników.
  - `/etc/gshadow` przechowuje zaszyfrowane hasła do grup.

Po dokonaniu zmian, wpisy od nowego konta administracyjnego powinny wyglądać mniej tak:

    # egrep roo /etc/passwd
    roocica:x:0:0:roocica:/roocica:/bin/zsh

    # egrep roo /etc/shadow
    roocica:zahashowane-haslo:16401:0:99999:7:::

    # egrep roo /etc/group
    roocica:x:0:

    # egrep roo /etc/gshadow
    roocica:*::

Pamiętajmy, że w przypadku nowego katalogu domowego, musimy go stworzyć i nadać mu odpowiednie
uprawnienia przy pomocy `chown` oraz `chmod` .

## Dostosowanie konfiguracji systemu

W tym momencie będziemy już w stanie zalogować się na nowe konto podając użytkownika `roocica` oraz
stare hasło. Nie będzie można się jednak zalogować już na konto `root` , bo ono zwyczajnie nie
istnieje. To nie jest jednak koniec naszej pracy. Musimy jeszcze dostosować konfigurację systemu,
tak by poszczególne usługi były wykonywane jako nowy użytkownik. W większości przypadków nic nie
będziemy musieli zmieniać. Poniżej znajdują się te usługi, które standardowo są dostępne w debianie
i po tym jak nastąpiła zmiana nazwy konta administratora powodują problemy.

### Dbus

Pierwszą usługą, która wymaga zmiany kilku rzeczy jest dbus. Jego pliki konfiguracyjne znajdują się
w `/etc/dbus-1/` i w nich musimy zmienić `user="root"` na `user="roocica"` . W zależności od
zainstalowanych pakietów w systemie, w tym powyższym katalogu mogą się znaleźć różne pliki. Dlatego
też najlepiej jest je przejrzeć przy pomocy `egrep` i pozmieniać odpowiednie pozycje, przykładowo:

    # egrep -r root /etc/dbus-1/
    /etc/dbus-1/system.d/org.freedesktop.systemd1.conf:
    /etc/dbus-1/system.d/org.freedesktop.locale1.conf:
    ...

### Cron i anacron

Wszystkie zadania, które są odpalane czasowo przez cron'a muszą być wykonane jako określony
użytkownik i zwykle jest to `root` . Po tym jak nastąpiła zmiana tej nazwy, musimy rzucić okiem na
pliki `/etc/crontab` oraz wszystkie te znajdujące się w `/etc/cron.d/` . Generalnie to struktura
tych plików wygląda mniej więcej tak:

    ...
    17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
    ...

Widzimy tutaj, że to powyższe polecenie zostanie uruchomione jako użytkownik`root` . Musimy
przepisać wszystkie te wpisy tak by zawierały nazwę nowego użytkownika `roocica` .

Nie można także zapominać o dostosowaniu skryptów, które znajdują się w katalogach
`/etc/cron.daily/` , `/etc/cron.hourly/` , itp. One również muszą zostać dostosowane pod kątem
nowego użytkownika.

### Logrotate

Jako, że bawimy się w zmianę nazwy administratora, to trzeba wziąć poprawkę na wszelkie narzędzia,
które generują czy też operują na logach. `logrotate` ma swoją konfigurację w pliku
`/etc/logrotate.conf` . Dodatkowo, szereg usług wgrywa swoje własne pliki do obróbki logów w
`/etc/logrotate.d/` . Problematyczne są tylko te wpisy, które mają za zadanie zmienić użytkownika i
grupę nowo tworzonych plików logów. Zwykle jest to `root` . Musimy zatem wszystkie te wystąpienia
odpowiednio przepisać.

### Skrypty /etc/init.d/

Szereg skryptów init, które znajdują się w katalogu `/etc/init.d/` również może operować na
uprawnieniach i sprawdzać czy określone pliki mają nadane odpowiednie prawa do plików, w tym też
użytkownika i grupę. Jeśli w tych skryptach jest odwołanie do konta `root` , będą one miały
problemy z uruchomieniem się.

Trzeba także pamiętać, że wszystkie demony odpalane za pomocą skryptów init mogą mieć dodatkową
konfigurację w katalogu `/etc/` i to w tych plikach może być określony użytkownik i grupa, które
następnie są przekazywane do skryptu init. Podobnie sprawa ma się w przypadku plików w katalogu
`/etc/default/` , gdzie zwykle są umieszczane parametry podawane demonom usług, tak by nie trzeba
było zmieniać bezpośrednio samego skryptu init.

### Systemd

W przypadku gdy korzystamy z systemd, to mogliśmy się natknąć na kilka ciekawych parametrów w pliku
`/etc/systemd/logind.conf` . Chodzi o zabijanie procesów tych użytkowników, których sesje przestają
istnieć. W ten sposób, po wylogowaniu się użytkownika, mamy pewność, że żaden proces, który on
uruchomił, nie działa w tle. Jeśli włączyliśmy sobie tę funkcjonalność, to musimy przepisać
użytkownika, który ma być wyjęty spod tego mechanizmu. Standardowo jest to `root` :

    ...
    KillUserProcesses=yes
    KillOnlyUsers=morfik
    KillExcludeUsers=root
    ...

### Sudo

W konfiguracji `sudo`, również musimy zmienić kilka rzeczy. Głównie chodzi o przepisanie pliku
`/etc/sudoers` , gdzie mamy określone komendy, które zwykli użytkownicy mogą wykonywać jako
użytkownik `root` , przykładowo:

    ...
    morfik     HOSTY = (root) NOPASSWD: /usr/sbin/pbuilder
    ...

## Konkluzja

Te powyższe zmiany, to tylko początek. Jest wielce prawdopodobne, że wiele innych usług ma również
podobne problemy z uprawnieniami, tj. operuje na nazwach konta użytkownika, a nie na UID/GID.
Dodatkowo, te numerki nie zawsze są stałe. Przy bardziej skomplikowanych systemach, tych z GUI,
zmiana nazwy konta administratora może być niemożliwa. W środowiskach tekstowych, przy małej ilości
usług systemowych, to jest raczej wykonalne. Problem w tym, że przy aktualizacji pakietów, szereg
plików może zostać nadpisanych bez naszej wiedzy. Niekoniecznie muszą to być pliki w katalogu
`/etc/` , niemniej jednak nie zaglądałem w inne katalogi. Udało mi się, co prawda, doprowadzić
system do ładu, tak by nie zwracał podczas startu żadnych błędów ale to był tylko testowy system i
na dobrą sprawę nie używałem go za długo by powiedzieć, czy zmiana nazwy użytkownika `root` będzie w
ogóle użyteczna. Jeśli martwimy się o bezpieczeństwo związane ze zdalnym logowaniem na shell, to
najlepiej jest tak skonfigurować sobie demona ssh, by zrzucał połączenia próbujące logować się na
konto `root` .
