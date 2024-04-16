---
title: "Własny zdecentralizowany Reddit, czyli jak stworzyć własną instancje Lemmy używając Dockera"
date: 2024-04-16T21:24:56+02:00
draft: false
cover: "images/2024-04-09-lemmy/cover.png"
tags: ['pl', 'fediverse']
---

W tym wpisie postaram się przybliżyć proces instalacji własnej instancji Lemmiego oraz opisać czym jest ten cały fediverse.

## Czym jest Lemmy?
Lemmy to zdecentralizowany odpowiednik Reddita/agregatora linków/forum. Istnieje podział na community, możliwość upvote/downvote oraz komentowania postów. 

## Czym jest ten fediverse?
Fediverse (Fediwersum) to w skrócie zbiór zdecentralizowanych serwisów społecznościowych które korzystają z ustandaryzowanego protokołu do komunikacji z sobą. Działaniem bardzo przypomina email - czyli przykładowo posiadając skrzynkę pocztową na gmailu, możemy bez problemu pisać i odbierać maile od użytkownika o2 - tak samo jest tutaj, np. jeżeli ktoś opublikuje wpis na Lemmy, to inny użytkownik który np. używa Mastodon, zobaczy ten post oraz może z nim prowadzić interakcje.
Plusem fediverse jest to, że nie ma jednego centralnego serwera.

| ![screenshot showing the same post on lemmy.ml and sh.itjust.works](images/2024-04-09-lemmy/Screenshot_20240409_234719.png) | 
| :--: |
| Ten sam post z community !linux@lemmy.ml widoczny na lemmy.ml oraz sh.itjust.works |

## Po co tworzyć własną instancje?
Jeżeli chcesz tylko spróbować fediverse, nie musisz tworzyć własnej instancji - najlepiej znaleźć instancje która ma otwartą rejestrację lub poprosić o dostęp. Jest wiele list instancji np. [joinfediverse.wiki](https://joinfediverse.wiki/Instances) lub [fediverse.to](https://www.fediverse.to/).

Tworzenie własnej instancji ma sens, jeżeli:
* chcesz prowadzić większe community i mieć nad nim pełną kontrolę
* chcesz mieć niezależną, prywatną instancje, za którą odpowiadasz tylko Ty

## Co jest potrzebne do stworzenia własnej instancji?
Do stworzenia własnej instancji nie potrzeba dużo - potrzebujesz tylko domeny (lub subdomeny) oraz serwer z wyjściem na świat (otwartym portem 80/443). Lemmy nie wymaga dużo zasobów, ja moją małą instancje hostuje na serwerze z 2vCPU, 2GB RAM oraz 20GB dysku.
Łącze nie ma dużego znaczenia, dla paru osób nawet 5Mb/s uploadu prawdopodobnie wystaczą.

## Instalacja Lemmy
Do stworzenia instancji Lemmy wykorzystam Dockera, ponieważ to najprostsza metoda instalacji oraz aktualizacji, dodatkowo nie ma większego znaczenia z jakiego systemu operacyjnego korzystasz na hoście. Ten tutorial będzie przedstawiał kroki na Debianie. 

1. Zainstaluj Dockera
    W zależności od systemu operacyjnego, będzie się to trochę różniło. Najlepiej będzie przejrzeć oficjalną dokumentacje Dockera [jak zainstalować Docker Engine](https://docs.docker.com/engine/install/) na systemie który wybrałeś.

    Przykład na Debianie/Ubuntu
    ```
    sudo apt update
    sudo apt install docker docker-compose
    ```

2. Utwórz folder pod pliki
    Aby zachować porządek, utworzymy folder w katalogu domowym gdzie będziemy trzymać wszystkie elementy lemmy - w zależności od konfiguracji dysków lub preferencji, możesz go również utworzyć w innym miejscu.
    ```
    mkdir ~/lemmy
    mkdir ~/lemmy/volumes
    ```

3. Utwórz docker-compose
    Lemmy dzieli się na pięć (lub cztery jeżeli chcemy wykorzystać oddzielne reverse proxy) elementy:
    * Lemmy - główna aplikacja która serwuje API oraz obsługuje wymianę informacji
    * Lemmy-ui - kontener z interfejsem webui do interakcji z instancją
    * pict-rs - usługa do przechowywania obrazków na instancji
    * Postgres - baza danych
    * Nginx - jako reverse proxy które kieruje ruch do odpowiednich kontenerów - jest on opcjonalny jeżeli mamy już inne reverse proxy wystawione w sieci.

    Na czas pisania tego poradnika, najnowsza stabilna wersja Lemmy to `0.19.3` - sprawdź czy na [Githubie](https://github.com/LemmyNet/lemmy/releases) nie ma dostępnej nowszej wersji.

    ```yaml
    # docker-compose.yml
    version: "3.7"

    x-logging: &default-logging
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "4"

    services:
      proxy:
        image: nginx:1-alpine
        ports:
          # You could use port 80 if you won't use a reverse proxy
          - "8536:8536"
        volumes:
          - ./nginx.conf:/etc/nginx/nginx.conf:ro,Z
        restart: unless-stopped
        depends_on:
          - pictrs
          - lemmy-ui
        logging: *default-logging

      lemmy:
        image: dessalines/lemmy:0.19.3
        hostname: lemmy
        restart: always
        logging: *default-logging
        volumes:
          - ./lemmy.hjson:/config/config.hjson:Z
        depends_on:
          - postgres
          - pictrs

      lemmy-ui:
        image: dessalines/lemmy-ui:0.19.3
        environment:
          - LEMMY_UI_LEMMY_INTERNAL_HOST=lemmy:8536
          - LEMMY_UI_LEMMY_EXTERNAL_HOST=your-domain.tld
          - LEMMY_UI_HTTPS=true
        volumes:
          - ./volumes/lemmy-ui/extra_themes:/app/extra_themes:Z
        depends_on:
          - lemmy
        restart: always
        logging: *default-logging

      pictrs:
        image: docker.io/asonix/pictrs:0.5.4
        # this needs to match the pictrs url in lemmy.hjson
        hostname: pictrs
        # we can set options to pictrs like this, here we set max. image size and forced format for conversion
        # entrypoint: /sbin/tini -- /usr/local/bin/pict-rs -p /mnt -m 4 --image-format webp
        user: 991:991
        volumes:
          - ./volumes/pictrs:/mnt:Z
        restart: always
        logging: *default-logging
        deploy:
          resources:
            limits:
              memory: 690m

      postgres:
        image: docker.io/postgres:15-alpine
        hostname: postgres
        environment:
          - POSTGRES_USER=lemmy
          - POSTGRES_PASSWORD=password
          - POSTGRES_DB=lemmy
        volumes:
          - ./volumes/postgres:/var/lib/postgresql/data:Z
        restart: always
        shm_size: 1g
        logging: *default-logging
    ```
    Podmień `your-domain.tld` we wszystkich miejscach na prawdziwą domenę.

4. Utwórz plik konfiguracyjny lemmy oraz nginx
    W folderze w którym utworzyłeś `docker-compose.yml`, utwórz plik `lemmy.hjson` - jest to główny plik konfiguracyjny lemmy, gdzie skonfigurujemy dane dostępowe do bazy danych, pict-rs, hostname oraz opcjonalne dane SMTP.

    W podstawowej konfiguracji, jedyną rzeczą jaką należy tutaj zmienić, jest `hostname` (powinien odpowiadać domenie pod którą chcesz wystawić instancję) oraz `pictrs.api_key` (powinien się pokrywać z api_key zdefiniowanym w `docker-compose`)
    ```json
    # lemmy.hjson
    {
      # for more info about the config, check out the documentation
      # https://join-lemmy.org/docs/en/administration/configuration.html

      database: {
        # name of the postgres database for lemmy
        database: "lemmy"
        # username to connect to postgres
        user: "lemmy"
        # password to connect to postgres
        password: "password"
        # host where postgres is running
        host: "postgres"
        # port where postgres can be accessed
        port: 5432
        # maximum number of active sql connections
        pool_size: 5
      }
      hostname: "your-domain.tld"
      pictrs: {
        url: "http://pictrs:8080/"
        api_key: "randomApiToken"
        # If you have a instance only for couple of people, you might want to disable image caching
        # from other instances
        cache_external_link_previews: false
      }
    }
    ```
    Dla bezpieczeństwa, sugeruję zmienić domyślne dane, np. hasło do bazy postgres oraz apiKey do pictrs
    Zajrzyj na [join-lemmy.org/docs/administration/configuration.html](https://join-lemmy.org/docs/administration/configuration.html) aby dowiedzieć się o reszcie dostępnych opcjach konfiguracjnych.

    Pobierz przykładową konfiguracje nginx
    ```bash
    wget https://raw.githubusercontent.com/LemmyNet/lemmy/1596aee724339c7112d5efa42fa37e838e87d93c/docker/nginx.conf -O ~/lemmy/nginx.conf
    ```
    Jeżeli masz inne reverse proxy, możesz pozbyć się kontenera proxy z `docker-compose` i wykorzystać ten `nginx.conf` jako baza.
    
    Aby federacja działała poprawnie, potrzebujemy mieć SSL, np. za pomocą LetsEncrypt, w zależności od tego jak chcesz wystawić Lemmy (za reverse proxy, wykorzystać nginx w docker-compose jako główny), musisz odpowiednio zmodyfikować konfiguracje nginx.

5. Ustaw poprawne uprawnienia dla volumes
    ```bash
    mkdir -p ~/lemmy/volumes/pictrs
    mkdir -p ~/lemmy/volumes/postgres
    # Set owner to user defined in pictr
    sudo chown -R 991:991 volumes/pictrs
    ```
6. Uruchom `docker-compose`
    ```bash
    docker-compose up -d
    ```
    | ![scrrenshot of lemmy setup screen](images/2024-04-09-lemmy/Screenshot_20240415_231154.png) |
    | :--: |
    | Po chwili interfejs lemmy-ui powinien być dostępny - powineneś mieć opcje aby założyć konto administratora oraz opcje konfiguracyjne site, jak nazwa, opis, czy instancja ma mieć otwartą rejestrację oraz ustawienia federacji. | 

    Domyślnie nie musisz zmieniać żadnych opcji federacji, sugeruję zostawić *Registration Mode* na *Require registration application*, aby nasza instancja nie została źródłem spamu


    Następnie powinniśmy już mieć możliwość subskrybowania społeczności 🎉
    ![screenshot of empty lemmy instance](images/2024-04-09-lemmy/Screenshot_20240415_231816.png)

7. Znajdowanie społeczności
    Domyślnie nie będziesz widział żadnych społeczności - musisz je najpierw znaleźć - do znajdowania społeczności Lemmy możesz wykorzystać [**Lemmy Explorer**](https://lemmyverse.net/communities)
    ![lemmy explorer](images/2024-04-09-lemmy/Screenshot_20240415_225802.png)
    Skopiuj link do społeczności (przykładowo `!nazwa@instacja.tld`), przejdź do wyszukiwarki na swojej instancji Lemmy i wyszukaj społeczność.
    Możliwe że za pierwszym razem od razu twoja instancja nie pokaże wyszukiwanej społeczności, spróbuj ponownie kliknąć na wyszukiwanie.   
    ![screenshot showing search screen of lemmy instance, in search result showing Technology community from lemmy.world instance](images/2024-04-09-lemmy/Screenshot_20240415_232056.png)
    Następnie przejdź do społeczności i ją zasubskrybuj, po pewnym czasie na twojej instancji powinny się pokazywać posty z danej społeczeności..
    ![community subscribe button](images/2024-04-09-lemmy/Screenshot_20240415_232342.png)

    Jeżeli po dłuższym czasie nadal nie pojawiają ci się posty z instancji zewnętrznych, sprawdź logi nginx, oraz czy na pewno twoja instancja jest poprawnie dostępna z internetu/posiada poprawnie skonfigurowane HTTPS

Cover background: [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [unsplash](https://unsplash.com/photos/a-close-up-of-a-purple-background-with-wavy-lines-hOYHAdgbTr0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
  
