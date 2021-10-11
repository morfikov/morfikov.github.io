---
author: Morfik
categories:
- Android
date:    2017-01-20 18:37:11 +0100
lastmod: 2017-01-20 18:37:11 +0100
published: true
status: publish
tags:
- smartfon
- prywatność
- factory-reset
- spflashtool
GHissueID: 62
title: Factory Reset Protection (FRP) w smartfonach z Androidem
---

Kupowanie telefonów czy smartfonów z Androidem z innych źródeł niż oficjalne punkty sprzedaży nie
zawsze jest bezpieczną opcją. Gdy nabywamy takie urządzenie od znajomego, to raczej nie powinniśmy
się martwić o to, że ten telefon może być kradziony. Niemniej jednak, po zakupie takiego urządzenia,
poprzedni użytkownik zwykle resetuje jego ustawienia do fabrycznych, by klient miał świeży system i
nie był w stanie uzyskać dostępu do prywatnych danych poprzedniego właściciela smartfona. Nie byłoby
w tym nic nadzwyczajnego, gdyby nie fakt, że nabywca tak odsprzedanego telefonu może mieć pewne
problemy ze skonfigurowaniem Androida, bo ten system zwróci mu komunikat: "Urządzenie zostało
zresetowane. Aby kontynuować, zaloguj się na konto Google, które było wcześniej synchronizowane na
tym urządzeniu", czyli telefon został zablokowany przez mechanizm Factory Reset Protection Lock (FRP
Lock). Jeśli znajomy mieszka blisko nas, to naturalnie możemy się przejść do niego w celu zdjęcia
tej blokady. A co w przypadku, gdy nabyliśmy urządzenie na odległość? Czy jest jakiś sposób na
obejście tej blokady w przypadku smartfonów Neffos od TP-LINK?

<!--more-->
## Factory Reset Protection Lock (FRP Lock)

Urządzenia takie jak smartfony zwykle zabezpieczone są jakąś formą blokady ekranu, np.kodem PIN. By
skorzystać z takiego telefonu trzeba ten kod pierw wprowadzić. Problem pojawia się w momencie, gdy
tego kodu z jakiegoś powodu nie znamy. W przypadku, gdy jedynie zapomnieliśmy prawidłowej sekwencji
odblokowującej ekran, to nic nie stoi na przeszkodzie, by taki telefon zresetować do ustawień
fabrycznych przez tryb recovery. Stracimy co prawda wszystkie dane przechowywane na flash'u
urządzenia ale będziemy w stanie sobie na nowo skonfigurować system.

Niemniej jednak, tego typu sytuacja zwykle nie ma miejsca, natomiast dużo częściej zdarzają się
kradzieże telefonów. Taki złodziej również byłby w stanie zresetować ustawienia telefonu do
domyślnych w celu jego późniejszej sprzedaży gdzieś na targu. By utrudnić ten proceder, w
Androidach począwszy od wersji 5.0 (Lollipop) został wprowadzony mechanizm Factory Reset Protection
Lock (FRP Lock). Mając taki system, po zresetowaniu smartfona do ustawień fabrycznych przez tryb
recovery, urządzenie w dalszym ciągu będzie zablokowane. Użytkownikowi podczas konfiguracji telefonu
zostanie pokazany jedynie komunikat "Urządzenie zostało zresetowane...". By móc korzystać z takiego
telefonu, trzeba podać dane do poprzedniego konta i innej opcji zwykle nie ma.

![](/img/2017/01/001.factory-reset-protection-frp-lock-smartfon-android-blokada-telefonu.png#medium)

Po aktywowaniu blokady FRP Lock, Android jest zwykle bezużyteczny. Możemy jedynie odbierać
połączenia przychodzące i wykonywać połączenia alarmowe ale w zasadzie nic poza tym. Nie mamy
dostępu do ustawień telefonu czy przeglądarki internetowej i z lwiej części funkcjonalności
telefonu nie będziemy mogli skorzystać do momentu podania danych do konta, które było skonfigurowane
na tym telefonie przed zakupem. W przypadku zakupu telefonu na odległość, raczej jest mało
prawdopodobne, że poprzedni właściciel poda nam dane do swojego konta Google byśmy sami mogli ten
telefon odblokować. Jakie inne opcje nam zatem pozostają?

## Zasada działania FRP Lock

Szukając informacji na temat zasady działania tego całego FRP Lock, na [jednym z forów Androida][1]
użytkownik piskorfa podesłał mi linki do dwóch chińskich blogów [[1][2]] i [[2][3]]. Chińskiego co
prawda nie znam ale zawartość tych stron można przetłumaczyć na angielski w Google Translate.

Zgodnie z informacjami zawartymi w tych powyższych artykułach, mechanizm FRP Lock działa w oparciu o
dedykowaną partycję na flash'u telefonu. Nazwa tej partycji może być różna, choć zwykle przyjmuje
wartość `frp` (od Factory Reset Protection). W smartfonach Neffos C5 i C5 MAX ta partycja figuruje
na liście partycji. Nie ma jej jednak w przypadku modelów Neffos Y5 i Y5L ale to nic nie szkodzi.
Nazwę tej partycji zawsze można ustalić przeglądając, np. za pomocą `adb` , plik
`/system/build.prop` w telefonie:

    shell@C5_Max:/ $ cat /system/build.prop | grep -i frp
    ro.frp.pst=/dev/block/platform/mtk-msdc.0/by-name/frp

    shell@Y5:/ $ cat /system/build.prop | grep -i frp
    ro.frp.pst=/dev/block/bootdevice/by-name/config

Widać, zatem że w tym drugim przypadku partycja, której szukamy, nazywa się `config` .

Blokada FRP Lock jest zakładana na telefon w momencie powiązania z nim konta Google, tj.
uzupełnienia formularza w celu zalogowania się, np. do sklepu Google Play. Blokadę tę można
dezaktywować usuwając konto z telefonu lub też resetując smartfon do ustawień fabrycznych z poziomu
działającego systemu. Zatem dodając lub usuwając konto Google, Android inicjuje pewną operację na
partycji, która została zwrócona w `ro.frp.pst` .

Jeśli teraz zresetujemy telefon do ustawień fabrycznych z poziomu trybu recovery, to system
urządzenia nie przepisze nam tej partycji w żaden sposób, bo Android w trym trybie nie jest
uruchomiony. Później jak przechodzimy przez proces wstępnej konfiguracji telefonu, system odczytuje
dane z partycji `frp` i na ich podstawie decyduje czy przepuścić użytkownika, czy zablokować dostęp
i zażądać uwierzytelnienia przez podanie danych do konta, które wcześniej było na tym telefonie
skonfigurowane.

W jaki sposób system rozpoznaje czy podaliśmy dane do odpowiedniego konta? Serwery Google w tym
procesie nie biorą udziału. Te dane są zapisywane również na partycji `frp` , z tym, że raczej w
formie jakieś hasha, który można uzyskać podając konkretny login i hasło. Podając prawidłowe dane,
system jest w stanie wygenerować taki hash i porównać go z tym co zostało zapisane na partycji
`frp` .

Pewności do końca nie mam jak ten proces weryfikacji przebiega ale patrząc na zrzuty partycji w
edytorze HEX, można dojść do wniosku, że system dodaje jakieś informacje na tej partycji po
zalogowaniu się na konto Google. Jest to około 20 KiB, zatem dość sporo. Te dane jednak są w formie
nieczytelnej dla człowieka, także nic więcej na ten temat nie powiem.

Wiedząc, że partycja `frp` odgrywa kluczową rolę w zablokowaniu użytkownikowi dostępu do telefonu,
można by się pokusić o ręczne wyczyszczenie tej partycji. Jeśli faktycznie serwery Google nie biorą
udziału w tym całym procesie, to FRP Lock można by obejść lokalnie.

## Jak odblokować smartfon z aktywowanym FRP Lock

Załóżmy, że zakupiliśmy sobie używanego smartfona na odległość oraz, że to urządzenie nie było
kradzione. Naturalnie telefon nie został poprawnie zresetowany do ustawień domyślnych i my jako nowy
właściciel mamy teraz problem, bo Android wyrzuca nam informacje o założeniu blokady FRP Lock.

W zasadzie wiemy co mamy robić, tj. trzeba wyczyścić partycję `frp` . Problem w tym, że w różnych
modelach smartfonów inaczej do tego przedsięwzięcia trzeba się zabrać. Kluczowe znaczenie ma
zainstalowany w urządzeniu SoC, wersja Androida oraz ewentualne zabezpieczenia wprowadzone przez
producenta telefonu.

W przypadku smartfonów Neffos C5 i C5 MAX mamy do czynienia z SoC od MediaTek, model nie jest aż tak
ważny. Android jest zaś w wersji 5.1 (Lollipop). Natomiast Neffos Y5 i Y5L mają SoC od Qualcomm i w
tych telefonach siedzi Android w wersji 6.0 . Mamy zatem dwie różne sytuacje do rozważenia.

### Odblokowanie Neffos C5 i C5 MAX

Jako, że te dwa modele smartfonów mają SoC od MediaTek, to nadpisanie partycji `frp` w ich przypadku
jest stosunkowo proste, bo możemy do tego celu zaprzęgnąć [SP Flash Tool][4]. Problematyczne może
być ustalenie gdzie na flash'u smartfona znajduje się partycja `frp` . Ja korzystałem ze [swojego
pliku scatter.txt][5], gdzie mam taki oto blok kodu:

    - partition_index: SYS18
      partition_name: frp
      file_name: NONE
      is_download: true
      type: NORMAL_ROM
      linear_start_addr: 0x6a00000
      physical_start_addr: 0x6a00000
      partition_size: 0x100000
      region: EMMC_USER
      storage: HW_STORAGE_EMMC
      boundary_check: true
      is_reserved: false
      operation_type: UPDATE
      reserve: 0x00

W zasadzie to, interesuje nas tutaj wartość `0x100000` , która wskazuje nam rozmiar partycji i jest
to 1048576 bajtów, czyli 1 MiB. Trzeba zatem stworzyć plik wypełniony samymi zerami, który będzie
miał dokładnie taki rozmiar. Możemy to zrobić przy pomocy `dd` z poziomu każdego linux'a:

    # dd if=/dev/zero of=./c5max-frp.orig bs=1K count=1024

Tak wygenerowany plik trzeba przy po mocy SP Flash Tool wgrać w odpowiednie miejsce na flash'u
smartfona. Odpalamy zatem narzędzie SP Flash Tool i przechodzimy na zakładkę `Download` i tam
zaznaczamy partycję `frp` i wskazujemy ścieżkę do pliku z zerami:

![](/img/2017/01/002.factory-reset-protection-frp-lock-smartfon-android-czyszczenie-partycji.png#huge)

Upewniamy się, że nad listingiem partycji mamy zaznaczone `Download Only` i wciskamy przycisk
`Download` . W tym momencie SP Flash Tool będzie oczekiwał na podłączenie smartfona do portu USB
komputera. Wyłączamy zatem telefon i podłączamy go do komputera. System powinien go automatycznie
wykryć i zaaplikować mu wskazany plik:

![](/img/2017/01/003.factory-reset-protection-frp-lock-smartfon-android-czyszczenie-partycji.png#huge)

Jeśli są jakieś problemy z działaniem SP Flash Tool, to prawdopodobnie nie ma on uprawnień do
urządzenia `/dev/ttyACM0` i trzeba będzie dodać naszego użytkownika do grupy `dialout` .

Po wgraniu pliku, włączamy smartfon i już nie powinniśmy mieć problemów z dodaniem nowego konta
Google na naszym smartfonie.

![](/img/2017/01/004.factory-reset-protection-frp-lock-smartfon-android-zdjecie-blokady.png#medium)

### Odblokowanie Neffos Y5 i Y5L

W przypadku smartfonów Neffos Y5 i Y5L sprawa nie wygląda tak różowo. Nie tylko nie mamy możliwości
skorzystania z SP Flash Tool, bo SoC jest od Qualcomm'a, to jeszcze wygląda na to, że blokada OEM
(ta w opcjach developerskich) działa i uniemożliwia odblokowanie bootloader'a. Bez odblokowanego
bootloader'a z kolei nie damy rady wgrać obrazu przy pomocy narzędzia `fastboot` .

Próbowałem obejść blokadę FRP Lock w tych telefonach na kilka różnych sposobów ale żadne ze
znalezionych przeze mnie rozwiązań nie dało rady sprostać zabezpieczeniom, które w tych Neffos'ach
zostały zaimplementowane.

W Neffos Y5 i Y5L mamy praktycznie gołego Androida 6.0 i jedyne co możemy próbować zrobić, to
uzyskać dostęp do ustawień telefonu w celu przeprowadzenia procesu Factory Reset z poziomu
działającego systemu. Przynajmniej tak wynika z tych materiałów, z którymi się zdążyłem
zapoznałem. Pytanie jest tylko jak wywołać ustawienia, skoro mamy zablokowaną możliwość operowania
na smartfonie i jedyny obrazek jaki widzimy, to ten poniżej:

![](/img/2017/01/005.factory-reset-protection-frp-lock-smartfon-android-blokada.png#medium)

#### Sposób z przeglądarką

Może i na pierwszy rzut oka nic nie da się zrobić i FRP Lock spełnia swoje zadanie ale nawet w tym
miejscu jesteśmy w stanie wywołać przeglądarkę internetową, która pozwoli nam uzyskać dostęp do
ustawień systemu. W jaki sposób? Wyżej widzimy formularz, w którym mamy wpisać adres email lub numer
telefonu. Generalnie nie wpisujemy tutaj tego, o co nas proszą. Zamiast tego wpisujemy dosłownie
cokolwiek. Na ten wpisany w formularzu wyraz możemy kliknąć i pojawi nam się proste menu:

![](/img/2017/01/006.factory-reset-protection-frp-lock-smartfon-android-przegladaka-ustawienia.png#medium)

Z tego menu wybieramy pozycję `Podpowiedzi` (przez te trzy kropki):

![](/img/2017/01/007.factory-reset-protection-frp-lock-smartfon-android-przegladarka-ustawienia.png#huge)

I jak widzimy, odpaliła nam się przeglądarka Chrome. Nie logujemy się tutaj i wciskamy "Nie Dzięki".

Na środku ekranu mamy standardowy formularz wyszukiwania, jak w każdej wyszukiwarce. W tym
formularzu wpisujemy w zależności od wykorzystywanego języka w telefonie: `ustawienia` (PL) lub
`settings` (EN). Jak tylko zaczniemy wpisywać kolejne znaki w polu formularza, na dole ekranu
powinny nam się pojawić podpowiedzi:

![](/img/2017/01/008.factory-reset-protection-frp-lock-smartfon-android-przegladarka-ustawienia.png#small)

Mamy pozycję `Ustawienia` i naturalnie klikamy w nią. Powinien nam się ukazać znajomy widok ustawień
systemowych. Zatem nawet mając aktywny mechanizm blokady telefonu, jesteśmy w stanie go ominąć i
wejść w ustawienia telefonu.

![](/img/2017/01/009.factory-reset-protection-frp-lock-smartfon-android-factory-reset.png#big)

W tak uzyskanym menu przechodzi do pozycji `Kopia zapasowa i reset` , a z niej wybieramy `Przywróć
ustawienia Fabryczne` :

![](/img/2017/01/010.factory-reset-protection-frp-lock-smartfon-android-factory-reset.png#big)

W ten sposób niby powinniśmy pozbyć się blokady, bo proces Factory Reset zostanie przeprowadzony z
poziomu działającego systemu. Niestety najwyraźniej w nowszych wersjach Androida partycja `frp` nie
jest przepisywana jeśli Factory Reset jest przeprowadzany z poziomu systemu, na którym nie ma
skonfigurowanego konta Google. Zatem może i zresetujemy ustawienia ale w dalszym ciągu system przy
konfiguracji telefonu będzie nas prosił o podanie danych do starego konta Google.

#### Sposób ze zdjęciem blokady OEM

Kluczem do zdjęcia blokady FRP Lock jest odblokowanie bootloader'a, a to można zrobić zdejmując
pierw blokadę OEM z poziomu opcji developerskich. Mając dostęp do opcji telefonu, możemy wejść w
"Informacje o telefonie" i spróbować postukać w numer kompilacji.

![](/img/2017/01/011.factory-reset-protection-frp-lock-smartfon-android-numer-kompilacji.png#big)

Problem w tym, że ten sposób również nie działa i w tym przypadku stukanie w numerek kompilacji nic
nie daje, a bez tego nie pojawią nam się opcje developerskie i nie ściągniemy blokady OEM.

#### Sposób z wyłączeniem WiFi

Innym sposobem, który znalazłem, miało być ogłupienie systemu przez rozłączenie sieci WiFi w
odpowiednim momencie. Gdy jesteśmy na pozycji "Wybierz WLAN" przy konfiguracji telefonu, to
naturalnie wskazujemy naszą sieć i uzupełniamy dane logowania do tej sieci. Później wracamy
przyciskiem Wstecz do ekranu wyboru sieci. Powinniśmy widzieć listę sieci WiFi w naszej lokalizacji
oraz powinniśmy być podłączeni do tej, którą sobie skonfigurowaliśmy:

![](/img/2017/01/012.factory-reset-protection-frp-lock-smartfon-android-wlan.png#medium)

W tym miejscu dajemy "Dalej" i gdy na ekranie pojawi się informacja "Sprawdzam połączenie" ale przed
"Aktualizuję oprogramowanie" (szybko przeskakuje) trzeba sieć WiFi rozłączyć. Można albo wyłączyć
router WiFi przyciskiem, albo też w ustawieniach routera wyłączyć samo WiFi.

Warto tutaj zaznaczyć, że w telefonie nie może być obecna karta SIM, bo wtedy dane mogą być
wymieniane po 3G/LTE. W takim przypadku, smartfon nie będzie w stanie połączyć się z serwerami
Google po uprzednim zapewnieniu, że połączenie działa. Taki stan rzeczy najprawdopodobniej sprawia,
że system głupieje i pomija proces uwierzytelniania zwracając informację "Nie można się zalogować"
i proces konfiguracji telefonu może być kontynuowany:

![](/img/2017/01/013.factory-reset-protection-frp-lock-smartfon-android-wlan-proces.png#huge)

Naturalnie klikamy Dalej i Dalej i w zasadzie wszystko wskazuje na to, że proces zostanie ukończony
z powodzeniem. Niemniej jednak, z jakiegoś powodu system stwierdza, że nie jesteśmy zalogowani i
każe nam cały proces powtórzyć.

![](/img/2017/01/014.factory-reset-protection-frp-lock-smartfon-android-wlan-blad.png#big)

#### Sposób z linkami w opcjach języka i klawiatury

Kolejnym sposobem, który może nam pomóc z ominięciem blokady FRP Lock, jest próba wywołania ustawień
systemowych za pomocą linków, do których mamy dostęp z menu różnych aplikacji systemowych. W
procesie wstępnej konfiguracji telefonu mamy dostęp do jednej takiej aplikacji, tj. ustawienia
języka i klawiatury (czy jak to się tam nazywa). W opcje tej aplikacji można wejść przyciskając
przez dłuższą chwilę znak `@` na klawiaturze ekranowej. W ten sposób powinno nam się pojawić małe
kółko zębate oferujące "Opcje wprowadzania":

![](/img/2017/01/015.factory-reset-protection-frp-lock-smartfon-android-opcje-klawiatura.png#big)

Po wejściu w te opcje, w prawym górnym rogu mamy trzy kropki z menu pomocy, które powinniśmy wywołać
w celu uzyskania dostępu do upragnionego linku, za pomocą którego można by wywołać przeglądarkę i za
jej pomocą wejść w główne ustawienia telefonu:

![](/img/2017/01/016.factory-reset-protection-frp-lock-smartfon-android-opcje-klawiatura.png#big)

Problem w tym, że żadna z tych opcji się nie da wcisnąć, czyli kolejna ślepa uliczka. Znacie jeszcze
jakieś ciekawe pomysły na obejście tej blokady? :D

## Jak uniknąć zablokowania smartfona

Wygląda na to, że w przypadku Neffos C5 i C5 MAX zabezpieczenie FRP Lock jest praktycznie
bezużyteczne. Oczywiście w dalszym ciągu zdjęcie tej blokady dla przeciętnego Kowalskiego może być
zbyt trudne ale jak widać jest ono możliwe. Natomiast póki co nie mam pojęcia jak obejść tę blokadę
w Neffos Y5 i Y5L i w przypadku tych telefonów raczej nie chcielibyśmy tego FRP Lock'a złapać.

Dlatego też zawsze przed odsprzedaniem komuś telefonu miejmy na uwadze to zabezpieczenie i manualnie
usuwajmy konto Google z systemu. Nie zaszkodzi też usunięcie blokady ekranu przed zresetowaniem
smartfona do ustawień fabrycznych.

Jeśli zaś kupujemy telefon od kogoś, to poprośmy tę osobę o wykonanie procesu Factory Reset z
poziomu trybu recovery, tak by ta czynność została wykonana przy nas. Po czym sprawdźmy czy w
procesie wstępnej konfiguracji telefonu nie złapiemy FRP Lock'a.


[1]: http://forum.android.com.pl/topic/307282-jak-dzia%C5%82a-mechanizm-factory-reset-protection-lock-frp-lock/#comment-4963533
[2]: http://blog.csdn.net/woshing123456/article/details/44524051
[3]: https://echuang54.blogspot.com/2015/03/factory-reset-protection.html
[4]: http://spflashtool.com/
[5]: /img/manual/mt6753-neffos-c5-max-tp-link-scatter.txt
