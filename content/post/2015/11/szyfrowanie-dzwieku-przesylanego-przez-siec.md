---
author: Morfik
categories:
- Linux
date: "2015-11-01T00:31:48Z"
date_gmt: 2015-10-31 22:31:48 +0100
published: true
status: publish
tags:
- pulseaudio
- ssh
- szyfrowanie
- dźwięk
GHissueID: 266
title: Szyfrowanie dźwięku przesyłanego przez sieć
---

[PulseAudio to serwer dźwięku](https://www.freedesktop.org/wiki/Software/PulseAudio/), który jest w
stanie otrzymywać zapytania ze zdalnych lokalizacji. Wobec czego, możemy realizować [przesyłanie
dźwięku przez sieć](/post/pulseaudio-i-przesylanie-dzwieku-przez-siec/) i usłyszeć
go tam, gdzie go sobie życzymy. Problem w tym, że taki dźwięk jest przesyłany przez sieć w formie
niezaszyfrowanej. Dlatego też jesteśmy narażeni na podsłuchanie wszystkiego co mówimy do mikrofonu
lub też tego co pojawia się w naszych głośnikach. Możemy jednak zabezpieczyć komunikację między
klientem i serwerem dźwięku wykorzystując do tego połączenie SSH. W ten sposób cały sygnał
dźwiękowy, jaki jest generowany przez danego hosta w sieci, zostanie wrzucony w szyfrowany kanał
TLS i nikt nie będzie w stanie go zinterpretować. Ten wpis ma na celu przedstawienie sposobu na
zaszyfrowanie dźwięku, bez którego większość z nas nie wyobraża sobie pacy przy komputerze.

<!--more-->
## Szyfrowanie dźwięku

Będąc w stanie przesłać dźwięk z jednej maszyny na drugą, możemy pokusić się o zaszyfrowanie go. Ten
krok zalecany jest tylko w przypadku przesyłania dźwięku przez internet, bo jakby nie patrzeć, w
sieci lokalnej podsłuch nam nie grozi. Trzeba jednak mieć na uwadze, że szyfrowanie zjada cenne
zasoby i nie każdy sprzęt jest w stanie obsłużyć te bardziej wymagające algorytmy, zwłaszcza gdy
danych, które trzeba zaszyfrować, jest bardzo dużo.

We wstępie podlinkowałem artykuł, który omawiał przesyłanie dźwięku przez sieć. Jak możemy w nim
wyczytać, implementacja tego rozwiązania nie jest szczególnie trudna i sprowadza się jedynie do
załadowania w PulseAudio modułu `module-native-protocol-tcp` oraz wyeksportowanie na kliencie
zmiennej `$PULSE_SERVER` . Do zaszyfrowania ruchu między klientem i serwerem dźwięku wykorzystamy
SSH, a konkretnie jedną z możliwości, które to oprogramowanie nam oferuje, tj. [port
forwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding).

### Przekierowanie portów (port forwarding)

Mechanizm port forwarding'u polega na przekierowaniu ruchu z danego portu na jakiś inny i w tym
przypadku, ruch który będzie kierowany na domyślny port serwera PulseAudio (4713), będzie szyfrowany
i kierowany na port SSH (22) i w ten sposób przesyłany przez sieć.

By skorzystać z przekierowania portów w SSH, musimy odpowiednio skonfigurować połączenie. Możemy to
zrobić bezpośrednio w wierszu poleceń, lub też wykorzystać do tego celu z pliku `~/.ssh/config` .
Jeśli szyfrowanie dźwięku ma dobywać się tylko sporadycznie, to możemy po prostu w terminalu wpisać
to poniższe polecenie:

    $ ssh -R 4713:localhost:4713 morfik@192.168.10.20

W przypadku gdy do serwera łączymy się dość często, to wpisywanie tej powyższej linijki może być
mało praktyczne. Dlatego też lepszym rozwiązaniem jest zdefiniowanie konfiguracji dotyczącej tego
połączenia SSH w pliku `~/.ssh/config` . W tym przypadku przekierowujemy port `4713` na zdalnej
maszynie, zatem konfiguracja połączenia ma wyglądać mniej więcej tak:

    Host 192.168.10.20
          Compression yes
          RemoteForward 127.0.0.1:4713 127.0.0.1:4713

### Zmienna $PULSE_SERVER

Trzeba jeszcze wskazać systemowi gdzie ma szukać serwera PulseAudio. Odpowiada za to, co prawda,
zmienna `$PULSE_SERVER` ale problem w tym, że ruch na zdalnej maszynie trzeba pierw przesłać na jej
lokalny port, z którego przy pomocy SSH zostanie przesłany do serwera dźwięku. Zatem wartość
zmiennej `$PULSE_SERVER` powinna wyglądać mniej więcej tak:

    morfik@viper:~$ export PULSE_SERVER="tcp:localhost:4713"

W taki oto sposób, pakiety będą szły na lokalny port `4713` maszyny zdalnej, po czym będą
forward'owane na port `4713` serwera dźwięku przez tunel SSH. Problem jednak w tym, że takie
zapytania nie zostaną zaakceptowane przez sam serwer PulseAudio.

### Moduł module-native-protocol-tcp

Jako, że dźwięk z aplikacji zdalnych będzie przesyłany do serwera dźwięku przy pomocy SSH, to z
perspektywy tego serwera będą to połączenia lokalne. Musimy zatem jeszcze skonfigurować moduł
`module-native-protocol-tcp` . Konkretnie chodzi o to, by przepuszczał on ruch z adresów
`127.0.0.0/8` , czyli pętli zwrotnej. W tym celu musimy dopisać tę poniższą linijkę do pliku
`/etc/pulse/default.pa` :

    load-module module-native-protocol-tcp auth-ip-acl=127.0.0.0/8

## Test szyfrowanego dźwięku

By mieć pewność, że dźwięk, który przesyłamy przez sieć jest faktycznie szyfrowany, podejrzyjmy
komunikację sieciową, tj. jakie porty są w niej wykorzystywane. Najprościej zrobić to przy pomocy
`iptables` , gdzie możemy zablokować ruch przychodzący i przepuścić pakiety na dwóch portach, tj.
jeden od serwera dźwięku PulseAudio, drugi zaś od SSH. W przypadku gdy ruch jest szyfrowany, tylko
ta druga reguła powinna łapać pakiety, przykładowo:

![](/img/2015/11/1.szyfrowanie-dzwieku-pulseaudio-iptables.png#huge)
