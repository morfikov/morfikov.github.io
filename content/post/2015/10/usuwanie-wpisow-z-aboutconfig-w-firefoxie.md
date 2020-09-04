---
author: Morfik
categories:
- Linux
date: "2015-10-27T20:58:39Z"
date_gmt: 2015-10-27 18:58:39 +0100
published: true
status: publish
tags:
- firefox
title: Usuwanie wpisów z about:config w Firefox'ie
---

Po wpisaniu w pasku adresu Firefox'a [about:config](http://kb.mozillazine.org/About:config) ,
zostanie nam zwrócona dość długa lista parametrów konfiguracyjnych, które możemy sobie dostosować
wedle uznania. Większość z nich ma spory wpływ na zachowanie samej przeglądarki ale są też i opcje,
które zostały dodane za sprawą różnych dodatków. Chodzi o to, że za każdym razem gdy instalujemy
nowy addon, to ten zwykle ma opcje konfiguracyjne i to właśnie one są widoczne w `about:config` . W
przypadku gdy już nie korzystamy z tego dodatku i wyrzuciliśmy go kompletnie z Firefox'a, wpisy w
konfiguracji dalej widnieją. Przydałoby się zatem nieco przeczyścić naszą przeglądarkę i usunąć te
wszystkie śmieci.

<!--more-->
## Ręczne czyszczenie about:config

Wszystkie wpisy w `about:config` , które zostały dodane lub zmienione w wyniku naszej świadomej
ingerencji, są pogrubione. Z kolei wszystkie wpisy od dodatków zaczynają się od `extensions` . Po
`.` jest nazwa dodatku, przykładowo:

![]({{< baseurl >}}/img/2015/10/1.about-config-firefox.png#big)

Wpisy po dodatkach z `about:config` możemy usunąć ręcznie. Polega to na klikaniu w każdą opcję z
osobna i resetowaniu ich wartości. Po tym jak dodatek będzie miał zresetowaną wartość jakiejś opcji
i Firefox zostanie ponownie uruchomiony, ta opcja zostanie usunięta z pliku konfiguracyjnego i już
więcej nie pojawi się w `about:config` .

Gdy addon jest dość rozbudowany, ilość opcji może być spora. To samo tyczy się sytuacji gdzie
testowaliśmy szereg dodatków i wybraliśmy jedynie te, które nam przypadły w jakiś sposób do gustu.
Może się zdarzyć tak, że będziemy musieli zresetować 100-200 albo nawet i więcej parametrów. Zatem
klikanie w każdy z nich było by bardzo męczące i czasochłonne. Istnieje na szczęście [plik
prefs.js](http://kb.mozillazine.org/Prefs.js_file) , który pozwoli nam uporać się z zadaniem
czyszczenia o wiele szybciej.

## Czyszczenie pliku prefs.js

Opcje które są nam udostępniane w `about:config` są wpisywane do pliku `prefs.js` , który znajduje
się w katalogu `~/.mozilla/firefox/profil.default` . Jest to zwykły plik tekstowy i możemy go
otworzyć w swoim ulubionym edytorze teksu. Poniżej jest wycinek tego pliku:

    ...
    user_pref("extensions.unmht.open.multiple.action", "index");
    user_pref("extensions.unmht.save.filename.dup.rename.digits", 2);
    user_pref("extensions.unmht.save.history.register", false);
    user_pref("extensions.unmht.save.insert.link", true);
    user_pref("extensions.unmht.save.insert.link.pos", "top");
    ...

Jak widać, namierzenie odpowiednich wpisów nie sprawia problemów. By teraz oczyścić ten plik,
zwyczajnie kasujemy wszystko co odnosi się do dodatków, których już nie używamy na tym profilu.
Trzeba pamiętać, by czyszczenia pliku `prefs.js` dokonywać przy wyłączonym Firefox'ie. Jeśli
wprowadzimy jakiś zmiany mając przy tym odpaloną przeglądarkę, zostaną one nadpisane przy jej
zamknięciu. Pamiętajmy też, by przed edycją tego krytycznego pliku, zrobić jego kopię zapasową, tak
na wszelki wypadek.
