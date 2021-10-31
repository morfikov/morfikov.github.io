---
author: Morfik
categories:
- Linux
date: "2016-06-07T17:07:54Z"
date_gmt: 2016-06-07 15:07:54 +0200
published: true
status: publish
tags:
- system-plików
- pliki
- foldery
- ext4
GHissueID: 355
title: Data i czas utworzenia pliku w linux'ie (crtime)
---

Systemy plików, które wykorzystujemy na partycjach swoich dysków, zawierają metadane opisujące
pliki. Domyślnym systemem plików w większości linux'ów (do nich zalicza się też debian) jest EXT4.
Gdy listujemy pliki przy pomocy narzędzia `ls` , jesteśmy w stanie uzyskać szereg informacji
opisujących konkretny plik. Mamy tam min. czas ostatniej modyfikacji i-węzła (i-node), czyli tzw.
`ctime` . Narzędzia takie jak `stat` są w stanie podać również inne czasy, tj. `atime` (ostatni czas
dostępu do pliku) oraz `mtime` (ostatni czas modyfikacji pliku). Jednak żaden z tych powyższych nie
przekłada się na czas utworzenia pliku. Co prawda, po stworzeniu pliku, wszystkie te czasy są ze
sobą zsynchronizowane ale po przeprowadzeniu szeregu różnych operacji na tym pliku, problematyczne
może być ustalenie pierwotnej daty jego utworzenia. Celem tego artykułu jest pokazanie, jak przy
pomocy `debugfs` uzyskać czas utworzenia dowolnie wskazanego pliku w systemie plików EXT4.

<!--more-->
## Czas utworzenia pliku w stat

Z tego co wyczytałem na necie, linux oferuje trzy znaczniki czasu, o których była mowa we wstępie.
Niemniej jednak, jeśli popatrzymy sobie na wyjście narzędzia `stat` , to mamy tam jedna pozycję,
która nie jest określona. Poniżej jest przykład:

![stat-data-czas-utworzenia-pliku](/img/2016/06/1.stat-data-czas-utworzenia-pliku.png#huge)

Mamy tutaj pozycje `Access:` (czas dostępu), `Modify:` (czas modyfikacji) oraz `Change:` (czas
zmiany). Niżej mamy także `Birth:` , który jest pusty. To właśnie tutaj powinna być zawarta data i
czas utworzenia pliku. Co się z nią stało i dlaczego jej tutaj nie ma? System plików EXT4 posiada
informację na temat daty stworzenia pliku ale nie jest ona udostępniana z jakiegoś powodu przez
kernel i dlatego `stat` nie zwraca żadnej informacji w tym przypadku. Czy ten stan rzeczy się zmieni
w najbliższym czasie? Tego nie wiem ale skoro EXT4 przechowuje w swojej strukturze czas utworzenia
pliku, to można pokusić się o wyciągnięcie tej informacji bezpośrednio z i-węzła opisującego dany
plik.

## Jak uzyskać czas utworzenia pliku za pomoca debugfs w EXT4

Każdy plik w systemie plików ma swój i-węzeł. Numer tego i-node możemy uzyskać na kilka sposobów.
Najprościej jest to zrobić przy pomocy `ls -i` lub `stat` . Wyżej na obrazku widzimy wyraźnie, że
widnieje tam `Inode: 132863` . To jest właśnie nasz punkt zaczepienia i musimy ten i-węzeł poddać
głębszej analizie. Odpalamy zatem terminal, logujemy się na kontro root i wydajemy to poniższe
polecenie:

    # debugfs -R 'stat <132863>' /dev/mapper/debian_laptop-home

W ten sposób powinniśmy uzyskać informacje podobne do tych, które zostały zwrócone za sprawą
`stat` :

![debugfs-data-czas-utworzenia-pliku-ext4](/img/2016/06/2.debugfs-data-czas-utworzenia-pliku-ext4.png#huge)

Widzimy tutaj już, że `crtime:` wskazuje na określoną datę i czas: `Tue Jun 7 16:21:23 2016` . Różni
się ona od tych trzech pozostałych i jak możemy zauważyć jest ona również najstarsza. To jest
właśnie data utworzenia pliku, której szukaliśmy. Niemniej jednak, uzyskiwanie tego czasu w taki
sposób jest niezbyt praktyczne i przydałoby się coś z tym mankamentem zrobić.

Szukając informacji na temat tego całego problemu z uzyskiwaniem daty/czasu utworzenia pliku,
[natknąłem się na ten
wątek](https://unix.stackexchange.com/questions/50177/birth-is-empty-on-ext4/131347#131347). Mamy
tam miłą funkcję, którą można dodać do konfiguracji shell'a (plik `~/.bashrc` lub `~/.zshrc` ).
Poniżej jest ta funkcja:

    get_crtime() {

        for target in "${@}"; do
            inode=$(stat -c %i "${target}")
            fs=$(df  --output=source "${target}"  | tail -1)
            crtime=$(debugfs -R 'stat ' "${fs}" 2>/dev/null |
            grep -oP 'crtime.*--\s*\K.*')
            printf "%s\t%s\n" "${target}" "${crtime}"
        done
    }

Zapisujemy plik i przeładowujemy konfigurację shell'a wpisując w terminalu to poniższe polecenie:

    # source ~/.zshrc

W tej chwili powinniśmy mieć dostęp do funkcji `get_crtime` . By uzyskać datę i czas utworzenia
przykładowego pliku, w terminalu wpisujemy poniższą komendę:

    # get_crtime file
    file    Tue Jun  7 16:21:23 2016

Naturalnie możemy podać w argumencie tyle plików ile chcemy. Możemy także uzyskać datę i czas
utworzenia wszystkich plików w danym katalogu przez podanie `*` w miejscu `file` .
