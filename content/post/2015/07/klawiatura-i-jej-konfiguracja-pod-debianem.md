---
author: Morfik
categories:
- Linux
date: "2015-07-15T01:21:22Z"
date_gmt: 2015-07-14 23:21:22 +0200
published: true
status: publish
tags:
- xserver
- initramfs
- locale
- klawiatura
title: Klawiatura i jej konfiguracja pod Debianem
---

W środowiskach graficznych, np. GNOME czy KDE, nie musimy zbytnio się zastanawiać nad tym jak
skonfigurować klawiaturę, bo wszystko możemy sobie szybko i w prosty sposób wyklikać z graficznego
panelu administracyjnego systemu. Natomiast jeśli korzystamy jedynie z odchudzonych instalacji
linux'a zawierających jedynie jakiś menadżer okien, np. OPENBOX, to sami musimy zadbać o
skonfigurowanie klawiatury, tak by układ się zgadzał, by były dostępne polskie znaki, no i
oczywiście by system potrafił rozpoznać ewentualne [klawisze multimedialne][1].

<!--more-->
## Klawiatura pod Xserver'em

Na Debianie klawiaturę można konfigurować w kilku miejscach. Jedną z opcji jest stworzenie
odpowiednich plików konfiguracyjnych dla Xserver'a. Niemniej jednak, trzeba pamiętać, że środowiska
graficzne kompletnie ignorują te ustawienia. Dla przykładu, stwórzmy plik Xorga w
`/etc/X11/xorg.conf.d/10-keyboard.conf` z konfiguracją klawiatury podobną do tej poniżej:

    Section "InputClass"
    Identifier              "Logitech Media Keyboard Elite"
    MatchIsKeyboard         "on"
    MatchDevicePath         "/dev/input/event*"
    Driver                  "evdev"
    Option                  "XkbModel"      "logimel"
    Option                  "XkbLayout"     "pl"
    #Option                 "XkbVariant"    ""
    Option                  "XkbOptions"    "kpdl:dot,lv3:ralt_switch,compose:rctrl,terminate:ctrl_alt_bksp,grp:alt_shift_toggle,grp_led:scroll"
    EndSection

Jeśli mamy inną klawiaturę, musimy dowiedzieć się jakie wartości powpisywać do powyższego pliku.
`XkbModel` można wyciągnąć z pliku `/usr/share/X11/xkb/rules/xorg.lst` o ile oczywiście znamy model
klawiatury:

    $ cat /usr/share/X11/xkb/rules/xorg.lst
    ...
       logimel         Logitech Media Elite Keyboard
    ...

Jeśli chodzi o `XkbLayout` , to można go odszukać w katalogu `/usr/share/X11/xkb/symbols/` . W
przypadku polskiego układu klawiatury jest to `pl` . Jeśli chodzi zaś o opcje, to jest ich dość
dużo, a te częściej używane [zostały wyszczególnione tutaj][2]. Ja jednak korzystam z tych
poniższych:

  - `kpdl:dot` -- ustawia `.` na klawiaturze numerycznej zamiast `,` .
  - `lv3:ralt_switch` -- wciśnięcie przycisku prawy Alt umożliwi wprowadzanie polskich znaków.
  - `compose:rctrl` -- wciśnięcie prawego Ctrl aktywuje klawisz compose
  - `terminate:ctrl_alt_bksp` -- przyciśnięcie Ctrl-Alt-Backspace spowoduje zresetowanie Xserver'a,
    nie jest to jednak zalecane, bo zagraża bezpieczeństwu systemu.
  - `grp:alt_shift_toggle` -- wciśnięcie klawiszy lewy Alt-Shift spowoduje, ze układ klawiatury
    ulegnie zmianie.
  - `grp_led:scroll` -- sygnalizowanie diodą scroll lock na klawiaturze, który układ aktualnie
    obowiązuje.

## Natywne narzędzia Debiana

Debian ma swoje własne narzędzia, które ułatwiają konfigurację klawiatury i służy do tego pakiet
`keyboard-configuration` . Prawdopodobnie mamy go zainstalowanego w systemie, jeśli jednak go
przeoczyliśmy, to czym prędzej go dograjmy. W taki sposób, podczas instalacji tegoż pakietu, lub
przy korzystaniu z narzędzia `dpkg-reconfigure` , będziemy w stanie zmienić układ klawiatury
praktycznie od razu i w każdej sytuacji. Zostaniemy jedynie poproszeni o udzielenie odpowiedzi na
szereg pytań i raczej nie powinno z nimi być żadnych problemów.

Możemy wybrać model klawiatury:

![](/img/2015/06/1.debian-konfiguracja-klawiatura.png#huge)

Jak i również język:

![](/img/2015/06/2.debian-konfiguracja-klawiatura.png#huge)

Oraz układ:

![](/img/2015/06/3.debian-konfiguracja-klawiatura.png#huge)

A także dostosować klawisz `AltGr` , który odpowiada za możliwość wprowadzania polskich znaków:

![](/img/2015/06/4.debian-konfiguracja-klawiatura.png#huge)

Jest też opcja dla klawisza `compose` , to ten co umożliwia wprowadzanie dodatkowych znaków utf-8,
np. `®` :

![](/img/2015/06/5.debian-konfiguracja-klawiatura.png#huge)

Możemy także zdefiniować kilka opcji dla Xserver'a, choć jedynie tylko te podstawowe:

![](/img/2015/06/6.debian-konfiguracja-klawiatura.png#huge)

Zaowocuje to stworzeniem pliku `/etc/default/keyboard` o poniższej zawartości:

    # KEYBOARD CONFIGURATION FILE

    # Consult the keyboard(5) manual page.

    XKBMODEL="logimel"
    XKBLAYOUT="pl"
    XKBVARIANT=""
    XKBOPTIONS="kpdl:dot,lv3:ralt_switch,compose:rctrl,terminate:ctrl_alt_bksp,grp:alt_shift_toggle,grp_led:scroll"
    BACKSPACE="guess"

Jeśli, któryś z parametrów nam nie odpowiada, zawsze możemy go ręcznie tutaj zmienić, tylko unikajmy
ponownej generacji tego pliku, bo ustawienia zostaną nadpisane.

## Sprawdzenie aktualnego układu klawiatury

Po dokonaniu wszelkich zmian, przydałoby się także zweryfikować czy aby na pewno ustawiliśmy
wszystko tak jak trzeba. Możemy w tym celu posłużyć się narzędziem `setxkbmap` :

    #  setxkbmap -print -verbose 10
    Setting verbose level to 10
    locale is C
    Trying to load rules file ./rules/evdev...
    Trying to load rules file /usr/share/X11/xkb/rules/evdev...
    Success.
    Applied rules from evdev:
    rules:      evdev
    model:      logimel
    layout:     pl,us
    options:    kpdl:dot,lv3:ralt_switch,compose:rctrl,terminate:ctrl_alt_bksp,grp:alt_shift_toggle,grp_led:scroll
    Trying to build keymap using the following components:
    keycodes:   evdev+aliases(qwerty)
    types:      complete
    compat:     complete+ledscroll(group_lock)
    symbols:    pc+pl+us:2+inet(evdev)+group(alt_shift_toggle)+level3(ralt_switch)+compose(rctrl)+kpdl(dot)+terminate(ctrl_alt_bksp)
    geometry:   pc(pc104)
    xkb_keymap {
            xkb_keycodes  { include "evdev+aliases(qwerty)" };
            xkb_types     { include "complete"      };
            xkb_compat    { include "complete+ledscroll(group_lock)"        };
            xkb_symbols   { include "pc+pl+us:2+inet(evdev)+group(alt_shift_toggle)+level3(ralt_switch)+compose(rctrl)+kpdl(dot)+terminate(ctrl_alt_bksp)"  };
            xkb_geometry  { include "pc(pc104)"     };
    };

## Odpowiedni układ klawiatury przy starcie systemu

W przypadku gdy system nam nie chce się uruchomić jak zwykle, możemy być zmuszeni do ratowania go z
konsoli w bardzo wczesnym stadium boot. W takim przypadku nie zostanie ustawiony poprawny układ
klawiatury i praca pod konsolą w takich warunkach nie będzie należeć do najprzyjemniejszych. By
ustawić odpowiedni układ klawiatury już podczas startu systemu, trzeba zmienić w pliku
`/etc/initramfs-tools/initramfs.conf` opcję `KEYMAP=y` . Po edycji tego pliku, trzeba będzie także
wygenerować nowy initramfs za pomocą:

    # update-initramfs -u -k all


[1]: /post/klawiatura-multimedialna-i-niedzialajace-klawisze/
[2]: https://wiki.archlinux.org/index.php/Keyboard_configuration_in_Xorg#Frequently_used_XKB_options
