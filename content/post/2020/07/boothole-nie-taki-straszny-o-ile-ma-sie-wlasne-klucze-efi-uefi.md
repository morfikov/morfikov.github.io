---
author: Morfik
categories:
- Linux
date: "2020-07-31T21:07:00Z"
published: true
status: publish
tags:
- debian
- efi
- uefi
GHissueID: 12
title: BootHole nie taki straszny, o ile ma się własne klucze EFI/UEFI
---

Dnia 29-07-2020 do publicznej wiadomości zostały podane informacje na temat podatności BootHole,
która to za sprawą bootloader'a GRUB2 w różnych dystrybucjach linux'a jest w stanie obejść
mechanizm bezpieczeństwa EFI/UEFI, tj. Secure Boot. [Z informacji][2], które opublikował Debian,
sprawa nie wygląda miło, jako że poza aktualizacją GRUB2, shim, jądra linux, Fwupdate oraz Fwupd,
unieważnieniu podlegają również klucze dystrybucji Debian/Ubuntu, przez co praktycznie cały soft
podpisany tymi kluczami (w tym systemy live) przestaną działać w trybie Secure Boot. Czy jest się
czego obawiać i co użytkownik korzystający z mechanizmu SB powinien w takiej sytuacji zrobić?

<!--more-->
## Co to ten cały BootHole

Zgodnie z informacjami jakie zostały [opublikowane przez Eclypsium][1], podatność BootHole daje
możliwość atakującemu wykonanie dowolnego kodu podczas startu systemu w skutek przepełnienia bufora
w GRUB2. Bez znaczenia przy tym jest fakt czy Secure Boot jest włączony oraz czy użytkownik
korzysta z GRUB2 jako bootloader, czy też używa innego oprogramowania np. [refind'a][3]). Niemniej
jednak, do wykorzystania tej podatności potrzebny jest albo fizyczny dostęp do sprzętu, albo też
uzyskanie praw administratora systemu, co może ździebko poprawić nam humor.

W skrócie, winny jest plik konfiguracyjny `grub.cfg` , który to zawiera ustawienia dla samego GRUB2
wliczając w to, który kernel załadować do pamięci RAM oraz jakie opcje podać temu kernelowi.
W przypadku, gdy dojdzie do przepełnienia bufora, to atakujący jest w stanie zmodyfikować kod GRUB2,
który został już załadowany do pamięci operacyjnej (a wcześniej jego sygnatura została
zweryfikowana przez Secure Boot) i wykonać przez to dowolny kod obchodząc tym samym cały mechanizm
bezpieczeństwa EFI/UEFI. Co ciekawe, tego typu atak standardowo zostałby przez Secure Boot wykryty.
Niemniej jednak, Secure Boot nie weryfikuje sygnatur w stosunku do tekstowych plików
konfiguracyjnych -- jedynie pliki binarne muszą być podpisane i to sygnatury złożone pod nimi są
weryfikowane. W taki sposób można dowolnie zmieniać plik `grub.cfg` , co nie powoduje problemów z
Secure Boot, np. w przypadku aktualizacji systemu.

Z racji, że GRUB2 jest podpisany, to stosowny certyfikat musi figurować w firmware EFI/UEFI, by
umożliwić start systemu w trybie Secure Boot. Praktycznie każda podpisana wersja GRUB2 jest podatna
na BootHole (oczywiście za wyjątkiem tej ostatnio zaktualizowanej, tj.  `2.02+dfsg1-20+deb10u1` ),
co sprawia, że każdy system z dowolną dystrybucją linux'a jest podatny na zagrożenie jakie niesie
ze sobą BootHole.

Nawet jeśli zaktualizujemy GRUB2, to w dalszym ciągu ktoś może nam podmienić binarkę GRUB2 na jej
starszą wersje. A biorąc pod uwagę fakt, że dalej mamy w firmware EFI/UEFI certyfikat, którym ta
podatna binarka może zostać zweryfikowana (zwykle cert używanej przez nas dystrybucji linux'a), to
w zasadzie ta aktualizacja na nic nie wpływa w kwestii podatności na BootHole.

Póki co, klucze Debiana i Ubuntu trafiły na czarną listę do bazy `dbx` , przez co te dwie
dystrybucje zostały zmuszone do wygenerowania i korzystania z nowych kluczy, o czym można
przeczytać w oświadczeniu [Debiana][4] i [Ubuntu][5].

Nawet jeśli [zastąpiliśmy klucze firmware EFI/UEFI swoimi własnymi][9], to i tak możemy
nieświadomie podpisać te podatne wersje GRUB2 (mając nieaktualny system) i umożliwić BootHole siać
spustoszenie w naszym linux'ie.

Problem także dotyczy Windows'ów korzystających z Secure Boot. Chodzi o dual/multiboot,
umożliwiający uruchamianie na jednej maszynie zarówno windows'a jak i linux'a. Generalnie to w
firmware każdego laptopa czy desktopa od blisko dekady jest zaszyty certyfikat `Microsoft Third
Party UEFI Certificate Authority` . Klucz prywatny odpowiadający temu certyfikatowi jest
wykorzystywany m.in. do podpisywania binarek shim. Binarki shim z kolei dostarczają certyfikaty
różnych dystrybucji linux'a -- m.in. te, które trafiły na czarną listę. Idąc dalej, certyfikaty
dystrybucji linux'a są wykorzystywane do weryfikacji podpisów złożonych kluczami prywatnymi od tych
certów i tak dochodzimy do podpisanych wadliwych binarek GRUB2 kluczami prywatnymi różnych
dystrybucji linux'a. W ten sposób, ogromna liczba domowych stacji roboczych i serwerów ma poważny
problem, a z nimi ten problem mają również ich użytkownicy.

## Jak załatać BootHole

Tej podatności BootHole nie da się załatać od razu. Nawet jeśli wszystkie wymienione wyżej binarki
zostaną zaktualizowane na docelowym systemie, to i tak problem z uruchomieniem trefnej binarki
GRUB2 pozostaje. Kompletne załatanie BootHole wiąże się z aktualizacją bazy `dbx` , która jest
obecna w firmware EFI/UEFI. Stosowna [aktualizacja tej bazy została już opublikowana][6] ale póki
co jest ona jeszcze w fazie testów. Chodzi generalnie o to, że gdyby ta aktualizacja została
zaaplikowana od razu dla wszystkich maszyn chwilę po publikacji zaistniałego problemu, to
prawdopodobnie popsułaby ona całą masę instalacji, co niezbyt by ucieszyło użytkowników. Dlatego
też aktualizację tej bazy trzeba na razie dokonać we własnym zakresie. Już nawet [pojawiają się
pierwsze problemy związane z aktualizacją GRUB2][10].

Plik `dbxupdate_x64.bin` jest podpisany kluczem `KEK` Microsoft'u, przez co zaktualizowanie bazy
danych `dbx` na przeciętnej maszynie powinno być w miarę łatwe i automatyczne. Problem jednak
zaczyna się w momencie, gdy nie mamy zaszytych w firmware EFI/UEFI certyfikatów MS (np. mamy
wspomniane wyżej własne klucze). W takiej sytuacji, nie damy rady od razu wgrać pliku aktualizacji
dostępnego w powyższym linku. Problemem jest właśnie ta sygnatura, która została złożona pod plikiem
`dbxupdate_x64.bin` . By ten plik zaimportować do bazy `dbx` , [trzeba pierw pozbyć się tej
sygnatury][7]. Można to zrobić w poniższy sposób:

    # dd if=dbxupdate_x64.bin of=dbxupdate_x64.esl bs=1 skip=3349
    11064+0 records in
    11064+0 records out
    11064 bytes (11 kB, 11 KiB) copied, 0.0921443 s, 120 kB/s

Dlaczego trzeba obciąć akurat `3349` bajtów i czy w przypadku każdej wersji tego pliku trzeba
będzie pozbyć się akurat tego kawałka? Tego póki co jeszcze nie udało mi się ustalić ale to
wymaga głębszego zbadania.

Tak otrzymany plik trzeba podpisać przy pomocy naszego klucza `KEK` :

    # sign-efi-sig-list -k KEK.key -c KEK.crt dbx dbxupdate_x64.esl dbxupdate_x64.auth
    Timestamp is 2020-7-30 23:06:36
    Authentication Payload size 11106
    Signature of size 1223
    Signature at: 40

Mając podpisany plik aktualizacji, możemy go wgrać do bazy `dbx` via `efi-updatevar` :

    # chattr -i /sys/firmware/efi/efivars/dbx-d719b2cb-3d3a-4596-a3bc-dad00e67656f
    # efi-updatevar -f dbxupdate_x64.auth dbx

Na koniec sprawdźmy jak wygląda zawartość zmiennej `dbx` :

    # efi-readvar -v dbx
    Variable dbx, length 11064
    dbx: List 0, type X509
        Signature 0, size 1076, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                C=GB, ST=Isle of Man, O=Canonical Ltd., OU=Secure Boot, CN=Canonical Ltd. Secure Boot Signing
            Issuer:
                C=GB, ST=Isle of Man, L=Douglas, O=Canonical Ltd., CN=Canonical Ltd. Master Certificate Authority
    dbx: List 1, type X509
        Signature 0, size 784, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Subject:
                CN=Debian Secure Boot Signer
            Issuer:
                CN=Debian Secure Boot CA
    dbx: List 2, type SHA256
        Signature 0, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:80b4d96931bf0d02fd91a61e19d14f1da452e66db2408ca8604d411f92659f0a
        Signature 1, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f52f83a3fa9cfbd6920f722824dbe4034534d25b8507246b3b957dac6e1bce7a
        Signature 2, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c5d9d8a186e2c82d09afaa2a6f7f2e73870d3e64f72c4e08ef67796a840f0fbd
        Signature 3, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1aec84b84b6c65a51220a9be7181965230210d62d6d33c48999c6b295a2b0a06
        Signature 4, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c3a99a460da464a057c3586d83cef5f4ae08b7103979ed8932742df0ed530c66
        Signature 5, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:58fb941aef95a25943b3fb5f2510a0df3fe44c58c95e0ab80487297568ab9771
        Signature 6, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5391c3a2fb112102a6aa1edc25ae77e19f5d6f09cd09eeb2509922bfcd5992ea
        Signature 7, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d626157e1d6a718bc124ab8da27cbb65072ca03a7b6b257dbdcbbd60f65ef3d1
        Signature 8, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d063ec28f67eba53f1642dbf7dff33c6a32add869f6013fe162e2c32f1cbe56d
        Signature 9, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:29c6eb52b43c3aa18b2cd8ed6ea8607cef3cfae1bafe1165755cf2e614844a44
        Signature 10, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:90fbe70e69d633408d3e170c6832dbb2d209e0272527dfb63d49d29572a6f44c
        Signature 11, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:106faceacfecfd4e303b74f480a08098e2d0802b936f8ec774ce21f31686689c
        Signature 12, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:174e3a0b5b43c6a607bbd3404f05341e3dcf396267ce94f8b50e2e23a9da920c
        Signature 13, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:2b99cf26422e92fe365fbf4bc30d27086c9ee14b7a6fff44fb2f6b9001699939
        Signature 14, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:2e70916786a6f773511fa7181fab0f1d70b557c6322ea923b2a8d3b92b51af7d
        Signature 15, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3fce9b9fdf3ef09d5452b0f95ee481c2b7f06d743a737971558e70136ace3e73
        Signature 16, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:47cc086127e2069a86e03a6bef2cd410f8c55a6d6bdb362168c31b2ce32a5adf
        Signature 17, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:71f2906fd222497e54a34662ab2497fcc81020770ff51368e9e3d9bfcbfd6375
        Signature 18, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:82db3bceb4f60843ce9d97c3d187cd9b5941cd3de8100e586f2bda5637575f67
        Signature 19, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:8ad64859f195b5f58dafaa940b6a6167acd67a886e8f469364177221c55945b9
        Signature 20, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:8d8ea289cfe70a1c07ab7365cb28ee51edd33cf2506de888fbadd60ebf80481c
        Signature 21, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:aeebae3151271273ed95aa2e671139ed31a98567303a332298f83709a9d55aa1
        Signature 22, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c409bdac4775add8db92aa22b5b718fb8c94a1462c1fe9a416b95d8a3388c2fc
        Signature 23, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c617c1a8b1ee2a811c28b5a81b4c83d7c98b5b0c27281d610207ebe692c2967f
        Signature 24, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c90f336617b8e7f983975413c997f10b73eb267fd8a10cb9e3bdbfc667abdb8b
        Signature 25, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:64575bd912789a2e14ad56f6341f52af6bf80cf94400785975e9f04e2d64d745
        Signature 26, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:45c7c8ae750acfbb48fc37527d6412dd644daed8913ccd8a24c94d856967df8e
        Signature 27, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:81d8fb4c9e2e7a8225656b4b8273b7cba4b03ef2e9eb20e0a0291624eca1ba86
        Signature 28, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:b92af298dc08049b78c77492d6551b710cd72aada3d77be54609e43278ef6e4d
        Signature 29, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e19dae83c02e6f281358d4ebd11d7723b4f5ea0e357907d5443decc5f93c1e9d
        Signature 30, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:39dbc2288ef44b5f95332cb777e31103e840dba680634aa806f5c9b100061802
        Signature 31, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:32f5940ca29dd812a2c145e6fc89646628ffcc7c7a42cae512337d8d29c40bbd
        Signature 32, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:10d45fcba396aef3153ee8f6ecae58afe8476a280a2026fc71f6217dcf49ba2f
        Signature 33, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:4b8668a5d465bcdd9000aa8dfcff42044fcbd0aece32fc7011a83e9160e89f09
        Signature 34, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:89f3d1f6e485c334cd059d0995e3cdfdc00571b1849854847a44dc5548e2dcfb
        Signature 35, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c9ec350406f26e559affb4030de2ebde5435054c35a998605b8fcf04972d8d55
        Signature 36, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:b3e506340fbf6b5786973393079f24b66ba46507e35e911db0362a2acde97049
        Signature 37, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9f1863ed5717c394b42ef10a6607b144a65ba11fb6579df94b8eb2f0c4cd60c1
        Signature 38, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:dd59af56084406e38c63fbe0850f30a0cd1277462a2192590fb05bc259e61273
        Signature 39, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:dbaf9e056d3d5b38b68553304abc88827ebc00f80cb9c7e197cdbc5822cd316c
        Signature 40, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:65f3c0a01b8402d362b9722e98f75e5e991e6c186e934f7b2b2e6be6dec800ec
        Signature 41, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5b248e913d71853d3da5aedd8d9a4bc57a917126573817fb5fcb2d86a2f1c886
        Signature 42, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:2679650fe341f2cf1ea883460b3556aaaf77a70d6b8dc484c9301d1b746cf7b5
        Signature 43, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:bb1dd16d530008636f232303a7a86f3dff969f848815c0574b12c2d787fec93f
        Signature 44, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0ce02100f67c7ef85f4eed368f02bf7092380a3c23ca91fd7f19430d94b00c19
        Signature 45, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:95049f0e4137c790b0d2767195e56f73807d123adcf8f6e7bf2d4d991d305f89
        Signature 46, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:02e6216acaef6401401fa555ecbed940b1a5f2569aed92956137ae58482ef1b7
        Signature 47, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:6efefe0b5b01478b7b944c10d3a8aca2cca4208888e2059f8a06cb5824d7bab0
        Signature 48, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9d00ae4cd47a41c783dc48f342c076c2c16f3413f4d2df50d181ca3bb5ad859d
        Signature 49, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d8d4e6ddf6e42d74a6a536ea62fd1217e4290b145c9e5c3695a31b42efb5f5a4
        Signature 50, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f277af4f9bdc918ae89fa35cc1b34e34984c04ae9765322c3cb049574d36509c
        Signature 51, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0dc24c75eb1aef56b9f13ab9de60e2eca1c4510034e290bbb36cf60a549b234c
        Signature 52, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:835881f2a5572d7059b5c8635018552892e945626f115fc9ca07acf7bde857a4
        Signature 53, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:badff5e4f0fea711701ca8fb22e4c43821e31e210cf52d1d4f74dd50f1d039bc
        Signature 54, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c452ab846073df5ace25cca64d6b7a09d906308a1a65eb5240e3c4ebcaa9cc0c
        Signature 55, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f1863ec8b7f43f94ad14fb0b8b4a69497a8c65ecbc2a55e0bb420e772b8cdc91
        Signature 56, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:7bc9cb5463ce0f011fb5085eb8ba77d1acd283c43f4a57603cc113f22cebc579
        Signature 57, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e800395dbe0e045781e8005178b4baf5a257f06e159121a67c595f6ae22506fd
        Signature 58, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1cb4dccaf2c812cfa7b4938e1371fe2b96910fe407216fd95428672d6c7e7316
        Signature 59, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3ece27cbb3ec4438cce523b927c4f05fdc5c593a3766db984c5e437a3ff6a16b
        Signature 60, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:68ee4632c7be1c66c83e89dd93eaee1294159abf45b4c2c72d7dc7499aa2a043
        Signature 61, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e24b315a551671483d8b9073b32de11b4de1eb2eab211afd2d9c319ff55e08d0
        Signature 62, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e7c20b3ab481ec885501eca5293781d84b5a1ac24f88266b5270e7ecb4aa2538
        Signature 63, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:7eac80a915c84cd4afec638904d94eb168a8557951a4d539b0713028552b6b8c
        Signature 64, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e7681f153121ea1e67f74bbcb0cdc5e502702c1b8cc55fb65d702dfba948b5f4
        Signature 65, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:dccc3ce1c00ee4b0b10487d372a0fa47f5c26f57a359be7b27801e144eacbac4
        Signature 66, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0257ff710f2a16e489b37493c07604a7cda96129d8a8fd68d2b6af633904315d
        Signature 67, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3a91f0f9e5287fa2994c7d930b2c1a5ee14ce8e1c8304ae495adc58cc4453c0c
        Signature 68, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:495300790e6c9bf2510daba59db3d57e9d2b85d7d7640434ec75baa3851c74e5
        Signature 69, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:81a8b2c9751aeb1faba7dbde5ee9691dc0eaee2a31c38b1491a8146756a6b770
        Signature 70, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:8e53efdc15f852cee5a6e92931bc42e6163cd30ff649cca7e87252c3a459960b
        Signature 71, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:992d359aa7a5f789d268b94c11b9485a6b1ce64362b0edb4441ccc187c39647b
        Signature 72, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9fa4d5023fd43ecaff4200ba7e8d4353259d2b7e5e72b5096eff8027d66d1043
        Signature 73, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d372c0d0f4fdc9f52e9e1f23fc56ee72414a17f350d0cea6c26a35a6c3217a13
        Signature 74, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5c5805196a85e93789457017d4f9eb6828b97c41cb9ba6d3dc1fcc115f527a55
        Signature 75, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:804e354c6368bb27a90fae8e498a57052b293418259a019c4f53a2007254490f
        Signature 76, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:03f64a29948a88beffdb035e0b09a7370ccf0cd9ce6bcf8e640c2107318fab87
        Signature 77, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:05d87e15713454616f5b0ed7849ab5c1712ab84f02349478ec2a38f970c01489
        Signature 78, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:06eb5badd26e4fae65f9a42358deef7c18e52cc05fbb7fc76776e69d1b982a14
        Signature 79, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:08bb2289e9e91b4d20ff3f1562516ab07e979b2c6cefe2ab70c6dfc1199f8da5
        Signature 80, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0928f0408bf725e61d67d87138a8eebc52962d2847f16e3587163b160e41b6ad
        Signature 81, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:09f98aa90f85198c0d73f89ba77e87ec6f596c491350fb8f8bba80a62fbb914b
        Signature 82, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0a75ea0b1d70eaa4d3f374246db54fc7b43e7f596a353309b9c36b4fd975725e
        Signature 83, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0c51d7906fc4931149765da88682426b2cfe9e6aa4f27253eab400111432e3a7
        Signature 84, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:0fa3a29ad05130d7fe5bf4d2596563cded1d874096aacc181069932a2e49519a
        Signature 85, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:147730b42f11fe493fe902b6251e97cd2b6f34d36af59330f11d02a42f940d07
        Signature 86, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:148fe18f715a9fcfe1a444ce0fff7f85869eb422330dc04b314c0f295d6da79e
        Signature 87, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1b909115a8d473e51328a87823bd621ce655dfae54fa2bfa72fdc0298611d6b8
        Signature 88, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1d8b58c1fdb8da8b33ccee1e5f973af734d90ef317e33f5db1573c2ba088a80c
        Signature 89, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1f179186efdf5ef2de018245ba0eae8134868601ba0d35ff3d9865c1537ced93
        Signature 90, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:270c84b29d86f16312b06aaae4ebb8dff8de7d080d825b8839ff1766274eff47
        Signature 91, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:29cca4544ea330d61591c784695c149c6b040022ac7b5b89cbd72800d10840ea
        Signature 92, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:2b2298eaa26b9dc4a4558ae92e7bb0e4f85cf34bf848fdf636c0c11fbec49897
        Signature 93, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:2dcf8e8d817023d1e8e1451a3d68d6ec30d9bed94cbcb87f19ddc1cc0116ac1a
        Signature 94, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:311a2ac55b50c09b30b3cc93b994a119153eeeac54ef892fc447bbbd96101aa1
        Signature 95, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:32ad3296829bc46dcfac5eddcb9dbf2c1eed5c11f83b2210cf9c6e60c798d4a7
        Signature 96, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:340da32b58331c8e2b561baf300ca9dfd6b91cd2270ee0e2a34958b1c6259e85
        Signature 97, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:362ed31d20b1e00392281231a96f0a0acfde02618953e695c9ef2eb0bac37550
        Signature 98, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:367a31e5838831ad2c074647886a6cdff217e6b1ba910bff85dc7a87ae9b5e98
        Signature 99, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3765d769c05bf98b427b3511903b2137e8a49b6f859d0af159ed6a86786aa634
        Signature 100, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:386d695cdf2d4576e01bcaccf5e49e78da51af9955c0b8fa7606373b007994b3
        Signature 101, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3a4f74beafae2b9383ad8215d233a6cf3d057fb3c7e213e897beef4255faee9d
        Signature 102, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3ae76c45ca70e9180c1559981f42622dd251bca1fbe6b901c52ec11673b03514
        Signature 103, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3be8e7eb348d35c1928f19c769846788991641d1f6cf09514ca10269934f7359
        Signature 104, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:3e3926f0b8a15ad5a14167bb647a843c3d4321e35dbc44dce8c837417f2d28b0
        Signature 105, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:400ac66d59b7b094a9e30b01a6bd013aff1d30570f83e7592f421dbe5ff4ba8f
        Signature 106, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:4185821f6dab5ba8347b78a22b5f9a0a7570ca5c93a74d478a793d83bac49805
        Signature 107, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:41d1eeb177c0324e17dd6557f384e532de0cf51a019a446b01efb351bc259d77
        Signature 108, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:45876b4dd861d45b3a94800774027a5db45a48b2a729410908b6412f8a87e95d
        Signature 109, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:4667bf250cd7c1a06b8474c613cdb1df648a7f58736fbf57d05d6f755dab67f4
        Signature 110, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:47ff1b63b140b6fc04ed79131331e651da5b2e2f170f5daef4153dc2fbc532b1
        Signature 111, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:47ff1b63b140b6fc04ed79131331e651da5b2e2f170f5daef4153dc2fbc532b1
        Signature 112, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5391c3a2fb112102a6aa1edc25ae77e19f5d6f09cd09eeb2509922bfcd5992ea
        Signature 113, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:57e6913afacc5222bd76cdaf31f8ed88895464255374ef097a82d7f59ad39596
        Signature 114, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5890fa227121c76d90ed9e63c87e3a6533eea0f6f0a1a23f1fc445139bc6bcdf
        Signature 115, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:5d1e9acbbb4a7d024b6852df025970e2ced66ff622ee019cd0ed7fd841ccad02
        Signature 116, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:61341e07697978220ea61e85dcd2421343f2c1bf35cc5b8d0ad2f0226f391479
        Signature 117, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:61cec4a377bf5902c0feaee37034bf97d5bc6e0615e23a1cdfbae6e3f5fb3cfd
        Signature 118, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:631f0857b41845362c90c6980b4b10c4b628e23dbe24b6e96c128ae3dcb0d5ac
        Signature 119, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:65b2e7cc18d903c331df1152df73ca0dc932d29f17997481c56f3087b2dd3147
        Signature 120, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:66aa13a0edc219384d9c425d3927e6ed4a5d1940c5e7cd4dac88f5770103f2f1
        Signature 121, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:6873d2f61c29bd52e954eeff5977aa8367439997811a62ff212c948133c68d97
        Signature 122, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:6dbbead23e8c860cf8b47f74fbfca5204de3e28b881313bb1d1eccdc4747934e
        Signature 123, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:6dead13257dfc3ccc6a4b37016ba91755fe9e0ec1f415030942e5abc47f07c88
        Signature 124, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:70a1450af2ad395569ad0afeb1d9c125324ee90aec39c258880134d4892d51ab
        Signature 125, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:72c26f827ceb92989798961bc6ae748d141e05d3ebcfb65d9041b266c920be82
        Signature 126, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:781764102188a8b4b173d4a8f5ec94d828647156097f99357a581e624b377509
        Signature 127, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:788383a4c733bb87d2bf51673dc73e92df15ab7d51dc715627ae77686d8d23bc
        Signature 128, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:78b4edcaabc8d9093e20e217802caeb4f09e23a3394c4acc6e87e8f35395310f
        Signature 129, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:7f49ccb309323b1c7ab11c93c955b8c744f0a2b75c311f495e18906070500027
        Signature 130, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:80b4d96931bf0d02fd91a61e19d14f1da452e66db2408ca8604d411f92659f0a
        Signature 131, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:82acba48d5236ccff7659afc14594dee902bd6082ef1a30a0b9b508628cf34f4
        Signature 132, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:894d7839368f3298cc915ae8742ef330d7a26699f459478cf22c2b6bb2850166
        Signature 133, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:8c0349d708571ae5aa21c11363482332073297d868f29058916529efc520ef70
        Signature 134, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:8d93d60c691959651476e5dc464be12a85fa5280b6f524d4a1c3fcc9d048cfad
        Signature 135, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9063f5fbc5e57ab6de6c9488146020e172b176d5ab57d4c89f0f600e17fe2de2
        Signature 136, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:91656aa4ef493b3824a0b7263248e4e2d657a5c8488d880cb65b01730932fb53
        Signature 137, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:91971c1497bf8e5bc68439acc48d63ebb8faabfd764dcbe82f3ba977cac8cf6a
        Signature 138, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:947078f97c6196968c3ae99c9a5d58667e86882cf6c8c9d58967a496bb7af43c
        Signature 139, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:96e4509450d380dac362ff8e295589128a1f1ce55885d20d89c27ba2a9d00909
        Signature 140, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9783b5ee4492e9e891c655f1f48035959dad453c0e623af0fe7bf2c0a57885e3
        Signature 141, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:97a51a094444620df38cd8c6512cac909a75fd437ae1e4d22929807661238127
        Signature 142, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:97a8c5ba11d61fefbb5d6a05da4e15ba472dc4c6cd4972fc1a035de321342fe4
        Signature 143, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:992820e6ec8c41daae4bd8ab48f58268e943a670d35ca5e2bdcd3e7c4c94a072
        Signature 144, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:992d359aa7a5f789d268b94c11b9485a6b1ce64362b0edb4441ccc187c39647b
        Signature 145, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9954a1a99d55e8b189ab1bca414b91f6a017191f6c40a86b6f3ef368dd860031
        Signature 146, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9baf4f76d76bf5d6a897bfbd5f429ba14d04e08b48c3ee8d76930a828fff3891
        Signature 147, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9c259fcb301d5fc7397ed5759963e0ef6b36e42057fd73046e6bd08b149f751c
        Signature 148, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9dd2dcb72f5e741627f2e9e03ab18503a3403cf6a904a479a4db05d97e2250a9
        Signature 149, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:9ed33f0fbc180bc032f8909ca2c4ab3418edc33a45a50d2521a3b5876aa3ea2c
        Signature 150, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:a4d978b7c4bda15435d508f8b9592ec2a5adfb12ea7bad146a35ecb53094642f
        Signature 151, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:a924d3cad6da42b7399b96a095a06f18f6b1aba5b873b0d5f3a0ee2173b48b6c
        Signature 152, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:ad3be589c0474e97de5bb2bf33534948b76bb80376dfdc58b1fed767b5a15bfc
        Signature 153, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:b8d6b5e7857b45830e017c7be3d856adeb97c7290eb0665a3d473a4beb51dcf3
        Signature 154, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:b93f0699598f8b20fa0dacc12cfcfc1f2568793f6e779e04795e6d7c22530f75
        Signature 155, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:bb01da0333bb639c7e1c806db0561dc98a5316f22fef1090fb8d0be46dae499a
        Signature 156, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:bc75f910ff320f5cb5999e66bbd4034f4ae537a42fdfef35161c5348e366e216
        Signature 157, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:bdd01126e9d85710d3fe75af1cc1702a29f081b4f6fdf6a2b2135c0297a9cec5
        Signature 158, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:be435df7cd28aa2a7c8db4fc8173475b77e5abf392f76b7c76fa3f698cb71a9a
        Signature 159, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:bef7663be5ea4dbfd8686e24701e036f4c03fb7fcd67a6c566ed94ce09c44470
        Signature 160, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c2469759c1947e14f4b65f72a9f5b3af8b6f6e727b68bb0d91385cbf42176a8a
        Signature 161, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c3505bf3ec10a51dace417c76b8bd10939a065d1f34e75b8a3065ee31cc69b96
        Signature 162, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c42d11c70ccf5e8cf3fb91fdf21d884021ad836ca68adf2cbb7995c10bf588d4
        Signature 163, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c452ab846073df5ace25cca64d6b7a09d906308a1a65eb5240e3c4ebcaa9cc0c
        Signature 164, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c69d64a5b839e41ba16742527e17056a18ce3c276fd26e34901a1bc7d0e32219
        Signature 165, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:cb340011afeb0d74c4a588b36ebaa441961608e8d2fa80dca8c13872c850796b
        Signature 166, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:cc8eec6eb9212cbf897a5ace7e8abeece1079f1a6def0a789591cb1547f1f084
        Signature 167, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:cf13a243c1cd2e3c8ceb7e70100387cecbfb830525bbf9d0b70c79adf3e84128
        Signature 168, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d89a11d16c488dd4fbbc541d4b07faf8670d660994488fe54b1fbff2704e4288
        Signature 169, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:d9668ab52785086786c134b5e4bddbf72452813b6973229ab92aa1a54d201bf5
        Signature 170, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:da3560fd0c32b54c83d4f2ff869003d2089369acf2c89608f8afa7436bfa4655
        Signature 171, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:df02aab48387a9e1d4c65228089cb6abe196c8f4b396c7e4bbc395de136977f6
        Signature 172, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:df91ac85a94fcd0cfb8155bd7cbefaac14b8c5ee7397fe2cc85984459e2ea14e
        Signature 173, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e051b788ecbaeda53046c70e6af6058f95222c046157b8c4c1b9c2cfc65f46e5
        Signature 174, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e051b788ecbaeda53046c70e6af6058f95222c046157b8c4c1b9c2cfc65f46e5
        Signature 175, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e36dfc719d2114c2e39aea88849e2845ab326f6f7fe74e0e539b7e54d81f3631
        Signature 176, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e39891f48bbcc593b8ed86ce82ce666fc1145b9fcbfd2b07bad0a89bf4c7bfbf
        Signature 177, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:e6856f137f79992dc94fa2f43297ec32d2d9a76f7be66114c6a13efc3bcdf5c8
        Signature 178, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:eaff8c85c208ba4d5b6b8046f5d6081747d779bada7768e649d047ff9b1f660c
        Signature 179, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:ee83a566496109a74f6ac6e410df00bb29a290e0021516ae3b8a23288e7e2e72
        Signature 180, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:eed7e0eff2ed559e2a79ee361f9962af3b1e999131e30bb7fd07546fae0a7267
        Signature 181, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f1b4f6513b0d544a688d13adc291efa8c59f420ca5dcb23e0b5a06fa7e0d083d
        Signature 182, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f2a16d35b554694187a70d40ca682959f4f35c2ce0eab8fd64f7ac2ab9f5c24a
        Signature 183, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f31fd461c5e99510403fc97c1da2d8a9cbe270597d32badf8fd66b77495f8d94
        Signature 184, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:f48e6dd8718e953b60a24f2cbea60a9521deae67db25425b7d3ace3c517dd9b7
        Signature 185, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:c805603c4fa038776e42f263c604b49d96840322e1922d5606a9b0bbb5bffe6f
        Signature 186, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:1f16078cce009df62edb9e7170e66caae670bce71b8f92d38280c56aa372031d
        Signature 187, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:37a480374daf6202ce790c318a2bb8aa3797311261160a8e30558b7dea78c7a6
        Signature 188, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:408b8b3df5abb043521a493525023175ab1261b1de21064d6bf247ce142153b9
        Signature 189, size 48, owner 77fa9abd-0359-4d32-bd60-28f4e78f784b
            Hash:540801dd345dc1c33ef431b35bf4c0e68bd319b577b9abe1a9cff1cbc39f548f

Jak widać, mamy tutaj 190 hash'y oraz dwa certyfikaty Debiana i Ubuntu. Żaden soft, który został
podpisany kluczami prywatnymi od tych certyfikatów, po zaaplikowaniu tej aktualizacji już nam się
nie uruchomi w trybie Secure Boot. W taki sposób nie zadziałają nam praktycznie żadne systemy live,
choć [Debian już zapowiedział aktualizację swoich obrazów][8] przy okazji wypuszczenia stabilnego
wydania 10.5. Także nie ma się co martwić, że nie będziemy mieli jak odratować swojego systemu, gdy
coś mu się złego przytrafi. Zawsze też można ten cały mechanizm Secure Boot wyłączyć na czas prac
ratowniczych.

## Podsumowanie

Stworzenie własnych kluczy do podpisu oprogramowania i umieszczenie certyfikatów w firmware
EFI/UEFI po uprzednim usunięciu starych, dostarczanych ze sprzętem kluczy w dość prosty sposób jest
nas w stanie zabezpieczyć przed podatnościami takimi jak BootHole. Warto tutaj dodać, że tego typu
sytuacje, czy chcemy czy nie, będą mieć miejsce w przyszłości, a mając w firmware własne
certyfikaty, te późniejsze problemy również nas nie będą dotyczyć. Jeśli faktycznie zależy nam na
bezpieczeństwie naszego linux'a, to powinniśmy przestać ufać zewnętrznym podmiotom podpisującym
oprogramowanie, które instalujemy na swoich maszynach. Nie chodzi tutaj o całkowitą utratę zaufania
do podmiotów typu Debian czy Ubuntu ale widać na żywym przykładzie z jakimi problemami takie
zaufanie się może wiązać. Jeśli pozwalamy organizacjom trzecim na kontrolowanie tego jakie binarki
firmware naszego komputera jest w stanie uruchomić, to prędzej czy później skończy się to dla nas
nie najlepiej. Warto jednak mieć aktualną bazę sygnatur `dbx` , tak byśmy przez przypadek nie
uruchomili binarek, których hash'e w tej bazie są obecne.


[1]: https://eclypsium.com/2020/07/29/theres-a-hole-in-the-boot/
[2]: https://www.debian.org/security/2020-GRUB-UEFI-SecureBoot/
[3]: https://www.rodsbooks.com/refind/
[4]: https://www.debian.org/security/2020-GRUB-UEFI-SecureBoot/#revocations
[5]: https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/GRUB2SecureBootBypass#DBX_Update
[6]: https://uefi.org/revocationlistfile
[7]: https://unix.stackexchange.com/questions/601093/
[8]: https://www.debian.org/security/2020-GRUB-UEFI-SecureBoot/#buster_point_release
[9]: /post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/
[10]: https://bugzilla.redhat.com/show_bug.cgi?id=1861977
