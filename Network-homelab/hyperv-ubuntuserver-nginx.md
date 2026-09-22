# Network+ Homelab -- Hyper-V, Ubuntu Server en Nginx

## 1. Laptop gecontroleerd

-   Windows-versie gecontroleerd met:

``` cmd
winver
```

-   Windows 10 Pro Education 22H2 aanwezig.
-   8 GB RAM aanwezig.
-   Virtualisatie gecontroleerd via **Task Manager → Performance →
    CPU**.
-   **Virtualization: Enabled**.

## 2. Hyper-V geïnstalleerd

-   `Windows + R` geopend.
-   Commando:

``` text
optionalfeatures
```

-   **Hyper-V** aangevinkt:
    -   Hyper-V Management Tools
    -   Hyper-V Platform
-   Windows opnieuw gestart.
-   **Hyper-V Manager** geopend.

## 3. External Virtual Switch aangemaakt

Via **Hyper-V Manager → Virtual Switch Manager**:

-   Type: **External**
-   Naam: `LAB-External`
-   Gekoppeld aan de fysieke Wi-Fi-adapter.
-   **Allow management operating system to share this network adapter**
    aangevinkt.
-   Geen VLAN ingesteld.

``` text
Internet
   │
Thuisrouter
192.168.178.1
   │
 Wi-Fi
   │
LAB-External
   │
Ubuntu VM
```

## 4. Probleem met Wi-Fi opgelost

Na het configureren van Hyper-V was de downloadsnelheid tijdelijk erg
laag.

Network Connections geopend:

``` text
ncpa.cpl
```

Hyper-V uitschakelen maakte geen verschil. De Wi-Fi-adapter opnieuw
gestart:

-   **Wi-Fi → Disable**
-   ±10 seconden gewacht.
-   **Wi-Fi → Enable**

Hierna was de downloadsnelheid weer normaal.

## 5. Ubuntu Server gedownload

-   Ubuntu Server LTS ISO gedownload.
-   Ubuntu Server gekozen in plaats van Ubuntu Desktop vanwege de
    beschikbare 8 GB RAM.

## 6. Ubuntu VM aangemaakt

Via **Hyper-V Manager → New → Virtual Machine**:

-   Name: `Ubuntu-NetworkLab`
-   Generation: **Generation 2**
-   Memory: **2048 MB**
-   **Dynamic Memory** ingeschakeld
-   Network: `LAB-External`
-   Virtual hard disk: **25 GB**
-   Ubuntu Server ISO gekoppeld als installation media.

## 7. Secure Boot ingesteld

Via **Ubuntu-NetworkLab → Settings → Security**:

-   **Enable Secure Boot**: aan
-   Template: **Microsoft UEFI Certificate Authority**

## 8. Ubuntu Server geïnstalleerd

VM gestart via **Connect → Start**.

Gekozen voor **Try or Install Ubuntu Server**.

Installatietaal: **English**.

## 9. Netwerkconfiguratie via DHCP

Ubuntu herkende:

``` text
eth0
DHCPv4
DHCPv6
```

Via DHCP werd toegewezen:

``` text
192.168.178.196/24
```

Netwerkconfiguratie:

``` text
IP address:       192.168.178.196
Subnet mask:      255.255.255.0
CIDR:             /24
Network:          192.168.178.0/24
Broadcast:        192.168.178.255
Default gateway:  192.168.178.1
```

DHCP DORA-proces: **Discover → Offer → Request → Acknowledgement**. DHCP
gebruikt UDP-poort 67 en 68.

## 10. Proxy-configuratie

Bij **Proxy configuration** niets ingevuld. **Proxy address** leeg
gelaten en **Done** gekozen.

## 11. Ubuntu package mirror

Ubuntu heeft tijdens de installatie de Ubuntu archive mirror getest.
Hiermee werd ook bevestigd dat internettoegang vanuit de VM werkte.

## 12. Storage geconfigureerd

Bij **Guided Storage Configuration**:

-   **Use an entire disk**
-   Virtuele disk van 25 GB geselecteerd
-   **LVM** ingeschakeld
-   Geen LUKS-encryptie

## 13. Ubuntu Pro overgeslagen

Bij **Upgrade to Ubuntu Pro** gekozen voor **Skip for now**.

## 14. OpenSSH Server geïnstalleerd

Bij **SSH Setup**:

-   **Install OpenSSH server** aangevinkt
-   Geen SSH identity geïmporteerd

SSH gebruikt standaard **TCP/22**.

## 15. Featured Server Snaps

Bij **Featured Server Snaps** niets geselecteerd en installatie laten
afronden.

## 16. Ubuntu opnieuw gestart

Na installatie **Reboot Now** gekozen. Bij de melding om installation
media te verwijderen via **VM Settings → SCSI Controller → DVD Drive**
gecontroleerd dat de ISO niet meer gekoppeld was (`None`). Daarna
`Enter` gedrukt.

## 17. Ingelogd op Ubuntu Server

IP-adres gecontroleerd:

``` bash
hostname -I
```

Resultaat:

``` text
192.168.178.196
```

Uitgebreide netwerkconfiguratie:

``` bash
ip addr
```

## 18. Routing gecontroleerd

``` bash
ip route
```

Default gateway:

``` text
default via 192.168.178.1
```

## 19. ARP/Neighbor table bekeken

``` bash
ip neigh
```

Hiermee zijn bekende koppelingen tussen lokale IP-adressen en
MAC-adressen zichtbaar.

## 20. Connectiviteit vanaf Windows getest

``` cmd
ping 192.168.178.196
```

Ubuntu reageerde op de ICMP Echo Requests.

## 21. SSH-verbinding vanaf Windows

``` powershell
ssh <username>@192.168.178.196
```

Bij de eerste verbinding de SSH host fingerprint geaccepteerd met `yes`,
daarna het Ubuntu-wachtwoord ingevoerd.

## 22. Luisterende netwerkservices bekeken

``` bash
ss -tulpn
```

Vanaf Windows TCP/22 getest:

``` powershell
Test-NetConnection 192.168.178.196 -Port 22
```

Resultaat:

``` text
TcpTestSucceeded : True
```

## 23. Ingelogde gebruikers bekeken

``` bash
who
```

Aantal sessies:

``` bash
who | wc -l
```

Meer informatie:

``` bash
w
```

-   `tty1` = lokale Hyper-V console
-   `pts/0` = SSH-sessie
-   `pts/1` = tweede SSH-sessie

Actieve TCP-verbindingen:

``` bash
ss -tn
```

## 24. Wireshark geïnstalleerd

Wireshark op een tweede Windows-machine geïnstalleerd. Capture
uitgevoerd op de actieve Wi-Fi-interface.

ICMP-filter:

``` text
icmp
```

Hiermee ICMP Echo Request en Echo Reply bekeken.

## 25. ARP bekeken met Wireshark

ARP-cache op Windows gewist:

``` cmd
arp -d *
```

Wireshark-filter:

``` text
arp
```

Daarna:

``` cmd
ping 192.168.178.196
```

ARP Request en ARP Reply bekeken. ARP-cache gecontroleerd met:

``` cmd
arp -a
```

## 26. TCP three-way handshake bekeken

Wireshark-filter:

``` text
tcp.port == 22
```

Nieuwe SSH-verbinding gemaakt:

``` powershell
ssh <username>@192.168.178.196
```

Hiermee de TCP three-way handshake bekeken: **SYN → SYN/ACK → ACK**.

## 27. DNS bekeken met Wireshark

DNS-cache geleegd:

``` cmd
ipconfig /flushdns
```

DNS-query:

``` cmd
nslookup example.com
```

Wireshark-filter:

``` text
dns
```

DNS-records:

-   `A` = IPv4-adres
-   `AAAA` = IPv6-adres
-   `MX` = Mail Exchange

``` cmd
nslookup -type=AAAA example.com
nslookup -type=MX gmail.com
```

Alleen DNS-requests:

``` text
dns.flags.response == 0
```

Zoeken naar een woord in een domeinnaam:

``` text
dns.qry.name contains "google"
```

Of:

``` text
dns.flags.response == 0 && dns.qry.name contains "google"
```

## 28. Nginx webserver geïnstalleerd

Repositories bijgewerkt:

``` bash
sudo apt update
```

Nginx geïnstalleerd:

``` bash
sudo apt install nginx -y
```

Status gecontroleerd:

``` bash
systemctl status nginx
```

Resultaat: **Active: active (running)**. Nginx luistert standaard voor
HTTP op **TCP/80**.

## 29. Webserver vanaf andere computer getest

In een browser geopend:

``` text
http://192.168.178.196
```

De standaardpagina **Welcome to nginx!** was zichtbaar.

## 30. Eigen website aangemaakt

Naar de Nginx webroot:

``` bash
cd /var/www/html
```

Bestanden bekeken:

``` bash
ls -la
```

Eigen homepage aangemaakt:

``` bash
sudo nano index.html
```

Opgeslagen met **Ctrl+O → Enter → Ctrl+X**.

Eventueel de standaard Nginx-pagina verwijderd:

``` bash
sudo rm /var/www/html/index.nginx-debian.html
```

Daarna opnieuw `http://192.168.178.196` geopend. De eigen `index.html`
werd weergegeven.

## 31. HTTP-verkeer bekeken met Wireshark

Filter voor verkeer tussen Windows en Nginx:

``` text
ip.addr == 192.168.178.196 && tcp.port == 80
```

Alleen HTTP:

``` text
http
```

Onder andere zichtbaar:

``` text
GET / HTTP/1.1
HTTP/1.1 200 OK
```

Omdat HTTP niet versleuteld is, zijn delen van het verkeer en de inhoud
in Wireshark leesbaar.

## Eindopstelling

``` text
                         INTERNET
                            │
                      Thuisrouter
                     192.168.178.1
                            │
               ┌────────────┴────────────┐
               │                         │
        Hyper-V Laptop             Windows Laptop
               │                     Wireshark
         LAB-External
               │
        Ubuntu Server
       192.168.178.196
               │
       ┌───────┴────────┐
       │                │
      SSH             Nginx
    TCP/22            TCP/80
       │                │
 Remote beheer      Website
                  index.html
```

## Behandelde Network+ onderwerpen

Hyper-V/virtualisatie, virtual switching, DHCP, IPv4, IPv6, CIDR,
subnetting, default gateway, routing, ARP, MAC-adressen, ICMP, DNS, TCP,
TCP three-way handshake, SSH, HTTP, poorten, client/server en packet
analysis met Wireshark.
