---
author: Morfik
categories:
- Linux
date:    2019-09-23 19:05:21 +0200
lastmod: 2020-09-02 17:50:00 +0200
published: true
status: publish
tags:
- apparmor
- debian
- initrd
- initramfs
title: Jak załadować profile AppArmor w fazie initrd/initramfs na Debian Linux
---

Zapewne wielu użytkowników Debiana zdążyło już zauważyć, że od wydania 10 (Buster), AppArmor jest
włączony domyślnie. Nie powinien on raczej sprawiać żadnych problemów po doinstalowaniu pakietów
`apparmor-profiles` oraz `apparmor-profiles-extra` , które zawierają szereg profili pod różne
aplikacje użytkowe. Niemniej jednak, pewnych procesów nie da się ograniczyć przez AppArmor,
przynajmniej nie w standardowy sposób. Chodzi o to, że jeśli mamy już odpalony jakiś proces, to nie
ma możliwości zamknąć go w profilu AA do momentu aż zakończy on swoje działanie i zostanie
uruchomiony ponownie. Profile AppArmor'a są ładowane podczas startu systemu w pewnym określonym
momencie ale szereg procesów systemowych startuje sporo wcześniej w stosunku do usługi AppArmor'a.
W taki sposób nawet jeśli w późniejszym czasie profile zostaną załadowane, to i tak część procesów
nie będzie ograniczona bez względu na to czy zdefiniowaliśmy im zestaw reguł. Oczywiście można
próbować restartować usługi lub szeregować je po `apparmor.service` ale nie zawsze tak się da
zrobić. Alternatywnym rozwiązaniem tego problemu jest ładowanie polityki AppArmor'a w fazie
initrd/initramfs, czyli w momencie, w którym nasz system nie ma jeszcze nawet uruchomionego procesu
z PID z numerkiem `1` .

<!--more-->
## Przeniesienie procesu ładowania profili do initrd/initramfs

Wiemy już, że nie możemy polegać na usługach systemowych mających za zadanie aplikować reguły
podczas startu systemu. Musimy zatem się zatroszczyć o ręczne ich ładowanie w fazie
initrd/initramfs. W tym celu potrzebne będą nam dwa skrypty.

Poniżej znajduje się zawartość pliku `/etc/initramfs-tools/hooks/apparmor` :

    #!/bin/sh

    set -e

    PREREQ=""
    prereqs()
    {
       echo "$PREREQ"
    }

    case $1 in
    prereqs)
       prereqs
       exit 0
       ;;
    esac

    [ -r /usr/share/initramfs-tools/hook-functions ] || exit 0
    . /usr/share/initramfs-tools/hook-functions

    copy_exec /sbin/apparmor_parser /sbin

    mkdir -p $DESTDIR/etc/apparmor/
    cp -a /etc/apparmor/ $DESTDIR/etc/
    mkdir -p $DESTDIR/usr/share/apparmor-features/
    cp -a /usr/share/apparmor-features/features $DESTDIR/usr/share/apparmor-features/
    mkdir -p $DESTDIR/etc/apparmor.d/
    cp -a /etc/apparmor.d/ $DESTDIR/etc/

    for i in $(ls /etc/apparmor.d/disable/); do rm $DESTDIR/etc/apparmor.d/$i ;done
    rm -R $DESTDIR/etc/apparmor.d/disable/

Ten powyższy skrypt ma za zadanie skopiować binarkę `apparmor_parser` oraz wszystkie niezbędne jej
pliki do obrazu initrd/initramfs. W tym obrazie mają znaleźć się także wszystkie profile AppArmor'a,
które mamy zdefiniowane w katalogu `/etc/apparmor.d/` . Jako, że cześć profili AA może być przez
nas wyłączona (linki do katalogu `/etc/apparmor.d/disable/` ), to musimy się ich pozbyć z obrazu,
tak by nie zostały one przez przypadek załadowane.

Niżej zaś znajduje się zawartość pliku `/etc/initramfs-tools/scripts/init-bottom/apparmor` :

    #!/bin/sh

    PREREQ=""
    prereqs()
    {
       echo "$PREREQ"
    }

    case $1 in
    prereqs)
       prereqs
       exit 0
       ;;
    esac

    export PATH=/sbin:/usr/sbin:/bin:/usr/bin

    mount -t securityfs none /sys/kernel/security

    echo -en "\n *** \033[5mApplying AppArmor profile list\033[0m ..."

    profile_list=$(find /etc/apparmor.d/ -maxdepth 1 -type f)
    apparmor_parser -a $profile_list
    if [ $? -eq 0 ]; then
        echo -e " \033[32mOK\033[0m"
    else
        echo -e " \033[31mFAIL\033[0m"
    fi

Ten skrypt z kolei ma za zadanie zamontować `/sys/kernel/security` w fazie initrd/initramfs oraz
zaaplikować reguły obecne w plikach profili. Warto tutaj zaznaczyć, by nie wywoływać
`apparmor_parser` dla każdego profilu z osobna, bo załadowanie ich w taki sposób może zająć nawet
kilka minut. Lepszym rozwiązaniem jest po prostu podanie wszystkich profili w argumencie dla
`apparmor_parser` , co zaoszczędzi nam sporo czasu podczas startu systemu.

Pamiętajmy by obu skryptom nadać prawa wykonywania:

    # chmod +x /etc/initramfs-tools/hooks/apparmor
    # chmod +x /etc/initramfs-tools/scripts/init-bottom/apparmor

### Generowanie obrazu initrd/initramfs

Generujemy teraz obraz initrd/initramfs przy pomocy `update-initramfs` :

    # update-initramfs -u -k all

Tak utworzony obraz powinien zawierać już wszystkie niezbędne pliki do zastosowania polityki
AppArmor'a w fazie initrd/initramfs. Możemy na wszelki wypadek podejrzeć jeszcze sam obraz:

    # lsinitramfs /boot/initrd.img-5.3.0-amd64| grep apparmor
    etc/apparmor
    etc/apparmor.d
    ...
    usr/sbin/apparmor_parser
    ...

## Usługa systemd apparmor.service

AppArmor dostarcza usługę dla systemd -- `apparmor.service` i to ona standardowo ładuje profile AA
podczas startu systemu. My ten krok ładowania profili mamy zrobiony już w fazie initrd/initramfs i
ponowne ładowanie profili podczas startu systemu jest już pozbawione sensu. Ta usługa jest dla nas
praktycznie bezużyteczna i możemy ją sobie spokojnie wyłączyć:

    # systemctl disable apparmor.service

## Test ładowania polityki AppArmor'a

Teraz wystarczy już jedynie restartować komputer. Zaraz na samym początku powinniśmy zauważyć
informację o ładowaniu profili AA:

![](/img/2019/09/001-debian-linux-boot-apparmor-initrd-initramfs.jpg#huge)

Po tym jak system się uruchomi, wpisujemy w terminal `aa-status` by sprawdzić czy profile zostały
załadowane oraz czy interesujące nas usługi systemowe są chronione:

![](/img/2019/09/002-debian-linux-apparmor-verify-profiles-load-initrd-initramfs.png#big)

Jak widać, zostało załadowanych nieco ponad 550 profili. Spośród wszystkich procesów aktualnie
działających w systemie, 51 jest ograniczonych przez politykę AA (49 enforce i 2 complain). Warto
zwrócić uwagę, że nie ma żadnego procesu który by miał zdefiniowany profil ale działał bez
ograniczeń ze strony AppArmor'a. W taki oto sposób można ograniczyć praktycznie dowolny proces
działający w systemie, nawet ten mający [PID z numerem 1][1].

## Cache profili

Standardowo usługa mająca na celu załadowanie wszystkich profili podczas startu systemu może czytać
całą masę plików, zwłaszcza gdy dość mocno oprofilowaliśmy sobie system. W takim przypadku
aplikowanie reguł AppArmor'a może zająć nawet dłuższą chwilę spowalniając tym samym start systemu.
Istnieje jednak sposób, by ten czas wczytywania profili dość mocno zredukować za sprawą cache,
który domyślnie jest wyłączony. By włączyć cache profili trzeba edytować plik
`/etc/apparmor/parser.conf` usuwając `#` w poniższej linijce:

    write-cache

W taki sposób `apparmor_parser` będzie tworzył i aktualizował cache ilekroć tylko będziemy dodać
czy przeładowywać profile aplikacji. Standardowo cache jest przechowywany w katalogu
`/var/cache/apparmor/` i by mieć możliwość skorzystania z niego w fazie initramfs/initrd, musimy
przekopiować ten katalog do obrazu initramfs/initrd. Musimy zatem do skryptu
`/etc/initramfs-tools/hooks/apparmor` dodać poniższy wpis:

    mkdir -p $DESTDIR/var/cache/apparmor/
    cp -a /var/cache/apparmor/ $DESTDIR/var/cache/

Teraz już wystarczy wygenerować cache, np. za pomocą usługi `apparmor.service` dla systemd oraz
zaktualizować obraz initramfs/initd:

    # systemctl restart apparmor.service
    # update-initramfs -u -k 5.8.5-amd64

Podczas startu systemu powinniśmy zauważyć dość mocny wzrost wydajności podczas wczytywania profili
AppArmor'a. W moim przypadku, gdzie mam tych profili już blisko koło tysiąca, czas potrzebny na ich
wczytanie został zredukowany z około 40s do poniżej jednej sekundy. Zatem moja maszyna startuje o
wiele szybciej niż w sytuacji, gdy nie robiło się użytku z tego cache.


[1]: https://gitlab.com/apparmor/apparmor/wikis/FullSystemPolicy
