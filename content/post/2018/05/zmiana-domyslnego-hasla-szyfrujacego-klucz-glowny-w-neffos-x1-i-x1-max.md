---
author: Morfik
categories:
- Android
date:     2018-05-16 19:33:54 +0200
lastmod:  2018-05-16 19:33:54 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- szyfrowanie
- neffos
- neffos-x1
- neffos-x1-max
GHissueID: 321
title: Zmiana domyślnego hasła szyfrującego klucz główny w Neffos X1 i X1 Max
---

Nie bawiłem się ostatnio Neffos'ami ale w końcu udało mi się doprowadzić szyfrowanie w X1 Max (i
pewnie X1 też) do ładu. Dla przypomnienia, to w tych modelach najwyraźniej system zapomniał by
pytać użytkownika o hasło podczas konfiguracji, a że partycja z danymi użytkownikami jest
zaszyfrowana w standardzie (bez możliwości zmiany), to ustawiane jest domyślne hasło tj.
`default_password` . W ten sposób z technicznego punktu widzenia wilk jest syty i owca cała,
no bo użytkownik nie jest dręczony dodatkowym hasłem przy uruchamianiu systemu (obok hasła blokady
ekranu), no i dane są zaszyfrowane, no chyba, że ktoś wpisze ten nieszczęsny
`default_password` .

<!--more-->
## Mechanizm szyfrujący dane na partycji /data/ w Android

Czytając sobie o ewolucji szyfrowania w Androidzie natrafiłem na [taki artykuł][1]. Technicznie
Neffos X1 Max (i X1) mają Androida 6.0 ale z możliwością upgrade do 7.0. Niemniej jednak od 7.0
Android wprowadził [mechanizm direct-boot][2]. Po pewnych cechach charakterystycznych można ustalić,
że ten direct-boot nie jest wspierany w Neffos'ach. Zatem obowiązuje ciągle stary mechanizm, czyli
ten znany z Androida 6.0. Szukając dalej informacji na temat szyfrowania stosowanego w Android 6.0,
a konkretnie jak na nim można by operować, np. w celu wymuszenia zmiany hasła, które powinno być
możliwe, natrafiłem na [taki artykuł][3].

Są tam wymienione w zasadzie dwa polecenia:

    # vdc cryptfs changepw password jakies_haslo
    # vdc cryptfs verifypw jakies_haslo

Pierwsze z nich ustawia hasło, a drugie jest je w stanie zweryfikować. Przy czym, w żadnym z tych
dwóch poleceń system nie pyta o stare hasło i jak się zapomni co się wpisało i zresetuje telefon,
to można pożegnać się z danymi, które znajdują się na partycji `/data/` . Poniżej jest przykładowe
wywołanie tych ww. poleceń:

    $ adb shell
    X1_Max:/ $ su
    X1_Max:/ # vdc cryptfs verifypw test
    200 6822 1
    X1_Max:/ # vdc cryptfs verifypw default_password
    200 6824 0

Wyżej widać, że pomyślnie weryfikację przechodzi `default_password` -- numerek 0 na końcu oznacza
pomyślne wykonanie polecenia, 1 zaś błąd.

A tutaj już pomyślnie została przeprowadzona zmiana hasła oraz również pomyślnie zostało ono
zweryfikowane:

    X1_Max:/ # vdc cryptfs changepw password test
    200 7033 0
    X1_Max:/ # vdc cryptfs verifypw test
    200 7066 0

Można zatem ponownie uruchomić smartfon i podczas jego startu, system powinien przywitać nas
okienkiem, w którym to hasło trzeba wpisać. Bez tego system smartfona się nawet nie uruchomi i nie
będzie można z nim w żaden sposób wejść w interakcję (za wyjątkiem połączeń alarmowych).

Nawet TWRP prosi teraz o podanie hasełka i już tak ochoczo nie potrafi zdeszyfrować danych na
partycji `/data/` :

![](/img/2018/05/twrp.png#big)

## Problemy z hasłem do klucza głównego

Jedyny problem z jakim użytkownicy mogą się spotkać, to fakt, że taka zmiana hasła wymaga uprawnień
root...

No i oczywiście, gdy zmienia się hasło do blokady ekranu, to również trzeba jeszcze raz osobno
ustawić hasło do klucza głównego szyfrującego dane na partycji `/data/` . Jeśli tego się nie zrobi,
to zostanie przywrócone `default_password` .


[1]: https://yourtechexplained.com/2016/12/08/explained-android-nougat-file-based-encryption/
[2]: https://developer.android.com/training/articles/direct-boot.html
[3]: https://www.xda-developers.com/how-to-manually-change-your-android-encryption-password/
