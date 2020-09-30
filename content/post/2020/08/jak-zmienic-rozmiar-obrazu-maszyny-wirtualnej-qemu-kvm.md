---
author: Morfik
categories:
- Linux
date: "2020-08-09T13:45:00Z"
published: true
status: publish
tags:
- debian
- wirtualizacja
- kvm
- qemu
- libvirt
title: Jak zmienić rozmiar obrazu maszyny wirtualnej QEMU/KVM
---

Bawiąc się ostatnio [maszynami wirtualnymi na bazie QEMU/KVM][1], zauważyłem, że sugerowany rozmiar
pliku `.qcow2` waha się w okolicach 25 GiB. Nie jest to może jakoś specjalnie dużo ale mało to też
nie jest, zwłaszcza jeśli tworzymy maszyny wirtualne na dysku swojego laptopa. Co jeśli
przeholowaliśmy z szacunkami co do rozmiaru takiego obrazu i po zainstalowaniu systemu operacyjnego
gościa okazało się, że w sumie to ten obraz można by zmniejszyć o połowę? Albo też i w drugą stronę,
tj. co w przypadku, gdy stworzony obraz maszyny wirtualnej okazał się zbyt mały i teraz zachodzi
potrzeba jego powiększenia? Czy w takiej sytuacji musimy na nowo tworzyć maszynę wirtualną
odpowiednio zwiększając lub zmniejszając jej przestrzeń na pliki? A może istnieje jakiś sposób na
zmianę rozmiaru tych istniejących już obrazów maszyn wirtualnych? Postaramy się ten fakt
zweryfikować, a cały proces zostanie opisany przy wykorzystaniu systemu Debian Linux.

<!--more-->
## Jak zmniejszyć rozmiar obrazu maszyny wirtualnej

Okazuje się, że możemy nieco zmniejszyć rozmiar obrazu maszyny wirtualnej ale potrzebne są do tego
odpowiednie narzędzia, no i oczywiście nie da się tego zrobić automatycznie. Przede wszystkim,
trzeba będzie uruchomić na maszynie wirtualnej system live, tak by móc przepartycjonować jej
wirtualny dysk twardy. Zatem do dzieła. Odpalamy `virt-manager` i ustawiamy w nim rozruch systemu z
cd-rom (zmieniając kolejność nośników), czy też zaznaczając opcję `Enable boot menu` :

![](/img/2020/08/052-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Podczas rozruchu wciskamy klawisz `ESC` , co powinno zaowocować pojawieniem się menu wyboru nośnika,
z którego wystartować system:

![](/img/2020/08/053-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Wybieramy cd-rom (pozycja druga).

Po chwili powinien nam się załadować system live Ubuntu:

![](/img/2020/08/054-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

W tym systemie domyślnie jest już zainstalowany `gparted` ale gdybyśmy korzystali z innego systemu
live, to trzeba będzie ten pakiet doinstalować sobie we własnym zakresie.

Logujemy się na root via `sudo su` i uruchamiamy `gparted` :

![](/img/2020/08/055-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Jak widać na fotce, mamy dysk o rozmiarze 20 GiB, z czego system zajmuje niecałe 8 GiB. Przydałoby
się zmniejszyć rozmiar pliku `.qcow2` o połowę. W tym celu zmieniamy rozmiar partycji w `gparted` i
ustawiamy go na 10 GiB:

![](/img/2020/08/056-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

![](/img/2020/08/057-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Czekamy cierpliwie aż proces dobiegnie końca. Może on zająć dłużej lub której, a wszystko zależy od
wielkości samego nośnika oraz od tego jak bardzo chcemy go skurczyć, tj. ile danych trzeba będzie
przenieść.

![](/img/2020/08/058-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Gdy proces zmiany rozmiaru partycji się zakończy, powinniśmy mieć jedną mniejszą partycję oraz
sporo wolnego miejsca:

![](/img/2020/08/059-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Warto tutaj odnotować ostatni sektor partycji ( `20973567` ), który to posłuży nam do wyliczenia
nowego obrazu dysku (pliku `.qcow2` ): ((20973567+1)×512)/1024/1024 = 10241 MiB.

Po skurczeniu partycji wyłączamy maszynę wirtualną. Na maszynie hosta zaś odpalamy terminal i
patrzymy jaki rozmiar ma obecnie obraz maszyny wirtualnej:

    # ls -alh ubuntu20.04.qcow2
    -rw------- 1 root root 21G 2020-08-04 19:08:36 ubuntu20.04.qcow2

Póki co, jeszcze ma te 21 GiB. Teraz przy pomocy narzędzia `qemu-img` (dostępnego w pakiecie
`qemu-utils` ) zmniejszamy rozmiar obrazu maszyny wirtualnej w poniższy sposób:

    # qemu-img resize --shrink ubuntu20.04.qcow2 10241M
    Image resized.

Gdybyśmy przy wydawaniu tego polecenia dostali taki komunikat:

    # qemu-img resize --shrink ubuntu20.04.qcow2 10241M
    qemu-img: Could not open 'ubuntu20.04.qcow2': Failed to get "write" lock
    Is another process using the image [ubuntu20.04.qcow2]?

oznacza to, że maszyna wirtualna wciąż działa i trzeba ją pierw zatrzymać, by móc wejść w
interakcję z jej obrazem.

Sprawdźmy ile obecnie zajmuje wirtualny dysk oraz sam plik `.qcow2` :

    # qemu-img info ubuntu20.04.qcow2

    image: ubuntu20.04.qcow2
    file format: qcow2
    virtual size: 10 GiB (10738466816 bytes)
    disk size: 10 GiB
    cluster_size: 65536
    Format specific information:
        compat: 1.1
        lazy refcounts: true
        refcount bits: 16
        corrupt: false

Wartość `virtual size` wskazuje `10738466816` bajtów, czyli 10241 MiB. Jest to dokładnie tyle ile
ustawiliśmy w `gparted` przy zmniejszaniu rozmiaru partycji systemowej. Natomiast rozmiar samego
pliku `.qcow2` różni się o niecałe 2 MiB w stosunku do wartości `virtual size` :

    # ls -al ubuntu20.04.qcow2
    -rw------- 1 root root 10740432896 2020-08-04 20:26:46 ubuntu20.04.qcow2

Nie udało mi się ustalić z czego wynika ta różnica ale obstawiam nagłówek pliku `.qcow2` , który
`qemu-img` bierze pod uwagę i automatycznie dodaje jego rozmiar w kalkulacji wynikowego rozmiaru
obrazu maszyny wirtualnej. Jeśli jednak boimy się, że za bardzo okroimy obraz, czego efektem będzie
uszkodzenie systemu plików maszyny wirtualnej i utrata danych, to dobrze jest zostawić te kilka
dodatkowych MiB. W razie czego systemową partycję maszyny wirtualnej będzie można powiększyć bez
problemu, tak by włączyć w nią te parę MiB wolnej przestrzeni.

Odpalmy teraz maszynę wirtualną, by sprawdzić czy jej system działa w porządku:

![](/img/2020/08/061-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Jak widać, nie mamy już żadnej wolnej przestrzeni, a systemowa partycja kończy się dokładnie na
20973567 sektorze:

![](/img/2020/08/062-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Nie ma przy tym żadnych błędów, zatem proces zmniejszenia rozmiaru dysku maszyny wirtualnej
zakończył się powodzeniem.

## Jak zwiększyć rozmiar obrazu maszyny wirtualnej

W przypadku, gdy nasze oczekiwania w stosunku do przechowywanych danych wewnątrz maszyny wirtualnej
wychodzą poza ramy rozmiaru wirtualnego dysku systemu gościa, to w takiej sytuacji przydałoby się
zwiększyć nieco przestrzeń dyskową oddelegowaną tej maszynie wirtualnej. Podobnie jak w przypadku
zmniejszania rozmiaru obrazu maszyny wirtualnej, tutaj również będziemy wykorzystywać narzędzie
`qemu-img` .

Załóżmy zatem, że potrzebujemy dodatkowe 15 GiB przestrzeni na pliki. By dodać te 15 GiB do pliku
`.qcow2` , korzystamy z poniższego polecenia:

    # qemu-img resize ubuntu20.04-small-comp.qcow2 +15G
    Image resized.

Operacja jest w zasadzie natychmiastowa, choć my nie dostrzeżemy praktycznie żadnej różnicy w
rozmiarze samego pliku `.qcow2` . Niemniej jednak, po uruchomieniu systemu maszyny wirtualnej i
podejrzeniu dysku w `gparted` , ta różnica będzie już dość dobrze widoczna:

![](/img/2020/08/081-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Jak widać, te 15 GiB, które dodaliśmy do obrazu, zostały przez maszynę wirtualną uwzględnione, choć
oczywiście póki co partycja systemowa rozciąga się na 10 GiB. By zrobić użytek z tych dodatkowych
GiB, trzeba rozciągnąć tę systemową partycję, tak by obejmowała również i tę wolną przestrzeń.

Do powiększania partycji systemowej obrazu maszyny wirtualnej nie musimy korzystać z systemu live,
tak jak to miało miejsce w przypadku zmniejszania jej rozmiaru. Cały proces można wykonać będąc
zalogowanym w systemie gościa. Zatem rozciągamy partycję główną, tak by zajmowała całą dostępną
wolną przestrzeń:

![](/img/2020/08/082-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Po czym rozpoczynamy proces zwiększenia rozmiaru partycji:

![](/img/2020/08/083-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Jak widać, ten zabieg trwał dosłownie parę sekund.

System też od razu jest w stanie dostrzec nowych rozmiarów partycję systemową bez potrzeby
ponownego uruchomienia maszyny wirtualnej:

![](/img/2020/08/084-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Jeśli teraz rzucimy okiem na rozmiar obrazu maszyny wirtualnej w systemie hosta, to niekoniecznie
musi on zajmować 25 GiB:

    # qemu-img info ubuntu20.04.qcow2
    image: ubuntu20.04.qcow2
    file format: qcow2
    virtual size: 25 GiB (26844594176 bytes)
    disk size: 10 GiB
    cluster_size: 65536
    Format specific information:
        compat: 1.1
        lazy refcounts: true
        refcount bits: 16
        corrupt: false

W tym przypadku, rozmiar pliku `.qcow2` uległ zwiększeniu jedynie o parę MiB ale to nie jest
ostateczny rozmiar tego pliku, bo zależy on od ilości danych, które będziemy pakować do maszyny
wirtualnej. Im więcej danych znajdzie się w wirtualnym dysku maszyny wirtualnej, tym bardziej jego
rozmiar się rozrośnie, oczywiście w granicach ustawionego limitu. Widoczny wyżej `virtual size:` to
wirtualny rozmiar dysku, zaś `disk size:` pokazuje ile ten ten wirtualny dysk zajmuje miejsca na
maszynie hosta w rzeczywistości.

## Odzyskiwanie wolnego miejsca z pliku .qcow2

Jak widzieliśmy wyżej przy zmniejszaniu obrazu maszyny wirtualnej, plik `.qcow2` na maszynie hosta
zajmował ~10 GiB, mimo, że pliki systemu gościa w rzeczywistości zajmowały mniej miejsca. Możemy tę
wolną przestrzeń również odzyskać. Do tego celu posłużymy się poniższym poleceniem:

    # qemu-img convert -O qcow2 ubuntu20.04.qcow2 ubuntu20.04-small.qcow2

Po chwili powinien zostać utworzony drugi, nieco mniejszy obraz:

    # ls -alh ubuntu20.04*
    -rw-r--r-- 1 root root 8.8G 2020-08-04 21:37:06 ubuntu20.04-small.qcow2
    -rw------- 1 root root  11G 2020-08-04 20:38:47 ubuntu20.04.qcow2

Trzeba jednak liczyć się z faktem, że odzyskane w ten sposób GiB będą tracone ilekroć tylko
będziemy coś zapisywać w systemie maszyny wirtualnej. Zatem prędzej czy później ten obraz urośnie
nam do swojego rzeczywistego rozmiaru. Warto jednak pamiętać, że w taki sposób można odchudzić
obraz maszyny wirtualnej, np. po skasowaniu sporej ilości danych z systemu gościa, tak by nie
marnował nam on miejsca w systemie hosta.

## Kompresja pliku .qcow2

Jeśli maszyna hosta dysponuje mocniejszym procesorem albo zwyczajnie zależy nam na ograniczeniu
przestrzeni dyskowej, która przeznaczona jest pod obrazy maszyn wirtualnych, to możemy jeszcze
bardziej zmniejszyć rozmiar tych obrazów stosując kompresję (przełącznik `-c` w `qemu-img` ),
przykładowo:

    # qemu-img convert -O qcow2 -c ubuntu20.04.qcow2 ubuntu20.04-small-comp.qcow2

No i tutaj przy zastosowaniu kompresji udało się zmniejszyć rozmiar obrazu maszyny wirtualnej na
dysku hosta o ponad 50%:

    # ls -alh ubuntu20.04-small*
    -rw-r--r-- 1 root root 4.2G 2020-08-04 21:55:38 ubuntu20.04-small-comp.qcow2
    -rw-r--r-- 1 root root 8.8G 2020-08-04 21:37:06 ubuntu20.04-small.qcow2

Tak utworzony obraz maszyny wirtualnej trzeba teraz podmienić w jej konfiguracji. Można oczywiście
zmienić nazwę pliku ale lepiej jest się pierw upewnić, czy aby na pewno ten okrojony obraz działa
zgodnie z oczekiwaniami. Nie damy rady zmienić obrazu via `virt-manager` i trzeba w tym celu
edytować plik XML maszyny wirtualnej (via `virsh edit ubuntu20.04` ):

![](/img/2020/08/063-virtualization-kvm-qemu-system-change-image-qcow2-size.png#huge)

Może i udało nam się dość znacznie zaoszczędzić miejsce na dysku stosując kompresję ale trzeba
liczyć się z większym wykorzystaniem procesora -- nie ma nic za free.

## Defragmentacja pliku obrazu

Po tych zabiegach z utworzeniem obrazu maszyny wirtualnej oraz zmianą jego rozmiaru, dobrze jest
też zdefragmentować sam plik `.qcow2` w przypadku, gdy w maszynie hosta mamy zainstalowany dysk HDD
(magnetyczny):

    # filefrag -ve ubuntu20.04-small-comp.qcow2
    # e4defrag -v ubuntu20.04-small-comp.qcow2

Pierwsze z powyższych poleceń sprawdzi stopień fragmentacji pliku i jeśli będzie on duży, to wtedy
korzystamy z drugiego polecenia. By dodatkowo zmniejszyć stopień fragmentacji obrazu maszyny
wirtualnej, dobrze jest to drugie polecenie wydać kilka razy.


[1]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
