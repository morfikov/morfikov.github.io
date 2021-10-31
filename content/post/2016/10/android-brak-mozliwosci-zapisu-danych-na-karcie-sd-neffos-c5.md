---
author: Morfik
categories:
- Android
date:    2016-10-14 20:56:20 +0200
lastmod: 2016-10-14 20:56:20 +0200
published: true
status: publish
tags:
- karta-sd
- smartfon
- root
GHissueID: 449
title: 'Android: Brak możliwości zapisu danych na karcie SD w Neffos C5'
---

Może i obecne smartfony dają nam w standardzie sporo wolnego miejsca na swoim flash'u ale dla
niektórych to ciągle za mało. Nie ważne ile tej pamięci będziemy mieć dostępnej, to i tak zawsze
będzie nam jej brakować. Mój [Neffos C5][1] ma na pokładzie 16 GiB flash, z czego około 10 GiB jest
udostępnione użytkownikowi. Mi by się przydał flash 64 GiB. Jako, że ten telefon obsługuje karty
SDHC, max 32 GiB, to postanowiłem dokupić tego typu kartę i zamontować ją w smartfonie. Problem
pojawił się w momencie próby przeniesienia danych aplikacji z pamięci wewnętrznej na pamięć
zewnętrzną jaką jest karta SD. Chodzi na przykład o zapisywanie zdjęć czy filmów z kamery
bezpośrednio na karcie SD. Okazuje się jednak, że [Android począwszy od wersji 4.4][2] zablokował
możliwość umieszczania danych aplikacji na karatach SD. Czy słusznie i czy istnieje jakiś sposób
by wybrnąć z tej sytuacji?

<!--more-->
## Prywatność i bezpieczeństwo danych na karcie SD

Wygląda na to, że cały problem z możliwością zapisu kart SD przez aplikacje wziął się z winy systemu
plików FAT, którym jest taka karta sformatowana. Ten system plików nie wspiera uprawnień, tak jak to
ma, np. miejsce w linux'owym systemie plików EXT4. W efekcie jeśli aplikacje mają możliwość zapisu
danych na karcie SD, to każda z nich jest w stanie odczytać informacje zapisane przez sąsiadujące
programy, a to nie jest już w porządku. Dlatego Android zablokował taką możliwość.

Oczywiście w dalszym ciągu jesteśmy w stanie zapisywać kartę SD ręcznie przez kopiowanie danych z
pamięci telefonu na kartę SD. Jest to jednak czasochłonne, mało wygodne i raczej niezbyt praktyczne
rozwiązanie. Jakby nie patrzeć trzeba dokonać dwukrotnego zapisu danych, raz na flash'u telefonu, a
drugi raz na karcie SD, a to pociąga za sobą większą utylizację zasobów.

## Sformatowanie karty SD jako pamięć wewnętrzna

Jednym z rozwiązań zaistniałego problemu może być sformatowanie karty SD jako pamięć wewnętrzna. Ten
pomysł [został mi podsunięty przez użytkownika @szopen][3] ale jak zawsze nie obyło się bez
problemów. Okazuje się, że opcja takiego formatowania karty jest dostępna jedynie w wybranych
modelach smartfonów. Z nieznanych mi przyczyn, TP-LINK nie umieścił takiego ficzera w Neffos'ie C5.
Stoją za tym jakieś argumenty?

## Sformatowanie karty SD z wykorzystaniem systemu plików EXT4

My linux'iarze zdajemy sobie sprawę z problemu praw dostępu do plików, bo bezpieczeństwo naszych
systemów operacyjnych jest właśnie o takie prawa oparte. Każdy plik musi mieć swojego właściciela
oraz grupę i odpowiednie prawa dla zapisu, odczytu czy wykonywania. To nic nowego dla osób
korzystających z alternatywnych systemów operacyjnych.

Przyglądając się bliżej Androidowi, a konkretnie systemowi plików jego partycji, możemy zauważyć, że
on również wykorzystuje system plików EXT4. Poniżej fotka:

![neffos-c5-karta-sd-system-plikow-ext4-android](/img/2016/10/001.neffos-c5-karta-sd-system-plikow-ext4-android.png#medium)

Niby nic wielkiego, podłączam zatem kartę SD do mojego Debiana i odpalam `gparted` . Tam zaś
formatuję kartę z wykorzystaniem systemu plików EXT4:

![neffos-c5-karta-sd-linux-format-ext4-gparted](/img/2016/10/002.neffos-c5-karta-sd-linux-format-ext4-gparted.png#huge)

Wsadzam tak sformatowaną kartę do telefonu, uruchamiam system i co widzą moje oczyska?

![neffos-c5-karta-sd-android-blad-uszkodzona-karta](/img/2016/10/003.neffos-c5-karta-sd-android-blad-uszkodzona-karta.png#big)

Jak to jest możliwe, że linux jakim jest Android, mający systemowe partycje sformatowane na EXT4, ma
problemy z czytaniem karty SD, która również ma taki sam system plików? :D Mając root na moim
Neffos'ie C5, postanowiłem, że sprawdzę, czy da radę ręcznie zamontować tę kartę w systemie. I co
się okazało?

![neffos-c5-karta-sd-reczne-zamontowanie-karty-mount-root](/img/2016/10/004.neffos-c5-karta-sd-reczne-zamontowanie-karty-mount-root.png#huge)

Oczywiście kartę SD można zamontować i jest ona widoczna w systemie ale Androidowe aplikacje tego
faktu zdają się nie zauważać.

![neffos-c5-karta-sd-problem-wykrycie-karty](/img/2016/10/005.neffos-c5-karta-sd-problem-wykrycie-karty.png#medium)

## Partycjonowanie karty SD z poziomu Android'a

Wygląda na to, że nie damy rady zrobić użytku z karty SD w sposób taki jak by tego oczekiwał
przeciętny linux'iarz. Karta będzie formatowana domyślnie na FAT, który ma całą masę ograniczeń i
podatności. Na necie można spotkać się z kilkoma sposobami obejścia tego problemu.

Jeśli zrobiliśmy root na naszym Neffos C5, to możemy spróbować [sformatować tę kartę SD z poziomu
Androida][4]. Problem z tym rozwiązaniem jest taki, że karta przestanie nam działać po przeniesieniu
do czytnika kart SD w komputerze. Więc albo będziemy korzystać z niej tylko i wyłącznie na
smartfonie albo musimy korzystać z systemu pliku FAT i zaakceptować problemy z nim związane.

W przypadku Neffos'a C5 sformatowanie karty SD z poziomu Androida nie pomaga. Czytając ten wyżej
podlinkowany watek, znalazłem tam informację, która mówi, że winny jest tutaj moduł FUSE, który w
stock'owym ROM'ie nie obsługuje systemu plików EXT4. W efekcie Android nie będzie w stanie
zamontować karty SD posiadającej na partycji taki system plików. Jedyna alternatywa to wgranie
niestandardowego ROM'u, np. [CyanogenMod][5]. Jak to zrobić wychodzi poza ramy tego artykułu i w
przypadku Neffos'a C5 nie obejdziemy tutaj niedogodności związanych z systemem plików FAT. Trzeba
zatem powrócić do głównego problemu jakim jest umożliwienie aplikacjom zapisu danych na karcie SD.

## Zdjęcie restrykcji co do zapisu danych na karcie SD

Ostatnią rzeczą, której możemy się podjąć jest zdjęcie restrykcji dotyczących zapisu danych przez
aplikacje na karcie SD. Krótko mówiąc, wyłączenie całego mechanizmu bezpieczeństwa i umożliwienie
każdej aplikacji na czytanie/zapisanie/zmianę wszystkich danych na karcie SD. Krok niezbyt rozważny
ale czego to się nie robi by poznać inne systemy operacyjne?

Do zdjęcia blokady będą nam potrzebne prawa administracyjne. Dlatego też upewnijmy się, że nasz
telefon ma root'a przed próbą podjęcia poniższych działań. Odpalamy zatem terminal logujemy się na
root'a i montujemy partycję `/system/` w trybie do zapisu:

    $ su
    # mount -o remount,rw /system

Następnie przy pomocy `vi` edytujemy plik `/system/etc/permissions/platform.xml` . Musimy w nim
odszukać `WRITE_EXTERNAL_STORAGE` i dopisać w nim grupę `media_rw` :

![neffos-c5-karta-sd-zmiana-uprawnien-aplikacji](/img/2016/10/006.neffos-c5-karta-sd-zmiana-uprawnien-aplikacji.png#huge)

Zapisujemy zmiany i restartujemy smartfona. Odpalamy teraz aplikację, która ma mieć możliwość
zapisywania karty SD (w tym przypadku jest to oprogramowanie aparatu), zmieńmy ścieżkę zapisu fotek
na kartę SD i przetestujemy czy wszystko działa w porządku:

![neffos-c5-karta-sd-test-zapisu-karty-kamera-aparat](/img/2016/10/007.neffos-c5-karta-sd-test-zapisu-karty-kamera-aparat.png#big)

Jak widać fotka została zapisana w folderze na karcie SD.

Wyłączenie tych zabezpieczeń rozwiązuje też problem przy wgrywaniu plików z komputera bezpośrednio
na kartę SD w telefonie. Niemniej jednak, szybkość zapisu karty spadła mi z 8,5 MiB/s do około 3
MiB/s.

## Aplikacja SDFIX

Jeśli chcielibyśmy zautomatyzować cały proces związany z modyfikacją uprawnień, to istnieje
[dedykowana aplikacja SDFIX][6], która pomoże nam tę kwestię ogarnąć.

![neffos-c5-karta-sd-sdfix-instalacja](/img/2016/10/008.neffos-c5-karta-sd-sdfix-instalacja.png#huge)

Teraz już wystarczy tylko odpalić program. Jego obsługa jest banalna i sprowadza się do zaznaczenia
jednej opcji, za której sprawą SDFIX poprosi nas o prawa root w celu zmiany piku
`/system/etc/permissions/platform.xml` :

![neffos-c5-karta-sd-sdfix-root-](/img/2016/10/009.neffos-c5-karta-sd-sdfix-root-.png#huge)

Aplikacja SDFIX tworzy backup pliku `/system/etc/permissions/platform.xml` przed dokonaniem w nim
zmian (nazwa backupu to `platform.xml.original-pre-sdfix` ). Gdy będziemy chcieli powrócić do
ustawień systemowych, wystarczy odinstalować ten program i przywrócić ww. plik backupu.


[1]: http://www.neffos.pl/product/details/C5
[2]: https://developer.android.com/about/versions/android-4.4.html
[3]: http://tplink-forum.pl/index.php?/topic/5307-kopiowanie-danych-na-kart%C4%99-sd-przez-usb-na-neffos-c5/#comment-45611
[4]: https://forum.xda-developers.com/galaxy-s3/general/tutorial-howto-convert-external-sd-card-t2480963
[5]: https://www.cyanogenmod.org/
[6]: https://forum.xda-developers.com/showthread.php?t=2684188
