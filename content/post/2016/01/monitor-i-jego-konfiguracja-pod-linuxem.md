---
author: Morfik
categories:
- Linux
date: "2016-01-04T18:02:20Z"
date_gmt: 2016-01-04 17:02:20 +0100
published: true
status: publish
tags:
- xserver
- monitor
- openbox
- debian
title: Monitor i jego konfiguracja pod linux'em
---

Obecnie w linux'ach spora cześć sprzętu jest rozpoznawana prawidłowo, a my nie musimy poświęcać
czasu na dodatkową konfigurację. Domyślne ustawienia sprawdzają się praktycznie za każdym razem, gdy
w grę nie wchodzi nic bardziej zaawansowanego. W tym wpisie rzucimy okiem na konfigurację Xserver'a,
która dotyczyć będzie wyświetlanego obrazu na monitorze. Pośrednio też będziemy musieli
skonfigurować sobie kartę graficzną, bo to ona jest odpowiedzialna w dużej mierze za to, co jest
odbierane przez nasz monitor.

<!--more-->
## Konfiguracja Xserver'a

Zainstalowanie sterowników do karty graficznej nie zawsze wystarcza. Zwłaszcza w przypadku, gdy mamy
do czynienia z tymi o zamkniętym kodzie źródłowym. Te otwarte odpowiedniki powinny działać
automatycznie. Tak czy inaczej, poniżej jest przykładowa konfiguracja, którą można obrać sobie jako
punkt wyjścia. Tworzymy zatem plik `/etc/X11/xorg.conf.d/20-monitor.conf` , w którym to dodajemy tę
poniższą treść:

    Section "Device"
          Identifier  "Device0"
          Driver      "intel"
          BusID       "PCI:0:2:0"
          Option      "Backlight" "acpi_video0"
    EndSection

    Section "Monitor"
          Identifier   "LVDS1"

    #     DisplaySize 344 193
          Modeline "1366x768x60.0-manual"  69.30  1366 1398 1422 1432  768 771 775 806  -HSync +Vsync
          Option "PreferredMode" "1366x768x60.0-manual"

    #     Option      "VendorName"      ""
    #     Option      "ModelName"       ""
          Option      "DPMS"            "true"
    #     Option      "HorizSync"       "30-81"
    #     Option      "VertRefresh"     "56-75"
    #     Option      "Position"  "0 0"
    #     Option      "LeftOf"    "VGA1"
    #     Option      "Rotate"    "normal"
    #     Option      "Gamma"     "1.0"
    #     Option      "Enable"    "true"
    #     Option      "Ignore"    "false"
    #     Option      "Primary"   "true"
    EndSection

    Section "Screen"
        Identifier     "Screen0"
        Device         "Device0"
        Monitor        "LVDS1"
        DefaultDepth    24
        SubSection     "Display"
                Depth     24
                Modes     "1366x768x60.0-manual"
        EndSubSection
    EndSection

Wyżej mamy określone trzy sekcje: `Monitor` , `Device` oraz `Screen` . By ta konfiguracja byłą
spójna i miała sens, musimy zwracać uwagę na identyfikatory, których używamy. W sekcji `Screen` ,
na pozycjach `Device` oraz `Monitor` są wartości ustawione w `Identifier` w poprzednich dwóch
sekcjach. Sekcja `Monitor` odpowiada za konfigurację monitora, zaś sekcja `Device` za konfigurację
karty graficznej.

### Różnica między Monitor i Screen

Kluczowe też jest zrozumienie różnicy między użytymi wyżej pojęciami `Monitor` i `Screen` .
`Monitor` to fizyczny obiekt, który zwykle swoi na biurku. Natomiast `Screen` wiąże ze sobą monitor
z kartą graficzną. Jest to generalnie obraz karty graficznej, który musi zostać gdzieś wyświetlony.
Do karty graficznej może być podpiętych szereg monitorów. Dlatego też wymagana jest konfiguracja
każdego Screen'a. Mając skonfigurowanych kilka Screen'ów, możemy je ze sobą łączyć i otrzymać w ten
sposób większy obraz, np. jego cześć może być wyświetlana na jednym monitorze, zaś druga część na
drugim monitorze, itp. W taki sposób, mimo, że pracujemy na dwóch fizycznych monitorach, to tak
naprawdę mamy do dyspozycji większą powierzchnię pulpitu.

Parametry dla poszczególnych monitorów powinny być w miarę zbliżone, a najlepiej by były takie same,
ewentualnie proporcjonalnie większe lub mniejsze. Wtedy nie ma większego problemu ze skalowaniem
obrazu czy rozdzielaniem go na poszczególne monitory. Jeśli parametry sprzętu różnią się, to wtedy
Screen może wyglądać tak jak ten poniżej:

![]({{< baseurl >}}/img/2016/01/1.monitor-screen-arandr.png#medium)

W taki sposób nie damy rady, np. rozdzielić filmu na oba monitory bez obcinania kawałka obrazu, lub
całkowitego wypełnienia obu monitorów. Podobnie sprawa ma się w przypadku powielania obrazu na oba
te urządzenia, bo na tym mniejszym monitorze, obraz będzie lekko obcięty. Niemniej jednak, jeśli
potrzebujemy jedynie większej przestrzeni pulpitu w celu rozmieszczenia większej ilości okien, to
różnice w parametrach monitorów nie mają większego znaczenia.

### Konfiguracja sekcji Device

W zależności od posiadanej karty graficznej, będziemy mieć do wyboru różne sterowniki. Te z kolei
podzielić można na dwie grupy: opensource oraz proprietary (własnościowe). Pierwsza grupa zwykle
cechuje się sporo gorszą wydajnością niż ta druga. Z kolei jeśli chodzi o sterowniki własnościowe,
to w debianie one oficjalnie nie są wspierane i generują całą masę problemów.

Po zainstalowaniu odpowiednich sterowników, będziemy musieli skonfigurować sobie sekcję `Device` w
utworzonym wcześniej pliku konfiguracyjnym. [W zależności od wybranych
sterowników](https://wiki.debian.org/Xorg), wartość parametru `Driver` ulegnie zmianie. To, jakie
wartości mogą zostać określone, można znaleźć w katalogu `/usr/lib/xorg/modules/drivers/` . Mamy tam
przykładowo `intel_drv.so` . Obcinamy `_drv.so` , a pozostałą cześć wpisujemy w linijce z `Driver` .

Przyjrzyjmy się zatem bliżej sekcji `Device` . Dla przypomnienia wygląda ona mniej więcej tak:

    Section "Device"
          Identifier  "Device0"
          Driver      "intel"
          BusID       "PCI:0:2:0"
          Option      "Backlight" "acpi_video0"
    EndSection

Wszystkie możliwe do skonfigurowania opcje są opisane w [man
xorg.conf](ftp://www.x.org/pub/X11R6.8.2/doc/xorg.conf.5.html#sect7). Tutaj opiszę tylko te wyżej
wymienione:

  - `Identifier` to identyfikator. Można wpisać dowolną frazę.
  - `Driver` odpowiada za użyty sterownik. Zwykle tylko tę opcję będziemy musieli dostosować.
  - `BusID` określa adres karty graficznej.

Możemy także definiować szereg opcji za pomocą `Option` . Wyżej mamy zdefiniowany jedynie
`"Backlight"` , który odpowiada za manipulację jasnością matrycy w laptopie. Czasem w katalogu
`/sys/class/backlight/` może znaleźć się kila dowiązań i system może korzystać nie z tego modułu co
potrzeba pociągając za sobą szereg problemów z zapisywaniem ustawień jasności monitora. Precyzując
tę opcję tutaj, możemy określić z którego sterownika system ma korzystać.

#### Jak ustalić BusID karty graficznej

Jeśli chodzi zaś o `BusID` , to zwykle system potrafi bez problemu określić ścieżkę do karty
graficznej. Niemniej jednak, nie stanowi problemu odszukanie jej w sposób manualny. Zapis jaki
musimy podać Xserver'owi ma być w formie `PCI:bus:device:function` . Potrzebne wartości możemy
odczytać z wyjścia `lspci` , przykładowo:

    $ lspci | grep VGA
    00:02.0 VGA compatible controller: Intel Corporation Core Processor Integrated Graphics Controller (rev 02)

W [man lspci](http://manpages.ubuntu.com/manpages/wily/en/man8/lspci.8.html) możemy odszukać
informację, że ten numerek na początku ma format `[domain:]bus:device.function` , czyli pasuje do
tego, który mamy podać Xserver'owi, z tym, że `.` zamieniamy na `:` . Dodatkowo, wartości na wyjściu
`lspci` są w hexach. Natomiast Xserver korzysta z systemu dziesiętnego i te wartości trzeba pierw
przekonwertować. Mamy zatem trzy pozycje: `00` -> `0` , `02` -> `2` oraz `0` -> `0` . W taki
sposób otrzymujemy `0:2:0` , gdzie na początku doklejamy `PCI:` i tę frazę dodajemy do pliku
konfiguracyjnego.

### Konfiguracja sekcji Monitor

Sekcja `Monitor` jest z reguły opcjonalna, gdy mamy do czynienia z jednym monitorem. Zwykle
wszystkie jego parametry są wykrywane poprawnie. Czasem jednak chcielibyśmy zmienić natywną
rozdzielczość monitora, czy też przyciemnić go nieco, lub też rozjaśnić. Być może orientacja obrazu
jest nie taka jak trzeba. To wszystko jesteśmy w stanie skonfigurować w tej sekcji. Dla
przypomnienia, wygląda ona mniej więcej tak:

    Section "Monitor"
          Identifier   "LVDS1"

    #     DisplaySize 344 193
          Modeline "1366x768x60.0-manual"  69.30  1366 1398 1422 1432  768 771 775 806  -HSync +Vsync
          Option "PreferredMode" "1366x768x60.0-manual"

    #     Option      "VendorName"      ""
    #     Option      "ModelName"       ""
          Option      "DPMS"            "true"
    #     Option      "HorizSync"       "30-81"
    #     Option      "VertRefresh"     "56-75"
    #     Option      "Position"  "0 0"
    #     Option      "LeftOf"    "VGA1"
    #     Option      "Rotate"    "normal"
    #     Option      "Gamma"     "1.0"
    #     Option      "Enable"    "true"
    #     Option      "Ignore"    "false"
    #     Option      "Primary"   "true"
    EndSection

Poniżej znajduje się wyjaśnienie użytych opcji, zaś opis wszystkich można naturalnie odszukać w [man
xorg.conf](ftp://www.x.org/pub/X11R6.8.2/doc/xorg.conf.5.html#sect9):

  - `Identifier` to identyfikator. Można wpisać dowolną frazę.
  - `DisplaySize` określa wymiary (mm) tej części monitora, na której jest wyświetlany obraz. Gdy
    ten parametr zostanie określony, na jego podstawie zostaną także wyliczone DPI.
  - `Modeline` odpowiada za [informacje EDID](https://pl.wikipedia.org/wiki/EDID).
  - `PreferredMode` określa preferowany tryb pracy monitora.
  - `VendorName` oraz `ModelName` określają producenta i model monitora.
  - `DPMS` włącza funkcje DPMS ([Display Power Management
    Signaling](http://www.tldp.org/HOWTO/Battery-Powered/displaytypes.html)) dla monitora.
  - `HorizSync` odpowiada za częstotliwość odchylania poziomego lub jej zakres.
  - `VertRefresh` odpowiada za częstotliwość synchronizacji pionowej lub jej zakres.
  - `Position` określa pozycję lewego górnego rogu obrazu monitora.
  - `LeftOf` , `RightOf` , `Below` i `Above` określają położenie względem innych monitorów.
  - `Rotate` umożliwia obrócenie obrazu o 90 (left), 180 (inverted) i 270 (right) stopni.
  - `Gamma` odpowiada za jasność obrazu. Zakres od 0.1 to 10.0 . Można także podać osobne wartości
    dla Red, Green, Blue.
  - `Enable` jeśli ustawiony, domyślnie włącza dany monitor.
  - `Ignore` jeśli ustawiony, ten monitor będzie ignorowany w całości.
  - `Primary` jeśli ustawiony, określa monitor jako główny.

Nie wszystkie opcje w tej sekcji mogą być wspierane przez dany sterownik urządzenia.

#### Modeline

Przydałoby się jeszcze powiedzieć kilka słów na temat `Modeline` . Generalnie rzecz biorąc,
wszystkie potrzebne wartości możemy odczytać z logu Xserver'a. Poniżej przykład:

    [ 37422.916] (II) intel(0): clock: 69.3 MHz   Image Size:  344 x 193 mm
    [ 37422.916] (II) intel(0): h_active: 1366  h_sync: 1398  h_sync_end 1422 h_blank_end 1432 h_border: 0
    [ 37422.916] (II) intel(0): v_active: 768  v_sync: 771  v_sync_end 775 v_blanking: 806 v_border: 0

Na wiki znalazłem rozpiskę na temat [formatu
Modeline](https://en.wikipedia.org/wiki/XFree86_Modeline). Wygląda on mniej więcej tak:

    pclk hdisp hsyncstart hsyncend htotal vdisp vsyncstart vsyncend vtotal [flags]
    Flags (optional): +HSync, -HSync, +VSync, -VSync, Interlace, DoubleScan, CSync, +CSync, -CSync

    Option      "Modeline" "1366x768x60.0"  69.30  1366 1398 1422 1432  768 771 775 806  -HSync -Vsync
                                (Label)     (clk)     (x-resolution)     (y-resolution)
                                              |
                                     (pixel clock in MHz)

Jeśli nie chce nam się ręcznie uzupełniać tych wartości w oparciu o log Xserver'a, to zawsze możemy
skorzystać z narzędzi `cvt` lub `gtf` . Częstotliwość odświeżania tego monitora wynosi 60Hz. Wiemy
to z powyższych parametrów. Wzór na wyliczenie częstotliwości jest następujący:
69.30MHz/(1432*806)=69300000/(1432*806)=60.04Hz.

Jak możemy wyczytać w podlinkowanym wyżej wpisie, ręczne ustawianie `Modeline` jest już
przestarzałe, bo Xserver sam wylicza konkretne wartości podczas swojego startu w oparciu o
[zapytania EDID](https://en.wikipedia.org/wiki/Extended_Display_Identification_Data) oraz pozostałą
konfigurację określoną w pliku konfiguracyjnym. Jeśli jednak chcemy ręcznie dodać nowy modeline, to
musimy także zainteresować się sekcją `Screen` .

### Konfiguracja sekcji Screen

Pozostała nam jeszcze jedna sekcja do skonfigurowania i jest nią `Screen` . Jak już wspomnieliśmy,
ma ona na celu spięcie monitora z kartą graficzną. Jakby ktoś zapomniał jak ona wygląda, to poniżej
przypomnienie:

    Section "Screen"
        Identifier     "Screen0"
        Device         "Device0"
        Monitor        "LVDS1"
        DefaultDepth    24
        SubSection     "Display"
                Depth     24
                Modes     "1366x768x60.0-manual"
        EndSubSection
    EndSection

Wszystkie parametry, jakie możemy skonfigurować w tej sekcji, również można odszukać w [man
xorg.conf](ftp://www.x.org/pub/X11R6.8.2/doc/xorg.conf.5.html#sect11). Poniżej wyjaśnienie tych
opcji, które zostały użyte wyżej:

  - `Identifier` to identyfikator.
  - `Device` musi pasować do pola `Identifier` w sekcji `Device` .
  - `Monitor` musi pasować do pola `Identifier` w sekcji `Monitor` .
  - `DefaultDepth` odpowiada za domyślną głębię kolorów.

Mamy również określoną podsekcję z dwoma parametrami: `Depth` oraz `Modes` . Ten pierwszy jest
podobny do `DefaultDepth` , zaś `Modes` przydaje się jedynie w przypadku definiowania własnych
wartości dla modeline.

### Display Power Management Signaling (DPMS)

Jeśli włączyliśmy w opcjach monitora DPMS, to potrzebujemy dodatkowej konfiguracji, która
skonfiguruje nam zarządzenie energią tego urządzenia. W tym celu stwórzmy sobie dwie dodatkowe
sekcje: `ServerLayout` oraz `ServerFlags` . Coś na wzór tych poniższych:

    Section "ServerLayout"
          Identifier "Main"
          Screen      0 "Screen0"
    EndSection

    Section "ServerFlags"
          Option "DefaultServerLayout" "Main"
          Option "BlankTime" "10"
          Option "StandbyTime" "10"
          Option "SuspendTime" "10"
          Option "OffTime" "10"
    EndSection

Wartość opcji `Screen` w sekcji `ServerLayout` musi pasować do wartości `Identifier` w sekcji
`Screen` , którą zdefiniowaliśmy wyżej. Dodatkowo wartość opcji `DefaultServerLayout` ma wskazywać
na `Identifier` w sekcji `ServerLayout` . W ten sposób jesteśmy w stanie definiować szereg flag dla
Xserver'a. Poniżej wyjaśnienie użytych opcji:

  - `BlankTime` to czas wygaszenia monitora.
  - `StandbyTime` to czas, po którym monitor przejdzie w stan czuwania.
  - `SuspendTime` to czas, po którym monitor przejdzie w stan uśpienia.
  - `OffTime` to czas, po którym nastąpi wyłączenie monitora.

Wszystkie określone wyżej czasy są w minutach. Zatem monitor powinien się wyłączyć po 10 minutach.
Różnica pomiędzy poszczególnymi trybami pracy jest w [poborze
prądu](http://www.shallowsky.com/linux/x-screen-blanking.html). Konfigurację DPMS możemy sprawdzić
za pomocą `xset` , przykładowo:

    $ xset -q
    ...
    Screen Saver:
      prefer blanking:  yes    allow exposures:  yes
      timeout:  600    cycle:  600
    ...
    DPMS (Energy Star):
      Standby: 600    Suspend: 600    Off: 600
      DPMS is Enabled
      Monitor is On

Z kolei tutaj czasy są w sekundach, a 600 sekund to 10 minut.
