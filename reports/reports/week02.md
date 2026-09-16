Hallinta  Tehtävä 2 - SNMP

1.	Johdanto

SNMP = Verkonhallintaprotokolla, jota käytetään IP-verkkojen laitteiden valvontaan, hallintaan ja tietojen keräämiseen

SNMP käyttö verkonhallinnassa = Verkon laitteiden valvontaan ja niiden tilan seuraamiseen. Esim seurataan vaikka laitteiden suorituskykyä ja toimintaa. 

2.	Asennus 

Miten SNMP-agentti asennettiin? 
Se asennettiin tälläisillä käskyillä web1 palvelimelle
-	apt update 
-	apt install snmp snmpd -y
Asennuksen jälkeen SNMP-palvelun toiminta tarkistettiin:
-	service snmpd status 
Tällä tarkistin että onko snmbp is running ja se ei ollut joten jouduin starttaamaan sen käskyllä 
-	service snmpd start

Mitä konfiguraatiomuutoksia tehtiin? 

Asenensin ekana nano:n komennolla 
-	apt update && apt install nano -y
Konfiguraatiotiedostoa muokattiin avaamalla se tekstieditorilla komennolla 
-	nano /etc/snmp/snmpd.conf
Lisäsin rocommunity public rivin tekstieditorin loppuun jonka jälkeen tallensin sen CTRL O + CTRL X
Käynnistin snmp uudelleen 
-	service snmpd restart
Jonka jälkeen tarkistin vielä prosessin komennolla 
-	ps aux | grep snmpd


3.	Kerätyt tiedot

SNMP-agentit muille laitteille

-	SNMP-agentti asennettiin täsmälleen samalla tavalla myös db1- ja
branch-client-laitteille.
Db1 päästiin komenolla 
-	docker exec -it clab-hamk-verkonhallinta-golden-db1 bash 
ja poistuttiin 
-	exit
branch-clientille pääsi komenolla 
-	docker exec -it clab-hamk-verkonhallinta-golden-branch-client bash
ja poistuttiin 
-	exit
Molemmille laitteille asennettiin `snmp`- ja `snmpd`-paketit,
jonka jälkeen tekstitiedostoon pistettiin sama
-	rocommunity public
Suoritetut SNMP-kyselyt 
Kirjaudutiin ekana ansible palveluun komenolla 
-	docker exec -it clab-hamk-verkonhallinta-golden-ansible bash
Asensin SNMP työkalu komenoilla 
-	apt update
-	apt install snmp -y
Yhteyttä ei voida testata ennenkun on ladattu MIB paketit komenolla 
-	apt update && apt install snmp-mibs-downloader -y
Sitten testataan yhteydet
-	snmpwalk -v2c -c public web1 system
-	snmpwalk -v2c -c public db1 system
-	snmpwalk -v2c -c public branch-client system
Saadan 

| Laite | Nimi | Käyttöjärjestelmä | UpTime |
| :--- | :--- | :--- | :--- |
| **Web1** | web1 | Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 0:04:37.46 |
| **Db1** | db1 | Linux db1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 0:05:07.37 |
| **Branch-client** | branch-client | Linux branch-client 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 0:04:11.65 |


4.	Verkkorajapinnat
SNMP:n avulla verkko rajapintatiedot
Web1-palvelimen verkkorajapintoja tutkittiin SNMP:n avulla seuraavalla komennolla
-	snmpwalk -v2c -c public web1 ifDescr
Saimme 3 verkkorajapintaa 
Lo
Eth0 
Eth1
5.	OID analyysi

Käytetyt OID-objektit. Mitä tietoa ne tarjoavat ja mihin niitä käytetään? 

| OID | Tarkoitus | Käyttö |
| :--- | :--- | :--- |
| **sysName.0** | Laitteen verkkonimi | Laitteen tunnistaminen ja inventointi valvontajärjestelmässä. |
| **sysDescr.0** | Yksityiskohtainen kuvaus laitteesta, käyttöjärjestelmästä ja ytimen versiosta. | Ohjelmistoversion ja käyttöjärjestelmän tarkistus haavoittuvuuksien tai päivitystarpeiden varalta. |
| **sysUpTime.0** | Laitteen käynnissäoloaika | Palvelun saatavuuden seuranta ja mahdollisten uudelleenkäynnistysten havaitseminen. |
| **ifDescr** | Verkkorajapinnan nimi tai kuvaus | Verkkotopologian kartoitus ja rajapintakohtaisen liikenteen seuranta. |
| **ifOperStatus** | Verkkorajapinnan senhetkinen toimintatila Up tai Down | Hälytysten tuottaminen, jos kriittinen verkkoyhteys katkeaa. |

6.	Pohdinta
Mitä opin tehtävän aikana
-	Tehtävän aikana opin SNMP:n perusperiaatteen sekä sen, miten SNMP-agentti asennetaan Linux-palvelimelle. Opin myös tekemään SNMP-kyselyitä komentoriviltä `snmpget`- ja `snmpwalk`-komennoilla.
Mitkä ovat SNMP tärkeimmät hyödyt
-	SNMP:n tärkein hyöty on verkkolaitteiden ja palvelimien automaattinen valvonta. Sen avulla voidaan kerätä esimerkiksi laitteiden perustietoja, käyttöaikaa, verkkorajapintojen tietoja ja niiden toimintatiloja.
Mitkä ovat SNMP:n rajoitukset tai haasteet?
-	SNMPv2c:n merkittävä rajoitus on tietoturva. Se perustuu community stringiin eikä tarjoa kunnollista salausta.

