---
author: Morfik
categories:
- blog
date:    2021-04-09 20:00:00 +0200
lastmod: 2021-04-09 20:00:00 +0200
published: true
status: publish
tags:
- disqus
- komentarze
- reklamy
GHissueID: 333
title: Disqus wprowadza reklamy, Disqus musi odejść
---

Sporo użytkowników przeglądających tego bloga zdążyło już zauważyć, że sekcja komentarzy została
usunięta. W efekcie czego nie można już komentować żadnych wpisów. Taki zabieg miał miejsce ze
względu na fakt, że sporo osób zaczęło mi zgłaszać, że na Morfitroniku pojawiły się reklamy.
Wszyscy znają raczej moje zdanie w kwestii serwowania ludziom reklam i mocno się zdziwiłem, że
pojawiły się one właśnie tutaj. Ja nikomu żadnego pozwolenia na wykorzystywanie mojego bloga jako
platformy reklamowej nie udzieliłem. Okazuje się jednak, że Disqus (bo to jego system komentarzy
był tutaj wykorzystywany) nawet nie pofatygował się, by zapytać mnie o zgodę na serwowanie reklam i
bezczelnie zaczął je w komentarzach umieszczać. Jak tylko się dowiedziałem, że te reklamy faktycznie
w tych komentarzach są oraz, że nie mam żadnego wpływu na to, by je usunąć, to postanowiłem
natychmiast wywalić Disqus'a.

<!--more-->
## Mail od Disqus

Jakieś dwa tygodnie temu Disqus wysłał mi na maila wiadomość o poniższej treści:

>Hi there,
>
>Thanks for using Disqus! Your site was recently approved for ads, but we noticed that you don’t
>currently having any campaigns running. If you’d prefer a non-ads supported plan, you can choose
>between either of our ads-free plans here:
>
>    Plus - $12/month or $11/month with our yearly discount
>
>    Pro - $115/month or $105/month with our yearly discount
>
>browse plans
>
>Each of these plans comes with an ads-free version of Disqus and our Pro plan offers a variety of
>additional engagement features. You can review these options from your Subscription and Billing
>page.
>
>Thank you,

Wynika z niej, że by mieć komentarze pozbawione syfu reklamowego, trzeba zapłacić jakieś 150 dolców
rocznie. Nawet VPS z własną domeną jest tańszy. Przyznam, że początkowo zakwalifikowałem tę w
iadomość jako mało ważną ale w swoich kręgach poinformowałem ludzi, że jakby tylko jakieś reklamy
się na blogu zaczęły pojawiać, to by mi dali znać. No i od paru dni ludziska zaczęli zgłaszać mi
pewne nieprawidłowości.

## Adblock'i są w stanie te reklamy wyciąć

Prawdą jest fakt, że jeśli ktoś korzysta z ublock'a czy też innego blokera reklam, to jest duże
prawdopodobieństwo, że reklamy w sekcji komentarzy zostaną w sporej części (albo też i całkowicie)
odfiltrowane (przynajmniej do momentu, w którym taka opcja nie będzie już możliwa). Niektórzy z nas
mają jednak jakieś zasady, którymi kierują się w swoim życiu. Jedną z moich zasad jest
nietraktowanie drugiego człowieka jak śmiecia (co Disqus właśnie uczynił). Drugą zaś jest
eradykacja reklam z otaczającej mnie przestrzeni, zarówno tej realnej, jak i wirtualnej -- reklamom
mówię stanowczo NIE. Tą pierwszą zasadę można by pewnie jeszcze jakoś przemilczeć ale jeśli chodzi
o tę drugą, to już tak łatwo nie będzie. Nie po to od ponad dekady staram się pozbyć ze swojego
otoczenia reklam wdrażając rozmaite rozwiązania mające za zadanie odfiltrować ten syf, by taki
proceder sygnować swoim imieniem. Dlatego też postanowiłem pozbyć się tego całego Disqus'a, jako że
nie pozostawił mi on praktycznie żadnego wyboru.

## Czy komentarze wrócą

Póki co nie zapowiada się by sekcja komentarzy wróciła. Ten blog jest zasilany przez Hugo, a on
generuje lokalnie statyczne pliki html, które później są przesyłane do GitHub'a, tam pobierane
przez ich serwer i publikowane. Z jednej strony niesamowity plus jeśli chodzi o bezpieczeństwo
takiej strony ale odbija się to na jej funkcjonalności, przez co trzeba szukać zewnętrznych
dodatków, które będą zajmować się obsługą dynamicznej treści, np. komentarzami. Trzeba też wziąć
poprawkę na wykorzystywany na blogu motyw, bo z ich funkcjonalnością też różnie bywa. Napisałem do
twórcy motywu z zapytaniem [czy planuje on może implementację innych systemów komentarzy][1] ale
odpowiedź była taka jak zawsze, czyli nic z tego.

Jeśli uda się jakieś rozwiązanie znaleźć, to sekcja komentarzy się pojawi, a póki co trzeba wrócić
do starego mechanizmu komunikacji, tj. zwykłego maila, którego adres jest, jak zawsze, na dole
strony.


[1]: https://github.com/Vimux/Binario/issues/44
