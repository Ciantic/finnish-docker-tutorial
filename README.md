# Docker tutorial

Tämän dokumentin on kirjoittanut Jari Pennanen, 2020.

Docker on ohjelmisto jolla voidaan ajaa valmiiksi koottuja levykuvia. Esim. oma ohjelmisto luodaan levykuvaksi, jolloin ohjelmiston ympäristö olisi mahdollisimman yhtenäinen. 

Termi levykuva tässä dokumentissa tarkoittaa tiedostoja joista järjestelmä muodostuu, eikä bitti bitiltä olevaa levykuvaa jossa olisi esim. boottisektori tai muita kiintolevykohtaisia asetuksia kuten blokkikokoja.

Asenna Docker for Windows (Huom! Ei mikään kevyt pieni ohjelma Windowssissa, katso alempaa):
https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe

## Levykuvan ajaminen Dockerilla

Ajaminen aloitetaan käskyllä:

```bash
docker run levykuvannimi
```

Käsky käynnistää levykuvan, jonka *ilmentymästä* käytetään nimitystä kontti (Container). Jos käynnistäisit kolme kertaa yllä olevan käskyn olisi sinulla sitten kolme konttia päällä levykuvasta "levykuvannimi".

Mutta yleensä annetaan enemmän vipuja, esimerkiksi:

```bash
# --rm tarkoittaa, että kontin pysähdyttyä se tuhotaan.
# Jollet halua jatkaa kontin ajamista myöhemmin käytä aina --rm, 
# muutoin se voi jäädä hengailemaan.
docker run --rm levykuvannimi

# -d tarkoittaa "detach" eli se ajetaan taustalla
docker run --rm -d levykuvannimi

# --name voidaan antaa oma nimi kontille, jotta se on helpompi esim pysäyttää 
# tms. koska tietää suoraan kontin nimen
docker run --rm --name omakontti levykuvannimi

# Defaulttina ajettu kontti on eristetty isäntäkoneesta, se ei 
# voi kiinnittyä kuuntelemaan portteja eikä se voi kirjoittaa isäntäkoneen 
# tiedostoihin. Kontti pääsee kyllä internettiin oletuksena.

# -p tarkoittaa että isäntäkoneen portti 1234 kiinnitetään kontin porttiin 80
docker run --rm -p 1234:80 levykuvannimi

# -v mounttaus, halutaan mountata C:\omakoodi kontin hakemistoon /var/omakoodi
docker run --rm -v C:\omakoodi\:/var/omakoodi levykuvannimi

# -e ympäristömuuttujan asettaminen -e NIMI=arvo, joka annetaan ajettavalle 
# kontille
docker run --rm -e FOO=123 levykuvannimi

# Käynnistetään levykuva ubuntu ja sen sisällä /bin/bash.
# -it = interactive, eli annetaan näppäimistösyöte kontille
docker run --rm -it ubuntu /bin/bash

# kokonaisempi esimerkki
docker run --rm --name omakontti -it -v C:\omakoodi\:/var/omakoodi ubuntu /bin/bash
apt-get update
apt-get install nano
ls /
uname -a
nano /var/omakoodi/test.txt
...
Ctrl+D poistuu


# HUOM! Jos runille antaa lipun:
--privileged
# tarkoittaa tämä sitä että kontilla on kaikki oikeudet isäntäkoneeseen! 
# Tämä ei ole hyvä asia luonnollisesti ja siitä ei ole esimerkkiä tässä.
```

## Valmiita levykuvia

Docker hakee levyvkuvat palvelimeltaan ja ne tulee defaulttina Docker hubista: 

https://hub.docker.com/

Viralliset, dockerin ylläpitämät levykuvat eivät sisällä kauttamerkkiä esim. `ubuntu` tai `mysql` tai `postgres`. Käyttäjien itse tekemät levykuvat sisältävät kauttamerkin esim: `kayttajanimi/levykuvannimi` Levykuvia voi hakea myös eri domaineista kuten [mcr.microsoft.comista](http://mcr.microsoft.com/).

### MySQL example

https://hub.docker.com/_/mysql

```bash
docker run -p 3306:3306 --name omamysql57 -e MYSQL_ROOT_PASSWORD=jorkki1234 --rm -d mysql:5.7
docker run -p 33060:33060 --name omamysqluusin -e MYSQL_ROOT_PASSWORD=jorkki1234 --rm -d mysql

# `exec` Suorita ajossa olevan omamysqluusin kontin sisällä bash
docker exec -it omamysqluusin /bin/bash

mysql -u root -p
jorkki1234
create database test;
use test;
create table person (name VARCHAR(256));
insert into person VALUES("jorkki");
# ...
```

### Postgres example

https://hub.docker.com/_/postgres

```bash
docker run --name omapostgres -e POSTGRES_ROOT_PASSWORD=1234 --rm -d postgres
docker exec -it omapostgres bash
psql -U postgres
create database test;
create table person (name VARCHAR(256));
insert into person VALUES("jorkki");
\?
# ...
```

### MSSQL

https://hub.docker.com/_/microsoft-mssql-server

```bash
docker run --rm --name omamssql -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=!Password1" -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-CU8-ubuntu
```

### ASP.Net core esimerkkiohjelmisto

```bash
docker run --rm -it -p 8000:80 mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

### Redis-muistitietokanta

https://hub.docker.com/_/redis

```bash
docker run --rm --name omaredis -d redis
```

## Peruskomentoja

```bash
# Suoritetaan *ajossa olevan kontin* sisällä /bin/bash
# tällä voidaan mennä esim tarkkailemaan mitä kontin sisällä tapahtuu
# (-it on taas interactive)
docker exec -it kontinnimi /bin/bash

# näytä kaikki kontit
docker ps -a 

# pysäytä tietty kontti
docker stop kontinnimi

# Jos kontti jää stopin jälkeen hengailemaan `Exited` tilaan, se pitää tuhota:
docker rm kontinnimi

# Näytä levykuvat
docker images

# Poista levykuva
docker rmi levykuvannimi

# Näytä reaaliaikaisia tietoja ajossa olevista konteista
docker stats

# Pysäytä/poista kaikki kontit tai kuvat
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker rmi $(docker images -q)

# Vähän erikoisempia:

# Voit kopioida omalta koneelta tiedoston ajossa olevan kontin sisään 
# tai toisinpäin. Kätevä jos esim kääntää kontin sisässä jotain ja kaipaa
# sieltä vain yhtä tiedostoa takaisin isäntäkoneelle.
docker cp --help

# Voluumit, tarvii vain jos on mennyt tekemään voluumeja:
docker volume ls
docker volume prune # Poista voluumit jotka ei ole käytössä

# Poista "kaikki käyttämätön data"
docker system prune
```

## Oman levykuvan tekeminen

Levykuvat tehdään `Dockerfile` nimisinä tiedostoina. Tiedostossa annetaan ohjeet joiden perusteella docker luo levykuvan, eli annetun ohjeistuksen täytyy pystyä pysähtymään jonka jälkeen docker tallentaa lopputuloksen levykuvaksi. Toisinsanoen levykuvan luontiscripteissä ei voi ajaa esimerkiksi komentorivitulkkia tai muita ohjelmia jotka vaativat interaktioita.

Luo tyhjä hakemisto jonne tiedostot:

`Dockerfile`

```Dockerfile
# Tämä levykuva peritään toisesta levykuvasta, esimerkiksi "ubuntu"
FROM ubuntu

# Eri arkkitehtuurit, esim:
# FROM arm64v8/ubuntu
#
# https://github.com/docker-library/official-images#architectures-other-than-amd64

# Levykuvaa luodessa ajetaan komentoja RUN ...
RUN apt-get update -qq -y && apt-get -y install \
  build-essential \
  gcc
  
RUN echo "foo" > /testfile

# Koita laittaa virhe tähän kohti:
# RUN efcho "virhe??"
# Sitten aja build uudestaan, huomaat kuinka build toiminto jatkuu 
# itseasiassa edellisestä komennosta, eikä sen tarvitse enää asentaa riippuvuuksia
# Tämä johtuu siitä että RUN ja muutkin komennot luovat joka rivillä snapshotin 
# josta voi jatkaa.

# kopiodaan omalta koneelta tiedosto levykuvan hakemistoon /jotain/
COPY main.c /jotain/main.c

# Levykuvassa siirrytään hakemistoon /jotain ja käännetään
WORKDIR /jotain
RUN gcc main.c

# Levykuvalle voi lopuksi antaa käskyn, joka suoritetaan kun kontti 
# suoritetaan. Tätä ei siis ajeta levykuvaa luodessa. 
#
# Esimerkiksi ubuntu ei määrittele mitään, mutta mysql serveri
# määrittelee kommenoksi mysql serverin. Tälle levykuvalle annetaan 
# komennoksi itse käännetty binääri jonka gcc tuotti.
CMD /jotain/a.out
```

`main.c`

```
#include <stdio.h>
int main() { 
    printf("Hello, World!\n");
    printf("Press enter to close...\n");
    getchar();
    return 0; 
}
```

Nyt tästä voi luoda oman levykuvan, joka tallennetaan vain omalle koneelle:

```bash
# build tässä hakemistossa oleva Dockerfile, ja anna sille -t nimeksi "omalevykuva"
docker build . -t omalevykuva

# Jos levykuva luotiin onnistuneesti, voit ajaa sen näin:
docker run --rm -it omalevykuva

# ja voi tarkastella omaa levykuvaa listassa näin
docker images
```

### Lisähuomioita

-   Oikeasti ohjelmaa ei kannata periä `ubuntu` tai `debian` distribuutioista, koska ne ovat liian isoja. Näihin käytetään erikoistuneita distribuutioita kuten esim [Alpine Linux](https://alpinelinux.org/) eli levykuva `alpine`. Tosin näissä pienemmissä distribuutioissa on se ongelma, että niistä puuttuu paljon, ja niillä alkuun pääseminen voi olla hankalampaa.
-   Dockerfilessä voi käyttää useampaa `FROM` käskyä. Tällöin voi esim. tehdä ohjelman kääntämisen omassa ja ajamisen toisessa osassa. Jolloin lopullisesta levykuvasta tulee huomattavasti pienempi. [Lisätietoja Dockerin sivustolta](https://docs.docker.com/develop/develop-images/multistage-build/).
-   Yleensäkkin kannattaa vähemmän muuttuvat asiat tehdä omaksi levykuvaksi, esimerkiksi jos jostain syystä tarvitsee tehdä levykuvan jossa on x-serveri ja sen päällä oleva ohjelmisto, niin tekee kaksi levykuvaa: x-serverille ja ohjelmistolle omansa. Sitten perii ohjelman levykuvan x-serveristä. Tällöin kun ohjelmisto muuttuu ei tarvitse välttämättä päivittää x-serverin levykuvaa lainkaan.
-   Viralliset levykuvat on saatavilla eri arkkitehtuureille, [lue lisää dockerin ohjeista](https://github.com/docker-library/official-images#architectures-other-than-amd64).

## Docker teknisesti Linuxissa

Käyttää [linuxin nimiavaruuksia](https://en.wikipedia.org/wiki/Linux_namespaces), eli mahdollistaa useamman root käyttäjän jotka näkee kukin vain tietyt prosessit, kahvat, verkot yms. Eli kukin kontti ajetaan tavallaan omassa nimiavaruudessaan, jolloin ne voi käyttää kaikki omaa eristettyä root käyttäjää. Linuxissa on tavallaan tuki konteille, joten niitä ei tarvitse virtualisoida mitenkään.

>   "With administrative assistance it is possible to build a container with seeming administrative rights without actually giving elevated privileges to user processes." -Wikipediasta napattu

Tästä on myös joitain ongelmia, esim. jos mounttaat hakemiston kontin sisään, ja käyttäjä kontin sisällä on root niin tiedostot tulevat root käyttäjälle isäntäkoneella. Tähän on kiertoja, ja kannattaa selvitellä niitä kun on tarpeen.

Docker vaatii myös konttien ajajalta käytännössä root oikeudet, joka voi olla ongelmallista jos palvelin on monikäyttöpalvelin. Tähän on parempia keinoja kuten [podman](https://podman.io/) joka on dockeria vastaava ja käyttää dockerin levykuvia ym. mutta sallii konttien ajamisen linuxin normaaleilla käyttäjillä.

Jos haluaa ajaa dockerpalvelinta tuotannossa, niin kannattaa sitä varten olla oma palvelin jossa pyörittää vain dockeria. Tarkoituksena on ajaa dockerpalvelimella vain kontteja eikä mitään muita sovelluksia. Kehityskoneella tällä ei ole niinkään merkitystä.

## Docker teknisesti Windowssissa

[Docker for Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows) ([lataa tästä linkistä](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)) ajaa yhtä Linux virtuaalikonetta Hyper-V:n kautta. Jokainen kontti jakaa siis yhden Linux palvelimen, jonka sisällä ne pyörivät. Microsoft on tehnyt myös Docker for Windowsiin tuen eri arkkitehtuureille, josta syystä voi esim. pyörittää ARM-pohjaista kontteja suoraan Windowssissa. Linuxissa tämä eri arkkitehtuurien tuki pitää asentaa erikseen.

## Docker Compose

`docker-compose` on yksinkertanen ohjelma, jolla voi käynnistää eri kontteja yhtä aikaa. Esim. ajamaan MSSQL serverin ja oman ASP.NET-ohjelman yhtäaikaisesti, sekä yhdistämään ne sisäverkossa keskenään. Sen oppiminen on aika nopeaa kun osaa edellä mainitut Dockerin perusteet. Kehittäjille käytännöllinen työkalu, tulee Docker for Windowssissa mukana.

## Kubernetes

Konttien orkesterointi ohjelmisto, esim Azure ja Google pyörittää kubernetestä jonne voi viedä suoraan omia levykuvia joista kubernetes sitten käynnistää kontteja. Ohjelmisto kehitetään Docker levykuvina (Dockerfileinä), sitten nämä levykuvat viedään Kubernetekseen ohjeistetusti, joka käynnistää automaattisesti tuotantoon kontteja.

Idea tässä on, että ei tarvitse miettiä enää palvelimen ylläpitoa. Vaan riittää pystyä muodostamaan toimivat levykuvat ohjelmistosta, jotka sitten viedään tuotantoon automatisoidusti. Kuulostaa yksinkertaiselta, mutta käytännössä tämän hallitseminen on kuitenkin aika paljon hankalampi asia kuin Docker itsessään, joten tästä ei sen enempää.