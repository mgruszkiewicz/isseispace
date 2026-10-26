---
title: "W poszukiwaniu idealnego docka USB-C (i wyciszanie HP Thunderbolt Dock G2)"
date: 2026-10-26T19:59:07+02:00
draft: false
tags: ['pl']
---

Jakiś czas temu zacząłem szukać nowego huba/docka USB-C do laptopa służbowego i prywatnego. Przez ostatnie lata korzystałem z prostych hubów od Ugreena i w większości przypadków działały w porządku, ale miałem z nimi dwa problemy.

Pierwszy to krótki kabel. Używam pionowej podstawki na laptopa, żeby nie zajmował miejsca na biurku, przez co hub wisi na samym kablu zamiast leżeć na blacie. Drugi pojawił się niedawno: po latach używania kabel chyba po prostu się zużył. Czasem były problemy z ładowaniem, ale przede wszystkim z monitorem. Laptop widział wyświetlacz, monitor udawał, że ma sygnał, a mimo to nie pokazywał nic.

Zacząłem więc szukać czegoś nowego. Skupiłem się na używanych dockach poleasingowych. Pewnie mógłbym kupić CalDigita, który spełniłby moje wymagania aż nadto, ale mój Ugreen kosztował do 200 zł i też robił to, czego od niego oczekiwałem. Wydawanie prawie dziesięć razy tyle nie miało dla mnie sensu.

Ostatecznie padło na **HP Thunderbolt Dock G2**. Na Allegro jest ich całkiem sporo - mi udało się go kupić za 150zł z zasilaczem 120W 

![screenshot of allegro offer of 120w version](images/2026-10-26-dock/allegro.jpeg)
Nie jest to wpis sponsorowany, więc nie podaję sprzedawcy, ale jest ich kilka - z tym konkretnym mam mieszane odczucia, początkowo przysłali mi zły zasilacz, sprzedawca poprosił o zwrot obecnego i dopiero po otrzymaniu, przesłał mi odpowiedni - więc poprawne zachowanie, ale cała akcje trwała prawie tydzień.

Jest jeszcze dostępna wersja 230W tego docka - różni się ona chyba głównie kablem i dołączonym zasilaczem. Kabel ma dodatkowy wtyk zasilania, który uniemożliwi podłączenie go do innych laptopów niż HP, ale mam podejrzenia, że po wymianie kabla USB-C na zwykły, prawdopodobnie będzie działać tak samo, jednak sam tego nie testowałem. Więcej o tym w sekcji o wyciszaniu docka.
![view of the 230w combined usb-c cable](images/2026-10-26-dock/230w.jpeg)

Co do samego docka - spełnia wszystko, czego chciałem:
* jeden, dłuższy kabel do laptopa,
* Ładowanie 100W.
* DisplayPort z obsługą 4K 60 Hz, natywnie, bez DisplayLinka,
* Cztery porty USB A oraz trzy porty USB-C (w tym thunderbolt pass-through),
* Ethernet 1Gb/s,
* niska cena.

W porównaniu z Ugreenem brakuje czytnika kart SD i microSD, ale dock ma za to wyjście słuchawkowe.

Po prawie pół roku używania działa bez zarzutu zarówno z MacBookiem z Apple Silicon, jak i z Lenovo Yogą (AMD). Podłączam kabel, wszystko się wybudza i dokładnie o to mi chodziło.

## Działanie z wieloma monitorami
Sam posiadam tylko jeden monitor 4K60hz i nie mam żadnych problemów z jego działaniem, ale podejrzewam że niektórych może ciekawić czy działa wiele monitorów, bo w internecie są sprzeczne informacje. Nie testowałem tylko wyjścia VGA, bo nie posiadałem akurat pod ręką monitora z VGA do testów. Wykonałem parę eksperymentów z róznymi konfiguracjami:
### 4K 60Hz
Tak jak wspomniałem wcześniej, działa bez problemu.
### 1080p 144Hz 
Działa
![screenshot of macos displays setting showing that 1080p screen is detected and working correctly at 144hz](images/2026-10-26-dock/dp-144hz.jpeg)
### Dwa monitory
Tutaj już zaczynają się schody. Po podłączeniu dwóch monitorów przez DisplayPort, jedyne co udało mi się uzyskać to mirror tych dwóch ekranów. Workaround tego problemu to użycie osobnego huba USB-c podpiętego do *lewego* portu docka - tego oznaczonego symbolem Thunderbolt. Monitor podłączony hubem do prawego portu - przynajmniej tym testowałem, nie był wykrywany.
Po podłączeniu monitorów w następujący sposób - jeden bezpośrednio do docka przez displayport, a drugi przez hub usb-c 
![connecting two screens to the HP dock require the use of seperate usb-c dongle](images/2026-10-26-dock/IMG_6937.jpeg)

Mogę potwierdzić że - licząc wbudowany ekran - możemy mieć trzy monitory na maku z apple silicon - w moim przypadku to M3Pro, ale wydaje mi się że od M2 to powinno działać na wszystkich
![photo showcasing three screens working, using my girlfriend _gaming setup_](images/2026-10-26-dock/IMG_6936.jpeg)


## Wyciszanie docka

Jedyne, co mi w nim przeszkadza, to wentylator. Nie jest jakoś bardzo głośny, ale gdy reszta stanowiska jest bezgłośna, to go słychać. Sam dock nie robi się przy tym szczególnie gorący i nie czuć mocnego nawiewu, więc postanowiłem go otworzyć i zobaczyć, co jest w środku.

Otwiera się go łatwo. Najpierw wykręcamy cztery śrubki od spodu i zdejmujemy pierwszy element obudowy, pod którym jest dostęp do kabla USB-C. Przy okazji sprawdziłem, że kabel nie musi być oryginalny od HP. Działa każdy kabel z obsługą Thunderbolta, więc jeśli oryginalny się uszkodzi, można go po prostu wymienić.

Następnie odkręcamy kolejne widoczne śrubki i rozdzielamy obudowę, podważając ją czymś cienkim, np. plastikową kartą. Spód powinien odejść, a płytki da się wysunąć z obudowy. Trzeba uważać, bo dock składa się z dwóch PCB połączonych taśmą. Jeśli płytka nie chce wyjść, prawdopodobnie zahacza o gniazdo VGA.

![](images/2026-10-26-dock/IMG_6852.jpeg)

Na samej górze jest mały wentylator, podobny do tych z laptopów, podpięty wtyczką do PCB. Wystarczy go odłączyć i złożyć dock z powrotem. Dock nie sprawdza, czy wentylator jest podłączony, więc nie zgłasza żadnych błędów.

![](images/2026-10-26-dock/IMG_6854.jpeg)

Po tej modyfikacji dock jest oczywiście trochę cieplejszy, ale nie na tyle, żeby wyglądało to niebezpiecznie. Poprzedni hub od Ugreena potrafił się nagrzać dużo bardziej. Można by jeszcze wydrukować bardziej przewiewną obudowę dla pasywnego oddawania ciepła albo wywiercić w obecnej kilka otworów, ale na razie nie czuję takiej potrzeby. Używam huba z odłączonym wentylatorem od paru tygodni i nawet podczas grania nie czuję aby się niebezpiecznie nagrzewał.

Początkowo liczyłem, że wentylator da się wyłączyć programowo. Dock ma aktualizowalny firmware, więc miałem nadzieję, że HP zostawiło gdzieś taką opcję. Próbowałem to ogarnąć z pomocą klaudiusza, ale po pewnym czasie obaj odpuściliśmy. Firmware jest podpisany, więc nie zanosiło się na to, że da się z nim coś łatwo zrobić. Dodatkowo soft od HP jest tylko pod windowsa, a akurat nie mam żadnego laptopa z tym systemem aby to potestować.

Dodatkowo dla ciekawskich, mogę potwierdzić że dock używa kontrolera Thunderbolt 3 Intel JHL7440 oraz chipu RTL8153AH do karty sieciowej

## Podsumowanie

Za 150 zł dostałem docka, który robi dokładnie to, czego potrzebowałem: jeden dłuższy kabel, ładowanie 100W, monitor 4K 60 Hz po DisplayPort i komplet portów. Z MacBookiem i Lenovo działa bez problemów od pół roku. Największe minusy to wentylator, który u mnie wylądował odłączony, oraz obsługa dwóch monitorów, która wymaga obejścia z dodatkowym hubem.

Polecam, jeśli szukasz taniego docka z Thunderboltem, używasz jednego monitora i nie boisz się odkręcić kilku śrubek. Jeśli potrzebujesz kilku monitorów podłączonych bez kombinowania albo gwarancji, lepiej dołożyć do czegoś nowszego.
