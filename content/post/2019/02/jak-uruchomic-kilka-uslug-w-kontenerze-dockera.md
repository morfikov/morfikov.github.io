---
author: Morfik
categories:
- Linux
date: "2019-02-02T07:20:43Z"
published: true
status: publish
tags:
- debian
- docker
title: Jak uruchomić kilka usług w kontenerze Docker'a
---

W kontenerach Docker'a nie powinno się uruchamiać więcej usług niż jedna. Czasami jednak zachodzi
potrzeba, by właśnie uruchomić kilka niezależnych od siebie procesów, które będą ze sobą
współpracować w obrębie takiego pojedynczego kontenera. Weźmy sobie na przykład serwer WWW Apache2
i bazę danych MySQL/MariaDB. Każda z tych usług posiada swój dedykowany kontener (nawet oficjalny)
i generalnie skonfigurowanie komunikacji między tymi dwoma kontenerami Docker'a nie jest niczym
trudnym. Jeśli jednak ktoś by się uparł, to może stworzyć sobie taki kontener, który będzie
uruchamiał obie te usługi. Oczywiście w tym przypadku raczej nikt nie będzie łączył tych dwóch
kontenerów w jeden ale są pewne sytuacje, w których będziemy chcieli uruchomić więcej niż jeden
proces wewnątrz kontenera i gdy ten czas nadejdzie, to wypadałoby wiedzieć jak się do tego
przedsięwzięcia zabrać.

<!--more-->
## Problem związany z usługami w kontenerze Docker'a

Docker nie bez powodu chce, by w nim uruchamiać tylko i wyłącznie jedną usługę. Oczywiście, jedna
usługa nie oznacza pojedynczego procesu. Taka usługa może sobie uruchomić tyle procesów ile tylko
zapragnie ale one wszystkie będą podpięte pod określony PID i to właśnie ten główny PID jest
monitorowany przez Docker. W przypadku, gdy główny proces z jakiegoś powodu wyrzuci błąd, to Docker
będzie wiedział czy na przykład zrestartować kontener w takiej sytuacji. Gdy tych procesów odpalamy
więcej, to co ma zrobić Docker w momencie, gdy jeden z nich przestanie nagle działać?

Dlatego właśnie budując sobie obraz kontenera, mamy możliwość zdefiniowania tylko jednej pozycji
`CMD` w pliku `Dockerfile` . Można naturalnie próbować obejść tę restrykcję za pomocą `&&` i tego
typu podobnych rozwiązań ale zdecydowanie nie jest to droga, którą powinno się podążać.

## Skrypt uruchamiający wiele usług/procesów

Szukając rozwiązania problemu uruchamiania wielu usług w jednym kontenerze Docker'a, natrafiłem
na [ten oto ciekawy artykuł](https://docs.docker.com/config/containers/multi-service_container/).
W nim z kolei jest zawarty jeszcze ciekawszy skrypt, którego treść znajduje się poniżej:

    #!/bin/bash

    # Start the first process
    ./my_first_process -D
    status=$?
    if [ $status -ne 0 ]; then
      echo "Failed to start my_first_process: $status"
      exit $status
    fi

    # Start the second process
    ./my_second_process -D
    status=$?
    if [ $status -ne 0 ]; then
      echo "Failed to start my_second_process: $status"
      exit $status
    fi

    # Naive check runs checks once a minute to see if either of the processes exited.
    # This illustrates part of the heavy lifting you need to do if you want to run
    # more than one service in a container. The container exits with an error
    # if it detects that either of the processes has exited.
    # Otherwise it loops forever, waking up every 60 seconds

    while sleep 60; do
      ps aux |grep my_first_process |grep -q -v grep
      PROCESS_1_STATUS=$?
      ps aux |grep my_second_process |grep -q -v grep
      PROCESS_2_STATUS=$?
      # If the greps above find anything, they exit with 0 status
      # If they are not both 0, then something is wrong
      if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 ]; then
        echo "One of the processes has already exited."
        exit 1
      fi
    done

Ten powyższy skrypt wywołuje dwa osobne procesy ale bez większego problemu można go przerobić tak,
by uruchamiał dowolną ilość usług. W zasadzie to w miejsce `./my_first_process -D` i
`./my_second_process -D` trzeba podać ścieżki do binarek, które zamierzamy uruchomić niezależnie w
kontenerze. Ta flaga `-D` ma za zadanie odpalić proces w tle. Zwykle jednak będzie na końcu tej
linijki coś na wzór `&` , ewentualnie jakiś natywny parametr, który jest w stanie zrzucić proces z
pierwszego planu tak, by dalsza część skryptu mogła się wykonać.

Po uruchomieniu procesów jest inicjowana pętla, która sprawdza w interwale 60s (czas można sobie
dowolnie określić) czy usługi cały czas działają. Jeśli jedna z nich zakończy się, to zniknie ona z
wyjścia `ps` , przez co skrypt zwróci błąd. Jako, że nasz skrypt jest inicjatorem usług, to jeśli
on zwróci błąd, to Docker będzie automatycznie wiedział, że coś poszło nie tak i od razu
wyłączy/zrestartuje kontener, czy też przeprowadzi tę akcję, którą mu wskazaliśmy w konfiguracji.

By ten skrypt zintegrować z kontenerem Docker'a, to do pliku `dockerfile` musimy dodać coś na
poniższy wzór:

    ...
    COPY my_wrapper_script.sh my_wrapper_script.sh
    CMD ./my_wrapper_script.sh
    ...

Linijka z `COPY` ma za zadanie skopiować plik z hosta do obrazu kontenera podczas jego budowy.
Ścieżkę do skryptu podajemy względem pliku `dockerfile` . W tym przypadku oba te pliki znajdują się
w tym samym katalogu. Z kolei ścieżka docelowa może być relatywna albo absolutna (poprzedzona `/` ).
Gdy nie zostanie użyty `/` , to w takim przypadku skrypt zostanie skopiowany w miejsce określone
przez zmienną `WORKDIR` . I to w zasadzie tyle.
