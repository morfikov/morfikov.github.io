---
author: Morfik
categories:
- Android
date:    2018-09-22 11:13:00 +0200
lastmod: 2018-09-22 11:13:00 +0200
published: true
status: publish
tags:
- smartfon
- lg
- g4c
- szyfrowanie
- factory-reset
GHissueID: 319
title: Kernel crash przy szyfrowaniu smartfona lub próbie resetu ustawień do fabrycznych
---

Parę dni temu dowiedziałem się o [projekcie /e/][1]. Z racji, że ten ROM jest dostępny na mój
smartfon LG G4C (jeszcze nieoficjalnie), to postanowiłem go sobie wgrać i zobaczyć jak się będzie
sprawował. Podczas testów nowego oprogramowania spróbowałem zaszyfrować partycję `/data/` . Problem
w tym, że po automatycznym zresetowaniu się systemu, urządzenie już nie chciało się uruchomić. Przez
dłuższy czas widniało logo LG, a po chwili pojawił się czarny ekran z informacją "Kernel Crash" lub
niebieski ekran z informacją "Subsystem Crash". Czy telefon w takiej sytuacji nadaje się jedynie do
wyrzucenia?

<!--more-->
## Kernel Crash, Subsystem Crash i TWRP recovery

Kernel Crash w przypadku tego smartfona LG G4C przytrafił mi się w zasadzie już drugi raz.
Pierwszym razem przy próbie zresetowania urządzenia do ustawień fabrycznych (factory reset) z
poziomu działającego systemu (LineageOS). Tym razem, Kernel Crash pojawił się przy próbie
zaszyfrowania smartfona. To co łączy te dwa przypadki, to tryb recovery. Na partycji `/recovery/`
mam wgrany obraz TWRP i to on jest prawdopodobnie winny, że telefon łapie crash przy
przeprowadzaniu tego tupu operacji. Pewności nie mam ale po przywróceniu stock'owego trybu recovery
(wgrywając go via `dd` przez TWRP recovery), Kernel Crash już nie występuje, a telefon jest w
stanie się już uruchomić.

Poniżej znajdują się fotki z objawami problemu:

![lg-g4c-logo-stuck](/img/2018/09/1.lg-g4c-logo-stuck.jpg#huge)

![lg-g4c-kernel-crash](/img/2018/09/2.lg-g4c-kernel-crash.jpg#huge)

![lg-g4c-subsystem-crash](/img/2018/09/3.lg-g4c-subsystem-crash.jpg#huge)

To czy dostaniemy komunikat z Kernel Crash czy Subsystem Crash zależy od tego czy będziemy się
starać przeczyścić telefon z poziomu TWRP recovery. Jeśli tak, to dostaniemy Subsystem Crash, w
przeciwnym wypadku Kernel Crash. W obu jednak przypadkach telefon nie będzie nam chciał wystartować.

## Stock'owy obraz partycji /recovery/

Te powyższe błędy można poprawić wgrywając stock'owy obraz na partycję `/recovery/` , oczywiście
jeśli nim dysponujemy, a z tym bywa różnie. Trzeba też pamiętać, że w tego typu sytuacjach, gdy nam
nie startuje system, przywracanie oryginalnego trybu recovery nie zawsze jest najlepszym pomysłem.
Co w przypadku, gdy po przywróceniu stock'owego trybu recovery system nam nie będzie chciał z
jakiegoś innego powodu wystartować?

Podczas bawienia się ROM'em /e/ , deweloper co ten ROM zbudował na mój model smartfona zapomniał w
nim wrzucić aplikacji launcher'a. Bez tej aplikacji nie można wejść w opcje systemu, a bez tego nie
da rady włączyć debugowania via ADB. Bez ADB nie da rady ponownie wgrać TWRP na partycję
`/recovery/` i robi się problem. Jeśli jednak jesteśmy pewni, że nasz system działał przed Kernel
Crash i mogliśmy na nim uzyskać prawa root, to wgranie stock'owego trybu recovery, by poprawić
zaistniałą sytuację, może być w miarę dobrym rozwiązaniem. Wciąż jednak trzeba się mieć na
baczności, by czasem przez przypadek nie pomylić partycji i by nie wgrać się na tę, na którą nie
powinniśmy.

## Partycja /misc/

Szukając informacji na temat tego jak wybrnąć z tej dość patowej sytuacji (przynajmniej w moim
przypadku), natrafiłem na to poniższe polecenie:

    # dd if=/dev/zero of=/dev/block/bootdevice/by-name/misc count=1 bs=32

Poszukałem zatem informacji na temat tego polecenia. [Znalazłem taki oto artykuł][2]. Możemy w nim
wyczytać, że:

> If the standard AOSP recovery image is being used, during boot the bootloader should read the
> first 32 bytes on the misc partition and boot into the recovery image if the data there matches:
> "boot-recovery" This allows any pending recovery work (e.g. applying an OTA, performing data
> removal, etc.) to resume until being finished successfully.

Czyli, na tych pierwszych 32 bajtach jest zapisywana informacja i system podczas startu wie czy
jakieś zaplanowane zadania zostały do przeprowadzenia w trybie recovery. Mając custom recovery (np.
TWRP), coś najwyraźniej poszło nie tak. Niemniej jednak, wyczyszczenie tych paru bajtów wydaje się
sensowne.

Na forum XDA jest cała masa wątków, w których ludziom udało się odblokować telefon wpisując to
polecenie przez ADB via TWRP recovery. U mnie ono nie przyniosło większych efektów. Próbowałem
również wydać to polecenie w kombinacji z czyszczeniem cache i dalvik ale i to nie poprawiło
problemu. Próbowałem też wyczyścić wszystkie partycje dostępne w menu TWRP za wyjątkiem partycji
`/system/` ale to też nic nie dało.

## Jak poprawić Kernel Crash/Subsystem Crash w LG G4C

To co poprawiło ten cały Kernel Crash/Subsystem Crash w przypadku mojego LG G4C, to uruchomienie
telefonu w trybie TWRP recovery, sformatowanie partycji `/data/` nowym systemem plików
(Wipe -> Format Data) i restart urządzenia. Kluczowe jest tutaj wybranie opcji formatowania
partycji, a nie tak jak zwykle się korzysta z "Advanced Wipe" czy też zwyczajnie przeciąga suwak
"Swipe to Factory Reset".


[1]: https://e.foundation/
[2]: https://source.android.com/devices/bootloader/flashing-updating
