---
author: Morfik
categories:
- Linux
date: "2015-07-11T10:12:11Z"
date_gmt: 2015-07-11 08:12:11 +0200
published: true
status: publish
tags:
- firefox
- klawiatura
GHissueID: 154
title: Klawisz Backspace w Firefox'ie
---

Przez cały czas korzystania z internetu, robiłem to za pomocą przeglądarki Opera. Nawet po tym jak
przeszedłem na linuxa, to wciąż nie mogłem się z nią rozstać i to pomimo faktu, że nie była ona
przecież opensource, przez co nie była także dostępna w repozytoriach debiana. Gdy deweloperzy z
zespołu Opery przestali rozwijać tę przeglądarkę dla linuxa, musiałem poszukać sobie czegoś innego.
Wybór padł na Firefox'a ale każdy kto używał tych dwóch przeglądarek wie, że różniły się one dość
znacznie parę lat temu i jedną z tych bardziej odczuwalnych różnic była inna obsługa klawisza
Backspace .

<!--more-->
## Domyślne ustawienia klawisza Backspace

Przy przeglądaniu stron internetowych w Operze, gdy człowiek chciał powrócić do poprzedniej strony,
wszystko czego potrzebował to wciśnięcie klawisza Backspace . Jeśli spróbujemy wcisnąć ten klawisz w
Firefox'ie, to na dobrą sprawę nic się nie stanie, a to z tego względu, że [od prawie 2007 roku][1],
klawisz Backspace jest niemapowany domyślnie, czego efektem jest brak reakcji na jego przyciśnięcie.

W powyższym linku możemy wyczytać, że akcja powrotu do poprzedniej strony była przypisana do
klawisza Backspace by zachować zgodność z przeglądarką Internet Explorer ale w trosce o
kompatybilność z klawiszami w innych aplikacjach na linuxie, deweloperzy postanowili ten klawisz
mapować w zależności od platformy, na której Firefox miał działać. Doprowadziło to do stworzenia
opcji `browser.backspace_action` , którą każdy użytkownik tej przeglądarki może sobie bez problemu
dostosować.

## Zmiana mapowania klawisza Backspace

Parametr `browser.backspace_action` może przybrać jedną z trzech wartości. Domyślnie jest ustawiony
na `2` , co oznacza kompletny brak mapowania tego klawisza przez Firefox. Jeśli chcemy przywrócić
zachowanie znane z Opery, czyli przyciśnięcie klawisza Backspace lub Shift-Backspace będzie
przenośić nas o jedną stronę do tyłu lub do przodu, to musimy ustawić `0` . Istnieje jeszcze
możliwość sprecyzowania wartości `1` , która sprawi, że przyciśnięcie klawisza Backspace lub
Shift-Backspace będzie symulować zachowanie klawiszy Page Down oraz Page Up .

Jeśli chcemy zmienić domyślne zachowanie klawisza Backspace , to przechodzimy do edycji ustawień
konfiguracyjnych Firefox'a, które kryją się pod `about:config` . Wpisujemy zatem ten adres w polu
adresu i wyszukujemy `backspace` , po czym przestawiamy jego wartość tej zmiennej wedle upodobań:

![](/img/2015/07/1.firefox-backspace.png#big)

[1]: http://kb.mozillazine.org/Browser.backspace_action
