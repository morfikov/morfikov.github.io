---
author: Morfik
categories:
- linux
date:    2021-04-30 22:10:00 +0200
lastmod: 2021-04-30 22:10:00 +0200
published: true
status: publish
tags:
- kernel
- bateria
- cpu
- sysfs
- intel
- thinkpad
- t430
GHissueID: 332
title: Energy Performance Bias (EPB) i jego wpływ na wydajność CPU Intela pod linux
---

Przeglądając ostatnio log systemowy, zauważyłem, że pojawia się w nim komunikat `kernel:
ENERGY_PERF_BIAS: Set to 'normal', was 'performance'` . Co prawda korzystam z laptopa i cokolwiek
związane z energią ustawione w trybie wydajności nie zawsze zdaje się być optymalnym rozwiązaniem
ale też moja maszyna zwykle jest podpięta do źródła zasilania i przydałoby się, by była ona
skonfigurowana właśnie bardziej w stronę profilu wydajności niż oszczędności energii. Ten powyższy
komunikat informuje nas zaś, że system zmienił ustawienia z `performance` (tryb wydajności) na
`normal` (jakiś bliżej nieokreślony tryb normalny). Chodzi naturalnie o ustawienia trybu pracy
procesora Intel. Postanowiłem zatem poszukać informacji na temat tego czym jest ten cały Energy
Performance Bias (EPB) i jak go skonfigurować w odpowiedni sposób pod linux.

<!--more-->
## Czym jest Energy Performance Bias (EPB)

Szukając informacji na temat Energy Performance Bias, [natrafiłem na dokumentację kernela][1],
która ten cały mechanizm w dość przystępny sposób opisuje. W skrócie, EPB pozwala aplikacjom
przestrzeni użytkownika (jak i samemu kernelowi) na określenie preferencji zarządzania energią
procesora w kontekście jego wydajności. Innymi słowy, wydajność procesora może zostać ograniczona,
co powinno przełożyć się na dłuższą pracę laptopa na baterii.

### Co dokładnie oznacza ENERGY_PERF_BIAS: Set to 'normal', was 'performance'

We wcześniejszych wersjach kernela (<5.1), komunikat `ENERGY_PERF_BIAS: Set to 'normal', was
'performance'` był w mechanizmie logowania podpięty pod poziom WARN, przez co wywoływał spore
zamieszanie, bo ten poziom zwykle jest wykorzystywana do zaznaczenia, że coś z systemem może być
nie tak. [Od wersji 5.1][3], priorytet tego komunikatu został przepisany z WARN na INFO. Niemniej
jednak, w logu systemowym, ta informacja wyróżnia się dość znacznie przez co może niepokoić mniej
zaawansowanych użytkowników linux'a.

Ten cały komunikat bierze się generalnie z faktu, że EPB może zostać ustawiony zarówno z
przestrzeni użytkownika (userspace), jak i również przez sam kernel. To drugie rozwiązanie zdaje
się być o wiele bardziej praktyczne ze względu na fakt niezachowywania wartości ustawionych przez
aplikacje przestrzeni użytkownika, np. przy przechodzeniu między stanami uśpienia maszyny. Dlatego
właśnie kernel stara się automatycznie dobrać odpowiednią wartość dla EPB ale nie zawsze ten zabieg
jest możliwy. Chodzi o to, że by ustawić odpowiedni EPB, kernel musi pozyskać pewne informacje z
firmware platformy. W sporej części przypadków, firmware maszyny zwraca `0` jako początkową/wstępną
wartość, co odpowiada za `performance` . Niemniej jednak, wartość `0` pojawi się także w przypadku
niezainicjowania EPB, bo firmware może przyjmować, że konfiguracja EPB będzie dokonywana z poziomu
przestrzeni użytkownika. Takie założenie nie zawsze jest w porządku i może prowadzić do szybszego
wyczerpywania się baterii laptopa. Dlatego też kernel linux'a zakłada, że jeśli firmware zwróci
wartość `0` , to lepiej jest przepisać profil z `0` na `6` albo mówiąc bardziej po ludzku z
`performance` na `normal` i stąd właśnie mamy taki komunikat.

Niekiedy BIOS/EFI/UEFI komputera zezwala na nieco bardziej zaawansowaną konfigurację pracy
procesora. W przypadku mojego laptopa ThinkPad T430 takiej opcji standardowo nie było ale po
[odblokowaniu zaawansowanego menu][4] w konfiguracji EFI/UEFI udało się uzyskać dostęp do ustawień
CPU, co wygląda mniej więcej tak:

![energy-performance-bias-epb-cpu-linux-bios-efi-uefi](/img/2021/04/001-energy-performance-bias-epb-cpu-linux-bios-efi-uefi.jpg#huge)

![energy-performance-bias-epb-cpu-linux-bios-efi-uefi](/img/2021/04/002-energy-performance-bias-epb-cpu-linux-bios-efi-uefi.jpg#huge)

![energy-performance-bias-epb-cpu-linux-bios-efi-uefi](/img/2021/04/003-energy-performance-bias-epb-cpu-linux-bios-efi-uefi.jpg#huge)

Jedyną opcją, którą można by powiązać z EPB jest `Boot Performance Mode` . Jak widać na fotkach
wyżej, ten parametr można ustawić na `Max Performance` , `Max Battery` albo `Auto` . Niestety żadne
z tych ustawień nie wpływa na pojawienie się (czy usunięcie) komunikatu `kernel: ENERGY_PERF_BIAS:
Set to 'normal', was 'performance'` w logu podczas startu systemu. Z opisu tego parametru można by
wnioskować, że ma on zastosowanie jedynie do momentu przekazania kontroli nad trybem pracy
procesora systemowi operacyjnemu i być może wtedy wartość EPB jest ustawiana na `0` (zakładając
oczywiście, że to jest ten szukany parametr).

## Konfiguracja EPB w userspace

Wcześniej zostało wspomniane, że Energy Performance Bias może być także konfigurowany za pomocą
aplikacji przestrzeni użytkownika (userspace). Jeśli chodzi o linux, to zostało tutaj stworzone
dedykowane narzędzie `x86_energy_perf_policy` , które na Debianie znajduje się w pakiecie
`linux-cpupower` . Ten cały `x86_energy_perf_policy` operuje jednak na rejestrach
MSR (Model-Specific Register) procesora i wymagane jest by w systemie były dostępne urządzenia
`/dev/cpu/*/msr` . Te urządzenia nie zawsze będą jednak widoczne. Nie chodzi tutaj już o sam fakt
zbudowania kernela z wyłączoną opcją `CONFIG_X86_MSR` ale bardziej o mechanizm EFI/UEFI Secure
Boot, który z kolei wymusza zastosowanie [kernel lockdown][2], przez co zmiana rejestrów MSR
jest zabroniona. W przypadku braku dostępu do MSR, `x86_energy_perf_policy` zwróci nam poniższy
błąd:

    # x86_energy_perf_policy
    x86_energy_perf_policy: no /dev/cpu/0/msr, Try "# modprobe msr" : No such file or directory

Oczywiście konfiguracja EPB przy pomocy narzędzia `x86_energy_perf_policy` nie jest jedynym
sposobem by ustawić odpowiedni profil energetyczny procesora. Możemy skorzystać z interfejsu
kernela `sysfs` i ręcznie ustawić odpowiedni profil. Interesuje nas plik `energy_perf_bias`
zlokalizowany w `/sys/devices/system/cpu/cpu*/power/` . Dla każdego CPU będziemy mieli osobny taki
plik. Trzeba jednak tutaj zaznaczyć, że czasami wartość określona w `energy_perf_bias` może być
wspólna dla kilku fizycznych rdzeni procesora lub/i też gdy procesor wykorzystuje technologię HT. W
takim przypadku, przepisanie `energy_perf_bias` dla jednego rdzenia będzie skutkowało ustawieniem
zapisanej wartości dla pozostałych rdzeni lub ich części. Plik `energy_perf_bias` można zapisać
liczbą od `0` do `15` . Im niższa jest ta wartość, tym mniejsza będzie energooszczędność procesora,
zatem ustawiając wartość `0` wybierzemy tryb `performacne` . Poza numerycznym definiowaniem profilu
możemy także skorzystać z jednej z następujących wartości: `performance` , `balance-performance` ,
`normal` , `balance-power` oraz `power` . Jeśli zdecydujemy się wybrać którąś z nich, to system
automatycznie dobierze dla niej numerek, przykładowo:

    # echo "performance" > /sys/devices/system/cpu/cpu*/power/energy_perf_bias

    #  cat /sys/devices/system/cpu/cpu*/power/energy_perf_bias
    0
    0
    0
    0

    # echo "power" > /sys/devices/system/cpu/cpu*/power/energy_perf_bias

    # cat /sys/devices/system/cpu/cpu*/power/energy_perf_bias
    15
    15
    15
    15

### Automatyczne ustawienie konfiguracji EPB

Zmiany, które wprowadziliśmy wyżej za sprawą pliku `energy_perf_bias` nie są permanentne, tj. np.
po restarcie maszyny trzeba będzie jeszcze raz określić stosowne wartości. Mamy jednak do
dyspozycji usługę systemd `sysfsutils.service` , która za nas może uzupełnić plik
`energy_perf_bias` podczas rozruchu linux'a. Trzeba tylko edytować plik `/etc/sysfs.conf` i dodać
do niego poniższą linijkę:

    devices/system/cpu/cpu*/power/energy_perf_bias = performance

Naturalnie zamiast `performance` możemy określić `balance-performance` , `normal` , `balance-power`
czy `power` , w zależności jaki tryb wydajności procesora nas najbardziej interesuje.


[1]: https://www.kernel.org/doc/html/latest/admin-guide/pm/intel_epb.html
[2]: https://man7.org/linux/man-pages/man7/kernel_lockdown.7.html
[3]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2ee27796f298b710992a677a7e4d35c8c588b17e
[4]: https://github.com/n4ru/1vyrain
