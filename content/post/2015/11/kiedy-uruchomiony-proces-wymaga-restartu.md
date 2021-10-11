---
author: Morfik
categories:
- Linux
date: "2015-11-13T14:22:21Z"
date_gmt: 2015-11-13 13:22:21 +0100
published: true
status: publish
tags:
- bezpieczeństwo
GHissueID: 244
title: Kiedy uruchomiony proces wymaga restartu
---

Linux słynie z tego, że nie są wymagane w nim częste restarty całego systemu operacyjnego. Nie ma
przy tym znaczenia czy aktualizujemy jakieś oprogramowanie, czy też wgrywamy nową wersję kernela.
Jeśli by to przenieść na środowisko windowsa, to tam system jest w stanie się automatycznie
zresetować kilka razy tylko podczas samego procesu aktualizacji. Można zatem kwestionować zasadność
twierdzenia, że linux nie wymaga restartu. Może nie koniecznie jesteśmy zmuszeni do dokonania
restartu w danej chwili, tak jak to ma miejsce w przypadku windowsa, ale czy aby na pewno po
instalacji jakichś pakietów w systemie, każdy proces powinien w dalszym ciągu działać bez restartu?
Na to pytanie postaramy się odpowiedzieć w tym wpisie.

<!--more-->
## To nie system wymaga restartu ale konkretny proces

Aktualizacje oprogramowania w obecnych czasach do podstawa. Systemy operacyjne potrafią te
aktualizacje pobierać i instalować automatycznie, o ile oczywiście posiadają dostęp do internetu.
Częstość aktualizacji się waha. Jedni użytkownicy aktualizują swoje systemy parę razy dziennie,
inni zaś parę razy w roku. W przypadku gdy nie instalujemy nic nowego, to faktycznie linux może
pracować naprawdę długo i nie trzeba go restartować w żaden sposób.

Problem zaczyna się w momencie gdy wgrywamy, np. poprawki bezpieczeństwa, czy też nowsze wersje
pakietów, które wcześniej mieliśmy już w systemie zainstalowane. W takim przypadku podmienianych
jest szereg plików wymaganych do poprawnego działania jakiegoś procesu. By program mógł się
uruchomić, trzeba załadować szereg rzeczy do pamięci. Gdy teraz aktualizujemy system, to szereg
plików ulega zmianie. Logiczne zatem jest, że pewne procesy trzeba ponownie uruchomić, by załadowały
sobie te nowsze pliki do pamięci. Jeśli tego nie zrobimy, to dany proces niekoniecznie będzie się
zachowywał tak jak powinien i na pewno prędzej czy później napotka jakiś błąd w odwołaniu do
nieistniejącego na dysku pliku. Taki proces może działać ale gdy w grę wchodzą poprawki
bezpieczeństwa, to nie zostaną one uwzględnione i ten proces w dalszym ciągu będzie zawierał
dziury, które zostały załatane w nowszej wersji konkretnego programu czy biblioteki.

Skąd zatem mamy wiedzieć, który proces w systemie wymaga restartu? Dobrą reguła jest restart całego
systemu po jego aktualizacji. W ten sposób mamy pewność, że każdy proces jest aktualny ale nie
zawsze taki restart może wchodzić w grę. Innym wyjściem jest restart środowiska graficznego. Jeszcze
innym jest ręczna próba ustalenia czy dany proces wymaga restartu przez wykorzystanie do tego celu
narzędzia `checkrestart` , który w debianie jest dostępny w pakiecie
[debian-goodies](https://packages.debian.org/pl/sid/debian-goodies).

## Narzędzie checkrestart

Jak możemy wyczytać w [man
checkrestart](http://manpages.ubuntu.com/manpages/wily/man1/checkrestart.1.html) , to narzędzie
używane jest czasem w audytach bezpieczeństwa pod kątem odnalezienia procesów, które korzystają z
nieaktualnych bibliotek. Jest też informacja, by nie ufać i nie polegać na `checkrestart` w 100%, bo
może on dawać fałszywe rezultaty. Tak czy inaczej, by uzyskać listę procesów, które należy
zrestartować po procesie aktualizacji, wpisujemy w terminalu to poniższe polecenie:

    # checkrestart -v
    Found 10 processes using old versions of upgraded files
    (8 distinct programs)
    ...
    [DEBUG] Process /lib/systemd/systemd-udevd (PID: 396)
    List of deleted files in use:
            /lib/modules/4.2.0-1-amd64/modules.symbols.bin
            /lib/modules/4.2.0-1-amd64/modules.alias.bin
            /lib/modules/4.2.0-1-amd64/modules.dep.bin
            /lib/modules/4.2.0-1-amd64/modules.builtin.bin

Widzimy zatem na powyższym przykładzie, że proces `systemd-udevd` korzysta z kilku plików, które
zostały usunięte. Termin "usunięte" w tym przypadku nie oznacza, że obecnie tych plików nie ma na
dysku. Jest to jedynie podpowiedź, że te pliki zostały prawdopodobnie wygenerowane na nowo, a to z
kolei automatycznie implikuje usunięcie starych wersji. I tak się stało w istocie, bo te powyższe
pliki zostały wygenerowane podczas aktualizacji pakietu `sysdig-dkms`, który wywołał [polecenie
depmod](http://manpages.ubuntu.com/manpages/wily/man5/modules.dep.5.html) przy instalowaniu modułu
kernela. Wszystkie powyżej wylistowane procesy należy uruchomić ponownie. W większości przypadków
mamy w systemie do tego celu odpowiednie usługi, które także zostaną wypisane w wyjściu
`checkrestart` .
