---
author: Morfik
categories:
- Linux
date: "2020-02-28T23:00:00Z"
published: true
status: publish
tags:
- debian
- acpi
- thinkpad
- lenovo
- t430
GHissueID: 28
title: Regulacja obrotów wentylatora w zależności od zmian temperatury w ThinkPad T430
---

Ostatnio udało mi się nabyć dość niedrogo laptop Lenovo, a konkretnie był to model ThinkPad T430.
Maszyna jakby nie patrzeć jest bardzo kompatybilna z linux i w zasadzie nie mogę jej nic zarzucić,
przynajmniej póki co. Niemniej jednak, jest pewien szkopuł, który mnie zaczął ździebko irytować od
praktycznie samego początku jak tylko podłączyłem ten komputer do prądu. Chodzi o wentylator
chłodzący radiator procesora, który no troszkę daje o sobie znać i to mimo faktu, że temperatura
CPU jest w granicach 40 stopni. Przeglądając opcje w BIOS nie natrafiłem na nic co mogło by te
obroty wyregulować. Na szczęście w przypadku laptopów Lenovo można programowo zdefiniować obroty
wiatraka posługując się narzędziem [thinkfan][1].

<!--more-->
## Moduł thinkpad_acpi i parametr fan_control

Zanim jednak zajmiemy się samym oprogramowaniem `thinkfan` , musimy sobie odpowiednio skonfigurować
system. Ze wstępnych ustaleń wynika, że zarządzanie wentylatorem pod linux w tym ThinkPad T430
zajmuje się moduł kernela `thinkpad_acpi` . Standardowo jednak ten moduł nie jest skonfigurowany w
taki sposób, aby było można ręcznie dostosować obroty wiatraka w zależności od temperatury
procesora, co pokazuje poniższa rozpiska wartości parametrów samego modułu:

    # systool -v -m thinkpad_acpi
    Module = "thinkpad_acpi"
    ...
      Parameters:
        ...
        fan_control         = "N"
        ...

Jak można zauważyć, wyżej mamy `fan_control="N"` . Trzeba zatem wartość tego parametru przepisać i
ustawić ją na `"Y"` . W tym celu trzeba stworzyć konfigurację dla modułu w pliku
`/etc/modprobe.d/thinkpad.conf` i dorzucić w nim tę poniższą linijkę:

    options thinkpad_acpi fan_control=1

Po uzupełnieniu powyższego pliku trzeba będzie zresetować maszynę, by nowe ustawienia modułu
zostały zaaplikowane.

Na necie można się spotkać także z zaleceniem/wymogiem przestawienia opcji `experimental` na `"Y"` .
Nie wiem czy to dobry pomysł. Jeśli z jakiegoś powodu nie będziemy mogli sterować obrotami
wentylatora, to sprawdźmy czy czasem aby na pewno napisaliśmy ten parametr prawidłowo. Ja z
początku wpisałem `fancontrol` zamiast `fan_control` i po tej drobnej poprawce wszystko już zaczęło
ładnie śmigać.

## Konfiguracja thinkfan

Mając przygotowany moduł kernela, możemy przejść do konfiguracji narzędzia `thinkfan` . Jeśli nie
mamy go jeszcze zainstalowanego, to znajduje się on w pakiecie `thinkfan` (przynajmniej na
Debianie). Z procesem instalacyjnym raczej nie powinno być problemów, zatem przejdźmy od razu do
pliku `/etc/thinkfan.conf` . Konfiguracja w zasadzie składa się z trzech części.

### Wybór wentylatora

Po pierwsze, musimy określić wentylator, na którym zamierzamy operować. Póki co możemy zdefiniować
tylko jedno urządzenie (choć nic nie stoi na przeszkodzie, by stworzyć kilka plików
konfiguracyjnych i uruchomić demona kilka razy). By określić pożądane urządzenie potrzebny nam
będzie odpowiedni sterownik ( `tp_fan` lub `pwm_fan` ) oraz ścieżka do pliku w katalogu `/proc/`
lub `/sys/` . To z którego sterownika skorzystamy, zależy głównie od naszych preferencji.  Niemniej
jednak, każdy z tych sterowników konfiguruje się nieco inaczej. Poniżej jest przykładowa
konfiguracja dla obu sterowników, którą trzeba dodać do pliku `/etc/thinkfan.conf` (wybrać należy
sobie jeden wpis):

    tp_fan /proc/acpi/ibm/fan
    pwm_fan /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/pwm1

O ile w przypadku sterownika `tp_fan` ścieżka zawsze będzie ta sama, tj. `/proc/acpi/ibm/fan` , o
tyle w przypadku `pwm_fan` odpowiednią ścieżkę trzeba sobie ustalić ręcznie przy pomocy tego
poniższego polecenia:

    # find /sys -type f -name "pwm?"
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/pwm1

### Wybór czujnika/czujników temperatury

Z reguły w każdym komputerze jest dostępnych wiele czujników temperatury. W przypadku, gdy w pliku
`/etc/thinkfan.conf` nie określimy żadnego czujnika temperatury, to `thinkfan` będzie operował na
najwyższych wartościach temperatury jakie uda mu się pozyskać z systemu. Tego typu zachowanie może
niekoniecznie być pożądane i przydałoby się określić te czujniki, które nas interesują najbardziej,
czyli procesor lub bezpośrednio jego rdzenie.

Podobnie jak w przypadku sterownika wentylatora, musimy wybrać sterownik czujnika temperatury oraz
podać odpowiedni plik w katalogu `/proc/` lub `/sys/` . Jeśli chodzi o sterowniki to mamy do wyboru
`tp_thermal` , `hwmon` , `atasmarti` oraz `nv_thermal` . Z reguły będą nas interesować tylko te dwa
pierwsze sterowniki, bo `atasmarti` odnosi się do temperatury dysku twardego odczytanej ze SMART
(trzeba podać ścieżkę typu `/dev/sda` ), zaś `nv_thermal` odnosi się do kart graficznych Nvidia
(potrzebne ID szyny PCI odczytane z `lspci` ). Jako, że nas tutaj interesuje głównie temperatura
głównego CPU, to w `/etc/thinkfan.conf` musimy określić sobie albo `tp_thermal` :

    tp_thermal  /proc/acpi/ibm/thermal

albo `hwmon` :

    hwmon /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp1_input
    hwmon /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp2_input
    hwmon /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp3_input

Jeśli nie wiemy jakie czujniki temperatur ma nasz ThinkPad, to możemy albo podejrzeć wyjście
polecenia `sensors` (potrzebny pakiet `lm-sensros` ):

    # sensors

    acpitz-acpi-0
    Adapter: ACPI interface
    temp1:        +39.0°C  (crit = +104.0°C)

    thinkpad-isa-0000
    Adapter: ISA adapter
    fan1:           0 RPM
    temp1:        +39.0°C
    temp2:         +0.0°C
    temp3:         +0.0°C
    temp4:         +0.0°C
    temp5:         +0.0°C
    temp6:         +0.0°C
    temp7:         +0.0°C
    temp8:         +0.0°C

    coretemp-isa-0000
    Adapter: ISA adapter
    Package id 0:  +41.0°C  (high = +87.0°C, crit = +105.0°C)
    Core 0:        +37.0°C  (high = +87.0°C, crit = +105.0°C)
    Core 1:        +43.0°C  (high = +87.0°C, crit = +105.0°C)

albo też skorzystać z poniższego polecenia:

    # find /sys -type f -name "temp*_input"
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp6_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp3_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp7_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp4_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp8_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp1_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp5_input
    /sys/devices/platform/thinkpad_hwmon/hwmon/hwmon3/temp2_input
    /sys/devices/virtual/thermal/thermal_zone0/hwmon0/temp1_input
    /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp3_input
    /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp1_input
    /sys/devices/platform/coretemp.0/hwmon/hwmon1/temp2_input

W tym przypadku moduł `coretemp-isa` jest w stanie podać nam dokładne wskazania temperatury dla
konkretnych rdzeni CPU oraz ogólnie dla całego procesora. Są to te trzy ostatnie wpisy wyżej i to
je określiłem w konfiguracji `thinkfan` . Jeśli interesują nas też i inne czujniki, to również
trzeba je dodać do konfiguracji w pliku `/etc/thinkfan.conf` .

W tym powyższym przykładzie monitorowaniu zostaną poddane trzy czujniki. Jeśli wskazania któregoś z
nich przekroczą 40 stopni, to wtedy wentylator powinien się załączyć. Z kolei wentylator wyłączy
się dopiero w momencie, gdy wszystkie trzy czujniki zejdą poniżej 38 stopni.

#### Regulacja czujników temperatury

Temperatura poszczególnych układów czy urządzeń wewnątrz obudowy laptopa może się dość drastycznie
różnić. Dlatego też `thinkfan` ma opcję regulacji czujników temperatury. Dla przykładu weźmy sobie
dysk twardy oraz główny procesor. Zakresy temperatur, w których te urządzenia mogą pracować jest
powiedzmy od 0-100°C dla CPU, natomiast dla dysku będzie to od 0-55°C. W spoczynku te dwa
urządzenia będą mieć zbliżoną temperaturę około 35-40°C. Niemniej jednak, jeśli zaczniemy
utylizować dość mocno procesor, to jego temperatura może wzrosnąć nam i do 70°C (i więcej). W
przypadku intensywnego wykorzystania dysku twardego, jego temperatura może wzrosnąć do 45-50°C.

Pojawia się tutaj problem różnicy temperatur i interpretacji wartości przez `thinkfan` -- nie wie
on z jakim urządzeniem ma do czynienia i jakie temperatury pracy są dla niego odpowiednie. Procesor
przy 45-50°C może pracować bez problemu i nawet nie trzeba kręcić wiatrakiem na full w takich
warunkach ale gdyby dysk już złapał taką temperaturę, to trzeba by podkręcić nieco obroty, by
wymusić przepływ powietrza w obudowie laptopa, co powinno schłodzić dysk. Dlatego też przy każdym
czujniku temperatury w pliku `/etc/thinkfan.conf` możemy podać korektę, tak by `thinkfan` zaczął
mocniej obracać wentylatorem, gdy temperatura dysku wzrośnie do 45°C, a temperatura procesora do
60°C. Poniżej jest przykład korekty:

    tp_thermal /proc/acpi/ibm/thermal (0, 10, 15, 2, 10, 5, 0, 3, 0, 3)

W tym przypadku korekta ma wiele wartości, bo sam plik `/proc/acpi/ibm/thermal` ma ich sporo.
Liczba wartości w nawiasie musi się zgadzać z tym co jest w pliku. Jeśli w pliku by była jedna
wartość, to w nawiasie precyzujemy tylko jedną liczbę. Same liczby zaś mogą być dodatnie lub
ujemne.

### Regulacja obrotów wentylatora

Z reguły dobór czujnika temperatury i wentylatora odbywa się bez większych problemów i raczej nie
będzie potrzeby określania ich. Oczywiście nic nie stoi na przeszkodzie by to zrobić ręcznie.
Niemniej jednak, trzeba jeszcze powiązać prędkość obrotów wentylatora ze wskazaniami czujników
temperatury. Trzeba mieć jednak na uwadze, że tutaj ma ogromne znaczenie to, z jakiego sterownika
wentylatora korzystamy, tj. czy mamy w użyciu `tp_fan` czy też `pwm_fan` . Dla każdego z nich
wartości, które trzeba zdefiniować są inne. Poniżej przykładowa konfiguracja.

Dla sterownika `tp_fan` :

    (0,	0,	20)
    (1,	18,	55)
    (2,	48,	60)
    (3,	58,	65)
    (4,	63,	70)
    (5,	68,	75)
    (6,	73,	80)
    (7,	78,	32767)

Dla sterownika `pwm_fan` :

    (0, 	0,	20)
    (36,	18,	55)
    (72,	48,	60)
    (109,	58,	65)
    (145,	63,	70)
    (182,	68,	75)
    (218,	73,	80)
    (255,	78,	32767)

Powyżej znajduje się moja konfiguracja pracy wentylatora. Mamy w zasadzie 8 trybów pracy (oznaczone
numerkami w pierwszej kolumnie). Druga z trzecią kolumną precyzują zakres temperatur, w których
danych tryb pracy wentylatora będzie obowiązywał. W tej powyższej konfiguracji wentylator włączy
się jedynie w momencie, gdy temperatura procesora przekroczy 20 stopni. Innymi słowy wiatrak będzie
obracał się cały czas na wolnych obrotach wymuszając przepływ powietrza przez obudowę. Ten stan
rzeczy jest lepszy według mnie niż ciągłe wyłączanie i załączanie wentylatora, co może ździebko
irytować. Sam wiatrak zaś będzie zwiększał obroty wraz ze wzrostem temperatury, osiągając w
ostatnim trybie około 5,4K obrotów na minutę.

Dodatkowo warto ustawić w drugiej kolumnie nieco niższe wartości niż w trzeciej kolumnie w
poprzednim wierszu. Czyli jak mamy w drugim trybie 18-55°C, to w trzecim dajemy 48-60°C.
Zapobiegnie to ciągłemu przełączaniu trybów wentylatora, gdy temperatura będzie na granicy 50
stopni.

Warto tutaj jeszcze zaznaczyć fakt, że domyślne ustawienia częstotliwości procesora umożliwiają mu
pracę z 3,3 GHz i można pokusić się o obniżenie tej częstotliwości maksymalnej do 2,5 GHz. W takiej
sytuacji i przy takich ustawieniach wentylatora, temperatura procesora nie powinna przekroczyć 60°C.
W efekcie wiatrak nie powinien nam się dawać zbytnio we znaki.

Trzeba także zwracać uwagę na wartości modułów, z których korzystamy. W przypadku `tp_fan` mamy
numery trybów `0-7`, natomiast w przypadku `hwmon` mamy wartości od `0-255` stopniowane
(przynajmniej w moim przypadku co `36` punków), zatem tryb `1` w `tp_fan` odpowiada wartości `36` w
`hwmon`, tryb `2` odpowiada zaś `72` , itd.

Problem w tym, że jeśli zapomnimy ustawić wyższych wartości, a będziemy jednocześnie korzystać z
`hwmon`, to nasz wiatrak może się w ogóle nie włączyć -- potrzebne są wartości zbliżone do
faktycznych wartości przełączających tryb wentylatora. Jeśli teraz mamy `1-7` i korzystamy z
`hwmon` , to system potraktuje je jako `0` i wentylatora nie włączy nawet, gdy ten będzie miał 100°C.
Zatem warto zwrócić uwagę na to jaki sterownik mamy w użyciu.

## Monitorowanie temperatury z thinkfan

Gdy już dostosujemy sobie konfigurację `thinkfan` , to trzeba uruchomić demona, który będzie nam
tym wentylatorem zarządzał w zależności o zmian temperatury. W pakiecie `thinkfan` była zawarta
usługa dla systemd ale zanim ją włączymy, to wypadałoby najpierw przetestować cały mechanizm i
sprawdzić czy aby działa on prawidłowo. W tym celu przechodzimy do terminala i wpisujemy w nim jak
root to poniższe polecenie:

    # thinkfan -n

Ma ono za zadanie uruchomić `thinkfan` i wypisać wszelkie komunikaty na konsoli. Na ekranie powinny
nam zacząć przeskakiwać linijki podobne do tych poniżej:

    sleeptime=5, tmax=40, last_tmax=40, biased_tmax=40 -> fan="level 1"
    sleeptime=5, tmax=38, last_tmax=40, biased_tmax=38 -> fan="level 0"

Jeśli tak się dzieje, to powinniśmy być także w stanie usłyszeć różnicę w pracy wentylatora. Jeśli
nie słyszymy, to prawdopodobnie mamy określoną złą ścieżkę do wentylatora w pliku
`/etc/thinkfan.conf` i jak najszybciej przydałoby się proces `thinkfan` zabić, bo w takiej sytuacji
procesor może się nam spalić jeśli popracuje nieco intensywniej przez dłuższy czas. Jeśli natomiast
słyszymy zmiany prędkości obrotu wiatraka, to znaczy, że `thinkfan` realizuje swoje zadanie
prawidłowo. Możemy zatem włączyć usługę, którą będzie uruchamiana wraz z każdym startem laptopa:

    # systemctl start thinkfan.service
    # systemctl enable thinkfan.service

Od tej pory wentylator w naszym ThinkPad T430 powinien już nieco ciszej się zachowywać. Jeśli
jednak w dalszym ciągu dość mocno hałasuje, to trzeba mu dobrać nieco inne wartości w pliku
`/etc/thinkfan.conf` ale należy uważać. Może i procesor jest w stanie wytrzymać 100°C ale dysk,
który siedzi zaraz obok w zamkniętej obudowie, już takiej temperatury nie wytrzyma. Im wyżej
podniesiemy progi temperatur, tym rzadziej wentylator będzie się załączał i równocześnie
temperatura wewnątrz obudowy laptopa wzrośnie, a więc i dysk twardy będzie się dodatkowo nagrzewał,
zwłaszcza, gdy mamy do czynienia z dyskiem magnetycznym. W przypadku HDD temperatura mocno wpływa
na błędy odczytu i zapisu danych z racji rozszerzania się ścieżek i sektorów.

## Zamiana modułów i zmiana ścieżek

Czasami ścieżki do plików, które trzeba określić w pliku `/etc/thinkfan.conf` stają się błędne po
restarcie maszyny. Dzieje się tak ze względu na obecność kilku modułów, które odpowiedzialne są za
dostarczanie informacji o czujnikach. W tym powyżej opisanym przykładzie mieliśmy do czynienia z
dwoma modułami: `thinkpad_acpi` oraz `coretemp` . Jeśli nie określimy w jakiej kolejności te moduły
będą ładowane na starcie systemu, to raz jeden może zostać załadowany jako pierwszy, a innym razem
ten drugi. Owocować to będzie ciągłą zamianą ścieżek, przez co my będziemy musieli je ręcznie
dostosować w pliku konfiguracyjnym. Jeśli mamy więcej modułów niż jeden, to określmy ich kolejność
w pliku `/etc/modules` :

    thinkpad_acpi
    coretemp


[1]: https://github.com/vmatare/thinkfan
