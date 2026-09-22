# Network+ Homelab -- UFW Firewall op Ubuntu Server

## Uitgangssituatie

Ubuntu Server draait als VM in Hyper-V.

``` text
Ubuntu Server: 192.168.178.196
SSH:            TCP/22
HTTP / Nginx:   TCP/80
HTTPS / Nginx:  TCP/443
```

Doel: een host-based firewall op de Ubuntu-server activeren waarbij
inkomend verkeer standaard wordt geblokkeerd en alleen benodigde
services worden toegestaan.

------------------------------------------------------------------------

## 1. UFW-status controleren

Via SSH ingelogd op de Ubuntu-server.

Status van UFW gecontroleerd:

``` bash
sudo ufw status verbose
```

Bij een nog niet geactiveerde firewall:

``` text
Status: inactive
```

UFW nog niet direct ingeschakeld, omdat eerst SSH moet worden toegestaan
om te voorkomen dat de actieve SSH-verbinding wordt geblokkeerd.

------------------------------------------------------------------------

## 2. Default firewall policy instellen

Inkomend verkeer standaard blokkeren:

``` bash
sudo ufw default deny incoming
```

Uitgaand verkeer standaard toestaan:

``` bash
sudo ufw default allow outgoing
```

Firewallprincipe:

``` text
INCOMING  → DENY tenzij expliciet toegestaan
OUTGOING  → ALLOW
```

Dit is een default-deny/allowlist-benadering.

------------------------------------------------------------------------

## 3. Beschikbare UFW application profiles bekijken

Beschikbare profielen opgevraagd:

``` bash
sudo ufw app list
```

Op de server waren onder andere de volgende profielen beschikbaar:

``` text
Nginx Full
Nginx HTTP
Nginx HTTPS
Nginx QUIC
OpenSSH
```

Betekenis:

  UFW-profiel   Protocol / poort   Functie
  ------------- ------------------ -------------------
  OpenSSH       TCP/22             SSH remote beheer
  Nginx HTTP    TCP/80             HTTP
  Nginx HTTPS   TCP/443            HTTPS
  Nginx Full    TCP/80 + TCP/443   HTTP en HTTPS
  Nginx QUIC    UDP/443            QUIC / HTTP/3

Voor het huidige lab is `Nginx QUIC` niet nodig.

------------------------------------------------------------------------

## 4. SSH toestaan

SSH eerst toegestaan voordat UFW wordt geactiveerd:

``` bash
sudo ufw allow OpenSSH
```

Hiermee wordt TCP-poort 22 toegestaan.

Alternatief kan de poort rechtstreeks worden toegestaan:

``` bash
sudo ufw allow 22/tcp
```

Slechts één van beide regels is nodig.

------------------------------------------------------------------------

## 5. HTTP en HTTPS toestaan

Omdat Nginx zowel HTTP als HTTPS gebruikt, het profiel `Nginx Full`
toegestaan:

``` bash
sudo ufw allow 'Nginx Full'
```

Hiermee worden toegestaan:

``` text
TCP/80  → HTTP
TCP/443 → HTTPS
```

Alternatief kunnen beide poorten afzonderlijk worden toegestaan:

``` bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

------------------------------------------------------------------------

## 6. Firewallregels controleren vóór activatie

Toegevoegde regels gecontroleerd:

``` bash
sudo ufw show added
```

Er moeten minimaal regels aanwezig zijn voor:

``` text
OpenSSH
Nginx Full
```

Hiermee is gecontroleerd dat SSH bereikbaar blijft nadat de firewall
wordt ingeschakeld.

------------------------------------------------------------------------

## 7. UFW activeren

Firewall geactiveerd:

``` bash
sudo ufw enable
```

Bevestigd met:

``` text
y
```

------------------------------------------------------------------------

## 8. Actieve firewallregels controleren

Status en uitgebreide informatie:

``` bash
sudo ufw status verbose
```

Regels met nummers weergeven:

``` bash
sudo ufw status numbered
```

De configuratie komt conceptueel neer op:

``` text
                 UFW
        ┌────────────────────┐
TCP/22  │ ALLOW → OpenSSH    │
TCP/80  │ ALLOW → HTTP       │
TCP/443 │ ALLOW → HTTPS      │
        │                    │
Overig  │ DENY → Incoming    │
        └────────────────────┘
                 │
                 ▼
           Ubuntu Server
          192.168.178.196
```

------------------------------------------------------------------------

## 9. Firewall vanaf Windows testen

Vanaf een andere Windows-machine gecontroleerd of SSH bereikbaar is:

``` powershell
Test-NetConnection 192.168.178.196 -Port 22
```

HTTP getest:

``` powershell
Test-NetConnection 192.168.178.196 -Port 80
```

HTTPS getest:

``` powershell
Test-NetConnection 192.168.178.196 -Port 443
```

Bij een toegestane poort hoort onder andere te staan:

``` text
TcpTestSucceeded : True
```

------------------------------------------------------------------------

## 10. Geblokkeerde poort testen

Een niet-toegestane TCP-poort kan worden getest, bijvoorbeeld Telnet op
TCP/23:

``` powershell
Test-NetConnection 192.168.178.196 -Port 23
```

Omdat hiervoor geen UFW allow-regel bestaat, hoort de TCP-verbinding
niet tot stand te komen.

Hiermee is het verschil zichtbaar tussen:

``` text
TCP/22  → toegestaan
TCP/80  → toegestaan
TCP/443 → toegestaan
TCP/23  → geblokkeerd
```

------------------------------------------------------------------------

## 11. SSH eventueel beperken tot het lokale netwerk

In plaats van SSH vanaf ieder netwerk toe te staan, kan toegang worden
beperkt tot het lokale subnet:

``` bash
sudo ufw allow from 192.168.178.0/24 to any port 22 proto tcp
```

Hiermee mag alleen een host uit:

``` text
192.168.178.0/24
```

verbinding maken met TCP/22.

Als eerst een algemene `OpenSSH`-regel is aangemaakt, moet die algemene
regel worden verwijderd voordat deze beperking daadwerkelijk effect
heeft. Controleer eerst de regelnummers:

``` bash
sudo ufw status numbered
```

Verwijder daarna alleen de betreffende algemene SSH-regel, bijvoorbeeld:

``` bash
sudo ufw delete 1
```

Controleer vervolgens opnieuw:

``` bash
sudo ufw status numbered
```

------------------------------------------------------------------------

## 12. UFW logging bekijken

Logging inschakelen:

``` bash
sudo ufw logging on
```

Status controleren:

``` bash
sudo ufw status verbose
```

UFW-logging kan worden gebruikt om toegestane/geblokkeerde
netwerkverbindingen te onderzoeken. Afhankelijk van de
Ubuntu-configuratie zijn UFW-events onder andere via de system
logs/journal terug te vinden.

------------------------------------------------------------------------

## Eindopstelling

``` text
                     Thuisnetwerk
                          │
                          ▼
                  192.168.178.196
                    Ubuntu Server
                          │
                    ┌──── UFW ────┐
                    │             │
          TCP/22 ──►│ OpenSSH     │
          TCP/80 ──►│ Nginx HTTP  │
         TCP/443 ──►│ Nginx HTTPS │
                    │             │
         Overig ──X │ DENY        │
                    └─────────────┘
```

## Network+ onderwerpen

Met deze configuratie worden praktisch behandeld:

-   Host-based firewall
-   Inbound en outbound traffic
-   Default deny
-   Allowlisting
-   TCP-poorten
-   SSH -- TCP/22
-   HTTP -- TCP/80
-   HTTPS -- TCP/443
-   QUIC / HTTP/3 -- UDP/443
-   IPv4
-   CIDR `/24`
-   Subnet-based firewallregels
-   Client/server-verkeer
-   Firewall troubleshooting
