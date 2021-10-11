---
author: Morfik
categories:
- Android
date:    2017-01-10 18:24:47 +0100
lastmod: 2017-01-10 18:24:47 +0100
published: true
status: publish
tags:
- smartfon
- root
GHissueID: 69
title: Root Integrity Check w smartfonach z Androidem
---

W smartfonach Neffos od TP-LINK, którymi mam możliwość się bawić, standardowo jest dostępny tryb
recovery, a telefon można uruchomić w tym trybie przez przyciśnięcie przycisków VolumeUP + Power. W
zasadzie jest to jeden z podstawowych trybów pracy smartfona, który może nam pomóc, gdy mamy
problemy z uruchomieniem urządzenia. Zwykle w trybie recovery przeprowadza się takie czynności jak
czyszczenie partycji `/cache/` i `/data/` (Factory Reset). Z poziomu trybu recovery jesteśmy także w
stanie przeprowadzić aktualizację firmware (tego oprogramowania, które zarządza naszym telefonem). W
tym artykule jednak nie będziemy dokonywać żadnych z tych powyżej opisanych czynności. W menu trybu
recovery jest jeszcze jedna ciekawa opcja, tj. Root Integrity Check. Do czego ona służy i jak
interpretować wynik skanowania?

<!--more-->
## Jak zainicjować Root Integrity Check

Po moich przygodach z przeprowadzaniem procesu root na Neffos'ach [C5][1], [C5 MAX][2], [Y5][3] i
[Y5L][4], każde z tych urządzeń trzeba było powrócić do stanu fabrycznego, tj. cofnąć wszelkie
wprowadzone w nich zmiany. W przypadku modelu Neffos Y5 miałem pewne problemy, bo coś musiałem
zrobić nie tak jak trzeba i nie mogłem odinstalować SuperSU. Do tej pory nie wiem co zrobiłem źle
ale grunt, że zmiany udało się ostatecznie cofnąć. W zasadzie, to tylko przypadek sprawił, że się
zainteresowałem opcją Root Integrity Check w menu recovery. Jedyne co chciałem zrobić pod kątem
partycji `/recovery/` , to sprawdzić czy przywrócenie oryginalnej partycji zakończyło się
powodzeniem. No i jak można zobaczyć na poniższej fotce, opcja Root Integrity Check jest zaraz obok
Power Off, którą to chciałem zaznaczyć. Wyszło tak, że zaznaczyłem tę niżej -- zwykły missclick.

![](/img/2017/01/001.root-integrity-check-smartfon-android-inicjacja-procesu.jpg#big)

Po aktywowaniu tej opcji rozpoczął się bliżej nie znany mi proces skanowania. Trwał on dłuższą
chwilę, a jako że ja nie lubię przerywać rozpoczętych rzeczy, to poczekałem aż ten proces
dobiegnie końca.

![](/img/2017/01/002.root-integrity-check-smartfon-android-skanowanie.jpg#big)

Ostatecznie proces zakończył się błędami. Z początku nie miałem pojęcia co jest nie tak. Niby
smartfona odratowałem i nie wykazywał on żadnych problemów w działaniu ale ten proces Root Integrity
Check nie chciał zakończyć się powodzeniem. Jakby nie patrzeć nazwa tego skanowania jest
jednoznaczna, choć nie do końca jasne jest do czego system się odwołuje przy zwracaniu wyniku.

## Root Integrity Check i partycje /system/, /recovery/ oraz /boot/

Szukając informacji na temat tego całego Root Integrity Check, w Google znalazłem coś na temat "OTA
update", czyli o aktualizacji firmware Over-The-Air. Okazuje się, że w przypadku, gdy to skanowanie
zwróci błąd integralności, aktualizacja firmware może albo nie być możliwa, albo może też uwalić nam
telefon. W sumie to nigdy nie przeprowadzałem aktualizacji firmware smartfona z poziomu trybu
recovery i dokładnie nie wiem jak ten proces przebiega i czy ten Root Integrity Check jest przed
aktualizacją inicjowany ale lepiej dmuchać na zimne i dbać o to, by ten proces zakończył się
sukcesem. Do czego zatem system referuje, gdy zwracany jest wynik skanowania Root Integrity Check?
Generalnie rzecz biorąc, w grę wchodzą trzy składowe.

Pierwszą z nich jest system plików na partycji `/system/` . Jeśli jakiś plik na tej partycji został
usunięty, dodany lub też zmieniony, proces zwróci błąd i wypisze na ekranie problematyczne pliki.
Poniżej przykład:

![](/img/2017/01/003.root-integrity-check-smartfon-android-blad-system.jpg#big)

Tutaj został dodany tylko jeden plik i jak możemy odczytać jest to `/system/etc/resolv.conf` .
Zwykle odtworzenie stock'owego obrazu partycji `/system/` poprawi ten błąd. Jeśli zaś nie mamy jak
odtworzyć oryginalnej partycji, to wtedy mamy problem. Można oczywiście utworzyć obraz tej partycji
i zamontować go na dowolnej dystrybucji linux'a. Wtedy mamy szansę pokasować nowe pliki lub cofnąć
zmiany w tych, które zmienialiśmy ręcznie. Niemniej jednak, w przypadku zmian dokonanych przez
zewnętrzne aplikacje, możemy zwyczajnie nie być w stanie odtworzyć wszystkich plików. Dlatego
właśnie ja zawsze staram się przeprowadzić backup całego flash'a telefonu, bo kto wie z jakimi
problemami przyjdzie mi się mierzyć w przyszłości.

Drugim elementem składowym jest suma kontrolna/ID partycji `/recovery/` . Tę partycję zwykle
zmieniamy wgrywając obraz TWRP podczas procesu root'owania Androida. Oczywiście niekoniecznie musimy
ukorzeniać swój system, możemy jedynie chcieć mieć nieco bardziej zaawansowany tryb recovery, który
oferuje TWRP, np. na potrzeby backup'u. Niemniej jednak, posiadanie takiej niestandardowej partycji
`/recovery/` może uniemożliwić nam dokonanie aktualizacji firmware z poziomu działającego systemu.

Trzecią składową jest suma kontrolna/ID partycji `/boot/` . Ta partycja jest zmieniana zwykle przy
wgrywaniu SuperSU. Różnie to wygląda w przypadku innych modeli smartfonów. Dla przykładu, SuperSU
przy root'owaniu Neffos'a C5 i C5 MAX nie tykał tej partycji zupełnie i nie trzeba było jej odtwarzać
przy cofaniu zmian. Natomiast w przypadku modeli Y5 i Y5L, SuperSU już jakieś czary na partycji
`/boot/` odprawiał. Okazało się, że może i przywróciłem partycję `/system/` i `/recovery/` do stanu
fabrycznego podczas procesu unroot ale najwyraźniej SuperSU namieszał coś w partycji `/boot/` i nie
posprzątał po sobie należycie. W efekcie partycja `/boot/` została zmieniona i Root Integrity Check
wyrzucał błąd, bo był w stanie wykryć właśnie te zmiany.

![](/img/2017/01/004.root-integrity-check-smartfon-android-blad-boot.jpg#big)

By Root Integrity Check zakończył się powodzeniem, te trzy partycje muszą pochodzić ze stock'owego
firmware. W zasadzie ten powyższy błąd nie przeszkadza w użytkowaniu telefonu ale należy liczyć się
z problemami podczas ewentualnych aktualizacji oferowanych przez producenta smartfona. Po
przywróceniu wszystkich trzech partycji z backup'u flash'a, Root Integrity Check przeszedł bez
problemu i zakończył się powodzeniem:

![](/img/2017/01/005.root-integrity-check-smartfon-android-sukces.jpg#big)


[1]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[2]: /post/android-root-smartfona-neffos-c5-max-od-tp-link/
[3]: /post/android-root-smartfona-neffos-y5-od-tp-link/
[4]: /post/android-root-smartfona-neffos-y5l-tp-link/
