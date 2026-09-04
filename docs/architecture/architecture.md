# Hosting Helpdesk Lab – architektúra

## Cél

A Hosting Helpdesk Lab célja egy egyszerű, reprodukálható hosting infrastruktúra
kialakítása, amelyben valósághű L1 helpdesk incidensek állíthatók elő,
diagnosztizálhatók és dokumentálhatók.

A projekt nem épít külön virtualizációs és hálózati infrastruktúrát.
A már működő Manufacturing IT Support Lab homelab-alapjait használja újra.

A két projekt ugyanazt a fizikai/virtuális laborkörnyezetet használhatja,
de szakmailag és repository szinten különálló marad.

## Kapcsolódó alapinfrastruktúra

A Hosting Helpdesk Lab az alábbi, már meglévő komponensekre épül:

- Windows host
- Hyper-V
- PANNON-LAB belső hálózat
- Windows Server alapú DC01
- DNS
- DHCP
- Windows kliensgépek
- WSL2
- Ubuntu
- Docker
- Docker Compose
- Windows host és WSL2 közötti porttovábbítás

A meglévő hálózat:

~~~text
PANNON-LAB
192.168.50.0/24

Windows Host / Gateway
192.168.50.1

DC01
192.168.50.10
DNS / DHCP / Active Directory
~~~

## A két projekt szerepe

### Manufacturing IT Support Lab

A Manufacturing IT Support Lab főként belső vállalati IT support folyamatokat modellez:

- Windows workstation support
- Active Directory
- felhasználó- és jogosultságkezelés
- GPO
- DHCP
- DNS
- hálózati hibakeresés
- workstation deployment
- Shop Floor Control támogatás
- asset management
- ticketing és eszkaláció

Repository:

https://github.com/gergelygreg/manufacturing-it-support-lab

### Hosting Helpdesk Lab

A Hosting Helpdesk Lab ugyanennek az infrastruktúrának az alapjaira épít,
de külső hosting ügyféltámogatási helyzeteket modellez:

- domain és DNS
- webhosting
- HTTP / HTTPS
- SSL/TLS
- e-mailhez kapcsolódó DNS
- Linux / VPS
- Nginx
- SSH
- hosting incidensek
- ügyfélkommunikáció
- tudásbázis
- L1 → L2 eszkaláció

## Logikai architektúra

~~~text
                       PANNON-LAB
                     192.168.50.0/24
                            |
          +-----------------+-----------------+
          |                                   |
        DC01                         Support workstation
   192.168.50.10                     Windows kliens
   DNS / DHCP
          |
          | DNS feloldás
          |
          v
   Windows Host
   192.168.50.1
          |
          | port forwarding
          v
        WSL2
   Ubuntu + Docker
          |
     +----+-------------------+
     |                        |
   Nginx                 későbbi szolgáltatások
     |                        |
 HTTP / HTTPS            WordPress / MariaDB
     |
     v
Hosting tesztoldal
~~~

## Infrastruktúra újrahasznosítása

A projekt fontos tervezési elve, hogy ne duplikáljuk a már meglévő
infrastruktúrát.

Ezért nem hozunk létre külön:

- Hyper-V hostot
- új Windows Server domain controllert
- új DHCP-szervert
- külön alap hálózatot
- külön Docker hostot

Ehelyett a Hosting Helpdesk Lab új szolgáltatási réteget épít a meglévő
homelab infrastruktúrára.

## Hosting-specifikus új komponensek

### 1. Hosting DNS névtér

A hosting incidensekhez külön teszt DNS-névtér készül.

Tervezett zóna:

~~~text
hosting.test
~~~

Tervezett rekordok például:

~~~text
web.hosting.test
www.hosting.test
mail.hosting.test
~~~

A `.test` tartomány kizárólag labor- és tesztelési célra használható névtérként
szolgál a projektben.

A DNS-zóna a meglévő DC01 DNS szolgáltatásán kerül kialakításra.

### 2. Support workstation

A meglévő Windows kliens használható L1 support workstationként.

A kliensről történik:

- DNS hibakeresés
- TCP kapcsolat ellenőrzése
- HTTP/HTTPS teszt
- szolgáltatás-elérhetőség vizsgálata
- SSL/TLS diagnosztika
- incidens reprodukció
- bizonyítékgyűjtés

Tervezett eszközök:

- nslookup
- ping
- tracert
- Test-NetConnection
- curl
- böngésző
- SSH

Linux oldalról később:

- dig
- curl
- openssl
- ssh

### 3. WSL2 Ubuntu

A már meglévő WSL2 Ubuntu környezet lesz a hosting szolgáltatások
Linux oldali futtatási környezete.

Feladata:

- Docker futtatása
- Nginx futtatása
- Linux oldali logok biztosítása
- HTTP/HTTPS szolgáltatások
- később további hosting komponensek

### 4. Docker

A szolgáltatásokat lehetőség szerint Docker konténerekben futtatjuk.

Ennek előnye:

- reprodukálható környezet
- egyszerű konfiguráció
- kontrollált hibák előállítása
- gyors visszaállítás baseline állapotba
- szolgáltatások elkülönítése

Az MVP első szolgáltatása:

~~~text
Nginx
~~~

Későbbi komponensek lehetnek:

- WordPress
- MariaDB

### 5. Nginx

Az Nginx lesz az MVP első webszolgáltatása.

Feladata:

- HTTP kiszolgálás
- statikus tesztoldal
- HTTPS
- HTTP státuszkódok vizsgálata
- logok biztosítása
- kontrollált webes hibák előállítása

## Baseline állapot

Minden incidens előtt szükségünk van egy bizonyítottan működő alapállapotra.

A baseline útvonal:

~~~text
Support workstation
        |
        | DNS query
        v
      DC01
        |
        | web.hosting.test
        v
192.168.50.1
Windows Host
        |
        | port forwarding
        v
      WSL2
        |
        v
      Docker
        |
        v
      Nginx
        |
        v
HTTP 200 OK
~~~

A baseline csak akkor tekinthető működőnek, ha:

- a DNS név feloldható
- a megfelelő IP-cím kerül visszaadásra
- a cél TCP port elérhető
- az Nginx válaszol
- a tesztoldal böngészőből elérhető
- `curl` használatával HTTP 200 válasz érkezik

## MVP incidensek

### INC-001 – hibás DNS A rekord

Normál állapot:

~~~text
web.hosting.test
        |
        v
192.168.50.1
        |
        v
Nginx
~~~

Hibás állapot:

~~~text
web.hosting.test
        |
        v
hibás IP-cím
        |
        v
weboldal nem érhető el
~~~

L1 vizsgálat:

- nslookup
- DNS rekord ellenőrzése
- IP-cím ellenőrzése
- Test-NetConnection
- curl
- DNS és webszolgáltatás hibájának elkülönítése

### INC-002 – hibás MX rekord

A második incidens az e-mail szolgáltatáshoz tartozó DNS konfigurációra koncentrál.

Az MVP-ben nem szükséges teljes mail szervert felépíteni.

Vizsgált témák:

- MX rekord
- cél hostname
- A rekord
- DNS-feloldás
- e-mail routing alapjai

### INC-003 – SSL/TLS probléma

Vizsgált réteg:

- HTTPS
- SSL/TLS
- tanúsítvány
- hostname
- certificate validation

Diagnosztikai eszközök:

- böngésző
- curl
- openssl

### INC-004 – HTTP 500

Ebben az incidensben:

- a DNS működik
- a hálózati kapcsolat működik
- a webszerver elérhető
- az alkalmazási réteg HTTP 500 választ ad

A cél annak felismerése, hogy a hiba nem DNS- vagy hálózati eredetű.

L1 feladat:

- hiba reprodukálása
- HTTP státuszkód rögzítése
- alap logvizsgálat
- bizonyítékgyűjtés
- érintett réteg meghatározása
- megfelelő eszkaláció

## L1 helpdesk folyamat

Az incidensek vizsgálatakor az első lépés nem a szerverkonfiguráció módosítása.

A követett folyamat:

1. ügyfél hibabejelentésének értelmezése
2. tünetek pontosítása
3. érintett szolgáltatás azonosítása
4. reprodukció
5. L1 diagnosztika
6. bizonyítékgyűjtés
7. érintett technikai réteg meghatározása
8. L1 szintű megoldás vagy eszkaláció
9. visszaellenőrzés
10. ügyfél tájékoztatása
11. incidens dokumentálása
12. szükség esetén tudásbázis frissítése

## Repository-határ

A két repository nem duplikálja egymás dokumentációját.

A Manufacturing IT Support Lab tartalmazza a közös alapinfrastruktúra
részletes felépítését.

A Hosting Helpdesk Lab csak:

- hivatkozik a meglévő homelab alapokra
- dokumentálja a hosting-specifikus kiegészítéseket
- tartalmazza a hosting incidenseket
- tartalmazza a hosting tudásbázist
- tartalmazza a szükséges konfigurációkat és scripteket

## Módosított megvalósítási sorrend

1. meglévő PANNON-LAB infrastruktúra ellenőrzése
2. DC01 és DNS működésének ellenőrzése
3. WSL2 Ubuntu ellenőrzése
4. Docker működésének ellenőrzése
5. hosting baseline Nginx szolgáltatás létrehozása
6. `hosting.test` DNS-zóna létrehozása
7. `web.hosting.test` normál működésének validálása
8. baseline dokumentálása
9. INC-001 – hibás DNS A rekord
10. INC-002 – hibás MX rekord
11. INC-003 – SSL/TLS probléma
12. INC-004 – HTTP 500
13. tudásbázis-bejegyzések elkészítése
14. MVP végső validálása

## MVP sikerkritérium

Az MVP akkor tekinthető elkészültnek, ha:

- a meglévő homelab infrastruktúrán működik a hosting szolgáltatás
- dokumentált a működő baseline állapot
- a `hosting.test` DNS-névtér működik
- mind a négy MVP incidens reprodukálható
- minden incidenshez tartozik diagnosztikai bizonyíték
- minden incidensnél dokumentált a gyökérok
- minden incidensnél szerepel L1 megoldási vagy eszkalációs döntés
- a javítások visszaellenőrzése dokumentált
- legalább két hosting tudásbázis-bejegyzés elkészül
