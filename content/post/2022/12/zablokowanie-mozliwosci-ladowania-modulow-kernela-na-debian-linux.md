---
author: Morfik
categories:
- Linux
date:    2022-12-13 13:03:00 +0100
lastmod: 2022-12-13 13:03:00 +0100
published: true
status: publish
tags:
- debian
- bezpieczeństwo
- kernel
- moduły-kernela
- modprobe
- sysctl
GHissueID: 593
title: Zablokowanie możliwości ładowania modułów kernela na Debian linux
---

Kernele oferowane przez różne dystrybucje linux'a, np. Debian czy Ubuntu, są tak budowane by
możliwie jak największa ilość sprzętu na takim jądrze operacyjnym nam zadziałała bez zbędnego
przerabiania systemu. Takie podejście sprawia, że nowo nabyty przez nas sprzęt możemy od razu
podłączyć do komputera, a stosowne moduły po rozpoznaniu urządzenia zostaną załadowane do pamięci
operacyjnej, przez co my będziemy mogli wejść w interakcję z takim kawałkiem elektroniki. Problem w
tym, że kernel ma bardzo dużo tych modłów. Dużo modułów, to więcej kodu, a więcej kodu to więcej
błędów, które na poziomie jądra mogą być bardzo opłakane w skutkach. Z zagrożeniem, jakie niosą
moduły zewnętrzne (te spoza drzewa kernela linux), można sobie poradzić [podpisując kernel kluczami
od firmware EFI/UEFI][2]. W takim przypadku załadowanie niepodpisanego modułu już się nie powiedzie.
Niemniej jednak, dystrybucyjne kernele mają całą masę modułów do różnorakiego sprzętu, które na
etapie budowania kernela są podpisywane, co otwiera im drogę do bycia załadowanymi w naszym
systemie nawet jeśli nie posiadamy urządzeń, które by użytek z tych modułów robiły. Przydałoby się
zatem zabezpieczyć możliwość ładowania modułów kernela w taki sposób, by jedynie administrator
systemu miał możliwość określenia jakie moduły i na którym etapie pracy systemu mogą one zostać
załadowane.

<!--more-->
##  Sysctl i parametr kernel.modules_disabled

Większość użytkowników Debiana zdaje sobie raczej sprawę z istnienia narzędzia `sysctl` , przy
pomocy którego to jesteśmy w stanie zmieniać wartości różnych parametrów kernela podczas jego pracy.
Samo narzędzie jest dość proste, bo sprowadza się do zapisu lub odczytu plików w
katalogu `/proc/sys/` . W tym katalogu mamy też dostępny plik `/proc/sys/kernel/modules_disabled` ,
który to po zapisaniu odpowiednią wartością sprawi, że w systemie nie będzie można już załadować
żadnego dodatkowego modułu. Ten mechanizm nie może też zostać odblokowany do momentu uruchomienia
komputera ponownie.

## Systemd i usługi systemd-modules-load.service oraz systemd-sysctl.service

W przypadku korzystania z systemd, mamy dostępne dwie usługi `systemd-modules-load.service` oraz
`systemd-sysctl.service` , z których ta pierwsza jest uruchamiana przed tą drugą. Można zatem
dopisać do pliku `/etc/modules` potrzebne nam moduły (względnie też stworzyć stosowny plik w
katalogu `/etc/modules-load.d/` ), po czym dopisać poniższą linijkę w pliku `/etc/sysctl.conf` :

    kernel.modules_disabled = 1

W przypadku takiej konfiguracji, system podczas startu powinien załadować wszystkie zdefiniowane
przez nas moduły i po chwili zablokować dalszą możliwość ładowania innych modułów, co znacznie
poprawia bezpieczeństwo działającego systemu.

## Alias na modprobe

Istnieje też [alternatywne rozwiązanie][1] w postaci stworzenia aliasu na `modprobe` , które
jest niezależne od wykorzystywanego systemu inicjacji procesów. By tego typu alias stworzyć,
wystarczy w katalogu `/etc/modprobe.d/` utworzyć plik `disable.conf` i wrzucić do niego poniższą
linijkę:

    install disable /sbin/sysctl kernel.modules_disabled=1

Następnie na końcu listy modułów w pliku `/etc/modules` trzeba dopisać moduł `disable` :

    ...
    disable

W ten sposób podczas startu systemu, gdy pojawi się wywołanie `modprobe` z nazwą konkretnego modułu,
to dany moduł zostanie załadowany do pamięci. Jak tylko system przetworzy listę modułów i dojdzie
do pozycji `disable` , to zostanie zapisana wartość `1` w pliku
`/proc/sys/kernel/modules_disabled` , co efektywnie wyłączy możliwość dalszego ładowania modułów.


[1]: https://outflux.net/blog/archives/2012/11/28/clean-module-disabling/
[2]: /post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/#podpisywanie-kernela
