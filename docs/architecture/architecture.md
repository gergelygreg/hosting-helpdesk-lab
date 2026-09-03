# Hosting Helpdesk Lab – architektúra

## Cél

A laborkörnyezet célja egy egyszerű, reprodukálható hosting infrastruktúra létrehozása,
amelyben valósághű L1 helpdesk incidensek előállíthatók, diagnosztizálhatók és
dokumentálhatók.

Az MVP nem teljes hosting szolgáltatói infrastruktúrát modellez. A fókusz azon van,
hogy a domain-, DNS-, HTTP/HTTPS-, web- és Linux/VPS-problémák első szintű
hibakeresése gyakorolható legyen.

## Logikai architektúra

~~~text
                    Kliens / Support workstation
                              |
               +--------------+--------------+
               |                             |
            DNS lookup                    HTTP/HTTPS
               |                             |
               v                             v
          DNS konfiguráció              Linux host
                                             |
                                  +----------+----------+
                                  |                     |
                               Nginx                  SSH
                                  |
                                  v
                           Webalkalmazás
                                  |
                               MariaDB
~~~

## MVP komponensek

### 1. Support workstation

A hibakeresést végző kliensgép.

Feladatai:

- DNS lekérdezések végrehajtása
- HTTP/HTTPS elérhetőség ellenőrzése
- TCP kapcsolat tesztelése
- SSL/TLS tanúsítvány vizsgálata
- SSH kapcsolat ellenőrzése
- hibakeresési eredmények dokumentálása

Használt eszközök:

- nslookup
- dig
- curl
- ping
- tracert / traceroute
- Test-NetConnection
- openssl
- ssh

### 2. Linux hosting host

Az MVP szerveroldali környezete.

Tervezett rendszer:

- Ubuntu Server

Feladata:

- webszolgáltatás futtatása
- Nginx reverse proxy / webserver biztosítása
- SSH hozzáférés
- alkalmazás- és rendszerlogok biztosítása
- később WordPress és MariaDB futtatása

### 3. Docker

A szolgáltatások elkülönített és könnyen reprodukálható futtatási környezete.

Tervezett konténerek:

- Nginx
- később WordPress
- később MariaDB

Az MVP első részében csak a szükséges komponenseket vezetjük be.

### 4. DNS

A DNS-réteg segítségével reprodukálhatók például:

- hibás A rekord
- hibás vagy hiányzó MX rekord
- hibás CNAME rekord
- később SPF / DKIM / DMARC konfigurációs problémák

A DNS vizsgálat során a kliensoldali diagnosztika a fontos:

- névfeloldás sikeressége
- visszaadott IP-cím
- rekord típusa
- authoritative válasz
- TTL
- DNS és alkalmazásszintű hiba elkülönítése

### 5. Nginx

A labor első webszolgáltatása.

Felhasználása:

- HTTP szolgáltatás
- HTTPS konfiguráció
- statikus tesztoldal
- HTTP státuszkódok vizsgálata
- logelemzés
- konfigurációs hibák reprodukálása

### 6. SSL/TLS

Az SSL/TLS réteg segítségével reprodukálható például:

- lejárt tanúsítvány
- nem megfelelő hostname
- hibás certificate chain
- HTTPS konfigurációs probléma

Vizsgálati eszközök:

- curl
- openssl s_client
- böngésző

## MVP incidensek és architektúra-kapcsolat

| Incidens | Érintett réteg | Elsődleges diagnosztikai eszközök |
|---|---|---|
| INC-001 – hibás DNS A rekord | DNS | nslookup, dig, ping, curl |
| INC-002 – hibás MX rekord | DNS / e-mail | nslookup, dig |
| INC-003 – SSL/TLS probléma | HTTPS / TLS | curl, OpenSSL |
| INC-004 – HTTP 500 | Nginx / alkalmazás | curl, Nginx logok |

## Support szemlélet

Az incidensek vizsgálatánál nem az a cél, hogy azonnal szerveroldali konfigurációt
módosítsunk.

Az L1 helpdesk folyamat elsődleges célja:

1. a tünet pontosítása
2. az érintett szolgáltatási réteg meghatározása
3. alapvető diagnosztika végrehajtása
4. bizonyítékok gyűjtése
5. annak eldöntése, hogy a hiba L1 szinten megoldható-e
6. megoldás vagy megfelelő eszkaláció
7. visszaellenőrzés
8. dokumentáció

## Tervezett megvalósítási sorrend

1. Linux hosting host előkészítése
2. Docker ellenőrzése
3. Nginx baseline webszolgáltatás
4. kliens–szerver kapcsolat ellenőrzése
5. normál HTTP működés dokumentálása
6. INC-001 DNS A rekord hiba
7. INC-002 MX rekord hiba
8. INC-003 SSL/TLS hiba
9. INC-004 HTTP 500 hiba
10. tudásbázis-bejegyzések elkészítése

## MVP sikerkritérium

Az MVP akkor tekinthető elkészültnek, ha:

- működik egy dokumentált baseline webszolgáltatás
- mind a négy MVP incidens reprodukálható
- minden incidenshez van diagnosztikai bizonyíték
- dokumentált a gyökérok és a megoldás
- minden esetben szerepel eszkalációs döntés
- legalább két tudásbázis-cikk elkészül
