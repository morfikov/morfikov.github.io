---
author: Morfik
categories:
- Android
date:     2017-05-02 17:19:22 +0200
lastmod: 2017-05-02 17:19:22 +0200
published: true
status: publish
tags:
- smartfon
- mediatek
- fastboot
- spflashtool
title: Jak sprawdzić czy smartfon z Androidem podlega gwarancji
---

Gdy kupujemy smartfon, to jego producent daje nam na ten sprzęt gwarancję i przez pewien okres czasu
możemy nie martwić się o koszty ewentualnej naprawy. Niestety żyjemy w takich czasach, że sprzęt
elektroniczny potrafi nam paść sam z siebie i to do tego jeszcze w niewyjaśnionych okolicznościach.
Cześć wad jest fabrycznych (zwykle fizycznych) i te z reguły są oczywiste i łatwe do zdiagnozowania
przez support producenta sprzętu, który nabyliśmy. Niemniej jednak, czasami wady są natury czysto
programowej i tu z kolei ustalenie, gdzie dokładnie leży problem może już sprawiać kłopoty. Dlatego
też producenci telefonów zabezpieczają się przed zmianą firmware przez użytkownika. W ten sposób
wgrywając, np. TWRP recovery czy przeprowadzając proces root Androida, zwykle pozbawiamy się
gwarancji i mamy problem, gdy telefon w późniejszym czasie popsuje się nie z naszej winy. Nawet
jeśli wgramy stock'owe oprogramowanie, to i tak producent sprzętu jest w stanie ustalić czy coś w
firmware mieszaliśmy. Zastanawialiście się może zatem skąd producent smartfona wie czy firmware
takiego urządzenia został w jakiś sposób przez nas zmieniony?

<!--more-->
## Sprawdzenie gwarancji

Do końca nie wiem jakie testy support producenta smartfona jest w stanie przeprowadzić po otrzymaniu
od klienta rzekomo wadliwej sztuki sprzętu ale jest jedna opcja, która identyfikuje czy firmware był
w jakiś sposób modyfikowany. Każdy z nas może sobie tę opcję podejrzeć przy pomocy narzędzia
`fastboot` . Przełączmy zatem telefon w tryb bootloader'a i podłączamy go do portu USB. W terminalu
wpisujemy zaś poniższe polecenie:

    # fastboot getvar all
    ...
    (bootloader)    warranty: no
    (bootloader)    unlocked: yes
    (bootloader)    secure: no
    ...

Jak widzimy wyżej, mamy opcję `unlocked` , która odnosi się do zdjętej blokady bootloader'a. Zwykle
smartfon posiada domyślnie zablokowany bootloader i w zasadzie bez zdjęcia tej blokady nic w
systemie nie możemy zmienić. W ten sposób nie możemy wgrać alternatywnego ROM'u czy też trybu
recovery, nie mówiąc już o przeprowadzaniu ryzykownych akcji na jakiejkolwiek części samego firmware
smartfona.

Gdy ściągniemy blokadę z bootloader'a, to automatycznie opcja `secure` jest przepisywana na `no` .
Na podstawie tej opcji Android może nam podczas startu systemu wyświetlić komunikat o tym, że
mechanizmy bezpieczeństwa zostały wyłączone oraz, że korzystanie z tego telefonu stoi pod znakiem
zapytania. Oczywiście nie jest tak źle jak tego typu komunikaty mogłyby wskazywać ale widząc taki
komunikat, zwłaszcza po zakupie telefonu od znajomego, to powinniśmy się zastanowić co z tym faktem
zrobić.

Ostatnią opcją jest zaś `warranty` i w przypadku telefonu z fabrycznym i do tego nie zmienionym
systemem będziemy mieli tutaj `yes` . Proces odblokowania bootloader'a przestawi nam tę opcję na
`no` .

Nawet jeśli nic nie zmienialiśmy w firmware i ostatecznie po zdjęciu blokady się rozmyśliliśmy i
założyliśmy ją z powrotem, to i tak pozycja `warranty` nie wróci nam na `yes` .

    # fastboot getvar all
    ...
    (bootloader)    warranty: no
    (bootloader)    unlocked: no
    (bootloader)    secure: yes
    ...

Zatem jeśli w telefonie ta blokada z bootloader'a została choćby raz zdjęta, to producent smartfona
bez większego trudu jest w stanie ten fakt ustalić i prawdopodobnie nie podejmie się on darmowej
naprawy widząc któryś z powyższych wyników.

## Jak przestawić warranty na "yes"

Ja eksperymentuję dość często (i co raz bardziej odważnie) ze swoimi smartfonami Neffos, które mi
podesłał TP-LINK. Czasami na dłużej lub krócej taki telefon uszkodzę (jedynie programowo) ale w
ostateczności wszystko wraca do normy. Podczas takich testów ciekawe rzeczy wychodzą na jaw,
zwłaszcza, gdy ma się pod ręką [SP Flash Tool][1].

Po ostatnim uszkodzeniu Neffos'a X1 byłem zmuszony do przywrócenia praktycznie całego stock'owego
ROM'u. Może i nie poprawiło to problemu, którego doświadczałem wtedy, ale po wywołaniu jeszcze raz
powyższego polecenia, wynik był nieco inny:

    # fastboot getvar all
    ...
    (bootloader)    warranty: yes
    (bootloader)    unlocked: no
    (bootloader)    secure: yes
    ...

Postanowiłem zatem poszukać partycji, która odpowiada za zapis ustawień dotyczących gwarancji.
Poniżej jest układ flash'a w Neffos X1:

    ~ # sgdisk --print /dev/block/mmcblk0
    Disk /dev/block/mmcblk0: 30535680 sectors, 14.6 GiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): 00000000-0000-0000-0000-000000000000
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 30535646
    Partitions will be aligned on 1-sector boundaries
    Total free space is 9758 sectors (4.8 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1              64           32831   16.0 MiB    0700  recovery
       2           32832           33855   512.0 KiB   0700  para
       3           33856           54335   10.0 MiB    0700  expdb
       4           54336           56383   1024.0 KiB  0700  frp
       5           56384           56895   256.0 KiB   0700  ppl
       6           56896          122431   32.0 MiB    0700  nvdata
       7          122432          187967   32.0 MiB    0700  metadata
       8          187968          204351   8.0 MiB     0700  protect1
       9          204352          229375   12.2 MiB    0700  protect2
      10          229376          229887   256.0 KiB   0700  seccfg
      11          229888          236031   3.0 MiB     0700  proinfo
      12          245760          262143   8.0 MiB     0700  oemkeystore
      13          262144          311295   24.0 MiB    0700  md1img
      14          311296          319487   4.0 MiB     0700  md1dsp
      15          319488          325631   3.0 MiB     0700  md1arm7
      16          325632          335871   5.0 MiB     0700  md3img
      17          335872          346111   5.0 MiB     0700  nvram
      18          346112          348159   1024.0 KiB  0700  lk
      19          348160          350207   1024.0 KiB  0700  lk2
      20          350208          382975   16.0 MiB    0700  boot
      21          382976          399359   8.0 MiB     0700  logo
      22          399360          409599   5.0 MiB     0700  tee1
      23          409600          419839   5.0 MiB     0700  tee2
      24          419840          432127   6.0 MiB     0700  secro
      25          432128          458751   13.0 MiB    0700  keystore
      26          458752         7716863   3.5 GiB     0700  system
      27         7716864         8601599   432.0 MiB   0700  cache
      28         8601600        30502878   10.4 GiB    0700  userdata
      29        30502879        30535646   16.0 MiB    0700  flashinfo

Metodą przywracania kolejnych partycji systemowych via SP Flash Tool (odrzucając te oczywiste),
udało mi się ustalić, że ustawienia dotyczące gwarancji są przechowywane na partycji `seccfg` .

Partycja `seccfg` ma jedynie 256 KiB ale z analizy struktury w edytorze HEX wychodzi, że tylko
pierwsze 60 bajtów jest w jakiś sposób zmienionych.

![](/img/2017/05/001.gwarancja-smartfon-android.png#big)

Co ciekawe, nawet nie posiadając backup'u tej partycji można bez większego problemu przestawić bit
odpowiadający za gwarancję. Wystarczy stworzyć przy pomocy `dd` obraz składający się z samych zer o
długości 256 KiB. Ten obraz następnie wgrywamy na partycję `seccfg` , np. przy pomocy SP Flash Tool
czy też z poziomu TWRP recovery na stosowne urządzenie blokowe. Po tym jak smartfon uruchomi się
ponownie, Android odtworzy partycję `seccfg` i przywróci jej ustawienia domyślne, a w logu
`fastboot` będziemy mieć już informacje, że gwarancja na smartfon obowiązuje. Tego typu zabieg
zakłada także automatycznie blokadę na bootloader.


[1]: http://spflashtool.com/
