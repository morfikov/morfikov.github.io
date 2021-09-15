---
author: Morfik
categories:
- Android
date:    2016-12-04 19:18:40 +0100
lastmod: 2016-12-04 19:18:40 +0100
published: true
status: publish
tags:
- karta-sd
- smartfon
- adb
- root
- adoptable-storage
title: 'Android: Formatowanie karty SD jako pamięć wewnętrzna'
---

Jakiś czas temu bawiąc się jednym z TP-LINK'owch smartfonów, konkretnie to był [model Neffos C5][1],
nie byłem zbytnio zadowolony z faktu, że karta SD w takim telefonie może być sformatowana jedynie
systemem plików z rodziny FAT. Takie rozwiązanie niesie ze sobą pewne niedogodności, bo [system
plików FAT ma dość spore ograniczenia][2] jeśli chodzi o przechowywanie informacji. Niekoniecznie
wszyscy musimy wgrywać na smartfona bardzo duże pliki czy też trzymać ich tam setki GiB, bo to jest
raczej rzadkością, ale brak wsparcia uprawnień do plików i katalogów w systemie plików FAT powoduje,
że aplikacje w Androidzie nie chcą zapisywać swoich danych na karcie SD, która taki system plików
wykorzystuje. W efekcie trzeba kombinować, by [aplikacja kamery/aparatu zapisywała zdjęcia czy
materiał video na karcie SD][3]. Na smartfonach TP-LINK'a, które mają zainstalowany Android 6.0
Marshmallow, np. [Y5][4] czy [Y5L][5]), jesteśmy w stanie sformatować karty SD jako pamięć
wewnętrzna za sprawą wprowadzonego w tej wersji Androida [mechanizmu Adoptable Storage][6].
Postanowiłem zatem sprawdzić jak taki proces formatowania karty SD przebiega i co dokładnie może
nam przynieść jego przeprowadzenie.

<!--more-->
## Jak sformatować kartę SD jako pamięć wewnętrzna

W starszych wersjach Androida, tj. 5.1 (Lollipop) i niższych, nie było możliwości sformatowania
karty SD jako pamięć wewnętrzna. Ta opcja jest dostępna jedynie w Androidach począwszy od wersji 6.0
(Marshmallow). Jeśli dysponujemy smartfonem, który ma zainstalowany taki system, i drażni nas
wykorzystywanie karty SD jako pamięć przenośna z systemem plików FAT, to możemy pokusić się o
sformatowanie jej w nieco inny sposób niż zazwyczaj. Proces jest dość prosty ale trzeba uważać na
kilka rzeczy.

Po wsadzeniu karty SD do slotu w smartfonie, Android powinien nam wyrzucić informację dotyczącą
wykrycia takiego nośnika i udostępnić nam opcje formatowania karty SD, ewentualnie zawsze możemy
wejść w Ustawienia => Pamięć plików:

![](/img/2016/12/001.karta-sd-pamiec-wewnetrzna-android-formatowanie.png#big)

W tym przypadku karta SD jest sformatowana w tradycyjny sposób i automatycznie zamontowana w
systemie. Jak widzimy wyżej, mamy opcję "Sformatuj jako pamięć wewnętrzna". Klikamy w nią:

![](/img/2016/12/002.karta-sd-pamiec-wewnetrzna-android-formatowanie.png#big)

Sformatowanie karty SD w taki sposób uniemożliwi nam korzystanie z niej na innych urządzeniach. Ten
proces kasuje także wszystkie dane zgromadzone na karcie SD. Dlatego też przed przeprowadzeniem go
upewnijmy się, że zrobiliśmy ewentualny backup zawartości karty SD:

Podczas procesu formatowania karty SD są również przeprowadzane testy zapisu/odczytu nośnika pod
kątem oceny jego prędkości. W przypadku, gdy karta nie należy do najszybszych i odbiega znacząco
parametrami od flash'a telefonu, to zostanie wyrzucony komunikat o możliwym spowolnieniu działania
systemu:

![](/img/2016/12/004.karta-sd-pamiec-wewnetrzna-android-wolna.png#medium)

Trzeba sobie zdawać sprawę, że nawet te szybsze karty SD są parokrotnie wolniejsze niż wbudowana w
smartfon pamięć flash, zatem i tak spowolnienie w działaniu systemu odczujemy. Dlatego też jeśli już
zamierzmy bawić się w formatowanie karty SD jako pamięć wewnętrzna, to dokupmy możliwie jak
najszybszą kartę SD, [min. klasa 10 lub UHS-1][7].

Po tym jak proces formatowania karty dobiegnie końca, zostaniemy poproszeni o określenie czy chcemy
zainstalowane w systemie aplikacje oraz ich prywatne dane przenieść na kartę SD:

![](/img/2016/12/005.karta-sd-pamiec-wewnetrzna-android-przenoszenie-danych.png#big)

### Przeniesienie aplikacji w późniejszym czasie

Proces przenoszenia aplikacji będzie można przeprowadzić w późniejszym czasie bez obawy o utratę
zgromadzonych już plików na karcie SD. Jeśli nie zdecydujemy się na przeniesienie danych w procesie
formatowania karty SD, to przestrzeń kary nie zostanie dodana do tej, którą mamy dostępną na flash'u
telefonu. W menadżerze plików nie zobaczymy też już pozycji karty SD i nie będziemy w stanie wgrać
na nią własnych plików:

![](/img/2016/12/006.karta-sd-pamiec-wewnetrzna-android-przenoszenie-danych.png#big)

### Przeniesienie aplikacji w procesie formatowania karty SD

W przypadku przeniesienia aplikacji z flash'a na kartę SD, stracimy dostęp do tej prawdziwej
wewnętrznej pamięci telefonu. By to nieco lepiej zobrazować, posłużmy się przykładem. W tym
przypadku flash w telefonie ma 16 G, z czego 12 G jest przeznaczone na partycję `/data/` . Karta SD
ma zaś 2 G. Po przeniesieniu danych, system będzie widział jedynie 2 G, a nie 14 G. Nie ma
możliwości, by te dwie przestrzenie połączyć ze sobą, no chyba, że sformatujemy kartę SD w
standardowy sposób z wykorzystaniem systemu plików FAT.

![](/img/2016/12/008.karta-sd-pamiec-wewnetrzna-android-przenoszenie-danych.png#big)

### Hybrydowa lokalizacja aplikacji

Po sformatowaniu karty SD jako pamięć wewnętrzna i przeniesieniu danych, nowe aplikacje przy
instalacji będą umieszczane na karcie SD tylko w momencie, gdy developer takiego programu wspiera
Adoptable Storage. Jeśli ten mechanizm nie jest wspierany przez aplikacje, to dane będą zapisywane
na flash'u telefonu. Tutaj warto też zaznaczyć, że pojedyncze aplikacje możemy przenieść sami na
kartę SD. Wystarczy, że przejdziemy w Ustawienia => Aplikacje => Informacje o aplikacji => Pamięć
plików i klikniemy przycisk "Zmień":

![](/img/2016/12/009.karta-sd-pamiec-wewnetrzna-android-przenoszenie-aplikacji.png#huge)

Wszystkie aplikacje, które zostaną w taki sposób przeniesione, są zapamiętywane przez Androida i w
przypadku odłączenia karty SD, te programiki przestaną nam działać (przestaną być widoczne przez
system) do momentu, aż ta karta SD zostanie podłączona ponownie.

W przypadku, gdy nie przenosiliśmy plików podczas procesu formatowania karty SD, aplikacje będą
zapisywane domyślnie na flash'u telefonu. Ten stan rzeczy może zostać zmieniony za sprawą narzędzia
`adb` . [Proces instalacji adb na linux został opisany tutaj][8]. Mając już dostęp do `adb` wydajemy
poniższe polecenia (wymagany root):

    # adb shell
    shell@Y5:/ # pm get-install-location
    0[auto]

    shell@Y5:/ # pm set-install-location 2

Wartości jakie mamy do wyboru to: `0[auto]` , `1[internal]` oraz `2[external]` .

Ta opcja z przenoszeniem danych na kartę SD jest użyteczna chyba jedynie w przypadku smartfonów,
które mają niewielkich rozmiarów pamięć flash, gdzie zwyczajnie brakuje nam miejsca na aplikacje,
nie wspominając już o innych danych, np. filmy czy zdjęcia. Jeśli nasz smartfon ma większy flash niż
16 G, to korzystanie z Adoptable Storage jest moim zdaniem pozbawione sensu.

## Przenoszenie danych z karty SD na flash smartfona

W przypadku, gdy rozczarowaliśmy się tym całym mechanizmem Adoptable Storage i zwyczajnie nam on nie
odpowiada ale przenieśliśmy już dane na kartę SD, to bez problemu możemy cały proces odwrócić i
przenieść dane z karty SD na flash smartfona. Wystarczy przejść w Ustawienia => Pamięć plików =>
Pamięć wewnętrzna. Tam z kolei w menu po prawej stronie na górze wybieramy "Przenieść dane":

![](/img/2016/12/010.karta-sd-pamiec-wewnetrzna-android-przenoszenie-danych.png#huge)

Trzeba jednak pamiętać, że ten proces przenoszenia danych z karty SD na pamięć telefonu może w
pewnych sytuacjach doprowadzić do bootloop, czyli zapętlenia się startu smartfona. Możemy do takiego
stanu doprowadzić przez nieuwagę, gdzie ilość przenoszonych danych z karty SD będzie zbliżona lub
większa niż rozmiar docelowego miejsca na flash'u smartfona na partycji `/data/` . Dlatego też
zwracajmy uwagę na to ile danych Android chce przenieść. Oczywiście z takiego bootloop'a da radę się
wybronić ale nie obejdzie się bez sformatowania partycji `/data/` z poziomu bootloader'a, a to, jak
zapewne wiemy, efektywnie zniszczy wszystkie nasze dane.

## Odłączanie karty SD sformatowanej jako pamięć wewnętrzna

Kartę SD sformatowaną jako pamięć wewnętrzna w dalszym ciągu jesteśmy w stanie odłączyć od systemu.
Niemniej jednak, nie jest to zalecane:

![](/img/2016/12/011.karta-sd-pamiec-wewnetrzna-android-odlaczanie.png#big)

Tutaj taka mała uwaga. Po przeprowadzeniu procesu formatowania karty SD zawsze zrestartujmy
smartfon. Bez tego kroku, Android może się zachowywać wręcz nieobliczalnie, co może być źródłem
różnych dziwnych problemów w działaniu systemu.

## Zasada działania mechanizmu Adoptable Storage

Podczas procesu formatowania karty SD jako pamięć wewnętrzna została nam wyświetlona informacja na
temat tego, że tej karty SD nie da rady wykorzystywać na innym urządzeniu niż to, na którym ten
proces został przeprowadzony. Taka karta zostanie sformatowana systemem plików EXT4 i dodatkowo dane
zostaną na tej karcie zaszyfrowane. Spójrzmy na tę poniższa fotkę obrazującą kilka partycji
widzianych przez Androida:

![](/img/2016/12/012.karta-sd-pamiec-wewnetrzna-android-widok-podzial.png#big)

Standardowa wielkość pamięci flash w smartfonie Neffos Y5, to 16 G, z czego tylko 12 G jest
przeznaczone dla partycji `/data/` , czyli danych użytkownika. Karta SD wykorzystywana w tym
przypadku ma jedynie 2 G. Po aktywowaniu mechanizmu Adoptable Storage, partycja `/data/` nie
zmieniła swojego rozmiaru i dalej ma 12 G. Na fotce mamy też sekcję "Karta SD" i tutaj już sprawa
wygląda ciekawie, bo mamy dwie partycje na tej karcie. Pytanie się nasuwa: czemu dwie a nie jedna?
Nie wiem jakie czary Android odprawia, by korzystać z Adoptable Storage ale najwyraźniej potrzebne
są do tego celu jakieś metadane (pierwsza partycja), które umożliwiają systemowi zamontowanie
pozostałej części karty SD.

Dane na karcie SD po sformatowaniu jej jako pamięć wewnętrzna są szyfrowane z wykorzystaniem
linux'owego narzędzia `dm-crypt` przy zastosowaniu algorytmu AES o rozmiarze klucza 128-bit
(aes-cbc-essiv:sha256). Jak się przyjrzymy bardziej uważnie, to na tej fotce dostrzeżemy "Device
Mapper" i to jest właśnie odszyfrowany i przemapowany obszar karty SD, na którym system jest w
stanie operować. Ta przestrzeń jest już sformatowana z wykorzystaniem natywnego systemu plików
wykorzystywanego w linux'ach, tj. EXT4. Niemniej jednak, możemy zapomnieć o wykorzystaniu
standardowego nośnika sformatowanego pod linux'em jako EXT4.

Może i mamy tutaj systemem plików EXT4 ale linux nie będzie w stanie tej karty SD odczytać,
przynajmniej nie bez uprzedniego odszyfrowania interesującego nas obszaru karty. Technicznie
jesteśmy w stanie wejść w interakcję z tą kartą SD na linux'ie ale w tym celu musielibyśmy wydobyć
klucz szyfrujący.

Klucz do takiego zaszyfrowanego kontenera jest generowany losowo podczas procesu formatowania karty
SD. Ten klucz jest także przechowywany gdzieś na flash'u smartfona. Gdzie dokładnie jest on
ulokowany i czy można go wydobyć bez ukorzeniania systemu (root), tego jeszcze nie wiem. Niemniej
jednak, wszystko wskazuje, że [root będzie niezbędny][9], by dostęp do tego klucza uzyskać. Z kolei
[tutaj ludzie piszą][10], że nie we wszystkich smartfonach szyfrowanie jest włączone domyślnie. [W
tym wątku][11] są zaś zebrane te bardziej użyteczne informacje dotyczące Adoptable Storage.

Warto w tym miejscu zaznaczyć, że jeśli flash telefonu nie jest zaszyfrowany, to ten klucz można
odzyskać i zdeszyfrować zawartość karty SD. Dlatego też jeśli zamierzamy korzystać z Adoptable
Storage, to rozważałbym zaszyfrowanie telefonu, co można zrobić przechodząc w Ustawienia =>
Zabezpieczenia.

## Pamięć przenośna i pamięć wewnętrzna na jednej karcie SD

Generalnie rzecz biorąc, ten cały Adoptable Storage średnio mi się podoba. Nie chciałbym powiedzieć,
że jest on kompletnie nieprzydatny ale raczej nie zamierzam z niego korzystać. Niemniej jednak,
szukając o nim informacji natrafiłem na ciekawy trik, który umożliwia sformatowanie karty SD w taki
sposób, by jedna część robiła za pamięć wewnętrzną, a druga za pamięć przenośną. W efekcie w
przypadku tych większych kart SD, np. 64 G czy 128 G, możemy stworzyć nieco bardziej zaawansowany
setup. Nie powiem, że jest to wyjście idealne ale zawsze daje ono nam nieco większe pole manewru
przy zarządzaniu danymi na karcie SD.

Taki zabieg możemy przeprowadzić z poziomu telefonu ale musimy uzbroić się w narzędzie `adb` . Mając
zainstalowany `adb` podłączamy się do telefonu i listujemy dostępne nośniki:

    # adb shell
    shell@Y5:/ $ sm list-disks
    disk:179,64

Wyżej został nam zwrócony `disk:179,64` . Całą tą frazę podajemy w poniższym poleceniu:

    shell@Y5:/ $ sm partition disk:179,64 mixed 75

Opcja `mixed 75` oznacza, że zamierzamy sformatować tę kartę SD jako pamięć wewnętrzno-przenośna (do
wyboru są jeszcze public i private). Liczba `75` to procent przestrzeni karty SD, który zostanie
przeznaczony na pamięć przenośną.

Gdy proces formatowania karty dobiegnie końca, możemy sprawdzić czy Android poprawnie rozpoznał
podział karty SD w Ustawienia => Pamięć plików:

![](/img/2016/12/013.karta-sd-pamiec-wewnetrzna-android-hybryda.png#medium)

Karta ma 2 G pamięci, z czego około 25% zostało przeznaczone na pamięć wewnętrzną, a reszta na
pamięć przenośną, czyli wszystko się zgadza.

Może i widzimy wyżej przycisk odmontowania karty SD ale pamiętajmy, że nie dotyczy on tej
przestrzeni, która jest sformatowana jako pamięć wewnętrzna. By być w stanie bezpiecznie usunąć
kartę z telefonu, musimy również wejść w opcje pamięci wewnętrznej (czerwona pozycja na powyższej
fotce) i tam z menu wybrać "Odłącz":

![](/img/2016/12/014.karta-sd-pamiec-wewnetrzna-android-hybryda-odlaczanie.png#big)

Jeśli ktoś jest ciekaw jak taka karta jest widziana pod linux'em, to poniżej jest fotka z
`gparted` :

![](/img/2016/12/015.karta-sd-pamiec-wewnetrzna-android-gparted.png#huge)

Jako, że partycja sformatowana systemem plików FAT jest pierwsza w szeregu, to nie powinno być
problemów z zamontowaniem jej nawet pod windows, choć ten nie zawsze może z taką kartą
współpracować. W przypadku dowolnego linux'a nie będzie z nią problemów, bo tutaj nawet kolejność
partycji nie ma większego znaczenia.

### Uszkodzenie karty SD sformatowanej jako pamięć wewnętrzna

W przypadku standardowego sformatowania karty SD jako pamięć wewnętrzna nie ma raczej problemu z
ponownym formatowaniem nośnika na wypadek, gdyby ten został w jakiś sposób uszkodzony, a jego
zawartość przestałaby być dla nas czytelna. Niemniej jednak, jeśli chodzi o sytuację sformatowania
karty SD jako wewnętrzno-przenośna, to tutaj już sprawa się komplikuje, bo przecież chcielibyśmy
jedynie sformatować tylko część nośnika, a nie cały. W opcjach Androida nie ma pozycji, które
pomogłyby nam uporać się z tym zadaniem i nie obejdzie się bez skorzystania z narzędzia `adb` :

    # adb shell

    shell@Y5:/ $ sm list-volumes all
    private:179,67 mounted ec91531a-61d7-4beb-adaa-272b2163b193
    private mounted null
    public:179,65 mounted 4B44-1110
    emulated mounted null
    emulated:179,67 unmounted null

Pozycja `private` opowiada za tę przestrzeń karty SD, która została zamontowana jako pamięć
wewnętrzna. Z kolei pozycja `public` jest zamontowana jako pamięć przenośna. Jeśli chcemy
sformatować jedynie część karty (niekoniecznie pamięć wewnętrzną), to wybieramy jedną z widocznych
wyżej pozycji, tj. `private:179,67` lub `public:179,65` i wydajemy poniższe polecenia:

    shell@Y5:/ $ sm format private:179,67
    shell@Y5:/ $ sm mount private:179,67

Jeśli nie zostaną zwrócone żadne błędy w procesie formatowania i montowania zasobu, to znaczy, że
proces ponownego formatowania części nośnika zakończył się powodzeniem. Dla uniknięcia ewentualnych
problemów, dobrze jest zrestartować smartfon tak, by jego system uruchomił się ponownie.


[1]: http://www.neffos.pl/product/details/C5
[2]: https://pl.wikipedia.org/wiki/FAT32
[3]: /post/android-brak-mozliwosci-zapisu-danych-na-karcie-sd-neffos-c5/
[4]: http://www.neffos.pl/product/details/Y5
[5]: http://www.neffos.pl/product/details/Y5L
[6]: https://source.android.com/devices/storage/adoptable
[7]: https://www.sdcard.org/developers/overview/speed_class/
[8]: /post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/
[9]: https://nelenkov.blogspot.fr/2015/06/decrypting-android-m-adopted-storage.html
[10]: https://www.reddit.com/r/Android/comments/3o6u80/psa_i_formatted_my_sd_card_as_internal_storage/
[11]: https://www.reddit.com/r/Android/comments/3oz7eu/guidelines_for_marshmallow_users_formatting/
