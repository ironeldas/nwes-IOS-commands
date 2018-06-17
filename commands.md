Cisco-IOS Befehle & Szenarien für die NWES-Matura
=================================================


Allgemeines
-----------

- `?` um context sensitive help aufzurufen
- Auto-Vervollständigung mit `TAB`
- Befehle können mit Keyword `no` am Anfang zurückgenommen/negiert werden
- Kommentare können in einer Zeile nach einem `!` (Rufezeichen) geschrieben werden
- Mit den folgenden Schlagwörtern nach einem pipe-character | ([Alt-Gr] + [</>]), kann der Output anhand von Bedingungen modifiziert werden
	- `section` - Shows entire section that starts with the filtering expression
	- `include` - Includes all output lines that match the filtering expression
	- `exclude` - Excludes all output lines that match the filtering expression
	- `begin` - Shows all the output lines from a certain point, starting with the line that matches the filtering expression
	- z. B. ´show ip int brief | include up` oder `show ip route | begin gateway`


Keyboard-Shortcuts
------------------

- Pfeiltasten: wie bei jeder CLI
- `Ctrl` + `A`: Spring zu Zeilenanfang
- `Ctrl` + `E`: Spring ans Ende der Zeile
- `Enter`: Zeile ausführen
- `Ctrl` + `Shift` + `6`: gerade ausführenden Befehl abbrechen ! meist ping, DNS query oder traceroute
- `Ctrl` + `P`: gleich wie Pfeiltaste nach oben
- `Ctrl` + `N`: gleich wie Pfeiltaste nach unten
- `Ctrl` + `C` in config mode: beendet config mode, wechselt auf priv exec mode
- `Ctrl` + `Z` in config mode: beendet config mode, wechselt auf priv exec mode


aus Windoof
-----------

```
ipconfig
ipconfig /all
ipconfig /reload
tracert <ip> ! ZB tracert 8.8.8.8 oder tracert www.orf.at !TODO auf IPv4 und oder IPv6
arp –a
ping
route print ! Routing table
netstat -r ! Routing table
netstat -a ! zeigt lokal offene Ports an
netstat -b ! zeigt entsprechende Anwendungen zu jeweiligen Ports an
netstat -p <protocol> ! filtert nach gewähltem L3-Protokoll (tcp/udp)
nslookup
```


aus user exec mode >
--------------------

```
enable (ena)
ping <ip> ! ZB ping 127.0.0.1 oder ping FE80::1
traceroute <ip> !TODO auf IPv4 und oder IPv6
end
```


aus priv exec mode #
--------------------

```
disable
exit
end
reload
configure terminal (conf t)
show history
show version
show flash
show interfaces (sh int)
show running-config
show startup-config

show ip route
show ip route <route> ! ZB show ip route 10.0.0.0 ! zeigt erweiterte Informationen zur jeweiligen Route an
show ip int brief
show ip protocols
show ipv6 route
show ipv6 route static ! zeigt nur **statische** ipv6 routen an

show arp
show ip arp ! Mappings welche IP welche MAC hat
show mac address-table
show vlan
show vlan brief
sh int <int> switchport
sh vlan summary

show history
show processes
show cdp neighbors
show protocols
show file systems
show hosts
show controllers
dir
debug
clock set 19:25:00
```


aus global configuration mode (config)
--------------------------------------

```
do <command> ! führt Befehle eine Ebene unterhalb (als priv exec mode) aus ZB do copy run start
hostname <hostname> !ZB hostname R1
enable secret <password> !ZB enable secret class
service password-encryption
security password min-length <length> ! ZB security password min-length 5
banner motd $ <motd> $ !ZB banner motd $  !!! NO HACKKKERS ALLOWED !!! $
copy running-config startup-config
copy running-config tftp
erase startup-config
copy tftp running-config
line vty 0 15
line console 0
ip route
```


aus line configuration mode (config-line)
-----------------------------------------

```
password cisco
login
exit
```


aus interface configuration mode (config-line)
----------------------------------------------

```
duplex {half | full}
speed 100
exit
```

Szenarien
---------

### Hostname, MotD, lokales CLI-Login Passwort setzen, config speichern

```
ena
erase startup-config
delete flash:vlan.dat
clear vtp counters
reload
ena
conf t
hostname <hostname> !ZB hostname R1
banner motd $ <motd> $ !ZB banner motd $  !!! NO HACKKKERS ALLOWED !!! $ ! $-Zeichen können auch durch andere, zb # ersetzt werden
enable password <password> ! ZB enable password cisco 
enable secret <password> ! ZB enable secret class
line con 0 ! con kurz für console
password cisco
login
line vty 0 15 ! 16 parallele vty connections möglich
password cisco
login
exit
service password-encryption
exit
copy running-config startup-config
```


### Ethernet-Interface mit IPv4, IPv6 konfigurieren

```
ena
conf t
int g0/0.20
clockrate <rate> ! ZB clockrate 64000 ! nur bei serieller Verbindung und dort auf DCE Seite (mit/ohne Uhr????) !TODO clock rate oder "clock rate"
ip add <ip-address> <subnetmask> ! ZB ip add 192.168.0.1 255.255.255.0
ipv6 add <ipv6-address>/<prefix-length> ! ZB ipv6 add 2001::1/48 ! statische IP Zuweisung
ipv6 address <ipv6-prefix>/<prefix-length> eui-64 ! ZB ipv6 address 2001::/64 eui-64
ipv6 address fe80::1 link-local ! link-local (nur im eigenen Netz verwendbare) Adresse zuweisen
ipv6 address dhcp ! bezug über DHCP von **anderem** Router
description <desc> ! ZB description Verbindung zum ISP
no shut
exit
show interfaces
show interface g0/0.20
show ipv6 interface g0/0.20
```


### SSH-Zugang vergeben

```
ena
conf t
ip domain-name <domain-name> ! ZB ip domain-name span.com
ip ssh authentication retries <anzahl> ! ZB ip ssh authentication retries 5
ip access-class in ? ! weiß nit wie es weitergeht, is da um Zugang nur aus bestimmten Netzen zu erlauben
ip ssh exec-out <dauer> ! ZB ip ssh exec-out 15 ! Timeout für SSH-connections
crypto key generate rsa general-keys modulus <length> ! ZB crypto key generate rsa general-keys modulus 1024
username <user> secret <password>
line vty 0 4
login local
transport input ssh
exit
```


### Lokalen HTTPS-Server am Router/Switch

```
ena
conf t
ip http server
ip http secure-server
ip http authentication local
username <username> privilege <level> secret <password> ! username meiuser privilege 15 secret class
line vty 0 4
privilege level 15
login local
transport input telnet ssh
exit
```
### Routing

#### statisches Routing

```
! default static route
ip route 0.0.0.0 0.0.0.0 {outgoing interface | next-hop address} ! ZB ip route 0.0.0.0 0.0.0.0 s0/0/0
ipv6 route ::/0 {outgoing interface | next-hop address}

! normal static route
ip route <network-address> <subnet-mask> {address of next hop | outgoing interface} [administrative distance][permanent] ! ZB ip route 172.16.3.0 255.255.255.0 S0/0/0
ipv6 route <network-address>/<prefix-length> {ipv6 address | outgoing interface} ! ZB ipv6 route 2001:DB8:ACAB:1::/64 2001:DB8:ACAB:4::2 ! bei Verwendung von link-local Adressen zwingend fully specified static routes, da link-local Adressen nicht in routing table eingetragen werden! (siehe unterhalb)

! fully specified static route
ip route <network-address> <subnet-mask> <outgoing interface> <address of next hop>
ipv6 route <network-address>/<prefix length> <outgoing interface> <address of next hop>

! summary static route wie normale static route

! floating static route
ip route <network-address> <subnet-mask> {address of next hop | outgoing interface} <administrative distance> ! ZB ip route 172.16.3.0 255.255.255.0 S0/0/0 120 ! 120 ist die AD, lower values wins -> Wertebereich [0;255]
```

#### dynamisches Routing

```
ena
conf t
router ? ! gibt mögliche routing protokolle auf CLI aus
```

##### RIP

```
ena
conf t
router rip

! v1:
network <network> ! ZB network 192.168.1.0 oder network 192.168.2.0

! v2:
version 2
no auto-summary

! allgemein wieder:
passive-interface <interface> ! ZB passive-interface g0/0
default-information originate ! für weitergabe der default-route
end
sh ip protocols
debug ip rip
```

##### RIPng (IPv6)

```
ena
conf t
ipv6 unicast-routing

! auf alle zu routenden interfaces
int <interface>
ipv6 rip <bezeichnung> enable ! ZB ipv6 rip RIP-AS enable
exit

sh ipv6 protocols
sh ipv6 route
debug ip rip
```

##### OSPF

```
ena
conf t
router ospf <process-nr> ! OSPF Konfiguration starten (Prozessnummer nicht wichtig)
log-adjacency-changes ! Nachbarschaftsänderungen mitloggen
router-id <ip-address> ! (Optional) Router-ID manuell einstellen
network <net-ip-address> <wildcard-mask> area <area-nr> ! Netzwerk lernen (Area ist bei Single-Area OSPF die BackboneArea (= 0))
network <outgoing-interface-ip-address> 0.0.0.0 area <area-nr> ! Netzwerk lernen mit Angabe von outgoing Interface

passive-interface default ! Alle Interfaces als Passiv einstellen
no passive-interface <interface> ! Nur bestimmte Interfaces nicht-passiv einstellen

default-information originate ! Static-Default-Route in OSPF übertragen
redistribute connected ! Verbundene Routen in OSPF weiterleiten
redistribute static ! Statische Routen in OSPF weiterleiten
exit

interface <interface>
ipv6 ospf <process-nr> area <area-nr> ! (Für IPv6) OSPF an diesem Interface aktivieren
ip ospf priority <priority 0-255> ! (Optional, eher unwichtig) Priorität einstellen für Wahl zum DR/BDR (stärker als Router-ID)
bandwidth <bandwidth in kbps> ! (Optional) Bandbreite für Cost-Errechnung (Metrik) einstellen
ip ospf cost <costvalue-kbps> ! (Optional) Cost-Value direkt einstellen
exit

show ipv6 protocols
show ipv6 ospf interface brief
show ipv6 ospf neighbor
show ipv6 route
show ip ospf neighbor
show ip ospf interface <interface>
```


### Switching, VLANs & Trunking

#### Switch-Management-SVI vergeben

```
ena
conf t
vlan <vlanid>
name <name>
int vlan <vlanid> ! ZB int vlan 99
ip add <ip-address> <subnetmask> ! ZB ip add 192.168.0.253 255.255.255.0
no shut
int <interface>
switchport mode access
switchport access vlan <vlanid>
no shut
exit
ip default-gateway <ip-address>
exit
sh vlan brief
sh ip int brief
```


#### Switch VLAN konfigurieren

```
ena
conf t
vlan <vlanid>
name <name>
apply
int <interface>
switchport mode access
switchport access vlan <vlanid>
sh vlan brief
sh ip int brief
```


#### Trunk am Switch

```
ena
conf t
vlan <vlanid>
name <name>
apply
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk native vlan 100
switchport trunk allowed vlan add 10,20,30,99,100
sh vlan brief
sh int <interface> switchport
```


#### Inter-VLAN Routing

##### Router-on-a-stick mit anliegender Trunkleitung

```
ena
conf t
int g0/0.10
encapsulation dot1q 10
ip add 10.0.10.254 255.255.255.0
int g0/0.30
encapsulation dot1q 30
ip add 10.0.30.254 255.255.255.0
int g0/0 ! nur physischen Port hochfahren, subinterfaces müssen nicht selbst hochgefahren werden
switchport mode trunk
no shut
end
```


##### SVI-L3-Switch-based

```
ena
conf t
int g0/0
ip routing
exit
int vlan 10
no shut
ip add 10.0.10.253 255.255.255.0
int vlan 30
no shut
ip add 10.0.30.253 255.255.255.0
exit
```


#### Switch Security

##### DHCP Spoofing

```
ena
conf t
ip dhcp snooping ! DHCP Snooping am Switch aktivieren
ip dhcp snooping vlan <vlan-id> ! DHCP Snooping per VLAN aktivieren

interface <interface>
ip dhcp snooping trust ! Interface als trusted einstellen (von dort werden DHCP Offers erlaubt)
```


##### DHCP Starvation

```
ena
conf t
ip dhcp snooping limit rate <rate> ! (am DHCP Server) Nur bestimmte Anzahl an DHCP Anfragen in einer gewissen Zeit erlauben
```


##### CDP

```
ena
conf t
no cdp run ! CDP am ganzen Switch deaktivieren
int <interface>
  no cdp enable ! CDP an einem Interface deaktivieren
```


##### DTP

```
ena
conf t
int <interface>
  switchport nonegotiate ! Am Trunk port DTP ausschalten (Trunk wird nicht mehr automatisch aktiviert)
```


##### Port Security

```
ena
conf t
int <interface>
  switchport mode access
  switchport port-security ! Port-Security auf diesem Interface erlauben
  switchport port-security maximum <number> ! Maximale Anzahl an erlaubten MAC Adressen einstellen
  switchport port-security violation <Shutdown|Restrict|Protect> ! Was passieren soll wenn Regeln verstoßen wurden
  switchport port-security mac-address <address> ! MAC Adresse statisch festlegen/erlauben
  switchport port-security mac-address sticky ! Automatisch gelernte MAC-Adressen werden in Running-Config gespeichert
  switchport port-security mac-address sticky <mac-address> ! Angegebene MAC-Adresse lernen und in Running-Config speichern
  exit
copy running-config startup-config ! nachdem MAC Adressen sticky gelernt wurden in NVRAM speichern: nach restart noch verfügbar
```

---

### ACLs

omitted weil i des spritz, ha. ha. ha.

---

### DHCP

### NAT/PAT

### STP
```
ena 
conf t
spanning-tree costs <value> ! Port Kosten festlegen (um schnellsten Link zu finden)
spanning-tree port-priority <value> ! (0-255, default 128) Port mit der niedrigsten Nummer wird zum Designated Port
	
spanning-tree portfast ! Zusatzfeature PortFast aktivieren
spanning-tree uplinkfast ! Zusatzfeature UplinkFast aktivieren
spanning-tree backbonefast ! Zusatzfeature BackboneFast aktivieren
spanning-tree bpduguard enable ! Zusatzfeature BPDUGuard aktivieren
spanning-tree bpdufilter enable ! Zusatzfeature BPDUFilter aktivieren
spanning-tree guard root ! Zusatzfeature RootGuard aktivieren
exit

spanning-tree vlan <vlan-id> priority <value> ! Priority für Root-Bridge-Ermittlung pro VLAN
spanning-tree vlan <vlan-id> root primary ! Switch wird zur Root-Bridge in diesem VLAN
	
show spanning-tree
show spanning-tree detail
show spanning-tree vlan
show spanning-tree summary
```

Eher für praktische Anwendung
-----------------------------

### running-config auf USB sichern &  wiederherstellen

```
ena
dir usbflash0:/
copy system:running-config usbflash0:
copy usbflash0:/running-config
```


### IOS mittels TFTP wiederherstellen

1. IOS file auf TFTP-Server laden
2. Ethernetverbindung zwischen Router mit TFTP-Server herstellen; TFTP-Server IP 192.168.1.1 zuweisen
3. Consoleverbindung zu Router herstellen und Terminal-Prog. starten
4. Router einschalten, bleibt im ROMmon-Mode stehen
5. Routerkonfiguration vornehmen:
```
rommon1> IP_ADDRESS=192.168.1.2 [ENTER]
rommon2> IP_SUBNET_MASK=255.255.255.0 [ENTER]
rommon3> DEFAULT_GATEWAY=192.168.1.1 [ENTER]
rommon4> TFTP_SERVER=192.168.1.1 [ENTER]
rommon5> TFTP_FILE=<filename> ! ZB rommon1> TFTP_FILE=c1841-ipbase-mz.123-14.T7.bin
```
6. Im ROMmon-Mode `rommon6> tftpdnld` eingeben, die beiden Fragen mit yes beantworten und die Übertragung abwarten.
7. Mittels `rommon7> reset` den Router mit dem neuen Image starten


### IOS mittels serieller Übertragung/xmodem wiederherstellen

1. IOS file auf Laptop laden
2. Consoleverbindung zu Router herstellen und Terminal-Prog. starten
3. Router einschalten, bleibt im ROMmon-Mode stehen
4. Mit `rommon1> xmodem -s 115200 –c <filename>` (Beispiel für filename s. o.) das gewünschte File zum Übertragen auswählen und mit yes die Frage zum Fortsetzen beantworten (115200 ist die Baudrate)
5. Im Terminal die Datei mit richtigem Namen, passendem Protokoll (xmodem) und passender Baudrate senden
6. Nach der Übertragung die Geschwindigkeit im Terminal wieder auf 9600 bps zurücksetzen.
7. Nach Neustart des Terminalprogramms das Konfigurationsregister auf 0x2102 (!TODO ?) zurücksetzen und mit `rommonX> reset` den Router neu starten.
