---
author: Morfik
categories:
- Linux
date: "2015-05-09T17:21:30Z"
date_gmt: 2015-05-09 15:21:30 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- xserver
- sysctl
title: Bezpieczeństwo Xserver'a pod linux'em
---

Prawdopodobnie każdy z nas korzysta, lub też i korzystał, ze skrótu klawiszy Ctrl-Alt-Backspace do
resetowania środowiska graficznego na linux'ie. Może i to jest wygodny sposób na szybki restart
[sesji graficznej Xserver'a]({{< baseurl >}}/post/konfiguracja-xservera-na-debianie-xorg/) ale
[nie do końca jest on bezpieczny](https://www.jwz.org/xscreensaver/faq.html#no-ctl-alt-bs). Nie
mówię tutaj o samej utracie danych towarzyszącej takiej akcji ale o możliwości obejścia ekranu
logowania o ile ta kombinacja klawiszy nie jest wyłączona. Jest też kilka innych rzeczy, które mogą
skompromitować źle zabezpieczony Xserver i tej kwestii będzie poświęcony niniejszy wpis.

<!--more-->
## Resetowanie Xserver'a

Na pierwszy ogień weźmy sobie wspomnianą kombinację Crtl-Alt-Backspace . W jaki sposób może ona
zagrozić bezpieczeństwu systemu spytacie? Bynajmniej nie chodzi mi o manualne odpalanie sesji
graficznych via `startx` , choć tu też jest pewien problem, bo po zresetowaniu takiej sesji,
zostaniemy zrzuceni do shell'a na TTY i będziemy mieć dostęp do konta użytkownika.

Groźniejsze jest jednak automatyczne logowanie użytkowników. Załóżmy, że mamy w pełni zaszyfrowany
system, odbezpieczamy kontener podając hasło chwilę po tym jak okno bootloader'a zniknie, po czym
już nie podajemy hasła do konta użytkownika i ten jest automatycznie logowany przez jakiś menadżer
logowania, np. LightDM. Jeśli odchodzimy od maszyny, możemy zablokować ekran, podobnie w przypadku
hibernacji/uśpienia komputera. Jak tylko powrócimy, to by uzyskać dostęp do systemu, musimy się
pierw zalogować. Pozornie nigdzie tutaj nie ma żadnego problemu ale co się stanie w przypadku gdy
mając zablokowany ekran wciśniemy Crtl-Alt-Backspace ? Nastąpi reset graficznej sesji, a że
korzystaliśmy z automatycznego logowania, zostaniemy natychmiast zalogowani na konto użytkownika,
który jest zdefiniowany w konfiguracji menadżera logowania. W ten sposób nie tylko obchodzimy
blokadę ekranu ale również i pełne szyfrowanie dysku i wszystkie zabezpieczenia szlag trafia.

Nie jestem fanem automatycznego logowania do systemu, choć ma to i swoje zalety, niemniej jednak,
jestem za bezwzględnym wyłączeniem tej kombinacji klawiszy. Jeśli naprawdę chcemy już resetować
sobie graficzną sesję i korzystamy przy tym jedynie z openbox'a, to ustawmy sobie jakiś konkretną
kombinację klawiszy, by wywoływała ona polecenie `openbox --exit` . Efekt będzie dokładnie taki sam
i nie naruszymy przy tym bezpieczeństwa systemu.

By wyłączyć skrót Crtl-Alt-Backspace , usuwamy z konfiguracji Xserver'a opcję
`terminate:ctrl_alt_bksp` . Pamiętajmy by przejrzeć katalog `/etc/X11/xorg.conf.d/` oraz plik
`/etc/default/keyboard` .

Samo usunięcie powyższego parametru, nas tak naprawdę przed niczym nie chroni. Chyba każde
środowisko graficzne zezwala na dodanie/usunięcie tej opcji z konfiguracji Xserver'a. By się
faktycznie zabezpieczyć przed tego typu kompromitacją, musimy zablokować globalnie możliwość
zresetowania Xserver'a. Robimy to przez pliki w katalogu `/etc/X11/xorg.conf.d/` . Przykładowo,
tworzymy plik `90-serverlayout.conf` i dopisujemy w nim poniższy kod:

    Section "ServerLayout"
          Identifier "Main"
          Screen      0 "Screen0"
    EndSection

    Section "ServerFlags"
          Option "DontZap" "true"
    EndSection

Parametr `DontZap` uniemożliwi zresetowanie sesji graficznej nawet jeśli w opcjach klawiatury
będziemy mieć zdefiniowany `terminate:ctrl_alt_bksp` .

## Zmiana terminala

Warto jedynie wspomnieć, że czasem można zapomnieć o zablokowaniu graficznej sesji przed
przełączeniem się na jedną z konsol TTY. W takim przypadku ciągle mamy dostęp do środowiska
graficznego, nawet po wylogowaniu się z TTY. Istnieje, co prawda, parametr `DontVTSwitch` , który
można podać Xserver'owi ale odradzam korzystanie z niego. Zamiast tego dużo rozważniejsza będzie
zwykła uwaga. Natomiast jeśli ktoś chciałby dodać i ten parametr do konfiguracji Xserver'a, to do
pliku `90-serverlayout.conf` niech sobie doda poniższą linijkę:

    Option "DontVTSwitch" "true"

## SysRq i OOM-killer

OOM to skrót od Out Of Memory, a killer, to raczej wszyscy wiedzą co oznacza. Razem zaś tworzą
mechanizm obrony przed żarłocznymi procesami, które mają niesamowity apetyt na zasoby systemowe.
Jeśli taki proces by się pojawił, zostanie zabity, by system mógł działać bez problemu. Nie zawsze
jednak zabijany jest ten proces, który my byśmy wybrali z naszej ludzkiej perspektywy i w przypadku
maszyn, te mogą zabić nie ten proces co trzeba, co może, np. zdjąć blokadę ekranu. By się
zabezpieczyć i na tę ewentualność, musimy określić, które akcje mogą być przeprowadzane za sprawą
[klawisza SysRq]({{< baseurl >}}/post/aktywacja-i-konfiguracja-klawisza-sysrq/).
