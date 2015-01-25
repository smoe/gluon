Setup of a new Freifunk network
===============================

This file summarises to varying degree of detail the creation
of a new Freifunk network. More explicitly, it is the history
of ostholstein.freifunk.net as a spin-off of the neighbouring
luebeck.freifunk.net.

Infrastructure setup
--------------------

*Hardware*

It all starts with a series of machines. There is
*   a web server
*   at least one, better two gateways
It is advisable to have more than one physical machine for
each gateway. It does not matter if these machines are true
physical instances or virtual hosts, as long as these may have
their own kernel with their own respective set of kernel modules
to support the networking, as e.g. facilitated by KVM.

There is a (German) description of demands expressed on
http://wiki.freifunk.net/Freifunk_Hamburg/Gateway/Anforderungen
that may be passed as a question to your provider. 
Critical: Start with IPv6 from day one and do not accept
any provider not supporting it.

*Distribution of IP network address space*

A typical Freifunk network will have a 10.XXX.0.0/16 network. The XXX should
not be shared with another freifunk network, such that the InterCity VPN can
be established. The page http://wiki.freifunk.net/IP-Netze
orchestrates the community, both for IPv4 (Ostholstein: 10.135.0.0/16) and
IPv6 (Ostholstein: fd73:111:e824::/48).

To describe later:

*   http://wiki.freifunk.net/AS-Nummern (ASN for Ostholstein: 65152)
*   api.freifunk.net einrichten

IPv4

Every gateway runs a DHCP-daemon for the assignment of IPv4 IP numbers to mobile
devices contacting the routers. The routers themselves all have static IPv6 numbers
and do not need any configuration beyond what is statically set in the firmware.
The firmware is compiled individually for every Freifunk community.

For the 32bittish IPv4, having the first 16 bit set leaves a seconda half of
16 bits to be assigned. We think of it as a series of 256 8bittish /24 networks.
Your milage may vary, but we decided to prepare every gateway to allow the management
of 8 of these /24 networks, i.e. just above 2000 clients. This is not too many, since
IP numbers are not immediately freed after a client disappeared, and when resurfacing,
a memorised IP number reduces latencies.

Thus - preparing for 8 gateways, the networks assigned to each DHCP on each gateway are
10.135.56.0/21, 10.135.48.0/21, 10.135.40.0/21, etc. down to 10.135.8.0/21. This leaves
10.135.0.0/21 unassigned, meant to be reserved for static addresses, such like the gateways
themselves, which reside on 10.135.0.8, 10.135.0.16, etc., with their last 8 bit matching
the subnet these distribute IP numbers to. This comes exceptionally useful whenever a
user with a particular IP numbers experiences issues.

It can also be observed that all the higher address space, i.e. 10.135.128.0/17 was not
yet touched. This is meant as a backup in case that the whole network is about to transition
to a new scheme, thus granting a reserve for the setup of a new network topology. Also, 
we had not spawned IP addressed beyond the first /18 of that first /17, yet. This grants
some extra freedom for e.g. the management of web cameras or temperature sensors - whatever
your neighbourhood may bring. With the advent
of IPv6 and an expected transition also of many clients towards that, too much restructuring
is not expected.

IPv6 - later
 
Prefix wird bei Gluon von den Knoten selbst announct.
Sollten aber RDNSS per radvd.


Server infrastructure
---------------------

Provider:  filoo.de provided subnet 2a00:12c0:1015:166::0/64 .

Add users:

    useradd --system fastd

Add packages

    apt-get install fastd batman-adv-dkms iptables-persistent tinc git resolvconf radvd lighttpd haveged openvpn

Gateway 1 : Dynamic node 141.101.36.19 1GB Mem

*   IPv4 intern 10.135.0.8
*   IPv6 extern 2a00:12c0:1015:166::1:1
*   IPv6 intern fd73:111:e824::8
*   fastd public hash: d63097eb426e8f5016957cd5ee184c60a535d001ec498115a828885568ba9e9c
*   MAC address intern d6:f3:ed:8a:00:01

Gateway 2: Dynamic node 141.101.36.67 1GB Mem

*   IPv4 intern 10.135.0.16
*   IPv6 extern 2a00:12c0:1015:166::1:2

Website: Dynamic node  512MB RAM

*   IPv4 extern 109.75.177.24
*   IPv6 extern 2a00:12c0:1015:166::1
*   add packages
       apt-get install fastd batman-adv-dkms git resolvconf
    and more packages because of web features
       apt-get install apache2 jekyll python3 exim4 (configured for ostholstein.freifunk.net)


Gateway configuration
---------------------

*Firmware*

All configuration of the gateways must be matched by the configuration of the firmware - otherwise
the firmware would not attempt to reach those gateways. To help with consistency across multiple
individuals contributing to the firmware, and to help with their consistency upon updates from the
firmware developers, it has been accustomed to clone the Luebeck "site" folder from 
https://github.com/freifunk-gluon/site-ffhl and localise IP addresses etc for the new community.

*Gatewayconfig*

The initial /etc/network/interfaces file will look like

      auto eth0
      iface eth0 inet static
               address 141.101.36.19
               netmask 255.255.255.0
               broadcast 141.101.36.255
               gateway 141.101.36.1
               dns-nameservers gw1.ostholstein.freifunk.net
     
     iface eth0 inet6 static
         address 2a00:12c0:1015:166::1:1/48
         up ip -6 route add 2a00:12c0:1015::1 dev eth0
         down ip -6 route del 2a00:12c0:1015::1 dev eth0
         up ip -6 route add default via 2a00:12c0:1015::1 dev eth0
         down ip -6 route del default via 2a00:12c0:1015::1 dev eth0

to then be extended for a few Freifunk-devices. Further instructions can be found on http://luebeck.freifunk.net/wiki/gatewayconfig
which comprise the installation of the following packages as mentioned above
    debfoster -u bird bird6 isc-dhcp-server radvd lighttpd haveged openvpn
Further, 
    apt-get install bind9 dnsutils
as a substitute for named and (yet missing in that description)
    apt-get install bridge-utils
for brctl.

/etc/modules: add batman-adv

/etc/hosts:

    10.135.0.8      gw1.ostholstein.freifunk.net gw1
    10.135.0.16     gw2.ostholstein.freifunk.net gw1
    10.135.0.24     gw3.ostholstein.freifunk.net gw1
    10.135.0.32     gw4.ostholstein.freifunk.net gw1
    10.135.0.40     gw5.ostholstein.freifunk.net gw1
    10.135.0.48     gw6.ostholstein.freifunk.net gw1
    10.135.0.56     gw7.ostholstein.freifunk.net gw1

At some point during startup, the gateway must initiate its role
as a server in the batman network by invocating
    batctl gw server
This could optionally be performed upon the initiation of a contact
with the anonymiser's in the respective init script - or elsewhere.

Freifunk-Mesh configuration on Gateway
--------------------------------------

*batman-adv*

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

*dhcpd*

The configuration of the dhcpd is straight forward - just two caveats:
*   there is a slightly unusual is the large number subnet, a /21 that
    the dhcpd distributes IPv4 numbers for, expressed by the range attribute.
    This is different for every gateway.
*   all gateways and dhcpd with them are on the very same network, which is
    a /18 if not a /17, i.e. 10.135.0.0 with netmask 255.255.192.0 . 
  
Examples:

Gateway 135.0.8

    subnet 10.135.0.0 netmask 255.255.192.0 {
        range 10.135.8.0 10.135.15.255;
        option routers 10.135.0.8;
        option domain-name-servers 10.135.0.8;
    }

Gateway 135.0.16

    subnet 10.135.0.0 netmask 255.255.192.0 {
        range 10.135.16.0 10.135.31.255;
        option routers 10.135.0.8;
        option domain-name-servers 10.135.0.8;
    }

DNS
---

Every gateway also serves as a DNS server. Their configuration is the
same for all instances and shared also by a github directory.

More on http://wiki.freifunk.net/DNS

fastd VPN
---------

The fastd provides the secured communication between the router and the gateway.

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

Anonymising internet traffic - external server: IPv4 exit
---------------------------------------------------------

install and configure mullvad

      - openvpn resolvconf
      - unzip
      - https://mullvad.net/en/setup/openvpn/ NICHT FOLGEN
      - http://wiki.freifunk.net/Freifunk_Hamburg/Gateway

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

# ICVPN eintragen, sobald ein Gateway fertig ist
  läuft über tinc
  Keys-Repo: https://github.com/freifunk/icvpn
  Konfiguration nach http://wiki.freifunk.net/IC-VPN


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
Configure mailing lists
*   MX-Record
*   PTR-Record
*   Mailman + z.B.Postfix


### Peculiarities with the Gluon/Lübecker Setup ###

#### Next-Node Adresse ####
in Lübeck: x.y.0.1 bzw. xxxx::1
Diese Adresse "freihalten". Vorschlag: IPv4 erstes /29 reservieren, also 0..7

