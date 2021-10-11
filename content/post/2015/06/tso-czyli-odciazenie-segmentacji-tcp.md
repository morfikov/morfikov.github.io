---
author: Morfik
categories:
- Linux
date: "2015-06-30T21:15:08Z"
date_gmt: 2015-06-30 19:15:08 +0200
published: true
status: publish
tags:
- tcp
- lxc
- sieć
GHissueID: 87
title: TSO, czyli odciążenie segmentacji TCP
---

Stawiając sobie środowisko testowe pod wireshark'a w celu analizy pakietów sieciowych, zauważyłem,
że coś mi się nie zgadza odnośnie wielkości przesyłanych pakietów między interfejsami kontenerów
LXC. Jakby nie patrzeć, środowisko testowe ma być odwzorowaniem środowiska produkcyjnego i w tym
przypadku wszelkie zasady dotyczące, np. podziału danych na segmenty, muszą być takie same.
Generalnie rzecz biorąc rozmiar pakietu powinien wynosić 1514 bajtów, a był parokrotnie większy.
Okazało się, że jest to za sprawą odciążenia segmentacji w protokole TCP ([TCP Segmentation
Offload](https://lwn.net/Articles/564978/)).

<!--more-->
## Jak działa TSO

Cały ten mechanizm TCP Segmentation Offload (TSO) został zaprojektowany w celu poprawienia
wydajności wychodzących strumieni danych. Bo jakby nie patrzeć, te dane przed transmisją muszą
zostać rozbite na mniejsze segmenty, tak by zmieściły się one w pakiecie. W przypadku interfejsów
sieciowych, które wspierają TSO, dane można przesłać do dużego bufora karty sieciowej i dopiero tam
przeprowadzić segmentację danych. Zmniejsza to ilość operacji jaką musi przeprowadzić główny
procesor komputera czyniąc cały proces bardziej efektywnym.

Poniżej fotka z wireshark'a obrazująca wymianę danych z serwerem www zamkniętym w kontenerze
LXC:

![](/img/2015/06/1.wireshark-odciazenie-segmentacji-tso.png#huge)

Długość kilku pakietów wynosi tam 7306 bajtów. Jest to parokrotnie więcej niż przewidziane 1514
bajtów. Jeśli przyjrzymy się procesowi witania (three-way handshake), to dostrzeżemy, że obie
strony zgodziły się słać pakiety o wartości MSS 1460. Fragmentacja pakietu IP nie występuje, w
przypadku gdyby ktoś był ciekaw:

![](/img/2015/06/2.brak-fragmentacji-tso.png#huge)

Ten pakiet widoczny wyżej może być oczywiście większy ale czemu on ma taki rozmiar? W całą zabawę
zamieszany jest rozmiar bufora (okna) odbiorczego TCP. Im jest on większy, tym można przesłać więcej
danych w jednym pakiecie. Zwykle ten mechanizm jest jak najbardziej użyteczny ale jeśli potrzebujemy
środowiska testowego, który ma mieć 1514 bajtowe pakiety, to musimy nieco dostosować interfejsy
poszczególnych systemów.

## Wyłączenie TSO

Połączenie z maszyną wirtualną odbywa się za sprawą dwóch interfejsów: jeden jest umieszczony
wewnątrz wirtualnego systemu, drugi zaś w systemie hosta. Kontenery LXC automatycznie tworzą taki
tunel i podłączają go do odpowiedniego mostka, tak by wirtualny system miał łączność ze światem za
pomocą naszego komputera wpiętego bezpośrednio do sieci, oczywiście jeśli wyrażamy zgodę na
forwardowanie pakietów.

W moim przypadku interfejs wewnątrz maszyny wirtualnej to `veth10` , jego drugi koniec to zaś
`veth10-sid` . Mostek z kolei nie ma tutaj znaczenia. Kluczowa sprawa to wyłączenie na tych dwóch
interfejsach opcji odpowiadających za odciążanie segmentacji. Na początek sprawdźmy jednak czy w
ogóle ta opcja jest na nich włączona. W tym celu posłużymy się narzędziem `ethtool` :

    # ethtool -k veth10-sid | grep segment
    tcp-segmentation-offload: on
            tx-tcp-segmentation: on
            tx-tcp-ecn-segmentation: on
            tx-tcp6-segmentation: on
    generic-segmentation-offload: on

By wyłączyć odciążanie segmentacji, wpisujemy w terminalu poniższe polecenie:

    # ethtool --offload veth10-sid tso off ufo off gso off gro off

Tę samą czynność przeprowadzamy w wirtualnym systemie na jego interfejsie.

Po tym zabiegu, pakiety powinny mieć rozmiar 1514 bajtów. Problem jednak jest taki, że te wirtualne
interfejsy w kontenerach LXC są tworzone na nowo za każdym razem gdy odpalamy wirtualny system.
Zatem taka konfiguracja nie przetrwa zbyt długo.

Jeśli korzystamy z konfiguracji w pliku `/etc/network/interfaces` , to wystarczy dopisać do
konfiguracji interfejsu na maszynie wirtualnej oraz na hoście poniższą linijkę:

    post-up ethtool -s veth10-sid --offload veth10-sid tso off ufo off gso off gro off

Oczywiście odpowiednio zmieniając nazwę interfejsu sieciowego.
