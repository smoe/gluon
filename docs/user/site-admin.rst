Setup of a new Freifunk network
===============================

This file summarises to varying degree of detail the creation
of a new Freifunk network. More explicitly, it is the history
of ostholstein.freifunk.net as a spin-off of the neighbouring
luebeck.freifunk.net.

Infrastructure setup
--------------------

It all starts with a series of machines. There is
 * a web server
 * at least one, better two gateways
It is advisable to have more than one physical machine for
each gateway. If these are virtual hosts or full emulations
of hosts (i.e. virtual machines that allow running their
own flavour of a linux kernel, like by KVM) does not matter.

There is a (German) description of demands expressed on
http://wiki.freifunk.net/Freifunk_Hamburg/Gateway/Anforderungen
that may be passed as a question to your provider. 
Critical: Start with IPv6 from day one and do not accept
any provider not supporting it.

Server infrastructure
---------------------

Provider:  filoo.de
 * Gateway 1 : Dynamic node 141.101.36.19 1GB Mem
  - IPv4 intern 10.135.0.8
  - IPv6 extern 2a00:12c0:1015:166::1:1
  - IPv6 intern fd73:111:e824::8
  - MAC Adresse intern d6:f3:ed:8a:00:01
  * add users:
    useradd --system fastd
  * add packages
     - fastd Public: d63097eb426e8f5016957cd5ee184c60a535d001ec498115a828885568ba9e9c
     - batman-adv-dkms
     - iptables-persistent
     - tinc
  * install and configure mullvad
      - openvpn resolvconf
      - unzip
      - https://mullvad.net/en/setup/openvpn/ NICHT FOLGEN
      - http://wiki.freifunk.net/Freifunk_Hamburg/Gateway
* Gateway 2: Dynamic node 141.101.36.67 1GB Mem
  - IPv4 intern 10.135.0.16
  - IPv6 extern 2a00:12c0:1015:166::1:2
* Website: Dynamic node  512MB RAM
  - IPv4 extern 109.75.177.24
  - IPv6 extern 2a00:12c0:1015:166::1
  - add packages
     fastd
     batman-adv-dkms
     apache2
     jekyll
     git
     python3
     exim4 konfiguriert fuer ostholstein.freifunk.net


Einsortieren des eigenen Netzes in der Community
------------------------------------------------

* http://wiki.freifunk.net/IP-Netze
  + IPv6 Subnetz "registrieren", z.B. ein /16
  Für Ostholstein: 10.135.0.0/16
  + IPv6 Subnetz "registrieren", z.B. ein /16
  Für Ostholstein: fd73:111:e824::/48
  
* http://wiki.freifunk.net/AS-Nummern
  ASN "registrieren"
  Für Ostholstein: 65152

* api.freifunk.net einrichten

Konfiguration der Gateways
--------------------------

Filoo.de hat das IPv6 Subnetz 2a00:12c0:1015:166::0/64 zugewiesen.

site.conf auf
https://github.com/smoe/site-ffoh
- Exit VPN

### Konzept zur Vergabe von IP Nummern ###

# IPs für Gateways
Vorschlag: IPv4 erstes /24 für statische Adressen (hauptsächlich Gateways) freihalten.
z.B. erster Gateway: 10.135.0.8
       zweiter Gateway: 10.135.0.16
       passend zu Subnetzen
## IPv6
"irgendwie" aus dem /64 verteilen. Prefix wird bei Gluon von den Knoten selbst announct.
Sollten aber RDNSS per radvd.
## Für Subnetze zum Verteilen
10.135.56.0/21
10.135.48.0/21
10.135.40.0/21
usw.. bis 
10.135.8.0/21
# Gatewayconfig
Fand http://luebeck.freifunk.net/wiki/gatewayconfig
debfoster -u bird bird6 isc-dhcp-server radvd lighttpd haveged openvpn
statt "named" installiert: bind9 dnsutils
fehlt in Beschreibung: bridge-utils (brctl)
/etc/modules:  batman-adv hinzugefuegt
/etc/hosts:
10.135.0.8      gw1.ostholstein.freifunk.net gw1
10.135.0.16     gw2.ostholstein.freifunk.net gw1
10.135.0.24     gw3.ostholstein.freifunk.net gw1
10.135.0.32     gw4.ostholstein.freifunk.net gw1
10.135.0.40     gw5.ostholstein.freifunk.net gw1
10.135.0.48     gw6.ostholstein.freifunk.net gw1
10.135.0.56     gw7.ostholstein.freifunk.net gw1



batctl gw server - wird von mullvad mit gestartet
# ICVPN eintragen, sobald ein Gateway fertig ist
  läuft über tinc
  Keys-Repo: https://github.com/freifunk/icvpn
  Konfiguration nach http://wiki.freifunk.net/IC-VPN
# IPv6 Addressspace aufteilen
Empfehlung: das erste /64 aus dem /48 für das Mesh verwenden. Weitere /64 evtl. für private Heimnetze verwenden.
# IPv4 Adressspace aufteilen
Empfehlung: Erstmal nur das erste /17 nehmen und das zweite für einen Umbau frei lassen.
Ansonsten IPv4 nur für Gateways und Clients (DHCP) verwenden. Alles andere v6.
Vorschlag: Aus dem ersten /17 das erste /18 für das Mesh verwenden.
# Beispiel:
IPv6 ULA: fdef:1234:5678::/48, für das Mesh dann: fdef:1234:5678:0000::/64
Dann wäre z.B.  fdef:1234:5678:0001::/64 frei
IPv4: 10.134.0.0/16, für das Mesh: 10.134.0.0/18, reserviert für später: 10.134.128.0/17
10.134.64.0/18 wäre frei für private Subnetze

### Besonderheiten bei Gluon/Lübecker Setup ###

#### Next-Node Adresse ####
in Lübeck: x.y.0.1 bzw. xxxx::1
Diese Adresse "freihalten". Vorschlag: IPv4 erstes /29 reservieren, also 0..7


### Installation der Freifunk-Mesh software auf Gateway ###

#### batman-adv ###
batman-adv legacy (von Gluon verwendet)
$ cat <<EOCAT > /etc/apt/sources.list.d/99matthias.list
deb http://repo.universe-factory.net/debian sid main
EOCAT 
gpg --keyserver pgpkeys.mit.edu --recv-key 16EF3F64CB201D9C
gpg --fingerprint 16EF3F64CB201D9C
#pub   4096R/CB201D9C 2014-01-08 [verfällt: 2016-01-08]
#  Schl.-Fingerabdruck = 6664 E7BD A6B6 6988 1EC5  2E75 16EF 3F64 CB20 1D9C
gpg --export -a 16EF3F64CB201D9C|apt-key add -
## radvd konfigurieren
Hauptsächlich RDNSS
$ cat /etc/radvd.conf
interface bat0
{
    AdvSendAdvert on;
    IgnoreIfMissing on;
    MaxRtrAdvInterval 200;
    prefix fd73:111:e824::/64
    {
    };
    RDNSS fd73:111:e824::1:1
    {
    };
};
## dhcpd
* Installation von isc-dhcp-server Paket
* Cave: Subnetz als /18 verteilen, nicht nur das /21.
Fuer Gateway 135.0.8
subnet 10.135.0.0 netmask 255.255.192.0 {
    range 10.135.8.0 10.135.15.255;
    option routers 10.135.0.8;
    option domain-name-servers 10.135.0.8;
}
Fuer Gateway 135.0.16
Subnetz als /18 verteilen, nicht nur das /21.
subnet 10.135.0.0 netmask 255.255.192.0 {
    range 10.135.16.0 10.135.31.255;
    option routers 10.135.0.8;
    option domain-name-servers 10.135.0.8;
}
## DNS
2..3 DNS Server wären toll. Auch per DHCP verteilen.
bind empfohlen. Siehe: http://wiki.freifunk.net/DNS
# fastd VPN
Tunnelinterface mit batctl if add $IF hinzufügen.
Beispielconfig (/etc/fastd/XXX/fastd.conf): z.B. XXX = ffoh-mesh-vpn
log to syslog level verbose;
user "fastd";
interface "ffoh-mesh-vpn";
method "salsa2012+gmac"; # WICHTIG!
method "xsalsa20-poly1305"; # evtl. nicht nötig
bind 0.0.0.0:10000;
include "secret.conf";
mtu 1426;
hide ip addresses yes;
include peers from "peers";
on up "
        ip link set up $INTERFACE
        batctl if add $INTERFACE
";


Dazu noch secret.conf anlegen, siehe: http://www.nilsschneider.net/2013/02/17/fastd-tutorial.html
ggf. ein paar Secrets im Vorraus generieren für geplante Gateways und die Public Keys in der Firmware hinterlegen.
Peers kommen dann in das Unterverzeichnis peers/. Bei Gateways noch eine remote Zeile eintragen! peers/ als GIT Repo ist praktisch. 

Anonymisierungs-Server: IPv4 exit
-----------------------------------

2. Routingtabelle anlegen (Policy Routing)
Dort defaultroute über das Exit-VPN eintragen.
Beispiel OpenVPN up script:
ip route replace 0.0.0.0/1 via $5 table freifunk
ip route replace 128.0.0.0/1 via $5 table freifunk
Das $5 wird hierbei automatisch ersetzt durch die IP Nummer des anonyisierers. Dies laesst sich auch bestimmen ueber "ifconfig mullvad".
Traffic aus dem Freifunk, z.B. vom Interface bat0 in Tabelle 42 (freifunk, siehe /etc/iproute2/rt_tables) umbiegen:
ip rule add iif bat0 table freifunk
# IPv6
Wieder: Eigene Routingtabelle anlegen, analog zu v4. Allerdings reicht als "defaultroute" 2000::/3 aus.
z.B über Sixxs Tunnel, ganzes /48 per NAT mappen. Stichwort: NPTV6
Beispiel mit neoraider's NPTV6 Modulen:
-A PREROUTING -d 2001:4dd0:ff00:9466::/64 -j MARK --set-xmark 0x2a/0xffffffff
-A PREROUTING -d 2001:4dd0:ff00:9466::/64 -j DNPTV6 --to-destination fdef:ffc0:3dd7::/64 
-A INPUT -s fdef:ffc0:3dd7::/64 -m mark --mark 0x2a -j SNPTV6 --to-source 2001:4dd0:ff00:9466::/64
-A OUTPUT -d 2001:4dd0:ff00:9466::/64 -j MARK --set-xmark 0x2a/0xffffffff
-A OUTPUT -d 2001:4dd0:ff00:9466::/64 -j DNPTV6 --to-destination fdef:ffc0:3dd7::/64 
-A POSTROUTING -d fc00::/7 -j RETURN
-A POSTROUTING -s fdef:ffc0:3dd7::/64 -m mark --mark 0x2a -j SNPTV6 --to-source 2001:4dd0:ff00:9466::/64
-A POSTROUTING -s fdef:ffc0:3dd7::/64 -o sixxs -j SNPTV6 --to-source 2001:4dd0:ff00:9466::/64
# next step: BGP / ICVPN

Gateway gw1 /etc/network/interfaces
    auto dummy
    iface dummy inet manual
        pre-up ip link add $IFACE address d6:f3:ed:8a:00:01 type dummy
        up ip link set up $IFACE
        up batctl if add $IFACE
        post-down ip link del $IFACE
    auto bat0
    iface bat0 inet static
        address 10.135.0.8/18
    iface bat0 inet6 static
        address fd73:111:e824::8/64

Initiation of IP forwarding

Some  may recall that "echo 1 > /proc/net/..." which had the same effect but was lost after reboot

    vim /etc/sysctl.conf 
    sysctl -p
    # net.ipv4.ip_forward = 1
    # net.ipv6.conf.all.forwarding = 1

Internet v6 packages are forwarded without constraints and without hiding
anything. This renders the router directly accessible from the outside - with IPv6.

Accession from the outside via the common IPv4 protocol is however not possible.
Outbound traffic is masqueraded by the IP number of the gateway. Use this line
    iptables -t nat -A POSTROUTING -s 10.135.0.0/18 -o eth0 -j MASQUERADE
to have a direct connection of the Freifunk network to the outside world, albeit
masqueraded. Use
    iptables -t nat -A POSTROUTING -s 10.135.0.0/18 -o mullvad -j MASQUERADE
to have all outbound traffic anonymised through your favorite external service.


DNS Config in named.local.conf

    zone "ffhl" IN {
        type master;
        file "ffhl/ffhl.zone";
        allow-transfer { any; };
    };
    zone "130.10.in-addr.arpa" IN {
        type master;
        file "ffhl/10.130.zone";
        allow-transfer { any; };
    };
    zone "7.d.d.3.0.c.f.f.f.e.d.f.ip6.arpa" IN {
        type master;
        file "ffhl/fdef:ffc0:3dd7.zone";
        allow-transfer { any; };
    };

/etc/radvd.conf

    interface bat0
    {
        AdvSendAdvert on;
        IgnoreIfMissing on;
        MaxRtrAdvInterval 200;
        prefix fd73:111:e824::/64
        {
        };
        RDNSS fd73:111:e824::1:1
        {
        };
    };

Intercity-VPN
-------------

Installation von bird .... Magie von Nils in /etc/bird
man will nicht neu starten, aber configure ist OK, sonst Verlust von Verbindung

    birdc6 configure
    birdc6 show protocols
    vim /etc/bird6.conf
    vim /etc/iproute2/rt_tables

Maps
----

Frankly speaking - the system to get all the data from nodes and have
this displayed on the www server is a mess. Get someone who has done
it before.

Alfred Installation from http://www.open-mesh.org
    cat >> /etc/rc.local
    /usr/sbin/alfred -i bat0 -m > /dev/null 2>&1 &
    /usr/sbin/batadv-vis -s > /dev/null 2>&1 &

Extra packages to install are rrdtool, python3, libjansson

https://github.com/tcatm/alfred-json

Optional for special community spirit
-------------------------------------
https://github.com/MetaMeute/ffhl-dns
# Configure mailing lists
* MX-Record
* PTR-Record
* Mailman + z.B.Postfix


