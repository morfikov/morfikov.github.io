---
author: Morfik
categories:
- Linux
date: "2019-09-16T18:20:15Z"
published: true
status: publish
tags:
- apparmor
- capability
- dac_read_search
- dac_override
GHissueID: 283
title: Capability dac_read_search i dac_override w profilu AppArmor'a
---

Od jakiegoś czasu tworzę dla aplikacji w moim Debianie [profile pod AppArmor][1], tak by ograniczyć
programom swobodny dostęp do plików czy urządzeń. Tych profili zebrało się już trochę i podczas
pisania jednego z nich, zacząłem się zastanawiać czy wszystkie CAP'y (linux capabilities), których
żądają procesy, są im faktycznie niezbędne do prawidłowego funkcjonowania. Chodzi póki co o
`dac_read_search` i `dac_override` . Co ciekawe, odmowa `dac_override` w części aplikacji nie
powodowała żadnych negatywnych konsekwencji. Idąc dalej tym tropem, postanowiłem z paru profili AA
usunąć linijkę zawierającą `capability dac_override,` zostawiając tym samym jedynie
`capability dac_read_search,`  i zobaczyć co się stanie. Okazało się, że sporo programów już o
`dac_override` nie prosi. Zatem co się zmieniło przez tych parę lat i czy ta zmiana dotyczy samych
aplikacji, a może kernela linux'a?

<!--more-->
## DAC i CAP

Każdy z nas wie raczej czym są uprawnienia do plików w linux, oraz zdaje sobie sprawę, że
użytkownicy w systemie niekoniecznie mają wgląd w każdą jego część. To, czy konkretny użytkownik
będzie mógł odczytać, zapisać lub też wykonać dany plik zależy od
[uprawnień jakie zostały nadane][2] na ten plik dla użytkownika, grupy i innych user'ów w systemie.
Wyjątkiem jest użytkownik `root` . On teoretycznie ma wgląd w każde miejsce w systemie i może
podglądać pliki innych użytkowników, nawet jeśli te mają wyraźnie ustawione uprawnienia typu
`0400` .

By procesy działające z uprawnieniami administratora systemu root nieco powściągnąć od zaglądania w
dowolne miejsce, stworzono dodatkowe uprawnienia, czyli capabilities. Przeciętny użytkownik linux'a
nie dostrzeże tej zmiany, bo po zalogowaniu się na root'a będzie miał on i tak dostęp do każdego
pliku w systemie, podobnie zresztą jak i same aplikacje. Wyjątkiem jest sytuacja, gdy
wykorzystywany jest jakiś dodatkowy moduł bezpieczeństwa w stosunku do tradycyjnego Unix'owego DAC
(Discretionary Access Control), a takim jest np. AppArmor czy SELinux.

## Różnica w CAP dac_read_search i dac_override

Gdy w grę wchodzi AppArmor lub SELinux, to trzeba liczyć się z tymi dodatkowymi
uprawnieniami/możliwościami, jakie oferuje już od dłuższego czasu linux'owy kernel. W skrócie,
uprawnienia root zostały rozbite na 64 części, z których póki co [około 40 jest w użytku][3]. Dwa z
nich to `dac_read_search` i `dac_override` .

Celem `dac_read_search` jest umożliwienie procesowi root obejście uprawnień odczytu (read) do
pliku/katalogu oraz wykonania (execute) ale tylko w stosunku do katalogu. Po nadaniu tego CAP'a,
proces root będzie w stanie odczytać plik oraz otworzyć folder, do którego nie ma uprawnień. Z
kolei (jak się można już łatwo domyśleć) `dac_override` ma za zadanie umożliwić obejście wszystkich
uprawnień do pliku/katalogu, czyli umożliwi procesowi root odczyt, zapis i wykonanie w stosunku do
nieswoich plików i katalogów. Jak widać ten drugi CAP jest o wiele potężniejszy.

## Problem w kernelu linux'a dotyczący dac_override

We wstępie wspomniałem, że szereg aplikacji w moim Debianie może działać bez `dac_override` , gdy
nada mu się jedynie `dac_read_search` . To powinno być raczej oczywiste, bo gdy proces z
uprawnieniami root chce jedynie odczytać plik lub katalog, do którego nie ma uprawnień, to powinien
skorzystać z `dac_read_search` , a nie z `dac_override` .

Problemem z żądaniem `dac_override` do samego odczytu plików tkwił w kernelu linux'a. Szukając
informacji na ten temat, natrafiłem na [ten oto artykuł][4]. Z niego z kolei dowiedziałem się , że
miała miejsce jakaś bliżej nieokreślona zmiana w kernelu, która się dokonała też nie wiadomo kiedy
dokładnie. Postanowiłem zatem poszukać co, kiedy i gdzie zmieniono. Przy pomocy serwisu
[elixir.bootlin.com][5] udało mi się odszukać odwołanie do `dac_override` i natrafiłem tam na plik
`fs/namei.c` . Szukając jeszcze trochę, trafiłem w końcu na [tego patch'a][6].

Zgodnie z tym co zostało napisane na podlinkowanym wyżej ML, sprawdzenie `dac_read_search` i
`dac_override` w nowszych kernelach (v4.12+) odbywa się w innej kolejności niż to miało miejsce w
starszych wydaniach. W starszych kernelach, najpierw był sprawdzany `dac_override` , a potem
`dac_read_search` , a obecnie jest odwrotnie. W ten sposób proces root dostaje nieco mniej
uprawnień w przypadku, gdy żąda jedynie odczytu nieswoich plików. Ta zmiana weszła w życie około
2017-07-12. Jest zatem niemal pewne, że profile AppArmor'a, które były pisane wcześniej i chciały
`dac_override` , obecnie powinny bez problemu działać z `dac_read_search` . Oczywiście w dalszym
ciągu znajdą się procesy, które będą chcieć `dac_override` i trzeba będzie je ręcznie zweryfikować.

W taki oto sposób z 54 profili AppArmor'a, w których był obecny `dac_override` , byłem w stanie się
go pozbyć w przypadku 25. Czyli prawie połowa aplikacji miała ten CAP nadany niepotrzebnie.

Warto też zaznaczyć, że AppArmor czy SELinux nie będzie prosił o `dac_read_search` czy też
`dac_override` w sytuacji, gdy uprawnienia do plików będą odpowiednie. Lepiej jest zadbać o to, by
plik/katalog, do którego proces z uprawnieniami root chce uzyskać dostęp, miał grupę `root` i w
niej stosowne uprawnienia, niż nadawać któryś z tych dwóch CAP'ów. Oczywiście nie zawsze będzie
można to uczynić i w pewnych sytuacjach AppArmor/SELinux będzie nas prosił o `dac_read_search`
lub/i `dac_override` i jeśli ich nie przyznamy, to nasza aplikacja będzie mieć defekty w działaniu.



[1]: https://gitlab.com/morfikov/debian-files/tree/master/configs/etc/apparmor.d
[2]: https://en.wikipedia.org/wiki/File_system_permissions#Notation_of_traditional_Unix_permissions
[3]: http://man7.org/linux/man-pages/man7/capabilities.7.html
[4]: https://danwalsh.livejournal.com/77140.html
[5]: https://elixir.bootlin.com/linux/v5.3/ident
[6]: http://kernsec.org/pipermail/linux-security-module-archive/2017-March/000029.html
