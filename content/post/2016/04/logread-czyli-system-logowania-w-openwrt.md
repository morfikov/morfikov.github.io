---
author: Morfik
categories:
- OpenWRT
date: "2016-04-28T21:04:26Z"
date_gmt: 2016-04-28 19:04:26 +0200
published: true
status: publish
tags:
- chaos-calmer
- logi
- router
- rsyslog
GHissueID: 404
title: Logread, czyli system logowania w OpenWRT
---

Każdy szanujący się system, nawet ten najmniejszy na bazie OpenWRT, musi posiadać mechanizm
logowania komunikatów. Logi routera to bardzo ważna rzecz. Jeśli coś dolega naszemu małemu
przyjacielowi, to jest niemal pewne, że właśnie wśród tych wiadomości znajdziemy przyczynę
problemów. Każda usługa systemowa działająca na routerze przesyła logi, które są zbierane przez
demon logowania. W Chaos Calmer odpowiadają za to `logd` oraz `logread` . Standardowa konfiguracja
logów w OpenWRT nie jest raczej skomplikowana ale niewiele osób wie, że logi routera można zapisywać
w pliku lub przesłać je przez sieć do innego hosta. W tym wpisie postaramy się właśnie zrealizować
te dwa zadania.

<!--more-->
## Odczytywanie logów za pomocą logread

Standardowo po instalacji OpenWRT, wszystkie komunikaty systemowe są gromadzone w pamięci routera.
Dlatego też, za każdym razem, gdy router zostanie uruchomiony ponownie, wszelkie informacje o
przeszłych zdarzeniach są wymazane. Utrudnia to trochę analizę problemów, zwłaszcza w przypadku,
gdy nie możemy z jakiegoś powodu zdalnie się dostać do urządzenia. Niemniej jednak, tego typu
zdarzenia są raczej rzadkie, dlatego też przyjrzyjmy się nieco bliżej narzędziu `logread` . Logujemy
się zatem na router i w terminalu wpisujemy polecenie `logread` :

![](/img/2016/04/1.logread-openwrt-komunikaty-log.png#huge)

Wyżej mamy szereg komunikatów, które zostały zwrócone przez działające na routerze usługi.
Oczywiście log uzyskany w taki sposób jest dość obszerny ale bez problemu może go przefiltrować
przy pomocy polecenia `grep` :

![](/img/2016/04/2.logread-openwrt-komunikaty-log-filtr.png#huge)

Każdy komunikat zawiera datę zalogowanej wiadomości `Sat Apr 16` , czyli w tym przypadku sobota, 16
kwietnia. Dalej mamy dokładny czas `08:29:29` . Następnie mamy rok `2016` chyba. Dziwny zapis. W
każdym razie już w oparciu o te informacje możemy przeszukiwać log. Dalej mamy obiekt i poziom
logowania `daemon.info` . [Wszystkie możliwe kombinacje są wyszczególnione
tutaj](https://en.wikipedia.org/wiki/Syslog#Facility). Potem mamy właściwą nazwę usługi oraz jej
numer PID `dnsmasq-dhcp[1134]` . No i oczywiście na końcu jest komunikat, który wysłała usługa.

## Logowanie komunikatów do pliku

Zwykle domowe routery nie udostępniają żadnych usług w sieci i nie są otwarte na świat, bo nie mają
stałego zewnętrznego adresu IP. Jeśli jednak obawiamy się o bezpieczeństwo naszego routera czy
sieci, to powinniśmy włączyć zapisywanie logów do pliku. Niemniej jednak, routery średnio się
nadają, by ciągle zapisywać dane u nich na flash'u. Raz, że mają go niewiele, a dwa, że bardzo
szybko nam ten flash padnie przy takim użytkowaniu. Dlatego też, jeśli rozważamy wdrożenie
logowania, które ma przetrwać proces restartu routera, to najlepiej [zaopatrzyć się w niewielki
pendrive i wykonać na nim
extroot'a](/post/extroot-whole_root-fullroot-pod-openwrt/). Taki mechanizm
logowania jest do skonfigurowania przez plik `/etc/config/system` . Poniżej znajdują się wpisy,
które trzeba do tego pliku dodać:

    config system
          ...
          option log_type 'file'
          option log_file '/mnt/log/messages'
          option log_size '64'

Przy pomocy opcji `log_type` mówimy routerowi, że życzymy sobie zapisywać logi do pliku. Lokalizacja
tego pliku jest określona w opcji `log_file` . Z kolei `log_size` odpowiada za rozmiar tego pliku, w
KiB. Zapisujemy plik i w terminalu wydajemy to poniższe polecenie:

    # /etc/init.d/log restart

Od tego momentu logi powinny wędrować również do wskazanego wyżej pliku. W dalszym ciągu logi możemy
odczytywać przy pomocy `logread` .

## Zdalne logowanie wiadomości

Istnieje także możliwość przesłania logów przez sieć do określonego hosta. Do końca nie wiem czy
takie logi można przesłać do maszyn mających na pokładzie windowsa. Niemniej jednak bez problemu ten
mechanizm współpracuje z linux'ami. Dlatego też ograniczę się do opisania konfiguracji w oparciu
właśnie o ten system operacyjny, jako że go używam.

Przede wszystkim, potrzebna nam jest maszyna, która będzie miała zainstalowany na swoim pokładzie
serwer logów. Obecnie większość dystrybucji linux'a korzysta z initu systemd. Jego logger, tj.
journal, nie będzie współpracował z OpenWRT. Potrzebne zatem będzie nam kompatybilne oprogramowanie.
Na szczęście nic nie stoi na przeszkodzie, by doinstalować sobie pakiet `rsyslog` . To on będzie
odpowiedzialny za odbieranie zdalnych komunikatów. Niemniej jednak, by był w stanie to zrobić,
musimy edytować plik `/etc/rsyslog.conf` i dopisać w nim tę poniższą konfigurację:

    $ModLoad imtcp
    $InputTCPServerRun 514
    $InputTCPServerStreamDriverPermittedPeer 192.168.1.1

Dodatkowo przyda nam się również rozdział logów z routera i zapisanie ich w osobnym pliku:

    if $fromhost-ip startswith "192.168.1.1" then -/var/log/home-network.log
    & stop

Teraz wracamy na router w celu skonfigurowania demona `logread` . Otwieramy plik
`/etc/config/system` i w przypadku, gdy mamy skonfigurowane logowanie do pliku, to musimy
wykomentować te powyżej opisane opcje. Na ich miejsce dodajemy te poniższe:

    config system
    ...
          option log_ip     '192.168.1.150'
          option log_port   '514'
          option log_proto  'tcp'
          option log_prefix 'router '

Trzeba mieć na uwadze, że te logi przesyłane za pomocą tego mechanizmu są w formie niezaszyfrowanej.
Standardowa konfiguracja OpenWRT nie daje możliwości zaszyfrowania przesyłanych logów. Niemniej
jednak, [szyfrowanie logów jest
możliwe](/post/szyfrowanie-logow-w-openwrt-syslog-ng/). Wyżej jednak mamy cztery
opcje, którymi musimy się zająć. Serwer, który ma przechowywać logi określamy w `log_ip` .
Standardowy port dla `rsyslog` to `514` . Dopisujemy go zatem w opcji `log_port` . Wybieramy
protokół TCP w `log_proto` . Na końcu w opcji `log_prefix` mamy prefiks, który będzie dołączany do
wiadomości, tak by szło w łatwy sposób odróżnić komunikaty generowane przez system od tych, które
pochodzą z OpenWRT na routerze. Zapisujemy plik i ponownie restartujemy skrypt `/etc/init.d/log` :

    # /etc/init.d/log restart

Od tego momentu, komunikaty powinny być przesyłane przez sieć. Poniżej jest fotka dowodząca, że tak
jest w istocie:

![](/img/2016/04/3.logread-rsyslog-openwrt-komunikaty-log-siec.png#huge)
