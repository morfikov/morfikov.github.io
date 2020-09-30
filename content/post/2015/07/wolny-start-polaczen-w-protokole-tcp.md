---
author: Morfik
categories:
- Linux
date: "2015-07-07T19:33:16Z"
date_gmt: 2015-07-07 17:33:16 +0200
published: true
status: publish
tags:
- tcp
- sysctl
- sieć
title: Wolny start połączeń w protokole TCP
---

Wolny start jest wynikiem braku zaufania maszyny nadawczej do utworzonego kanału przesyłowego --
połączenia. Nie wie ona czy to łącze jest bowiem w stanie obsłużyć taką porcję danych, którą ma
zamiar przesłać bez czekania na pakiet `ACK` . W przypadku bufora odbiorczego, wszystko jest proste,
bo dane dotyczące wielkości okien są zdefiniowane w nagłówku pakietów, do których ma wgląd druga ze
stron. W przypadku gdy nadawca przesyła dane, prędkość z jaką to robi zależy głównie od niego.

<!--more-->
## Wolny start -- zasada działania

Po ustanowieniu połączenia, maszyny na dwóch jego krańcach próbują zwiększyć rozmiar okna TCP, tak
by dane były przesyłane z prędkością obsługiwaną przez co najmniej jedną ze stron. Podczas tego
procesu, połączenie przechodzi w fazę wykładniczego wzrostu (exponential growth). Rozmiar okna TCP
jest zwiększany z każdym otrzymanym pakietem `ACK` o liczbę zaakceptowanych segmentów (dwukrotnie co
kolejny RTT). Trwa to do chwili gdy jakieś potwierdzenie nie zostanie dostarczone lub gdy zostanie
osiągnięty próg wielkości okna TCP. Gdy to nastąpi, połączenie przechodzi w fazę liniowego wzrostu
(linear growth), podczas której wielkość okna jest zwiększana o rozmiar jednego segmentu z każdym
RTT i trwa to do momentu utraty potwierdzenia. Cały ten proces od ustanowienia połączenia do końca
drugiej fazy nazywa się wolnym startem.

Kluczowym elementem wolnego startu jest CWND (Congestion Window Size), czyli bufor nadawczy hosta
wysyłającego dane. Odpowiada on za ilość segmentów składających się na początkowe okno TCP.
Domyślnie ten parametr ma różną wartość i zależy od konkretnego systemu operacyjnego. Na linuxach
było to z początku [4*MSS](https://lwn.net/Articles/427104/) (Maximum Segment Size). Obecnie jest
to wartość 10*MSS. Jako, że MSS to zwykle 1460 bajtów, to rozmiar okna, który jest zgłaszany w
pakietach `SYN` wynosi `14600` bajtów.

## Zmiana wielkości okna początkowego

Wraz ze wzrostem przepustowości łącz oraz przez to, że połączenia są zwykle krótsze niż kiedyś,
sporo z nich nigdy nie rozwija pełnej prędkości, właśnie ze względu na procedurę wolnego startu, co
pociąga za sobą zwiększenie opóźnień. Na dobrą sprawę, optymalny rozmiar początkowego okna to około
16 do 20 pakietów. Nie ma jednak prostego sposobu, by zweryfikować jaki rozmiar okna jest ustawiony
na danym linuxie. No chyba, że wywnioskujemy to ze sniffera pakietów lub zajrzymy w źródła kernela.

Jeśli ktoś chce odszukać wartość w kodzie kernela, to wystarczy pobrać pakiet `linux-source` i
wypakować paczkę `.xz` , która zostanie wgrana do katalogu `/usr/src/` . Następnie, posługując się
poniższym poleceniem, zostanie nam objawiona poszukiwana wartość:

    # grep -A 2 initcwnd `find /usr/src/linux/include -type f -iname '*h'`
    ...
    /usr/src/linux/include/net/tcp.h-#define TCP_INIT_CWND          10

Zatem jest to 10 pakietów. Możemy zwiększyć tę wartość do 20, np. przez dopisanie do pliku
`/etc/network/interfaces` poniższej zawartości:

    post-up ip route change default via 192.168.1.1 dev eth0 initcwnd 20 initrwnd 20

Jeśli teraz odpalimy sniffer sieciowy, np. wireshark, to powinniśmy ujrzeć wartość `29200` w opcjach
okna TCP w pakietach `SYN` , poniżej fotka:

![](/img/2015/06/1.wolny-start-okno-poczatkowe-wireshark.png#huge)

## Zagubione pakiety oraz połączenia IDLE

Faza wolnego startu może pojawić się w kilku miejscach połączenia, pomijając oczywiście jej
występowanie na samym początku transferu danych. Jedną z takich sytuacji jest przypadek, w którym
następuje retransmisja pakietu, który uległ zagubieniu. Ogólnie rzecz biorąc, chodzi o to, że gdy
nadawca przesyła pakiet do odbiorcy, to ma przy tym pewien przedział czasu, w obrębie którego
odbiorca musi odesłać pakiet `ACK` potwierdzający odbiór danych. W chwili gdy upływa ten czas i
nadawca nie otrzymał takiego potwierdzenia, retransmituje pakiet. Z chwilą gdy to robi połączenie
przechodzi w stan wolnego startu w celu ustalenia optymalnej prędkości przesyłu danych. Z tym, że [w
zależności od zastosowanego
algorytmu](http://www.roman10.net/2011/11/10/tcp-tahoe-reno-newreno-and-sacka-brief-comparison/),
ten proces może przebiegać nieco inaczej.

Inną sytuacją jest stan bezczynności połączenia. To taki stan, gdzie sesja jest aktywna ale przez
dłuższy okres czasu nie zostały przesłane żadne dane za wyjątkiem czegoś co się nazywa pakiety
keepalive, np. połączenia SSL/TLS. Gdy dochodzi do ponownej transmisji danych, trzeba na nowo
ustalić dla nich rozmiar okna, czyli połączenie znowu przechodzi w stan wolnego startu. W tym
przypadku mamy jednak nieco większe pole manewru i jesteśmy w stanie zadecydować czy wolny start w
przypadku połączeń IDLE ma mieć miejsce czy też nie. Jeśli zdecydujemy się wyłączyć tę opcję, to po
wznowieniu transmisji dane będą przesyłane w oknach o rozmiarach ustalonych wcześniej, zanim
połączenie weszło stan IDLE.

Poniżej jest parametr wyłączający wolny start dla połączeń IDLE. Trzeba go dopisać do pliku
`/etc/sysctl.conf` :

    net.ipv4.tcp_slow_start_after_idle = 0

By zmiany weszły w życie, wydajemy poniższe polecenie:

    # sysctl -p
