---
author: Morfik
categories:
- Linux
date:    2020-10-18 10:45:00 +0200
lastmod: 2020-10-18 10:45:00 +0200
published: true
status: publish
tags:
- debian
- deduplikacja
- wirtualizacja
- kvm
- qemu
- libvirt
- kernel
GHissueID: 15
title: Zastosowanie KSM w maszynach wirtualnych QEMU/KVM
---

Użytkownicy linux'a, którzy korzystają z [mechanizmu wirtualizacji QEMU/KVM][14], wiedzą, że takie
maszyny wirtualne potrafią zjadać dość sporo pamięci operacyjnej. Im więcej takich maszyn zostanie
uruchomionych w obrębie danego hosta, tym większe ryzyko, że nam tego RAM'u zwyczajnie zabraknie.
Można oczywiście ratować się dokupieniem dodatkowych modułów pamięci ale też nie zawsze taki zabieg
będzie możliwy, zwłaszcza w przypadku domowych stacji roboczych pokroju desktop/laptop. Szukając
rozwiązania tego problemu natrafiłem na coś, co nazywa się Kernel Samepage Merging. W skrócie, KSM
to mechanizm, który ma na celu współdzielenie takich samych stron pamięci operacyjnej przez kilka
procesów. W ten sposób można (przynajmniej teoretycznie) dość znacznie obniżyć zużycie RAM,
zwłaszcza w przypadku korzystania na maszynach wirtualnych z tych samych systemów operacyjnych.
Przydałoby się zatem ocenić jak bardzo KSM wpłynie na wykorzystanie pamięci i czy będzie z niego
jakiś większy użytek zarówno przy korzystaniu z maszyn wirtualnych, czy też w codziennym użytkowaniu
komputera.

<!--more-->
## Jak działa KSM

[Kernel Samepage Merging][1] (KSM) to mechanizm kernela linux, mający na celu zoptymalizować
wykorzystanie pamięci RAM przez hiperwizor (w tym przypadku QEMU/KVM). Działa on na zasadzie
współdzielenia stron pamięci przez wiele maszyn wirtualnych działających w obrębie jednego hosta.
Zwykle współdzieleniu podlegać będą biblioteki oraz inne często używane dane, których kontent
pozostaje w miarę stały w czasie. Zatem KSM pozwala na uruchomienie większej liczby systemów
gościa opartych o ten sam system operacyjny. W takiej sytuacji nie trzeba duplikować całej masy
informacji, przez co mniej pamięci RAM jest potrzebne do obsługi maszyn wirtualnych.

Pierwotnie mechanizm KSM był stworzony głównie na potrzeby maszyn wirtualnych, by można było ich
uruchomić więcej niż pozwalały na to ograniczenia związane z zainstalowaną pamięcią fizyczną. W
obecnych czasach, KSM znajduje zastosowanie również poza zwirtualizowanym środowiskiem, tj.
wszędzie tam gdzie zużycie pamięci RAM ma dla nas ogromne znaczenie (nawet w systemach wbudowanych).

[Koncept współdzielenia pamięci nie jest niczym nowym][8] w obecnych systemach operacyjnych. Dla
przykładu, gdy uruchamiamy jakąś aplikację po raz pierwszy, współdzieli ona całą swoją pamięć z
procesem swojego rodzica (parent proccess). Gdy któryś z tych dwóch procesów (rodzic albo dziecko)
próbuje zmodyfikować tę pamięć, kernel alokuje nowy region pamięci, kopiuje tam oryginalny kontent
i zezwala programowi na modyfikację tego nowego obszaru pamięci. Tego typu operacja jest znana pod
nazwą [Copy on Write][5] (CoW), czyli kopiowanie przy zapisie.

Mechanizm KSM w linux'ie korzysta dokładnie z tego samego rozwiązania tyle, że w odwrotnej
kolejności. KSM umożliwia kernelowi zbadanie dwóch (lub więcej) uruchomionych programów i porównanie
ich pamięci. Jeśli którekolwiek strony pamięci operacyjnej są identyczne, to KSM redukuje je do
jednej strony. Ta strona jest następnie oznaczana jako CoW. Jeśli w późniejszym czasie dany proces
maszyny wirtualnej będzie chciał zmodyfikowana którąś z współdzielonych stron pamięci, to dla tego
gościa zostanie utworzona nowa strona pamięci, w której te zmiany zostaną odzwierciedlone. Pozostałe
maszyny wirtualne będą współdzielić w dalszym ciągu tę starą stronę pamięci.

Kiedy maszyna wirtualna jest uruchamiana, dziedziczy ona tylko pamięć z procesu QEMU/KVM. Po
uruchomieniu systemu gościa, zawartość obrazu systemu operacyjnego gościa może być współdzielona
przy założeniu, że inne systemy gościa korzystają dokładnie z tego samego systemu operacyjnego lub
aplikacji. KSM umożliwia KVM zażądanie współdzielenia tych identycznych regionów pamięci gościa,
przez co poprawia szybkość i wykorzystanie pamięci operacyjnej. W KSM wspólne dane procesu są
przechowywane w pamięci podręcznej lub w pamięci głównej. Zmniejsza to liczbę błędów pamięci
podręcznej (cache miss) dla systemów gościa, co może poprawić wydajność niektórych aplikacji i
systemów operacyjnych.

### Większe obciążenie procesora

Może i w pewnych sytuacjach będziemy w stanie "zwiększyć" nawet parokrotnie ilość dostępnej pamięci
operacyjnej w systemie ale taki zabieg ma swoje konsekwencje. Obsługa mechanizmu KSM, dzięki
któremu możemy uruchomić więcej maszyn wirtualnych, jest bardzo kosztowa jeśli chodzi o
wykorzystanie procesora. Generalnie to im intensywniejsze będzie skanowanie stron pamięci w
poszukiwaniu kandydatów do współdzielenia, to tym wyższy stopień deduplikacji uda nam się uzyskać
ale też i procesor będzie wykorzystywany w większym stopniu. Dlatego KSM nie zawsze znajdzie
zastosowanie i czasem lepszym wyjściem jest po prostu dokupienie większej ilości pamięci RAM.

### Ryzyko wyczerpania się pamięci

Trzeba sobie także zdawać sprawię, że takie współdzielone strony pamięci mogą powodować zagrożenie
związane z wyczerpaniem się pamięci operacyjnej. Załóżmy, że mamy dwa procesy i jeden z nich będzie
chciał zmienić współdzieloną stronę pamięci. W takim przypadku system będzie musiał tę stronę
pamięci skopiować. W efekcie, w pamięci RAM będą figurować dwie osobne strony (po jednej dla każdego
z tych dwóch procesów), czyli dokładnie taka sama sytuacja, co w przypadku braku korzystania z KSM.
Jeśli teraz uruchomimy wiele maszyn wirtualnych i zaistnieje jakieś zdarzenie, które sprawi, że
spora ilość stron współdzielonych przestanie być współdzielona, to może się okazać, że nagle
zabraknie nam pamięci operacyjnej, by te nowe strony pomieścić i system albo się powiesi, albo
pozabija część maszyn wirtualnych.

### Problemy z bezpieczeństwem KSM

W przeszłości, mechanizm KSM [miewał poważne problemy z bezpieczeństwem][8], gdzie kolizje hash'ów
mogły doprowadzić do wstrzyknięcia kodu w procesy należące do innych użytkowników, czy też maszyn
wirtualnych. Później hash'e zostały zastąpione [drzewami czerwono-czarnymi][9], przez co udało się
zaadresować ten problem z kolizją hash'ów.

Wygląda jednak na to, że nie wszystkie obawy związane z bezpieczeństwem stosowania KSM zostały
rozwiane, bo w systemach typu hardened, ten mechanizm jest wyłączony. Dla przykładu w [CLIP OS][10],
powód jaki został podany za tym, by KSM wyłączyć, to możliwość ułatwienia [ataków typu side
channel][11], takich jak [FLUSH + RELOAD][12], które są wymierzone w pamięci cache L3 głównego
procesora komputera. Generalnie chodzi o problem współdzielenia stron pamięci operacyjnej przez
niezaufane procesy, a tych w maszynach wirtualnych nie brakuje. Wykorzystując mechanizm KSM
jesteśmy zatem narażeni na wyciek informacji i kompromitację bezpieczeństwa systemu.

## Jak włączyć/wyłączyć KSM na Debianie

Teoretycznie KSM wygląda na bardzo ciekawą opcję, która jest nam w stanie pozwolić dość znacznie
zoptymalizować wykorzystanie pamięci w naszym linux'e. Jak można przeczytać na wiki, [na hoście z
16G RAM udało się odpalić 52 maszyny wirtualne][2] z przydziałem 1G wirtualnej pamięci operacyjnej.
Zatem z 16G udało się zrobić 52G RAM. Przydałoby się sprawdzić jak ten mechanizm radzi sobie w
realnych warunkach i czy faktycznie jest taki dobry jak to ludzie piszą na necie.

KSM jest zwykle obecny w dystrybucyjnych kernelach. Jeśli jednak trafi nam się takie jądro, które
nie wspiera KSM, to trzeba będzie w nim włączyć opcję `CONFIG_KSM` . Do obsługi tego mechanizmu
oddelegowane zostały pliki w katalogu `/sys/kernel/mm/ksm/` i to przy ich pomocy możemy włączyć i
wyłączyć KSM oraz kontrolować jego zachowanie podczas pracy systemu.

By włączyć KSM, wystarczy przesłać `1` do pliku `/sys/kernel/mm/ksm/run` :

    # echo "1" > /sys/kernel/mm/ksm/run

KSM posiada predefiniowaną konfigurację i w zasadzie w sporej części przypadków, te domyślne
ustawienia powinny nam wystarczyć. Rozruch KSM zajmie zwykle parę minut, także nie stresujmy się z
początku, jeśli statystyki pamięci nie zmienią się w żaden sposób. Po prostu trzeba trochę poczekać,
aż system dojdzie do wniosku, że pewne strony pamięci można skonsolidować.

Gdy w późniejszym czasie zajdzie potrzeba wyłączenia KSM, to można ten zabieg przeprowadzić na dwa
sposoby. Pierwszym jest zwykłe zatrzymanie demona `ksmd` przez przesłanie `0` do pliku
`/sys/kernel/mm/ksm/run` . W takim przypadku, aktualnie  współdzielone strony pamięci w dalszym
ciągu będą mogły być współdzielone między procesami, przynajmniej do momentu aż któryś z tych
procesów nie zmieni tych stron. Jeśli jednak chcielibyśmy przy wyłączaniu demona `ksmd` usunąć przy
okazji wszystkie współdzielone strony pamięci, to trzeba przesłać wartość `2` :

    # echo "2" > /sys/kernel/mm/ksm/run

Włączanie KSM powyższym sposobem, czy też ogólnie operowanie na tym mechanizmie ma jedną wadę -- po
restarcie maszyny trzeba będzie tę konfigurację zaaplikować jeszcze raz. Oczywiście by sobie ulżyć
z tym zadaniem, możemy zamieścić stosowne wpisy w pliku `/etc/sysfs.conf` , przykładowo:

    kernel/mm/ksm/run = 1

### Usługi ksm.service oraz ksmtuned.service

W Debianie dostępny jest pakiet `ksmtuned` , który dostarcza usługi `ksm.service` oraz
`ksmtuned.service` . Zadaniem usługi `ksm.service` jest jedynie uruchomienie mechanizmu KSM przez
przesłanie `1` do pliku `/sys/kernel/mm/ksm/run` . Opcjonalnie też można przez plik
`/etc/default/ksm` ustawić maksymalną ilość stron pamięci, których kernel nie będzie w stanie
przenieść do SWAP i to te właśnie strony będą do wykorzystania przez KSM. Wygląda jednak na to, że
ten ficzer jest jakimś reliktem przeszłości, bo już od dość dawna [KSM może operować na stronach
pamięci, które zostaną przeniesione do SWAP][4], choć współdzielenie tych stron zostanie usunięte
po wydobyciu ich ze SWAP. Oczywiście w dalszym ciągu te strony będą mogły być współdzielone ale
trzeba będzie chwilę poczekać aż demon `ksmd` na nowo je oznaczy. Jeśli zaś chodzi o usługę
`ksmtuned.service` , to ma ona za zadanie uruchamiać i zatrzymywać usługę `ksm.service` w zależności
od tego czy współdzielenie pamięci jest potrzebne. Ta usługa jest także w stanie dostosować
konfigurację KSM. Demon `ksmtuned` jest także powiadamiany przez libvirt o tym czy maszyna wirtualna
została uruchomiona lub zatrzymana.

Próbowałem te dwie usługi skonfigurować według informacji zawartych na [stronie RedHat'a][6] ale nie
przyniosło to żadnego efektu. Wygląda na to, że te usługi nie działają poprawnie (albo też nie
potrafię ich poprawnie skonfigurować). Zamiana jakiejkolwiek wartości w pliku konfiguracyjnym
`/etc/ksmtuned.conf` nie przynosi żadnego efektu w kwestii dostosowania zachowania mechanizmu KSM.
Brak rzeczowej dokumentacji sprawia, że nie chce mi się zbytnio zastanawiać co tutaj może być nie
tak. Dlatego też lepiej odpuścić sobie te narzędzia i konfigurować KSM bezpośrednio przez katalog
`/sys/kernel/mm/ksm/` (via plik `/etc/sysfs.conf` ) lub też przy pomocy własnych skryptów
shell'owych.

## Mechanizm KSM w akcji

Zobaczmy zatem jak mechanizm KSM sprawuje się na naszym linux'ie. Żeby mieć jakieś porównanie,
dobrze jest uruchomić parę maszyn wirtualnych przed włączeniem KSM, tj. przed przesłaniem `1` do
pliku `/sys/kernel/mm/ksm/run` . W tym przypadku zostały uruchomione dwie maszyny wirtualne z Ubuntu
na pokładzie. Jedna maszyna jest klonem drugiej. Powinniśmy mieć zatem dokładnie ten sam system na
obu maszynach. Statystyki utylizacji pamięci RAM prezentują się mniej więcej tak:

    # ps_mem
     Private  +   Shared  =  RAM used       Program

      3.9 GiB +  11.0 MiB =   3.9 GiB       qemu-system-x86_64 (2)

Mamy zatem 3.9 GiB zajętego RAM'u przez procesy maszyn wirtualnych. Warto zwrócić uwagę, że ilość
danych w kolumnie `Shared` jest niewielka. Co ciekawe pewne dane będą współdzielone zawsze bez
względu na to, czy korzystamy z KSM czy też nie. Dzieje się tak dlatego, że współdzieleniu zawsze
podlega [segment .text programu][13].

Po przesłaniu `1` do pliku `/sys/kernel/mm/ksm/run` , otrzymujemy coś takiego:

    # ps_mem
     Private  +   Shared  =  RAM used       Program

      3.7 GiB +  56.4 MiB =   3.7 GiB       qemu-system-x86_64 (2)

No nie widać jakoś specjalnie różnicy. Trochę więcej danych jest w kolumnie `Shared` . Jeśli jednak
poczekamy kilka minut, to będziemy mieć poniższy wynik:

    # ps_mem
     Private  +   Shared  =  RAM used       Program

      3.2 GiB + 155.6 MiB =   3.4 GiB       qemu-system-x86_64 (2)

Jak widać, wartość w kolumnie `Shared` urosła przy jednoczesnym obniżeniu się pozostałych wartości.
Gdybyśmy jeszcze trochę poczekali, to uzyskamy znacznie lepszy wynik:

    # ps_mem
     Private  +   Shared  =  RAM used       Program

      2.6 GiB + 479.6 MiB =   3.0 GiB       qemu-system-x86_64 (2)

Wartość w kolumnie `Shared` dość mocno wzrosła, przez co każda z tych dwóch maszyn wirtualnych zjada
trochę mniej pamięci operacyjnej, bo przecież część danych jest współdzielona. Im dłużej te maszyny
wirtualne pochodzą, tym więcej danych będzie współdzielonych, bo system ostatecznie przyjmie, że
pewne dane w przewidywalnej przyszłości raczej nie zostaną w żaden sposób zmienione, a że dodatkowo
te dane są takie same, to można je współdzielić.

Czasami `ps_mem` może niedokładnie prezentować wartości zużycia RAM'u. Zawsze możemy podejrzeć pliki
w katalogu `/sys/kernel/mm/ksm/` , a konkretnie `pages_sharing` oraz `pages_shared`, by zobaczyć ile
udało nam się tej pamięci zaoszczędzić. Parametr `pages_shared` odpowiada za ilość stron pamięci,
które podlegają współdzieleniu, a parametr `pages_sharing` mówi ile pamięci RAM udało się faktycznie
odzyskać. W tym przypadku mamy wartości `137077` i `278260` . Obie te wartości są w stronach
pamięci, a każda strona to 4096 bajtów. Zatem mamy odpowiednio (137077×4096)/1024/1024=535.46 MiB
oraz (278260×4096)/1024/1024=1086.95 MiB, czyli udało się zaoszczędzić około 1,1 GiB RAM.

Prawdę mówiąc oczekiwałem trochę lepszego wyniku. Te dwie maszyny wirtualne mają przydział po 2 GiB
RAM. Bez KSM utylizacja pamięci operacyjnej przez tych gości powinna być na poziomie 4 GiB. Z KSM
mamy około 3 GiB, czyli zysk około 25%. W tym przypadku mamy do czynienia ze sklonowanymi systemami,
na których w zasadzie nic nie było robione. Jest to bardzo rzadko spotykana sytuacja i w realnych
warunkach raczej trzeba by się spodziewać, że te maszyny wirtualne będą się nieco różnić, np. przez
inny stopień aktualizacji czy uruchomione aplikacje. Zatem faktyczna oszczędność pamięci będzie
jeszcze niższa.

Dlaczego zatem w przypadku windows'ów można zaoszczędzić tak dużo pamięci, a na linux tak niewiele?
Wygląda na to, że windows [zeruje nieużywaną pamięć][7], przez co ta niewykorzystana pamięć maszyny
wirtualnej zawiera same zera. Strony pamięci zawierające same zera są takie same, więc można je
współdzielić i to dlatego mamy tak ogromny boost w przypadku systemów gości mających uruchomionego
windows'a.

Jeśli zaś chodzi o zużycie procesora przez proces `ksmd` , to mamy tutaj do czynienia z dodatkowym
narzutem na poziomie około 5% (wahania w zakresie 3% - 7%). Nie wydaje się to być dużą wartością,
no ale mało to też nie jest, zwłaszcza gdy pod uwagę weźmiemy odzyskanie jedynie 1 GiB RAM. Trzeba
mieć na uwadze też fakt, że im bardziej będziemy wykorzystywać maszyny wirtualne, tj. zmieniać ich
dane w RAM podczas ich pracy (uruchamiać/zamykać aplikacje), tym mniej danych będzie
współdzielonych, a zużycie procesora na potrzeby `ksmd` będzie mniej więcej stałe. Może się zdarzyć
zatem tak, że procesor będzie utylizowany, a my przy tym nie zauważymy praktycznie różnicy w
wykorzystywaniu pamięci RAM lub też ta różnica będzie niewielka.

### Tuning parametrów pages_to_scan i sleep_millisecs

Domyślna konfiguracja demona `ksmd` działa raczej w porządku. Możemy jednak nieco podbić lub obniżyć
wartości parametrów przechowywanych w `/sys/kernel/mm/ksm/` , tak by dobrać konfigurację KSM pod
nasz system indywidualnie. Możemy zwiększyć ilość stron pamięci ( `pages_to_scan` ) z domyślnych 100
do 1000, które `ksmd` będzie skanował w jednym podejściu. Dodatkowo możemy zmniejszyć interwał
między kolejnymi skanami ( `sleep_millisecs` ) z domyślnych 20 ms na 10 ms. W ten sposób więcej
stron będzie skanowanych i ten proces będzie przebiegał częściej. Ostatecznie udało mi się uzyskać
ten poniższy wynik:

    # ps_mem
     Private  +   Shared  =  RAM used       Program

      2.2 GiB + 765.5 MiB =   2.9 GiB       qemu-system-x86_64 (2)

Jak widać, wartość w kolumnie `Shared` skoczyła w górę jeszcze bardziej. Podobnie z wykorzystaniem
procesora, tj. z ~5% mamy wzrost na około 15%. Zatem bardziej intensywne skanowanie jest w stanie
nam zwrócić trochę więcej pamięci ale dość sporym kosztem procesora. Warto też zaznaczyć tutaj, że
system będzie nam tę pamięć szybciej odzyskiwał niż w przypadku wolniejszych skanów.

### Efektywność mechanizmu KSM

Efektywność KSM możemy sprawdzić korelując dane parametrów `pages_shared` , `pages_sharing` ,
`pages_unshared` oraz `pages_volatile` . Poniżej są wartości, które udało mi się wyłapać:

                   jedna maszyna | dwie maszyny | dwie maszyny po 20 minutach
    pages_shared        7284           12434                168067
    pages_sharing      28539           90630                299539
    pages_unshared    100180          173383                482783
    pages_volatile    312701          600636                 23884

Wysokie ratio `pages_sharing`/`pages_shared` oznacza, że KSM działa bardzo dobrze, bo wiele stron
jest współdzielonych między procesami. W tym przypadku mamy odpowiednio 3.92/7.29/1.72. Zatem widać,
że po uruchomieniu drugiej maszyny wirtualnej sporo tych stron jest współdzielonych ale po chwili
ten współczynnik dość mocno spada. Wysokie ratio `pages_unshared`/`pages_sharing` oznacza
marnotrawstwo zasobów i tutaj mamy 3.51/1,91/1.61 , czyli im dłużej te maszyny działają, tym lepiej.
Wartości w `pages_volatile` oznaczają strony, które się [zmieniają zbyt szybko][3], by system je
rozważał jako nadające się do współdzielenia. Jak widać, po inicjacji maszyn wirtualnych, ta wartość
spadła dość znacznie.

## Podsumowanie

Jakby nie patrzeć trochę pamięci RAM można odzyskać stosując mechanizm KSM na linux. W moim
systemie, jedyną aplikacją, która jest w stanie robić użytek z KSM, to QEMU/KVM, zatem w sporej
części przypadków ten mechanizm będzie miał zastosowanie jedynie przy wirtualizacji lub też będzie
kompletnie nieużywany. Na szczęście jeśli nie korzystamy z KSM (nawet jeśli włączony jest on w
kernelu), to system nie zużywa praktycznie żadnych dodatkowych zasobów. Gdy jednak już z KSM
zaczniemy korzystać, to sporo mocy procesora trzeba będzie przeznaczyć na obsługę tego mechanizmu.
Im więcej RAM'u będziemy chcieli odzyskać, tym więcej mocy procesora trzeba będzie poświęcić. Odzysk
pamięci RAM w postaci 25% może dla jednych być sporą wartością ale dla mnie nie jest on wart utraty
cykli procesora rzędu 10-15%. Do tego dochodzą jeszcze problemy z bezpieczeństwem, przez co ogólny
całokształt nie renderuje się zbyt pozytywnie. Na moich systemach, gdzie bezpieczeństwo jest
priorytetem, KSM nie znajdzie zastosowania. Lepiej jest dokupić ten 1 GiB RAM na każde 4 GiB
zainstalowanej pamięci fizycznej, tak by nie katować procesora niepotrzebnie i nie martwić się o
sprawy związane z bezpieczeństwem systemu i danych w nim przechowywanych.


[1]: https://www.kernel.org/doc/Documentation/vm/ksm.txt
[2]: https://en.wikipedia.org/wiki/Kernel_same-page_merging
[3]: https://developer.ibm.com/tutorials/l-kernel-shared-memory/
[4]: https://bugzilla.redhat.com/show_bug.cgi?id=558281
[5]: https://en.wikipedia.org/wiki/Copy-on-write
[6]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_tuning_and_optimization_guide/chap-ksm
[7]: https://lwn.net/Articles/306704/
[8]: https://lwn.net/Articles/330589/
[9]: https://pl.wikipedia.org/wiki/Drzewo_czerwono-czarne
[10]: https://docs.clip-os.org/clipos/kernel.html
[11]: https://en.wikipedia.org/wiki/Side-channel_attack
[12]: https://eprint.iacr.org/2013/448.pdf
[13]: https://en.wikipedia.org/wiki/Data_segment#Text
[14]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
