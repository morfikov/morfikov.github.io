---
author: Morfik
categories:
- Linux
date: "2016-01-03T16:24:45Z"
date_gmt: 2016-01-03 15:24:45 +0100
published: true
status: publish
tags:
- hdd
- ssd
- systemd
- temperatura
GHissueID: 536
title: Temperatura dysku twardego (hddtemp)
---

Obecnie dyski twarde nie są potrzebne do prawidłowego działania komputera. Mając sporo pamięci
operacyjnej oraz [system live][1], możemy bez problemu korzystać z takiego sprzętu. Niemniej jednak,
systemy live są nieco ograniczone. Najdotkliwszą ich wadą jest brak zapisywania wprowadzanych zmian.
W pewnych przypadkach ta cecha może być bardzo pożądana ale jeśli chodzi o przeciętnego użytkownika,
to chciałby on raczej mieć możliwość zapisu swojej pracy. Dlatego też zewnętrzny magazyn danych w
postaci dysku twardego nie prędko wyjdzie z użytku. Te urządzenia, jak i większość tych, które
podłączamy do naszego komputera, wydzielają ciepło. Temperatura jest wrogiem numer 1 w przypadku
maszyn i musi stale być monitorowana, tak by czasem nie doszło do przegrzania sprzętu.
[Dyski HDD][2] mają ten problem, że im wyższa jest temperatura, tym obszar magnetyczny się bardziej
rozszerza, a to powoduje błędy odczytu i zapisu. Podobnie jest ze zbyt niską temperaturą, gdzie
ścieżki i sektory ulegają skurczeniu. Taki dysk musi pracować w odpowiednich warunkach termalnych.
W tym wpisie postaramy się ustalić aktualną temperaturę dysku oraz spróbujemy ją monitorować, tak
by wiedzieć czy czynnik temperaturowy nie zagraża czasem dyskom podpiętym do naszego PC.

<!--more-->
## Narzędzie hddtemp

W linux'ie do pomiaru temperatury dysku wykorzystuje się z reguły [narzędzie hddtemp][3]. Jest ono
dostępne w oficjalnym repozytorium debiana w pakiecie o tej samej nazwie, zatem nie powinno być
problemów z jego instalacją. Ten pakiet nie ma usługi pod systemd, przynajmniej w przypadku
debiana, i jest odpalany za pośrednictwem skryptu init. Konfiguracja dla tego skryptu trzymana jest
zaś w pliku `/etc/default/hddtemp` . Rzućmy zatem okiem na ten plik:

    RUN_DAEMON="true"
    DISKS="/dev/sda"
    DISKS_NOPROBE=""
    INTERFACE="127.0.0.1"
    PORT="7634"
    DATABASE="/etc/hddtemp.db"
    SEPARATOR="|"
    RUN_SYSLOG="0"
    OPTIONS=""

Wyjaśnienie użytych opcji:

  - `RUN_DAEMON` uruchamia demona `hddtemp` .
  - `DISKS` określa, które dyski mają być monitorowane.
  - `DISKS_NOPROBE` to przeciwieństwo `DISKS` .
  - `INTERFACE` odpowiada za adres, a raczej za interfejs, na którym będzie nasłuchiwał demon
    `hddtemp` . Standardowo jest to 127.0.0.1 . Można także wpisać 0.0.0.0 i wtedy demon będzie
    nasłuchiwał na każdym dostępnym interfejsie.
  - `PORT` definiuje port. Domyślnie jest to 7634.
  - `DATABASE` określa plik bazy danych. Domyślnie `/etc/hddtemp.db` .
  - `SEPARATOR` określa jaki separator zostanie użyty przy rozdzielaniu konkretnych pól w wyjściu
    `hddtemp` .
  - `RUN_SYSLOG` ustawia logowanie temperatury w logu systemowym. Wartość jest podawana w sekundach
    i jeśli ustawiona na "0", logowanie jest wyłączone.
  - `OPTIONS` ustawia dodatkowe opcje.

### Usługa dla systemd

W przypadku korzystania z systemd, możemy pokusić się o napisanie dla tego init'u odpowiedniej
usługi. Nie jest to oczywiście wymagane, bo systemd bez problemu potrafi odpalać skrypty init ale
też nic nie stoi na przeszkodzie, by taką usługę sobie zrobić. Tworzymy zatem plik
`/etc/systemd/system/hddtemp.service` i dodajemy w nim tę poniższą treść:

    [Unit]
    Description=Hard drive temperature monitor daemon
    Documentation=man:hddtemp(8)

    [Service]
    Type=simple
    ExecStart=/usr/sbin/hddtemp /dev/sda \
          --daemon \
          --foreground \
          --listen=127.0.0.1 \
          --port=7634 \
          --file=/etc/hddtemp.db \
          --separator=| \
          --unit=C \
          -4

    [Install]
    WantedBy=multi-user.target

Użyte opcje są mniej więcej takie same co w przypadku skryptu init, za wyjątkiem `-4` . Ta opcja
sprawia, że demon `hddtemp` będzie nasłuchiwał jedynie w protokole IPv4. Niech nas też nie zmylą
opcje `--daemon` oraz `--foreground` , gdyż demon odpowiada za nasłuchiwanie na określonym adresie i
porcie, a nie za działanie usługi w tle. W przypadku systemd nie ma też potrzeby, by usługa sama
fork'owała się. Dlatego korzystamy z opcji `--foreground` i tę kwestię pozostawiamy samemu init'owi.

## Monitorowanie temperatury

Mamy zatem demona działającego w tle. By podejrzeć aktualną temperaturę, musimy wysłać zapytanie pod
adres, który został określony wyżej, przykładowo:

    # hddtemp /dev/sda
    /dev/sda: WDC WD2500BEKT-60A25T1: 41°C

W tym przypadku, temperatura dysku `/dev/sda` wynosi 41 stopni. Oczywiście nie będziemy w ten sposób
jej monitorować, bo do tego celu są odpowiednie narzędzia.

### Temperatura dysku w conky

[Conky][4] to tekstowy monitor systemu, który zwykle jest wyświetlany na pulpicie. Posiada on też
odpowiednie opcje od temperatury dysku. Wymagana jest jednak poprawna konfiguracja demona
`hddtemp` . By ta temperatura zwracana przez `hddtemp` została wyświetlona w `conky` , musimy do
jego pliku konfiguracyjnego dopisać `${hddtemp /dev/sda}˚C` . Temperatura dysku w conky prezentuje
się mniej więcej tak:

![conky-hddtemp-temperatura-dysku](/img/2016/01/1.conky-hddtemp-temperatura-dysku.png#small)

### Monitorowanie temperatury w monitorix

Monitor systemu `conky` daje nam jedynie możliwość obserwowania zmian temperatury w czasie
rzeczywistym. Nie mamy jednak żadnej historii dotyczącej jej zmian. Istnieje za to [narzędzie
monitorix][5], które jest w stanie monitorować nie tylko stan temperatury dysku twardego ale
również całą masę innych parametrów. Na potrzeby tego artykułu ograniczymy się jedynie do
temperatury dysku. Trzeba jednak zaznaczyć, że w debianie nie ma pakietu monitorix i nie damy rady
go zainstalować z oficjalnego repozytorium. Na stronie projektu jest za to pakiet `.deb` ,
którą możemy bez problemu dociągnąć i zainstalować ręcznie. [Alternatywnie zawsze możemy taki
pakiet .deb zrobić samodzielnie][6].

Konfiguracja sprowadza się do edycji pliku `/etc/monitorix/monitorix.conf` , gdzie przechodzimy do
sekcji grafów ( `graph_enable` ) i włączamy `hddtemp` , przykładowo:

    ...
    <graph_enable>
          ...
          hptemp            = y
          ...

Zapisujemy zmiany i restartujemy usługę. Monitor systemu jest dostępny z poziomu przeglądarki
internetowej pod adresem `:8080/monitorix/` . Wykres temperatury wygląda mniej
więcej tak:

![hddtemp-temperatura-monitoring-monitorix](/img/2016/01/2.hddtemp-temperatura-monitoring-monitorix.png#huge)

Obserwując go możemy w bardzo prosty sposób wywnioskować czy dyski w naszych komputerach mogą ulec
przegrzaniu w wyniki zbyt wysokiej temperatury i czy już najwyższy czas, by rozkręcić laptopa i go
nieco przeczyścić.


[1]: https://pl.wikipedia.org/wiki/Live_CD
[2]: https://pl.wikipedia.org/wiki/Dysk_twardy
[3]: https://manpages.ubuntu.com/manpages/wily/en/man8/hddtemp.8.html
[4]: https://github.com/brndnmtthws/conky
[5]: http://www.monitorix.org/
[6]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
