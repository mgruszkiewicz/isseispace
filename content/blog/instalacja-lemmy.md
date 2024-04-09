---
title: "Tworzenie własnej instancji Lemmy"
date: 2024-04-09T00:24:56+02:00
draft: false
cover: "images/2024-04-09-lemmy/cover.jpg"
tags: ['pl', 'fediverse']
---

W tym wpisie postaram się przybliżyć proces instalacji własnej instancji Lemmiego oraz opisać czym jest ten cały fediverse.

## Czym jest Lemmy?
Lemmy to zdecentralizowany odpowiednik Reddita/agregatora linków/forum. Istnieje podział na community, możliwość upvote/downvote oraz komentowania postów. 

## Czym jest ten fediverse?
Fediverse (Fediwersum) to w skrócie zbiór zdecentralizowanych serwisów społecznościowych które korzystają z ustandaryzowanego protokołu do komunikacji z sobą. Działaniem bardzo przypomina email - czyli przykładowo posiadając skrzynkę pocztową na gmailu, możemy bez problemu pisać i odbierać maile od użytkownika o2 - tak samo jest tutaj, np. jeżeli ktoś opublikuje wpis na Lemmy, to inny użytkownik który np. używa Mastadon, zobaczy ten post oraz może z nim prowadzić interakcje.
Plusem fediverse jest to, że nie ma jednego centralnego serwera.

## Po co tworzyć własną instancje?
Tak na prawdę - jeżeli chcesz tylko spróbować fediverse, nie musisz tworzyć własnej instancji - najlepiej znaleźć instancje która ma otwartą rejestrację lub poprosić o dostęp. Jest wiele list instancji np. [joinfediverse.wiki](https://joinfediverse.wiki/Instances) lub [fediverse.to](https://www.fediverse.to/).

Tworzenie własnej instancji ma sens, jeżeli:
* chcesz prowadzić większe community i mieć nad nim pełną kontrolę
* chcesz mieć niezależną, prywatną instancje

## Co jest potrzebne do stworzenia własnej instancji?
Do stworzenia własnej instancji nie potrzeba dużo - potrzebujesz tylko domeny (lub subdomeny) oraz serwer z wyjściem na świat (otwartym portem 80/443). Lemmy nie wymaga dużo zasobów, ja moją małą instancje hostuje na serwerze z 2vCPU, 2GB RAM oraz 20GB dysku.
Łącze nie ma dużego znaczenia, dla paru osób nawet 5Mb/s uploadu prawdopodobnie wystaczą.

## Instalacja Lemmy
Do stworzenia instancji Lemmy wykorzystam Dockera, ponieważ to najprostsza metoda instalacji oraz aktualizacji, dodatkowo nie ma większego znaczenia z jakiego systemu operacyjnego korzystasz na hoście. Ten tutorial będzie przedstawiał kroki na Debianie. 

1. Zainstaluj Dockera
W zależności od systemu operacyjnego, będzie się to trochę różniło.

Przykład na Debianie/Ubuntu
```
sudo apt update
sudo apt install docker docker-compose
```

2. Utwórz folder pod pliki
Aby zachować porządek, utworzymy folder w katalogu domowym gdzie będziemy trzymać wszystkie elementy lemmy - w zależności od konfiguracji dysków lub preferencji, możesz go również utworzyć w innym miejscu.
```
mkdir ~/lemmy
```

3. Utwórz docker-compose
Lemmy dzieli się na pięć (lub cztery jeżeli chcemy wykorzystać oddzielne reverse proxy) elementy:
* Lemmy - główna aplikacja która serwuje API oraz obsługuje wymianę informacji
* Lemmy-ui - kontener z interfejsem webui do interakcji z instancją
* pict-rs - usługa do przechowywania obrazków na instancji
* Postgres - baza danych
* Nginx - jako reverse proxy które kieruje ruch do odpowiednich kontenerów - jest on opcjonalny jeżeli mamy już inne reverse proxy wystawione w sieci.

```yaml

```

4. Utwórz plik konfiguracyjny lemmy
W folderze w którym utworzyłeś `docker-compose.yml`, utwórz plik `lemmy.hjson` - jest to główny plik konfiguracyjny lemmy, gdzie skonfigurujemy dane dostępowe do bazy danych, pict-rs, hostname oraz opcjonalne dane SMTP.

W podstawowej konfiguracji, jedyną rzeczą jaką należy tutaj zmienić, jest `hostname` (powinien odpowiadać domenie pod którą chcesz wystawić instancję) oraz `pictrs.api_key` (powinien się pokrywać z api_key zdefiniowanym w `docker-compose`)
```
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
