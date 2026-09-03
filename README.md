# Hosting Helpdesk Lab

Gyakorlati L1 hosting support és hibakeresési labor, amely a webtárhelyhez, DNS-hez,
e-mailhez, SSL/TLS-hez és Linux/VPS környezethez kapcsolódó gyakori hibák
strukturált vizsgálatát és dokumentálását mutatja be.

A projekt célja egy valósághű helpdesk munkafolyamat modellezése az ügyfél
hibabejelentésétől az első szintű diagnosztikán keresztül a megoldásig vagy
eszkalációig, majd a visszaellenőrzésig és tudásbázis-dokumentációig.

> **Projekt státusz:** MVP fejlesztés alatt

## A projekt célja

A labor célja valós hosting support helyzetek gyakorlása és dokumentálása,
különös tekintettel az alábbi feladatokra:

- ügyfél által jelzett probléma pontosítása
- első szintű technikai diagnosztika
- az érintett szolgáltatási réteg azonosítása
- hibakeresési lépések dokumentálása
- L1 szinten megoldható problémák elhárítása
- szükség esetén eszkaláció
- a javítás visszaellenőrzése
- újra felhasználható tudásbázis-dokumentáció készítése

## Helpdesk munkafolyamat

~~~text
Ügyfél hibabejelentése
        |
        v
Tünetek pontosítása
        |
        v
Érintett szolgáltatás azonosítása
        |
        v
L1 diagnosztika
        |
        +------ L1 szinten megoldható ------> Megoldás
        |                                      |
        |                                      v
        |                                Visszaellenőrzés
        |
        +------ Nem oldható meg -----------> Eszkaláció
                                                |
                                                v
                                         Technikai átadás
~~~

Minden incidens reprodukálható és egységes formában kerül dokumentálásra.

## MVP hatóköre

Az első MVP négy gyakori hosting support problémára koncentrál.

| Incidens | Probléma | Fő témák |
|---|---|---|
| INC-001 | A weboldal hibás DNS A rekord miatt nem érhető el | DNS, A rekord, dig, nslookup |
| INC-002 | Bejövő e-mail hiba hibás MX konfiguráció miatt | DNS, MX, e-mail routing |
| INC-003 | HTTPS / tanúsítvány probléma | SSL/TLS, HTTPS, OpenSSL, curl |
| INC-004 | A weboldal HTTP 500 hibát ad | HTTP, Nginx, alkalmazásnaplók |

A későbbi incidensek további területeket is lefedhetnek:

- SPF, DKIM és DMARC
- SSH kapcsolódási hibák
- Linux jogosultsági problémák
- lemezterület- és tárhelyproblémák
- WordPress hibakeresés
- szolgáltatás-elérhetőség és eszkaláció

## Tervezett laborkörnyezet

Az MVP egy kis saját hosting környezetre épül.

~~~text
                     Kliens
                       |
                       v
                  DNS feloldás
                       |
              +--------+--------+
              |                 |
          DNS rekordok       Linux VPS
                                |
                         +------+------+
                         |             |
                       Nginx           SSH
                         |
                      HTTPS
                         |
                 Webes szolgáltatás
~~~

Tervezett technológiák:

- Ubuntu Server
- Docker
- Nginx
- DNS
- SSH
- HTTP / HTTPS
- OpenSSL
- curl
- dig
- nslookup
- WordPress
- MariaDB

Az első MVP elkészítéséhez nem szükséges minden felsorolt komponens használata.

## Incidens-dokumentáció

Minden incidens ugyanazt a support-orientált struktúrát követi:

1. Ügyfél hibabejelentése
2. Üzleti / felhasználói hatás
3. Tünetek
4. Első pontosító kérdések
5. L1 diagnosztikai lépések
6. Bizonyítékok és mérési eredmények
7. Gyökérok
8. Megoldás
9. Visszaellenőrzés
10. Eszkalációs döntés
11. Ügyfélnek küldött válasz
12. Kapcsolódó tudásbázis-bejegyzés

## Repository struktúra

~~~text
hosting-helpdesk-lab/
|
+-- docs/
|   +-- architecture/
|   +-- incidents/
|   +-- knowledge-base/
|
+-- lab/
|
+-- scripts/
|
+-- README.md
+-- .gitignore
~~~

## A projektben demonstrált készségek

- L1 IT support
- strukturált hibakeresés
- hibajegy- és incidensdokumentáció
- DNS hibakeresés
- HTTP / HTTPS hibakeresés
- Linux alapismeretek
- webhosting alapismeretek
- ügyfélorientált technikai kommunikáció
- eszkalációs döntéshozatal
- tudásbázis készítése

## Kapcsolódó projekt

A vállalati / workstation IT support területet egy külön projekt mutatja be:

https://github.com/gergelygreg/manufacturing-it-support-lab

A két projekt eltérő support környezetet fed le:

- **Manufacturing IT Support Lab** – Windows workstation, Active Directory,
  DHCP/DNS, vállalati hálózat és belső IT support
- **Hosting Helpdesk Lab** – domain, DNS, webhosting, e-mail, SSL/TLS,
  Linux/VPS és külső ügyféltámogatás

## Szerző

Gulácsi Gergely

GitHub: https://github.com/gergelygreg
