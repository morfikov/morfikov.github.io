---
author: Morfik
categories:
- Android
date: "2017-01-02T17:19:47Z"
date_gmt: 2017-01-02 16:19:47 +0100
published: true
status: publish
tags:
- smartfon
- lollipop
- zram
title: Przenoszenie danych ze ZRAM do RAM na smartfonie z Androidem
---

Jak często zdarza się wam uruchamiać ponownie telefon czy smartfon? Raczej nikt z nas nie robi tego
zbyt często, tak jak ma to miejsce w przypadku desktopów, laptopów i innych tego typu standardowych
komputerów. System w smartfonie zwykle działa bez resetowania wiele tygodni czy nawet miesięcy, bo
niewiele rzeczy instalujemy, no i praktycznie nic nie aktualizujemy. Dlatego też nie ma potrzeby
uruchamiać ponownie Androida. Niemniej jednak są pewne przypadki, w których taki system jest w
stanie zachowywać się bardzo dziwnie i restart smartfona zwykle poprawia zaistniałe problemy. Chodzi
generalnie o utylizowanie baterii w większym stopniu, czy o ogólne uczucie spowolnienia pracy
systemu. Przyczyn takiego stanu rzeczy może być cała masa ale w tym konkretnym przypadku znaczenie
zdaje się mieć zbyt duże wykorzystanie pamięci RAM. W efekcie Android robi użytek z [urządzenia
ZRAM,](https://www.kernel.org/doc/Documentation/blockdev/zram.txt) które jest niczym innym jak tylko
skompresowanym wycinkiem pamięci operacyjnej, gdzie system stara się upchnąć dane w przypadku
kończenia się zasobów pamięci, a kompresja to przecież bardzo zasobożerny proces. Możemy taki
telefon wyłączać co jakiś czas ale istnieje prostsza metoda wyeliminowania problemu.

<!--more-->
## Urządzenie ZRAM w smartfonach

Mając zainstalowaną dowolną aplikację, która jest nam w stanie pokazać dostępne partycje naszego
smartfona, na liście powinniśmy być w stanie odszukać urządzenie ZRAM. Poniżej przykład z [aplikacji
DiskInfo](https://play.google.com/store/apps/details?id=me.kuder.diskinfo&hl=pl):

![](/img/2017/01/001.zram-swap-smartfon-urzadzenie-disk-info.png#medium)

W przypadku mojego smartfona Neffos C5 MAX, to urządzenie ma pojemność 512 MiB (oznaczone wyżej jako
SWAP). Generalnie tyle można tych danych tutaj upchnąć, choć trzeba uwzględnić stopień kompresji,
przez co faktycznych informacji zawsze wejdzie więcej.

Z biegiem czasu jak korzystamy z telefonu, ta przestrzeń ZRAM zaczyna nam się zapełniać. W jakim
stopniu i jak szybko, to zależy głównie od sposobu wykorzystywania telefonu. Jeśli telefon służy nam
głównie do rozmów i pstrykania fotek, to jest nawet spore prawdopodobieństwo, że urządzenie ZRAM w
ogóle nie będzie wykorzystywane. No, chyba, że nasz smartfon dysponuje niewielką ilością pamięci
operacyjnej, gdzie sam system tuż po uruchomieniu już może zacząć robić użytek z tego wirtualnego
urządzenia.

Tak czy inaczej, w każdym innym przypadku, uruchamiając na smartfonie dodatkowe aplikacje, zwykle
kilka naraz, sprawiamy, że ilość dostępnej wolnej pamięci RAM ulega zmniejszeniu. Im więcej takich
aplikacji będziemy otwierać bez zamykania tych już uruchomionych wcześniej, tym więcej danych
powędruje do urządzenia ZRAM:

![](/img/2017/01/002.zram-swap-smartfon-intensywne-wykorzystanie.png#medium)

Nawet jeśli w późniejszym czasie pozamykamy wszystkie uruchomione przez nas programy, to i tak część
danych w dalszym ciągu pozostanie w formie skompresowanej.

![](/img/2017/01/003.zram-swap-smartfon-zamykanie-aplikacji.png#medium)

Dzieje się tak dlatego, że system w sytuacji wyczerpywania się zasobów pamięci operacyjnej musi
podjąć decyzję, które dane zrzucić do tego urządzenia ZRAM. Zwykle są to informacje zawarte w
obszarach pamięci, do których nie było odwołania przez pewien czas i system uznał je za mniej
potrzebne, przynajmniej w tej konkretnej chwili i to one zostały skompresowane.

Ładując do pamięci kolejne aplikacje, do urządzenia ZRAM mogą powędrować dane programów systemowych,
czyli tych które standardowo działają po załadowaniu się Androida. W efekcie system będzie teraz
odwoływał się do tego urządzenia i przeprowadzał proces dekompresji danych, by móc na nich operować,
a to pociąga większą utylizację procesora, przez co cierpi bateria. Dodatkowo, te zbędne operacje
nie pozostają bez wpływu na responsywność całego systemu.

## Przenoszenie danych ze ZRAM do pamięci RAM

Jak wspomniałem we wstępie, takiemu niepożądanemu stanu rzeczy można zapobiec uruchamiając ponownie
smartfon, tak by wszystkie podstawowe programiki były załadowane bezpośrednio do pamięci RAM, a nie
przesiadywały w urządzeniu ZRAM. Tego typu rozwiązanie nie zawsze jednak jest praktyczne i może
raczej przydać się tylko niezbyt wymagającym użytkownikom telefonów. W przypadku osobników nieco
bardziej zżytych ze smartfonem, ciągłe restartowanie telefonu nie wchodzi praktycznie w ogóle w grę.

Alternatywnym rozwiązaniem jest przeniesienie danych zgromadzonych w ZRAM do pamięci RAM (choć jedne
i drugie dane są w pamięci operacyjnej). Tutaj jednak będzie wymagany ukorzeniony Android (root).
Jeśli mamy zapewniony dostęp root do systemu w naszym telefonie, to wszystko czego nam trzeba, to
ścieżka do urządzenia ZRAM. Standardowo jest to `/dev/block/zram0` . To urządzenie musimy
zdezaktywować czasowo. To właśnie podczas tego procesu, wszystkie dane ze ZRAM zostaną
zdekompresowane i przeniesione do standardowej pamięci RAM.

Odpalamy zatem terminal, wszystko jedno czy to w telefonie, czy też na komputerze (via `adb` ) i
logujemy się na użytkownika root. Następnie wpisujemy w terminalu poniższe polecenie:

    # swapoff /dev/block/zram0

W zależności od ilości danych w tym urządzeniu blokowym, cały proces będzie trwał dłużej lub krócej
ale zwykle nie więcej niż kilkadziesiąt sekund. Gdy to urządzenie ZRAM zostanie wyłączone, musimy je
aktywować ponownie:

    # swapon /dev/block/zram0

I to w zasadzie wszystko. Nic więcej nie trzeba robić, no może za wyjątkiem monitorowania ilości
danych w tym urządzeniu, tak by co jakiś czas je opróżniać.
