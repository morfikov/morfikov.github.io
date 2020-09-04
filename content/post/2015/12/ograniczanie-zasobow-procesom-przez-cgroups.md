---
author: Morfik
categories:
- Linux
date: "2015-12-08T21:21:53Z"
date_gmt: 2015-12-08 20:21:53 +0100
published: true
status: publish
tags:
- systemd
- cgroups
title: Ograniczanie zasobów procesom przez cgroups
---

Nasze komputery obecnie mają dość pokaźne zasoby obliczeniowe. Jeszcze nie tak dawno temu
wyposażenie maszyny w 32 GiB pamięci RAM, czy też 8 rdzeni było czystą abstrakcją. Wydawałoby się,
że te powyższe parametry zaspokoją każdego. Niemniej jednak, nie ważne jak szybki i rozbudowany
będzie nasz PC, to nam zawsze będzie mało. Mamy dwa rdzenie, to chcemy cztery. Mamy cztery, to
chcemy osiem, itd. Poza tym, szereg aplikacji realizuje co raz więcej zadań i staje się bardziej
wymagająca z każdym mijającym rokiem. Jeśli nie przeprowadzamy modernizacji sprzętu, to może się
okazać, że w niedługim czasie zabraknie nam pamięci albo pewne operacje będą wykonywane bardzo
wolno. W sporej części przypadków nie obędzie się bez wymiany podzespołów ale nawet w przypadku, gdy
mamy spory zapas zasobów systemowych, to poszczególne procesy rywalizują o nie ze sobą. Często bywa
tak, że chcielibyśmy, aby konkretny proces wykonał się szybciej, a to pociąga za sobą, np. zmianę
priorytetów w dostępie do rdzeni procesora. W linux'ie jest mechanizm zwany
[cgroups](https://en.wikipedia.org/wiki/Cgroups), który potrafi ograniczyć zasoby całym aplikacjom
bez względu na to ile ona by miała procesów. W tym wpisie postaramy się przebrnąć przez proces
konfiguracji tego mechanizmu i spróbujemy wyprofilować sobie nasz system.

<!--more-->
## Czym jest cgroups

[Cgroups](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt) ma postać wirtualnego
systemu plików, w którego skład wchodzą:
[blkio](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/ch-Subsystems_and_Tunable_Parameters.html#sec-blkio),
[cpu](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-cpu.html),
[cpuacct](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-cpuacct.html),
[cpusets](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-cpuset.html),
[devices](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-devices.html),
[freezer](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-freezer.html),
[memory](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-memory.html),
[net_cls](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-net_cls.html),
[net_prio](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/net_prio.html)
oraz
[perf_event](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-perf_event.html).
Linki są do strony RedHat'a, bo tam jest dużo przyjemniejszy opis niż na stronie kernela.

Z grubsza moduły odpowiadają kolejno kontrolę I/O dysków, za przydział procesora, za statystyki CPU,
za przydział konkretnego rdzenia i nodów pamięci, za dostęp do urządzeń, za zatrzymywanie i
wznawianie procesów, za przydział pamięci i statystyki pamięci, za przypisywanie pakietom sieciowym
odpowiednich klas dla [kontroli ruchu (traffic
control)](https://pl.wikipedia.org/wiki/Tc_%28Linux%29), za ustawianie priorytetu pakietom sieciowym
generowanym przez określone aplikacje w oparciu o soket SO_PRIORITY i ostatnia pozycja za
monitorowanie grup przy pomocy narzędzia `pref` .

W przeszłości debian miał pewne problemy z obsługą cgroups. [Obecnie cgroups jest wykorzystywany
przez systemd](https://www.freedesktop.org/wiki/Software/systemd/ControlGroupInterface/) i nie
musimy dodatkowo przeprowadzać żadnych czynności pod kątem jego konfiguracji, by podsystem cgroups
został automatycznie zamontowany w katalogu `/sys/fs/cgroup/` . Trzeba jednak pamiętać, że wymagane
są odpowiednie moduły, które muszą być włączone w kernelu:

    $ grep -i cgroup /boot/config-4.2.0-1-amd64
    CONFIG_CGROUPS=y
    # CONFIG_CGROUP_DEBUG is not set
    CONFIG_CGROUP_FREEZER=y
    CONFIG_CGROUP_DEVICE=y
    CONFIG_CGROUP_CPUACCT=y
    # CONFIG_CGROUP_HUGETLB is not set
    CONFIG_CGROUP_PERF=y
    CONFIG_CGROUP_SCHED=y
    CONFIG_BLK_CGROUP=y
    # CONFIG_DEBUG_BLK_CGROUP is not set
    CONFIG_CGROUP_WRITEBACK=y
    CONFIG_NETFILTER_XT_MATCH_CGROUP=m
    CONFIG_NET_CLS_CGROUP=m
    CONFIG_CGROUP_NET_PRIO=y
    CONFIG_CGROUP_NET_CLASSID=y

W tym przypadku, by móc skorzystać w pełni z możliwości oferowanych przez cgroups, trzeba będzie
załadować dodatkowo dwa moduły: `xt_cgroup` oraz `cls_cgroup` , oczywiście jeśli ich potrzebujemy.
Najlepiej ładować je wraz ze startem systemu, dlatego też do pliku `/etc/modules` dopisujemy te
poniższe wpisy:

    xt_cgroup
    cls_cgroup

W debianie musimy także dodatkowo dopisać do linijki kernela (w `extlinux` lub `grub` ) poniższy
parametr umożliwiający włączenie zarządzania pamięcią w cgroups:

    cgroup_enable=memory

By sprawdzić poprawność konfiguracji cgroups, posłużymy się narzędziem `lxc-checkconfig` dostępnym w
pakiecie `lxc` . Po weryfikacji, pakiet `lxc` możemy zwyczajnie usunąć, chyba, że interesują nas
[kontenery LXC]({{< baseurl >}}/post/konfiguracja-kontenerow-lxc/). Sprawdzamy zatem, czy wszystko
jest w porządku. Generalnie rzecz biorąc, to wszystkie pozycje powinny nam się zapalić na zielono,
tak jak to widać na fotce poniżej:

![]({{< baseurl >}}/img/2015/12/1.cgroups-konfiguracja-linux-debian.png#big)

Operowanie na cgroups odbywa się przez identyfikację procesu i aplikowanie reguł, co do tego ile
zasobów ten proces może wykorzystać. Tylko RAM można ustawić na sztywno. Pozostałe parametry nie
będą limitować zasobów w taki sposób jak człowiek może przypuszczać. Jeśli ustawimy max 50%
procesora pod jakiś proces, to ten proces będzie zjadał dostępne zasoby jak gdyby nigdy nic ale w
przypadku obciążenia maszyny, gdy ten proces chciałby zjadać więcej niż 50%, to mu zostanie to
zabronione. Dzięki temu inny proces będzie mógł wykorzystać swój przydział i nie zostanie zduszony
przez żarłoczną aplikację.

## Konfiguracja cgroups w systemd

W stosunku do praktycznie wszystkich demonów i usług systemowych, systemd jest w stanie dość
przyzwoicie skonfigurować przydział zasobów dla tych procesów. Mamy do dyspozycji szereg opcji,
które możemy umieścić w pliku `.service` . Wszystkie z nich można znaleźć [w
manualu](https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html). Poniżej
zaś przykład:

    [Service]
    ...
    CPUShares=256
    StartupCPUShares=256
    MemoryLimit=50M
    BlockIOWeight=128
    ...

Nie wszystkie kontrolery cgroups są jeszcze zaimplementowane. Niemniej jednak, te które są, w dużej
mierze działają przyzwoicie. Więcej przykładów konfiguracji cgroups w usługach systemd można znaleźć
[tutaj](http://0pointer.de/blog/projects/resources.html). Generalnie rzecz biorąc, konfiguracja
cgroups w systemd jest dość banalna. Problemy jednak zaczynają się gdy w grę wchodzą procesy
użytkownika. Nasuwa się zatem pytanie, jak ograniczyć zasoby przeglądarkom internetowym, np.
Firefox'owi?

## Procesy użytkownika w cgroups

Póki co [nie ma przyzwoitego
rozwiązania](https://lists.freedesktop.org/archives/systemd-devel/2015-April/031191.html) w
przypadku ograniczenia zasobów poszczególnym procesom użytkowników w systemie. Być może kiedyś taka
opcja zostanie dodana. Na razie zostaje nam startowanie usług ze zdefiniowanym użytkownikiem w pliku
`.service` , przykładowo:

    [Service]
    ...
    User=morfik
    Group=p2p
    ...

Istnieje jeszcze inna opcja ale do jej zaimplementowania potrzebować będziemy dwóch rzeczy. Pliku
konfiguracyjnego z regułami, na których to podstawie będą ograniczane zasoby, oraz demona, który
będzie identyfikował procesy jak tylko te zostaną zainicjowane. W ten sposób taki demon uzupełni
plik `tasks` o odpowiednie pid'y. By móc zastosować to rozwiązanie, potrzebujemy narzędzi z pakietu
`cgroup-tools` . Po zainstalowaniu tego pakietu tworzymy dwie usługi dla systemd.

Plik `/etc/systemd/system/cgrulesengd.service` :

    [Unit]
    Description=CGroup Rules Engine
    Documentation=man:cgrulesengd
    ConditionPathIsReadWrite=/etc/cgrules.conf
    DefaultDependencies=no
    Requires=cgconfig.service
    Before=basic.target shutdown.target
    After=local-fs.target cgconfig.service
    Conflicts=shutdown.target

    [Service]
    Type=simple
    ExecStart=/usr/sbin/cgrulesengd -n -Q

    [Install]
    #WantedBy=multi-user.target
    WantedBy=sysinit.target

Plik `/etc/systemd/system/cgconfig.service` :

    [Unit]
    Description=Control Group configuration service
    Documentation=man:cgconfigparser man:cgclear
    ConditionDirectoryNotEmpty=/sys/fs/cgroup/
    ConditionPathIsReadWrite=/etc/cgconfig.conf
    DefaultDependencies=no
    Before=basic.target shutdown.target
    After=local-fs.target
    Conflicts=shutdown.target

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/usr/sbin/cgconfigparser -l /etc/cgconfig.conf
    ExecStop=/usr/sbin/cgclear -l /etc/cgconfig.conf

    [Install]
    #WantedBy=multi-user.target
    WantedBy=sysinit.target

Potrzebna jest także konfiguracja dla tych usług. Tworzymy zatem dwa dodatkowe pliki:
`/etc/cgconfig.conf` oraz `/etc/cgrules.conf` .

Plik `/etc/cgconfig.conf` ma zawierać bloki podobne do tego poniżej:

    group users/firefox {
        perm {
            task {
                uid = root;
                gid = root;
                dperm = 775;
                fperm = 664;
            }
            admin {
                uid = root;
                gid = root;
                dperm = 775;
                fperm = 664;
            }
        }
        cpu {
            cpu.shares = "512";
        }
        memory {
            memory.limit_in_bytes = 512M;
            memory.soft_limit_in_bytes = 128M;
        }
        net_cls {
            net_cls.classid = 0x00010003;
        }
    }

W pliku `/etc/cgrules.conf` definiujemy zaś takie oto wpisy:

    *:firefox            cpu,memory,net_cls    users/firefox/
    *:firefox*           cpu,memory,net_cls    users/firefox/

Przeładowujemy konfigurację i odpalamy usługi:

    # systemctl daemon-reload
    # systemctl start cgrulesengd.service

Odpalmy teraz Firefox'a i sprawdźmy czy określone wyżej kontrolery cgroups ograniczają mu jakieś
zasoby:

    # cat /sys/fs/cgroup/memory/users/firefox/tasks | wc -l
    43

    # cat /sys/fs/cgroup/memory/users/firefox/memory.usage_in_bytes
    364240896

    # cat /sys/fs/cgroup/memory/users/firefox/memory.limit_in_bytes
    419430400

Oraz:

    # lscgroup | grep -i firefox
    cpu,cpuacct:/users/firefox
    net_cls,net_prio:/users/firefox
    memory:/users/firefox

    # cat  /proc/`pidof firefox`/cgroup
    9:memory:/users/firefox
    8:perf_event:/
    7:net_cls,net_prio:/
    6:cpuset:/
    5:freezer:/
    4:blkio:/user.slice/user-1000.slice/session-1.scope
    3:cpu,cpuacct:/users/firefox
    2:devices:/user.slice
    1:name=systemd:/user.slice/user-1000.slice/session-1.scope

Zatem wszystko działa w porządku.

## Konfiguracja cgroups z wykorzystaniem cgrulesengd i cgconfig

W zależności od konfiguracji określonej w pliku `/etc/cgconfig.conf` , w różnych podsystemach będą
tworzone grupy i podgrupy, którym będą nadawane okreslone ograniczenia dotyczące wykorzystania
zasobów systemowych. W tym przypadku zostanie utworzona grupa główna `users` i jej podgrupa
`firefox` w podsystemie `cpu` , `memory` oraz `net_cls` .

Ważne jest by w pliku `/etc/cgconfig.conf` nie umieszczać bloku `mount { }` , który odpowiada za
montowanie określonych zasobów cgroups. Tym ma się zając systemd, a on robi to automatycznie bez
naszej ingerencji.

Cały czas będzie nasłuchiwał demon `cgrulesengd` , który zidentyfikuje konkretne procesy w oparciu o
konfigurację w pliku `/etc/cgrules.conf` . Dla przypomnienia, przykładowa linijka w nim wygląda
następująco:

    *:firefox            cpu,memory,net_cls    users/firefox/

Pierwsza kolumna może się składać z użytkownika, w tym przypadku `*` odnosi się do wszystkich
użytkowników w systemie. Może być także określona grupa za pomocą `@` , przykładowo `@users` .
Proces nie jest wymagany ale można go sprecyzować po `:` w formie pełnej ścieżki do programu albo
nazwy procesu widocznego, np. w `ps` . Następna kolumna odpowiada za moduły cgroups, do których
proces będzie przypisywany. Jeśli proces ma być przypisany do wszystkich podsystemów cgroups, można
posłużyć się `*` . Ostatnia kolumna odnosi się do grupy, czyli tam gdzie plik `tasks` się powinien
znajdować. To ten plik będzie uzupełniany przez `cgrulesengd` .

### Uprawnienia do plików w /sys/fs/cgroup/

Możemy nadawać uprawnienia do plików, które będą tworzone w katalogu `/sys/fs/cgroup/` . Możemy
ustawić nie tylko prawa zapisu i odczytu ale także zmienić grupę i właściciela. Poniżej dla
przypomnienia stosowny kod:

    perm {
          task {
                uid = root;
                gid = root;
                dperm = 775;
                fperm = 664;
          }
          admin {
                uid = root;
                gid = root;
                dperm = 775;
                fperm = 664;
          }
    }

Blok `tasks` tyczy się plików `tasks` w konkretnej podgrupie. Czasami może się zdarzyć tak, że
chcemy, by konkretny użytkownik (czy też grupa) był w stanie ten plik zapisywać. Z kolei blok
`admin` odpowiada za użytkownika, który będzie administrował tą grupą. Prawa do katalogów i plików w
takiej grupie określamy zaś przez `dperm` i `fperm` . Można także wywołać `cgconfigparser` z opcją
`-s 1664` w usłudze systemd. Te uprawnienia nie przechodzą na podgrupy.

Dalej w pliku `/etc/cgrules.conf` są już definiowane bloki dotyczące konkretnych podsystemów
cgroups, które zawierają parametry i przypisane im wartości. Parametry noszą nazwę konkretnego
pliku, które można podejrzeć zaglądając do katalogu `/sys/fs/cgroup/` .

### Limitowanie czasu procesora

W przypadku pliku `cpu.shares` , wartość 512 oznacza 512/1024, czyli około 1/2 czasu procesora.
Domyślną wartością dla wszystkich procesów jest 1024 i tą wartością można się dowolnie bawić. Jeśli
ustawimy więcej niż 1024, np. 2048, proces dostałby 2/3 czasu procesora w przypadku wysokiego
obciążenia maszyny, bo 2048+1024=3072 i 2048/3072=2/3 . Oczywiście to wszystko przy założeniu, że
tylko dwa procesy by wykorzystywały procesor w 100% ale życie jest trochę bardziej skompilowane.

Trzeba uważać trochę w przypadku podgrup. Grupa główna, podobnie jak i podgrupy, może mieć własny
przydział procesora. W takim przypadku jego czas będzie liczony trochę inaczej. Dla uproszczenia,
załóżmy, że mamy 2 grupy główne: `A` oraz `B` , które mają `cpu.shares` odpowiednio 1024 i 2048.
Grupa `A` ma dwie podgrupy `A1` i `A2` , a grupa `B` ma tylko jedną podgrupę `B1` . Te podgrupy zaś
mają następujące wartości w `cpu.shares` : 512, 1024 i 2048 . Jak się rozłoży czas procesora przy
maksymalnym obciążeniu? Najpierw analizujemy grupy główne, czas procesora na te grupy rozłoży się w
stosunku 1/3 i 2/3, odpowiednio dla grup `A` i `B` (1024+2048=3072, 1024/3072 oraz 2048/3072).

Teraz podgrupy. Jeśli w grupie `A` nie będzie żadnych procesów (mogą być ale załóżmy, że nie ma), to
procesor przypadnie w 1/3 na grupę `A1` i w 2/3 na grupę `A2` (512/(512+1024) oraz 1024/(512+1024)
Jeśliby były jakieś procesy w grupie `A`, to rozkład mocy procesora przybrałby następującą postać:
dla grupy `A` 2/5 , dla `A1` 1/5 i dla `A2` 2/5 . Czemu tak? trzeba wziąć pod uwagę przydziały
wszystkich trzech grup (grupy głównej i dwóch podgrup), co daje nam 1024 ( `A` ) + 512 ( `A1` ) +
1024 ( `A2` ) = 2560 i odpowiednio 1024/2560 , 512/2560 , 1024/2560, co daje 2/5, 1/5 i 2/5 . To
tyle jeśli chodzi o grupę `A` , została jeszcze grupa `B` .

W grupie `B` jest tylko jedna podgrupa `B1` i w przypadku, gdy procesy będą tylko w podgrupie `B1` ,
cały przydział procesora jej przypadnie. Gdyby procesy także były w grupie `B` , podział procka
rozłoży się 1/2 dla `B` i 1/2 dla `B1` , bo obie mają takie same wartości `cpu.shares` (2048).

I chyba najbardziej złożony model, który można by rozpisać, biorąc pod uwagę powyższy przykład,
czyli gdy procesy trafiają do grup `A` , `A1` , `A2` , `B` oraz `B1` . Jak w takim przypadku rozłoży
się czas procesora? Jak już wiemy, grupy `A` i `B` podzielą procesor w stosunku 1/3 i 2/3. Rozkład w
grupie `A` wynosi 2/5, 1/5 i 2/5. Mnożymy to przez ratio wyższej grupy (1/3) i dostajemy wartości
2/15, 1/15 i 2/15. Łącznie daje nam to 5/15, czyli 1/3. Podobnie postępujemy z grupami `B` i `B1` ,
które mają współczynniki przydziału 1/2 i 1/2, mnożymy przez ratio 2/3, co daje 2/6 i 2/6, razem
4/6=2/3 . Jako, że 1/3+2/3=1, to wszystko się zgadza. Jeszcze dajemy to na wspólny mianownik 90
(6*15) i mamy współczynniki 12/90, 6/90, 12/90, 30/90 i 30/90, odpowiednio dla `A` , `A1` , `A2` ,
`B` i `B1` . Łącznie również 1. Przykład zaczerpnięty z [tej
strony](https://oakbytes.wordpress.com/2012/09/02/cgroup-cpu-allocation-cpu-shares-examples/) ale
odrobinę został zmieniony.

Trzeba jeszcze tylko pamiętać, że `cpu.shares` odnosi się do całego procesora, czyli wszystkich
rdzeni. Jeśli będzie ustawimy przydział, powiedzmy 512, to na dwurdzeniowym procesorze, proces
mógłby zjeść jeden rdzeń w pełni.

### Limitowanie zasobów pamięci RAM

Następny użyty kontroler to `memory.limit_in_bytes`. Ustawia on limit pamięci dla grupy i w tym
przypadku jest to 300 MiB. Po przekroczeniu tej wartości, dane będą zrzucane do SWAP. Dalej mamy
`memory.soft_limit_in_bytes` . Ten parametr jest podobny do tego powyżej i ma znaczenie głównie przy
zbyt dużym wykorzystaniu pamięci RAM, czyli, gdy brakuje zasobów. W takim przypadku trzeba będzie
zwolnić zasoby pamięci. To, które zostaną zwolnione, zależy od tego parametru właśnie. W tym
przypadku, na pierwszy ogień pójdzie 128 MiB tego procesu. Przez zwolnienie, ma się rozumieć, że
dane trafią do SWAP, a nie, że wylecą permanentnie z pamięci.

Jeśli ktoś jest ciekaw jak prezentują się statystyki pamięci, może je podejrzeć w dwóch poniższych
plikach:

    # cat /proc/`pidof firefox`/status
    # cat /sys/fs/cgroup/memory/users/firefox/memory.stat

W tych plikach są również zawarte informacje na temat tego ile dany proces zajmuje miejsca w
SWAP'ie.

### Oznaczanie pakietów sieciowych

Ostatnim użytym kontrolerem cgroups jest `net_cls.classid`. Moduł `net_cls` odpowiada za nadawanie
pakietom sieciowym odpowiednich klas, które są używane przy kontroli ruchu sieciowego (traffic
control). Jeśli chodzi o Firefox'a, ta aplikacja korzysta z internetu i przydałoby się odrobinę
kontrolować jej ruch. W tym celu nadawany jest ID grupy pakietom tworzonym przez procesy Firefox'a.
Oznaczenie `0x00010003` jest zapisem hexalnym i w jego skład wchodzą dwie liczby: 0001 oraz 0003. Po
przeliczeniu tego na system decymalny otrzymujemy 1 i 3, które narzędzie `tc` traktuje jako grupę
1:3 i tam właśnie wysyła pakiety, którym można nadać wyższy priorytet, określić przepustowość i tego
podobne rzeczy. Jeśli nie mamy pojęcia czym jest kształtowanie ruchu sieciowego, to raczej nie
potrzebujemy tego parametru.

## Błędy

I to w zasadzie tyle jeśli chodzi o implementację cgroups w debianie. W przypadku, gdyby coś nie
działało, to można użyć poniższych poleceń w celu sprawdzenia czy cgroups jest poprawnie montowany,
w których miejscach i czy procesy są przypisane do odpowiednich grup. I tak, np. sprawdźmy czy jest
coś w ogóle łapane przez cgroups:

    # lscgroup | grep -i firefox
    cpu,cpuacct:/users/firefox
    net_cls,net_prio:/users/firefox
    memory:/users/firefox

Jak widać powyżej, grupy są utworzone prawidłowo i cgroups je widzi w strukturze katalogów. Jeśli
jednak byśmy mieli więcej aplikacji kontrolowanych przez cgroups, możemy ograniczyć się do
wyszukania konkretnego procesu i sprawdzenia czy z nim jest wszystko w porządku. W tym celu trzeba
odszukać pid w katalogu `/proc/` :

    # cat /proc/`pidof firefox`/cgroup
    9:memory:/users/firefox
    8:perf_event:/
    7:net_cls,net_prio:/users/firefox
    6:cpuset:/
    5:freezer:/
    4:blkio:/user.slice/user-1000.slice/session-1.scope
    3:cpu,cpuacct:/users/firefox
    2:devices:/user.slice
    1:name=systemd:/user.slice/user-1000.slice/session-1.scope

Jeśli nie wiemy czy wirtualny system plików cgroups jest w ogóle montowany, możemy to sprawdzić
przez:

    # lssubsys -am
    cpuset /sys/fs/cgroup/cpuset
    cpu,cpuacct /sys/fs/cgroup/cpu,cpuacct
    blkio /sys/fs/cgroup/blkio
    memory /sys/fs/cgroup/memory
    devices /sys/fs/cgroup/devices
    freezer /sys/fs/cgroup/freezer
    net_cls,net_prio /sys/fs/cgroup/net_cls,net_prio
    perf_event /sys/fs/cgroup/perf_event

Jeśli chcemy mieć trochę więcej info, możemy odpytać `/proc/mounts` . Zaś informacje na temat
dostępnych podsystemów cgroups i ich hierarchii można odnaleźć pod `/proc/cgroups` .

Jeśli problem leży gdzieś indziej i powyższe polecenia zwracają pożądane wartości, być może problem
tkwi w demonie `cgrulesengd` . Najlepszym wyjściem jest sprawdzenie czy, aby na pewno pid'y procesów
trafiają do plików `tasks` . W przypadku ich braku, oznacza to, że demon albo nie działa, albo jest
źle skonfigurowany. W takiej sytuacji najlepiej jest uruchomić tego demona ręcznie z opcjami `-v -d
-f /var/log/cgrulesengd` .
