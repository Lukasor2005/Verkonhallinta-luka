Tehtävä 1.1 – Topologian kartoitus

Laite	Tarkoitus
r1	Käyttäjäverkon reititin. Yhdistää käyttäjäverkon r2:een.
r2	Verkon keskimmäinen reititin. Yhdistää r1:n, r3:n, palvelinverkon ja hallintaverkon.
r3	Sivukonttorin reititin. Yhdistää sivukonttorin r2:een.
client1	Käyttäjäverkon Linux-asiakaslaite.
attacker	Kali Linux -pohjainen testaus- ja tietoturvalaitteena käytettävä asiakas.
web1	Palvelinverkon web-palvelin.
db1	Palvelinverkon tietokantapalvelin.
branch-client	Sivukonttorin Linux-asiakaslaite.
ansible	Automaatio- ja hallintapalvelin.
prometheus	Monitorointipalvelu, joka kerää metriikkatietoja.
grafana	Monitorointitietojen visualisointipalvelu.
zabbix	Verkon ja järjestelmien monitorointipalvelu.
cadvisor	Konttien suorituskyky- ja resurssimittareita keräävä palvelu.
syslog	Keskitetty lokipalvelin, jonne reitittimien lokitietoja lähetetään.
srv-bp	Palvelinverkon L2-sillan toteutukseen käytettävä kontti.
mgmt-bp	Hallintaverkon L2-sillan toteutukseen käytettävä kontti.

Tehtävä 1.2 – Verkkokaavio

 Downloads valmis verkkokaavio 

Tehtävä 1.3 – IP-osoitteiden dokumentointi

Verkko	Tarkoitus	Yhdyskäytävä
10.10.10.0/24	Käyttäjäverkko	r1 – 10.10.10.1
10.10.20.0/24	Palvelinverkko	r2 – 10.10.20.1
10.10.30.0/24	Sivukonttorin verkko	r3 – 10.10.30.1
10.10.99.0/24	Hallintaverkko	r2 – 10.10.99.1
10.255.12.0/30	r1–r2-reititinyhteys	r1 10.255.12.1, r2 10.255.12.2
10.255.23.0/30	r2–r3-reititinyhteys	r2 10.255.23.1, r3 10.255.23.2


Tehtävä 1.4 – Reitityksen tutkiminen
	löytyykö yhteys kaikkiin verkkoihin

Kyllä yhteydet löytyi koska ping meni läpi ilman packetlossia

	mitä reittiä liikenne kulkee ja mitä reitittimiä reitille kuulu

liikenne kulkee yhteensä kolmen reititinhyppyjen (hops) läpi ennen määränpäätä:
	Hyppy 1 (10.10.10.1): Client1:n oletusportti (Default Gateway), eli oman paikallisverkon reititin (eth1-liitännän kautta).
	Hyppy 2 (10.255.12.2): Runkoverkon / Yhdysverkon ensimmäinen välireititin (Transit router 1).
	Hyppy 3 (10.255.23.2): Runkoverkon toinen välireititin (Transit router 2 / Branch-verkon yhdysreititin).
	Hyppy 4 (10.10.30.101): Kohdelaite (Branch-client).
TTL-analyysi:
Määränpäästä vastaanotettu TTL-arvo ping-testissä oli 61. Kun standardi aloitus-TTL Linuxissa on 64, kaavan 64-3=61 mukaan paketti kulki tismalleen kolmen reitittimen läpi.

root@client1:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if27: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 06:fe:56:8c:b3:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.20.20.11/24 brd 172.20.20.255 scope global eth0
       valid_lft forever preferred_lft forever
34: eth1@if35: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP group default
    link/ether aa:c1:ab:d9:07:63 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    altname clab-o-05031180f95d8850
    inet 10.10.10.101/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fed9:763/64 scope link
       valid_lft forever preferred_lft forever

root@client1:/# ip route
default via 10.10.10.1 dev eth1
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.101
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.11

root@client1:/# ping -c 4 10.10.20.101
PING 10.10.20.101 (10.10.20.101) 56(84) bytes of data.
64 bytes from 10.10.20.101: icmp_seq=1 ttl=62 time=0.304 ms
64 bytes from 10.10.20.101: icmp_seq=2 ttl=62 time=0.180 ms
64 bytes from 10.10.20.101: icmp_seq=3 ttl=62 time=0.112 ms
64 bytes from 10.10.20.101: icmp_seq=4 ttl=62 time=0.227 ms

--- 10.10.20.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3064ms
rtt min/avg/max/mdev = 0.112/0.205/0.304/0.069 ms

root@client1:/# ping -c 4 10.10.30.101
PING 10.10.30.101 (10.10.30.101) 56(84) bytes of data.
64 bytes from 10.10.30.101: icmp_seq=1 ttl=61 time=0.780 ms
64 bytes from 10.10.30.101: icmp_seq=2 ttl=61 time=0.220 ms
64 bytes from 10.10.30.101: icmp_seq=3 ttl=61 time=0.144 ms
64 bytes from 10.10.30.101: icmp_seq=4 ttl=61 time=0.135 ms

--- 10.10.30.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3078ms
rtt min/avg/max/mdev = 0.135/0.319/0.780/0.267 ms

root@client1:/# traceroute 10.10.30.101
traceroute to 10.10.30.101 (10.10.30.101), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  0.663 ms  0.471 ms  0.457 ms
 2  10.255.12.2 (10.255.12.2)  0.449 ms  0.341 ms  0.327 ms
 3  10.255.23.2 (10.255.23.2)  0.317 ms  0.259 ms  0.243 ms
 4  10.10.30.101 (10.10.30.101)  0.231 ms  0.159 ms  0.141 ms



