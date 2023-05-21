---
title: "'Hackowanie' ZOOMki - czyli co można zrobić z ramką na zdjęcia Zoom.me"
date: 2020-11-24T00:24:56+02:00
draft: false
cover: "images/2020-11-24-zoomka/cover.webp"
tags: ['pl', 'android']
---

Jakiś czas temu od mojego przyjaciela (dzięki [Oskar!](https://www.instagram.com/oskarhyung/)) dostałem ramkę [zoom.me](https://zoom.me) ponieważ nie miał do niej zasilacza, a po szybkim sprawdzeniu w internecie co to w ogóle jest, oboje się zainteresowaliśmy. Tak, jak widać po zdjęciu wyżej, można na niej uruchomić DOOMa ;)

## Ale od początku - co to jest w ogóle ta ramka zoom.me?

Jak możemy przeczytać na stronie producenta:

> ZOOM.ME to nowe podejście do komunikacji z bliskimi. Dzięki ZOOM.ME każdy może dzielić się z przyjaciółmi wspaniałymi chwilami uchwyconymi na zdjęciach.

Czyli w skrócie - jest to cyfrowa ramka na zdjęcia na którą można wysłać zdjęcia z dowolnego miejsca na świecie, korzystając z aplikacji lub wprost z maila. Została wydana w 2014 roku w współpracy z firmą Goclever i niestety nie okazała się takim sukcesem na jaki liczyli jego twórcy, po roku od wydania można było ją kupić za około 200zł. Tutaj pojawia się pierwszy problem, ponieważ jak można zobaczyć w [Sklepie Play](https://play.google.com/store/apps/details?id=me.zoom.asender) aplikacja ma aż 1.8 gwiazdki, do tego większość opinii wspomina o problemami z połączeniem do serwera  
![](https://i.issei.space/LNeGaGHf.png) 

No dobra, ale może już to naprawili, przecież strona jeszcze działa... Niestety ale próba stworzenia konta przez aplikacje jest niemożliwa, ponieważ nieważne jakiego maila bym tam wpisał, on zawsze jest już zajęty. Z ciekawości spróbowałem wysłać zdjęcie mailem na adres który wyświetla się na ramce, niestety - dostaje wiadomość zwrotną, że mój email nie istnieje w systemie (?!)  


![](https://i.issei.space/36RWEPbI.png)


Okej, skoro nie mogę w taki sposób skorzystać z ramki to pewnie jest inny sposób? Tak - można kopiować zdjęcia bezpośrednio przez złącze microUSB z komputera. Po podłączeniu do komputera można zauważyć dwa foldery - DCIM oraz Pictures. Aby zdjęcia były widoczne na ramce należy je skopiować do folderu DCIM/ZOOM.ME. I tutaj można zauważyć kolejną ciekawostkę - **mimo przywrócenia ustawień fabrycznych w menu ramki zdjęcia się nie usuwają**. No dobra, zdjęcia skopiowałem, wyświetlają się, fajnie, ale jednak to trochę mało.  

## Dostanie się do androida

Tak, ta ramka na zdjęcia działa pod kontrolą androida w wersji 4.2.2\. W środku znajduje się procesor Allwinner A23 wraz z 512MB pamięci ram oraz 4GB pamięci flash. Samo to, że ramka działa na androidzie można samemu dosyć szybko zauważyć - ponieważ podczas wpisywania hasła do sieci WiFi wysuwa się typowa androidowa klawiatura. No to do dzieła - udało mi się w internecie znaleźć instrukcję do ramki, jak i [poradnik od producenta w jaki sposób zaktualizować jej oprogramowanie](http://downloadfiles.goclever.com/Tablet/Zoom_me/ZOOM.ME_WiFi_Zmiana_Oprogramowania.pdf) oraz [post na forum XDA z prośbą o pomoc w wgraniu gapps](https://forum.xda-developers.com/android/help/goclever-zoom-gapps-t3873801). Z instrukcji wynika, że domyślnie powinno być włączone debugowanie USB, więc dostanie się do systemu powinno być bardzo proste. Niestety, ale moja ramka posiadała troszkę nowszą wersje oprogramowania i trik z ośmiokrotnym kliknięciem na wersje oprogramowania aby dostać się do ustawień nie zadziałał a debugowanie USB też nie jest uruchomione.  


![](https://i.issei.space/Eipv7J9R.png) 

W mojej ramce nie ma oryginalnej wersji oprogramowania - oryginalne oprogramowanie zostało wydane 07.10.2014, u mnie jest wersja z 17 grudnia 2014\. W instrukcji aktualizacji dostępny jest rom z wersją 17.07.2015 - nie wiem czym się dokładniej różnią.  
Na samym początku postanowiłem że skoro mam dostęp do instrukcji aktualizacji softu i pliku z systemem oraz jest to urządzenie z procesorem Allwinner to zmodyfikuję sobie rom i do build.prop dodam opcje aby debugowanie było włączone. Sama ta operacja się powiodła, ale przy głębszym spojrzeniu do instrukcji zauważyłem, że do aktualizacji softu potrzebne jest włączone debugowanie USB (?!). Teoretycznie rozwiązanie tego jest proste - wystarczy wyłączyć urządzenie, przytrzymać przycisk głośności i podłączyć kabel USB. Dlaczego teoretycznie? Ponieważ to jest ramka na zdjęcia która nie ma głośnika oraz przycisków głośności. Zastanawiałem się nad otworzeniem obudowy i poszukaniem na płycie głównej pinów odpowiedzialnych za UART lub/oraz głośności, ale dostałem oświecenia...  

## Hm, przecież to Allwinner

...ciekawe czy obsługuje USB OTG. Wziąłem przejściówkę USB OTG, podłączyłem klawiaturę, i.. Działa! No okej, teraz mogę sterować urządzeniem klawiaturą, ale co mi to daje? Ano właśnie dużo - mogę wykorzystać skrót klawiszowy WIN+spacja aby przejść do wbudowanej przeglądarki - tutaj zauważyłem drobny problem - przeglądarka ma problemy z witrynami korzystającymi z HTTPS, ale jestem już za blisko aby mnie to pokonało - wrzuciłem na prosty serwer www plik APK z [Olauncherem](https://github.com/tanujnotes/Olauncher) i go pobrałem. Huh - sprawdź pasek powiadomień z postępem pobierania pliku - ale ja nie mam dostępu do paska powiadomień.  

Z ciekawości wszedłem na stronę gdzie można wybrać plik do uploadu i pokazał mi się Open Manager - to było już spore ułatwienie ponieważ znałem ścieżkę do pliku. Próbowałem "wydostać się" do zwykłego trybu menadżera plików ale nie udało mi się. Przypomniałem sobie, że przecież znam teraz ścieżkę i w przeglądarce mogę wpisać lokalizacje tego pliku, więc wpisałem `file://storage/emulated/0/Downloads/launcher.apk` i moim oczom ukazało się okienko z pytaniem, czy na pewno chcę zainstalować tą aplikacje  
Po zainstalowaniu launchera wystarczy kliknąć parę razy "ESC" aby przejść do wyboru programu startowego - ja wybrałem aby domyślnie startował launcher - jeżeli ktoś nadal by chciał korzystać z aplikacji zoom.me może ją po prostu uruchomić z listy. 

![](https://i.issei.space/00f1fFSU.jpg)

## Jestem w androidzie ale co dalej?

Android który jest w tym urządzeniu jest bez aplikacji google (google-apps gapps) przez co swoją drogą działa lepiej niż się spodziewałem. Nie będę tutaj instalował aplikacji google ponieważ miejsca już jest mało, szkoda dodatkowo zamulać urządzenia, a do tego będą tutaj zbędne. Postanowiłem po prostu skorzystać z stron typu apkmirror oraz ewentualnie instalować aplikacje ze sklepu na moim telefonie a następnie eksportowanie ich apk na to urządzenie.  
Swoją drogą - nadal nie mamy przycisków nawigacyjnych - przydały by się aby nie trzeba było podłączać klawiatury. W tym celu potrzebujemy zmodyfikować plik build.prop, a żeby go zmodyfikować musimy mieć roota.  
Na szczęście rootowanie androida do wersji 5 na takich prostych konstrukcjach jest banalne, ale nadal potrzebujemy debugowania USB. Teraz kiedy już mamy launcher możemy przejść do ustawień, zjechać w dół -> Opcje programistyczne i zaznaczyć debugowanie USB. Na komputerze powinnyśmy mieć zainstalowane sterowniki ADB - ja zawsze korzystam z [tej paczki](https://forum.xda-developers.com/showthread.php?t=2588979). Do rootowania wykorzystam skryptu pod nazwą "rootV2" [która jest do pobrania tutaj](https://www.mediafire.com/download/12b1y3aw0k5dhkx/RootV2.rar) - wystarczy wypakować pliki, przejść do folderu, uruchomić skrypt, poczekać chwilę i zaraz powinnyśmy mieć na naszej ramce roota oraz aplikacje SuperSU.  
Teraz możemy wykorzystać adb aby zmodyfikować nasz build.prop. W tym celu przechodzimy do wiersza poleceń i wykonujemy komendę `adb shell`, a następnie  
`su  
echo "qemu.hw.mainkeys=0" >> /system/build.prop`  

Po tej operacji należy zrestartować ramkę i powinniśmy już widzieć pasek powiadomień oraz przyciski nawigacyjne  


![](https://i.issei.space/TjqlR0Qq.png) 

![](https://i.issei.space/zvhXlQMt.png)

## Zakończenie

Teraz kiedy już mamy dostęp do przeglądarki, menadżera plików, ADB, roota oraz normalnego launchera możemy robić wszystko co chcemy na tym urządzeniu.