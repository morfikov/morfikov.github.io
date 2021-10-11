---
author: Morfik
categories:
- Linux
date: "2015-10-20T18:27:46Z"
date_gmt: 2015-10-20 16:27:46 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- online-storage
GHissueID: 178
title: Kontener TrueCrypt trzymany na dropbox'ie
---

Żyjemy w czasach, w których mobilność jest tak samo ważna albo może i ważniejsza (dla niektórych
ludzi na pewno) jak i bezpieczeństwo i poufność danych. Użyteczność zwykle nie idzie w patrze z
bezpieczeństwem, bo im prostszy jest dla nas dostęp do danych, tym bardziej zagraża ich
bezpieczeństwu. W tym przypadku chcielibyśmy mieć możliwość dostępu do plików, np. naszego domowego
PC, z dowolnego miejsca na ziemi. Czy można w prosty i w miarę bezpieczny sposób coś takiego
osiągnąć? [Jakiś czas temu opisywałem implementację encfs na
dropbox'ie](/post/implementacja-encfs-na-dropboxie/), w tym artykule zostanie zaś
opisane sprzęgnięcie dropbox'a z kontenerem TrueCrypt.

<!--more-->
## Połączenie dropbox'a z TrueCrypt'em

Przeglądając sieć, przypadkiem natrafiłem na informacje dotyczące połączenia funkcjonalności
TrueCrypt'a z dropbox'em. Nie byłem jednak pewien, co do tego jak to ma działać. Pierwsze co mi
przyszło do głowy, to utworzenie kontenera na pliki w TrueCrypt'cie i umieszczeniu go w lokalnym
folderze dropbox'a. Następnie zostałby on wysłany do chmury i zsynchronizowany na każdej podpiętej
maszynie. Ok ale gdy mamy kontener, powiedzmy 4 GiB, to przesyłanie/pobieranie 4 GiB za każdym razem
troszeczkę by nam nadwyrężyło łącze. Oczywiście, chcemy być mobilni, a ciągnięcie 4 GiB przez WiFi
na fonie raczej odpada.

## Zasada działania synchronizacji plików

Na szczęście dropbox inaczej zarządza synchronizacją danych niż się może początkowo wydawać. Na
pewno konieczne będzie przesłanie całego pliku ale tylko z jednej maszyny. Ponad to, na każdym
dodatkowym sprzęcie trzeba będzie ten plik umieścić. Oczywiście, możemy przesłać plik lokalnie. Nie
musimy tego robić przy pomocy dropbox'a, co zaoszczędzi nam sporo czasu. Gdy przekopiujemy pliki na
porządne urządzenia, zostanie nam konfiguracja dropboxa ale to nie powinno sprawić problemu.

Dropbox pobiera plik z jednej maszyny i wysyła go do chmury. Po podłączeniu nowej maszyny, dropbox
sprawdza czy pliki są takie same. Jeśli się różnią, synchronizuje katalogi na maszynach z tymi u
siebie. Niby nic zaawansowanego. Jak to ma się do kontenera TrueCrypt'a? Mając synchronizowany plik
kontenera 4 GiB, można dość do wniosku, że każda zmiana w jego strukturze (zapisanie danych w
kontenerze) doprowadzi do ponownego przesłania całego pliku. Tak się jednak nie stanie. [Dropbox
dzieli pliki na kawałki 4
MiB](https://blogs.dropbox.com/tech/2014/07/streaming-file-synchronization/) i tworzy ich sumy
kontrolne, które przechowuje w bazie danych. Zatem nasz kontener będzie miał 1000 części po 4 MiB. W
przypadku modyfikacji którejkolwiek z części, zostanie ona przesłana przez sieć.

Nasuwa się zatem pytanie: co w przypadku gdy wrzucimy do kontenera plik, powiedzmy, 1 MiB? Na dobrą
sprawę zostanie przesłane nieco ponad 1 MiB danych. Czemu nie 4 MiB? Faktyczne dane zajmują 1 MiB,
reszta części zostanie wypełniona zerami, a następnie wszystko zostanie skompresowane. Część części
pliku zawierającego zera (3 MiB), po kompresji będzie zawierać mniej więcej tyle samo miejsca. Temu
też zostanie przesłane przez sieć 1 MiB, a nie 4 MiB, co bardzo odciąża łącze.

## Bezpieczeństwo i poufność danych

Nie jestem zwolennikiem powierzania bezpieczeństwa swoich plików komuś trzeciemu, dlatego też wadą
przesłania kontenera TrueCrypt do chmury dropbox'a jest jego jawność, a fakt przesłania komuś
plików, o których nie powinien wiedzieć, nie jest do pogodzenia z moja polityką poufności i
bezpieczeństwa danych. Najprościej założyć zatem, że panowie z dropbox'a będą próbowali się dostać
do naszych plików. Kontenery TrueCrypt mają z reguły hasło. By usługa zaszyfrowanego voluminum na
dropbox'ie była w miarę niezbyt upierdliwa nie będziemy stosować tutaj hasła 50 znakowego. Hasło nie
zostanie też zlikwidowane. Skorzystamy z kombinacji w miarę normalnego hasła oraz pliku klucza.
Posiadając lokalnie plik klucz, panowie z dropbox'a nie będą mogli wyciągnąć z kontenera żadnych
danych, nawet gdy uzyskają hasło.

Jak będą wyglądać nasze pliki klucze i po co nam tak naprawdę hasło? Za pliki klucze nie będą nam
robić standardowe maluśkie pliki wypełnione losowymi danymi. Będą to normalne pliki, np. ulubiona
mp3. TrueCrypt dopuszcza taką możliwość i nie zmienia struktury danych w plikach, zatem możemy
odtwarzać plik mp3 i jednocześnie używać go jako klucza. Trzeba pamiętać, że w przypadku utraty
pliku klucza lub zmiany jakiegokolwiek bitu w pierwszych 1024 KiB, otworzenie kontenera stanie się
niemożliwe, a co się z tym wiąże, wszelkie dane na nim zostaną bezpowrotnie utracone. Hasło z kolei,
służy jako ostatnia deska ratunku, gdyby ktoś przechwycił mp3. Oczywiście ma ono też za zadanie
zmylić potencjalnego napastnika, budując u niego przeświadczenie, że kontener jest tylko na hasło i
odciągnąć go od keyfile. A bez pliku klucza, pozostanie mu tylko łamanie szyfru.

## Problem z obsługą kontenera na dropbox'ie

Kilka uwag co do korzystania z TrueCrypt'a w połączeniu z dropbox'em. Kontener może być zmieniany
tylko przez jedną osobę naraz. Gdy dwie (lub więcej) osoby będą chciały w tym samym czasie
zmodyfikować kontener, dropbox stworzy kilka kopii w zależności od tego ile osób edytowało kontener.
Oczywiście spowoduje to późniejsze pobranie każdej kopi na każdą maszynę, co raczej może wyprowadzić
człowieka z równowagi.

Dropbox traktuje otwarty kontener jako plik w fazie edycji i nie synchronizuje go do momentu jego
zamknięcia. Dlatego też, jeżeli dzielimy z kimś konto, np. w celu przesłania mu zmodyfikowanego
obrazu ISO z linux'em, ważne jest by poinformować drugą stronę, aby odmontowała u siebie kontener,
przynajmniej na czas wgrywania danych. Po wgraniu plików i odmontowaniu kontenera, nastąpi
synchronizacja danych. Gdy druga osoba będzie chciała podmontować volumin u siebie, w chwili gdy ten
się synchronizuje, nie będzie miała takiej możliwości i zostanie o tym fakcie powiadomiona.

Przy usuwaniu danych z dropbox'a, trzeba pamiętać, że struktura kontenera nie ulega zmianie, za
wyjątkiem tylko metadanych systemu plików. Nadpisywanie zerami plików w kontenerze będzie
skutkowało wgrywaniem tak naprawdę losowych danych i one zostaną przesłane przy synchronizacji.
Kontener TrueCrypt'a można wyzerować tylko przez nadpisanie go zerami, omijając oczywiście sam
nagłówek.

## Inne serwisy

Oprócz dropbox'a mamy jeszcze inne usługi: skydrive, google drive i icloud. Nie będę się zbytnio
tutaj rozpisywał na temat konfiguracji wszystkich tych usług, a jedynie spróbuję odpowiedzieć na
pytanie, czy podobny zabieg można przeprowadzić w pozostałych usługach. Na necie możemy wyczytać, że
dropbox, skydrive i google drive dostarczają aplikacje desktopowe dla windowsa i mac'a. Ale tylko
dropbox ma klienta dla linux'a. Bez klienta desktopowego raczej nie ma co podchodzić do mechanizmu
szyfrującego z wykorzystaniem TrueCrypt'a.

Jeśli uważasz, że któryś z moich artykułów jest ciekawy i przydał ci się do czegoś oraz nie masz
konta na dropbox'ie, możesz się zrewanżować zakładając konto z tego linku
[http://db.tt/90XFs5H](https://db.tt/90XFs5H), zawsze mi to doda trochę miejsca (500MiB up to
16GiB).
