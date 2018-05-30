# Software Defined Infrastructre

### DNS

##### Installing bind
Zunächst muss bind9 auf den Servern installiert werden:
`apt-get install bind9 bind9utils`
#### Basic Configuration
Anschließend wird eingestellt, dass der DNS-Server nur für IPv4 arbeitet. Das geschieht über den Eintrag `OPTIONS="-4 -u bind"` in der Datei __/etc/default/bind9__

Dann muss die globale Konfigurationsdatei den Bedürfnissen entsprechend angepasst werden.
Unsere fertig konfigurierte Datei sieht für den Server mit der IP 141.62.75.106 wie folgt aus:
```
options {
  directory "/var/cache/bind";

  # Disable recursive DNS queries
  recursion yes;

  allow-recursion { any; };
  allow-query { any; };
  allow-query-cache { any; };

  # Listen on the private network only (local IP)
  listen-on { 141.62.75.106; };

  # Disable zone transfers, because we don't have a
  # redundant infrastructure (primary / secondary dns)
  allow-transfer { none; };

  # Currently, we don't want to forward requests to stable nameservers
  forwarders {
	141.62.64.128;
  };

  # Use the default settings for validation and ipv6
  dnssec-validation auto;
  auth-nxdomain no;
  listen-on-v6 { any; };
};
```
#### Zones, Forwarders & Mail exchange record
Jetzt müssen sogenannte zones angelegt werden. Hierfür legen wir im Ordner __/etc/bind/zones/__ folgende Dateien an:
__db.mi.hdm-stuttgart____.de__ 
```
$TTL    604800
@       IN      SOA     ns6.mi.hdm-stuttgart.de. root.mi.hdm-stuttgart.de. (
                     2018040401         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;

; name servers - NS records
        IN      NS      ns6.mi.hdm-stuttgart.de.

; name servers - A records
ns6.mi.hdm-stuttgart.de.        IN      A       141.62.75.106

; mail servers - MX records
mi.hdm-stuttgart.de.	IN	MX	10 	mx1.hdm-stuttgart.de.
```
__db.141.62.75__
```
; BIND reverse data file 
;
$TTL    604800
@       IN      SOA     ns4.mi.hdm-stuttgart.de. root.mi.hdm-stuttgart.de. (
                     2018040402         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;

; name servers - NS records
      IN      NS      ns6.mi.hdm-stuttgart.de.

; PTR Records
106   IN      PTR     sdi6a.mi.hdm-stuttgart.de.    ; 141.62.75.106
```

In der Datei __/etc/bind/named.conf.local__ legen wir die Verwendung der erstellten Zonen fest:
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "mi.hdm-stuttgart.de" {
    type master;
    file "/etc/bind/zones/db.mi.hdm-stuttgart.de"; # zone file path
};

zone "75.62.141.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.141.62.75";  # 141.62.75.0/24 class-C subnet
};
```

Um für unseren Server sich selbst als Default-DNS-Server einzustellen, muss noch die Datei __/etc/resolv.conf__ angepasst werden. Hier fügt man die Zeile `nameserver 141.62.75.106` hinzu. Fortan verwenden Tools wie _nslookup_, _dig_ oder _host_ automatisch den selbst bereitgestellten DNS-Service.


## WICHTIG/noch einarbeiten:
- generelle Funktion: `service bind9 reload` / `systemctl status bind9.service` etc.
- `dig`, `nslookup` und `host`
- Wie kommt die Serial zu Stande?
- Was ist die Zahl beim MX Eintrag (Priorität)
- Unterschied von A, AAA, C, MX Eintrag, ...
- Erklären was zu tun ist, damit DNS von Außen erreichbar ist (allow-query {any} ...)
- Forwarding (recursion)
