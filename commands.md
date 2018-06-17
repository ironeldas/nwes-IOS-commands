Cisco-IOS Befehle & Szenarien für die NWES-Matura
=================================================

Allgemeines
-----------

- `?` um context sensitive help aufzurufen
- Auto-Vervollständigung mit `TAB`
- Befehle können mit Keyword `no` am Anfang zurückgenommen/negiert werden
- Kommentare können in einer Zeile nach einem `!` (Rufezeichen) geschrieben werden

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
show processes
show cdp neighbors
show arp
show ip arp ! Mappings welche IP welche MAC hat
sh ip route ! Routing table
sh ip int brief
show ipv6 route ! Routing table
show mac-address-table
show vlan
show running-config
show startup-config
show protocols
show file systems
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


Szenarien
---------

### Hostname, MotD, lokales CLI-Login Passwort setzen, config speichern

```
ena
erase startup-config
reload
ena
conf t
hostname <hostname> !ZB hostname R1
banner motd $ <motd> $ !ZB banner motd $  !!! NO HACKKKERS ALLOWED !!! $ ! $-Zeichen können auch durch andere, zb # ersetzt werden
enable secret <password> ! ZB enable secret class
service password-encryption
exit
copy running-config startup-config
```


### Ethernet-Interface mit IPv4, IPv6 konfigurieren

```
ena
conf t
int g0/0.20
clock rate <rate> ! ZB clock rate 64000 ! nur bei serieller Verbindung und dort auf DCE Seite (mit/ohne Uhr????) !TODO
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

### Switch-SVI vergeben

```
int vlan 1
ip add <ip-address> <subnetmask> ! ZB ip add 192.168.0.253 255.255.255.0
no shut
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
