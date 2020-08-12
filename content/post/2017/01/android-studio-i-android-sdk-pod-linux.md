---
author: Morfik
categories:
- Android
date: "2017-01-29T18:29:36Z"
date_gmt: 2017-01-29 17:29:36 +0100
published: true
status: publish
tags:
- debian
- smartfon
- lollipop
- marshmallow
title: Android Studio i Android SDK pod linux
---

Rozpoczynając przygodę z Androidem (tylko taką nieco bardziej deweloperską) trzeba posiadać w
systemie szereg niezbędnych narzędzi. Chodzi tutaj oczywiście o Android SDK. Metod na instalację
tego pakietu na linux'ie, a konkretnie w dystrybucji Debian, jest co najmniej kilka. Chodzi o to, że
Google udostępnia paczkę `.zip` z Android SDK, którą można pobrać sobie z oficjalnej strony
Androida. Dodatkowo, na tej samej stronie mamy coś o nazwie Android Studio, które również jest w
stanie nam potrzebne narzędzia dostarczyć. Poza tym, te narzędzia można także skompilować sobie ze
źródeł Androida, jak i również zainstalować bezpośrednio z repozytorium samego Debiana. Niemniej
jednak, część z tych sposobów nie jest zbytnio wygodna, a pozostała część zakłada, że korzystamy z
najnowszej wersji Androida (obecnie Nougat). A co w przypadku, gdybyśmy chcieli operować na
Androidzie 5.1 (Lollipop) czy 6.0 (Marshmallow)? Jak zainstalować pasujące wersje narzędzi, by nic
nam się nie gryzło ze sobą?

<!--more-->
## OpenJDK Runtime Environment dla Lollipop i Marshmallow

Przed przystąpieniem do jakichkolwiek prac i budowania czegokolwiek, upewnijmy się, że mamy
zainstalowany kompilator `javac` . Trzeba też doinstalować OpenJDK Runtime Environment, z tym, że w
odpowiedniej wersji i nie zawsze jest to wersja najnowsza. Wszystko zależy od wersji Androida. Dla
przykładu Android 5.1 (Lollipop) oraz 6.0 (Marshmallow) wymagają OpenJDK w wersji 1.7 . W przypadku,
gdy mamy nowszą wersję OpenJDK, to zostanie nam zwrócony poniższy
    komunikat:

    Your version is: openjdk version "1.8.0_111" OpenJDK Runtime Environment (build 1.8.0_111-8u111-b14-3-b14) OpenJDK 64-Bit Server VM (build 25.111-b14, mixed mode).
    The required version is: "1.7.x"

Trzeba zatem postarać się o wersję 1.7 , która w Debianie siedzi w pakiecie `openjdk-7-jdk` i to ten
pakiet musimy zainstalować u siebie w systemie.

## Narzędzia deweloperskie dla Androida

[W Debianie panuje straszny
nieporządek](https://wiki.debian.org/AndroidTools#Android.27s_upstream_version_names) w pakietach
mających dostarczyć Androidowe narzędzia i w zasadzie nie udało mi się wypracować działającego
rozwiązania korzystając z tych pakietów. Na wypadek, gdyby w niedalekiej przyszłości Debian ogarnął
swoje pakiety, to warto zaznaczyć, że te narzędzia z repozytorium są instalowane do katalogu
`/usr/lib/android-sdk/` . Ścieżka do narzędzi zarówno w przypadku manualnej instalacji, jak i przez
pakiety z repozytorium Debiana, będzie miała znaczenie w późniejszej części artykułu.

Warto też dodać, że w przypadku instalacji pakietów z repozytorium Debiana, trzeba będzie także
doinstalować trochę zależności. Jeśli nie chcemy zaśmiecać sobie systemu, to zawsze możemy [stworzyć
i skonfigurować kontener LXC]({{< baseurl >}}/post/konfiguracja-kontenerow-lxc/), który będziemy
wykorzystywać jedynie w celu budowania modułów czy całego Androida.

Mając na uwadze powyższe informacje, nie będziemy tutaj instalować Androidowych narzędzi z
repozytorium Debiana. Dlatego też postanowiłem wybrać inną drogę na pozyskanie wymaganych rzeczy.
Potrzebne narzędzia pobierzemy sobie bezpośrednio ze strony Androida. Mamy tam do wyboru Android
Studio lub Android SDK ([do pobrania stąd](https://developer.android.com/studio/index.html)).

### Android SDK

Dla naszych potrzeb wystarczy ta druga opcja i w zasadzie po wypakowaniu paczki z Android SDK trzeba
uruchomić skrypt `android` , który znajduje się w katalogu `android-sdk-linux/tools/` . Za jego
sprawą zostanie uruchomiony Android SDK Manager, który da nam możliwość zainstalowania min. Tools,
Platform-tools oraz Build-tools w zależności od potrzebnego nam API Androida. Ja potrzebuję zarówno
Androida 5.1 (API 22) jak i Androida 6.0 (API 23):

![]({{< baseurl >}}/img/2017/01/001.android-studio-sdk-narzedzia.png)

### Android Studio

Jeśli zaś chodzi o Android Studio, to jest to pełne środowisko dla programistów. We wcześniejszych
wersjach Androida wykorzystywany był Eclipse z dodatkiem ADT (Android Developer Tools). W nowszych
wersjach Androida nie korzysta się z Eclipse ADT, bo nie jest on już rozwijany i został on w pełni
zastąpiony właśnie przez Android Studio, który nieco upraszcza operowanie na Androidzie. W zasadzie
wszystkie te rzeczy, które można zrobić za pomocą Android SDK Manager, można także zrobić w Android
Studio.

Póki co w dystrybucji Debian nie ma stosownego pakietu i by Android Studio zainstalować w tym
systemie, musimy pobrać paczkę `.zip` ze strony Androida. Paczkę naturalnie wypakowujemy i
przechodzimy do katalogu `android-studio-ide-linux/android-studio/bin/` . Z tego katalogu
uruchamiamy plik `studio.sh` . W ten sposób będziemy w stanie pobrać szereg niezbędnych nam rzeczy.

![]({{< baseurl >}}/img/2017/01/003.android-studio-sdk-narzedzia.png)

![]({{< baseurl >}}/img/2017/01/004.android-studio-sdk-narzedzia.png)

![]({{< baseurl >}}/img/2017/01/006.android-studio-sdk-narzedzia.png)

Wszystkie te składniki będą pobierane do katalogu `/tmp/` . Niektóre z nich ważą ponad 0,5 GiB, a do
tego instalator będzie chciał jeszcze te paczki w tym katalogu `/tmp/` wypakować. W niektórych
linux'ach może nam zwyczajnie zabraknąć miejsca na te pliki tymczasowe, bo od jakiegoś czasu katalog
`/tmp/` jest montowany w pamięci operacyjnej RAM i zwykle maksymalny rozmiar tego katalogu to 50%
pamięci. Dlatego też przed instalacją tych powyższych narzędzi upewnijmy się, że mamy wystarczającą
ilość wolnego miejsca przeznaczoną na katalog `/tmp/` .

![]({{< baseurl >}}/img/2017/01/007.android-studio-sdk-narzedzia.png)

Po puszczeniu instalatora, rozpocznie się pobieranie wszystkich uprzednio zaznaczonych rzeczy.
Będzie tam również zawarty Android SDK i nie będziemy musieli go instalować oddzielnie. Jedyny
problem w tym, że ten Android Studio zakłada, że zamierzamy operować na najnowszej wersji Androida,
co nie zawsze jest prawdą. W tym przypadku potrzebne nam są wersje 5.1 (Lollipop) i 6.0
(Marshmallow) i trzeba będzie dociągnąć stosowne wersje narzędzi dla tych Androidów po ukończeniu
procesu instalacyjnego.

![]({{< baseurl >}}/img/2017/01/008.android-studio-sdk-narzedzia.png)

Gdy proces instalacyjny dobiegnie końca, uruchamiamy Android Studio i przechodzimy do Configure =\>
SDK Manager. Tam z kolei zaznaczamy narzędzia pasujące do API interesujących nas Androidów:

![]({{< baseurl >}}/img/2017/01/009.android-studio-sdk-narzedzia.png)

![]({{< baseurl >}}/img/2017/01/010.android-studio-sdk-narzedzia.png)

Czekamy aż proces dobiegnie końca. W moim przypadku, katalog z tymi narzędziami zajmuje nieco ponad
4 GiB, także trochę tego jest.

![]({{< baseurl >}}/img/2017/01/011.android-studio-sdk-narzedzia.png)

### Własnoręczna kompilacja narzędzi

Alternatywą dla instalacji tych paczek/pakietów zawierających Androidowe narzędzia jest własnoręczne
skompilowanie SDK ze źródeł Androida (pozyskiwanie źródeł Androida jest poza zakresem tego
artykułu). W tym przypadku wszystkie niezbędne rzeczy będą w kompatybilnych wersjach z Androidem,
na którego źródłach będziemy operować. By zbudować narzędzia ze źródeł Androida, przechodzimy do
głównego katalogu z repozytorium GIT i wydajemy w terminalu poniższe polecenia:

    $ . ./build/envsetup.sh
    $ lunch sdk-eng
    $ make sdk

Przy budowaniu SDK ze starszych źródeł Androida na nowszych systemach linux może pojawić się błąd
uniemożliwiający ukończenie procesu. Objawia się on komunikatami `unsupported reloc 43` . By
poprawić ten problem trzeba zmodyfikować plik `build/core/clang/HOST_x86_common.mk` i [zaaplikować
poniższego
patch'a](https://oopsmonk.github.io/blog/2016/06/07/android-build-error-on-ubuntu-16-04-lts):

    diff --git a/core/clang/HOST_x86_common.mk b/core/clang/HOST_x86_common.mk
    index 0241cb6..77547b7 100644
    --- a/core/clang/HOST_x86_common.mk
    +++ b/core/clang/HOST_x86_common.mk
    @@ -8,6 +8,7 @@ ifeq ($(HOST_OS),linux)
     CLANG_CONFIG_x86_LINUX_HOST_EXTRA_ASFLAGS := \
       --gcc-toolchain=$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG) \
       --sysroot=$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/sysroot \
    +  -B$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/x86_64-linux/bin \
       -no-integrated-as

     CLANG_CONFIG_x86_LINUX_HOST_EXTRA_CFLAGS := \

## Zmienne środowiskowe

Niezależnie od wybranego sposobu instalacji, narzędzia deweloperskie powinniśmy mieć już wgrane w
systemie i ulokowane w znanych nam katalogach. W tym przypadku są to `Android/SDK/build-tools/` ,
`Android/SDK/tools/` oraz `Android/SDK/platform-tools/` . Te narzędzia muszą się znaleźć w zmiennej
PATH. Musimy także wskazać położenie katalogu z Android SDK. Robimy to przez wyeksportowanie
poniższych zmiennych:

    $ export ANDROID_HOME=/media/Kabi/Android/SDK
    $ export PATH=$ANDROID_HOME/tools:$PATH
    $ export PATH=$ANDROID_HOME/platform-tools:$PATH

Oczywiście takie eksportowanie zmiennych za każdym razem, gdy tylko chcemy bawić się Androidem jest
pozbawione sensu. Lepiej jest te zmienne dopisać sobie do pliku `~/.bashrc` czy `~/.zshrc` , tak by
były one inicjowane za każdym razem, gdy odpalamy terminal.

## Źródła Androida

W zasadzie ten artykuł jest na temat przygotowania Androidowych narzędzi deweloperskich do pracy pod
linux i ten cel został osiągnięty. Niemniej jednak, to nie koniec, bo przecież potrzebne nam są
jeszcze źródła Androida, na których będziemy pracować. Problem w tym, że temat dostosowywania
źródeł, ich budowy czy w ogóle cały aspekt dotyczący operowania na źródłach Androida jest
zagadnieniem ździebko skompilowanym. Nie chodzi tutaj tylko o źródła tego Androida od Google ale
również o wszelkie Custom ROM'y czy też obrazy recovery. Na pewno w następnych artykułach te tematy
zostaną poruszone.
