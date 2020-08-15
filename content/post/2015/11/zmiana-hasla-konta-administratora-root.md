---
author: Morfik
categories:
- Linux
date: "2015-11-25T13:55:44Z"
date_gmt: 2015-11-25 12:55:44 +0100
published: true
status: publish
tags:
- użytkownicy
- grupy
title: Zmiana hasła konta administratora root
---

W pewnych skrajnych przypadkach może się nam zdarzyć tak, że zapomnimy hasła do konta administratora
systemu. Jak wiadomo bez użytkownika root w naszych linux'ach nie da się zbytnio nic zrobić. Na
pewno nie da się przeprowadzić żądnych prac administracyjnych. Zwykle w takim przypadku moglibyśmy
przeinstalować system i ustawić nowe hasło ale to wydaje się lekką przesadą. Poza tym, co w
przypadku gdy nie możemy zwyczajnie zainstalować na nowo systemu lub nie mamy akurat pod ręką płytki
czy pendrive live? Czy w linux'ie jest w ogóle możliwość odzyskania hasła użytkownika root bez
rozkręcania komputera biorąc pod uwagę te wszystkie mechanizmy bezpieczeństwa, które czynią ten
system tak bezpiecznym? Oczywiście zmiana hasła do konta administratora jest możliwa i nawet nie
trzeba się przy tym zbytnio wysilać.

<!--more-->
## Zmiana hasła przy pomocy init=/bin/bash

Płytki live wymagają od nas po pierwsze posiadania zewnętrznego nośnika jakim jest sam krążek, a do
tego jeszcze potrzebujemy napędu CD/DVD. W przypadku pendrive live sprawa ma się podobnie, z tym, że
praktycznie każdy komputer ma już port USB, do którego to urządzenie można podpiąć. Jeśli nie
dysponujemy żadnym z tych powyższych, to pozostaje nam jeszcze rozkręcenie maszyny, wyciągnięcie
dysku i podpięcie go na drugim komputerze. Jest jeszcze jedna opcja, mianowicie możemy skorzystać z
bootloader'a, który uczestniczy w procedurze startu systemu.

W przypadku tego ostatniego rozwiązania nie trzeba nawet edytować żadnych plików konfiguracyjnych
samego bootloader'a. Możemy zwyczajnie edytować linijkę kernela przyciskając klawisz Tab w momencie,
gdy pojawi się okienko z wyborem jądra operacyjnego podczas startu systemu. Po przyciśnięciu tego
klawisza, zostanie nam wyświetlona konkretna pozycja, którą możemy poddać edycji. Zmiany mają efekt
tylko tymczasowy i po restarcie maszyny ulotnią się. Zatem jeśli coś wpiszemy nie tak, to warto
pamiętać, że nic złego się tak naprawdę nie stanie.

Będąc w stanie edytować linijkę kernela w okienku bootloader'a, możemy dopisywać lub też kasować
szereg parametrów. Tym, który nas interesuje najbardziej jest `init=/bin/bash` . Jak możemy
wyczytać, np. [tutaj](https://www.redhat.com/archives/rhl-list/2005-March/msg04089.html), dopisanie
tego powyższego parametru sprawi, że kernel uruchomi `/bin/bash` jako program init, czyli ten
pierwszy program, który zwykł startować wszystkie pozostałe usługi. Jako, że jest to zwykły shell,
to uruchomi się tylko on i nic poza nim. Zaletą tego rozwiązania jest fakt, że zostaniemy od razu
zalogowani na konto root bez pytania nas o jakiekolwiek hasło. Wszelkie usługi będą wyłączone, a
partycje niezamontowane. No oczywiście wyjątkiem jest partycja `/` , która, co prawda, będzie
zamontowana ale w trybie tylko do odczytu.

Zmiana hasła do konta administratora w tym przypadku może się odbyć za sprawą polecenia `passwd` .
Potrzebuje ono jednak uzyskać dostęp do pliku `/etc/shadow` , a konkretnie musi go zapisać. Dlatego
też musimy pierw przemontować partycję `/` w tryb do zapisu, a dopiero po jej przemontowaniu wydać
polecenie `passwd` , przykłądowo:

    # mount -o remount,rw /
    # passwd

## Usuwanie hasła do konta root

Zmian hasła to nie jedyna opcja, z której możemy skorzystać będą zalogowanym w systemie podając
bootloader'owi parametr `init=/bin/bash` . Mianowicie, możemy także usunąć hasło do konta root
całkowicie przez dokonanie ręcznej edycji pliku `/etc/shadow` . By usunąć hasło, odszukujemy w tym
pliku pozycję odnosząca się do użytkownika `root` , przykładowo:

    root:zahashowane-haslo:16764:0:99999:7:::

Jeśli skasujemy w niej wszystkie znaki, które znajdują się między pierwszym i drugim `:` (fraza
`zahashowane-haslo` ), to usuniemy hasło do konta root całkowicie. W takim przypadku, po tym jak
uruchomimy komputer ponownie, logowanie na użytkownika root będzie odbywać się już bez hasła.
Oczywiście zawsze możemy skorzystać z `passwd` i w dowolnym momencie ustawić hasło dla konta
administratora.
