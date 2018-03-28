

### FORK BOMB

`:() {:|:& };:`

### [Locate & Find](https://wiki.gentoo.org/wiki/Mlocate)

use `locate [filename]` to find stuff and create crownjob to use is daily
or from time to tome `updatedb`


### How to connect to vm

Du hast nen alias angelegt, `sdia` oder `sdib`

# Dns Server


### named.conf.options

```
options {
  directory "/var/cache/bind";

  # Disable recursive DNS queries
  recursion yes;

  # Listen on the private network only (local IP)
  listen-on { 141.62.75.116; };

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

### named.conf.local

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
### named.conf.local

```
$TTL    604800
@       IN      SOA     ns6.mi.hdm-stuttgart.de. root.mi.hdm-stuttgart.de. (
                     2018032800         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;

; name servers - NS records
        IN      NS      ns6.mi.hdm-stuttgart.de.


; name servers - A records
ns6.mi.hdm-stuttgart.de.          IN      A       141.62.75.116
;www6_1.mi.hdm-stuttgart.de.       IN      CNAME   ns5.mi.hdm-stuttgart.de.
;www6_2.mi.hdm-stuttgart.de.       IN      CNAME   ns5.mi.hdm-stuttgart.de.
```
