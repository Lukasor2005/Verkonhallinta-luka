Viikko 5 – Tietoturva, lokit ja verkkoliikenteen analysointi

Johdanto

Tietoturva verkonhallinnan näkökulmasta tarkoittaa verkkoinfrastruktuurin, laitteiden ja dataliikenteen jatkuvaa suojaamista, valvontaa ja hallintaa. Verkonhallinnan tehtävänä on varmistaa verkon luottamuksellisuus, eheys ja käytettävyys sekä estää luvaton pääsy verkkoon ja sen resursseihin.
Tehtävä 1 – Liikenteen kaappaus
Avataan kaksi erillistä terminal ikkunaa jossa toisessa kirjaudutaan web1-konttiin ja aloitetaan liikenteen kaappaus ja toisessa taas kirjaudutaan attacker-konttiin 
-	Web1 konttiin pääsy = docker exec -it clab-hamk-verkonhallinta-golden-web1 bash
-	Attacker konttiin pääsy = docker exec -it clab-hamk-verkonhallinta-golden-attacker bash
-	Web1 aloitetaan kaappaus tcpdump -i eth1 -w web1.pcap

Tehtävä 2 – Porttiskannaus

Attacker koneella ajetaan nmap 10.10.20.101 ja keskeytetään kaappaus web1
Varmistetaan että web1.pcap tallentui ajamalla 
-	ls -l web1.pcap

Tehtävä 3 – Wireshark-analyysi

Mistä IP-osoitteesta skannaus tuli
-	Skannaus tuli osoitteesta 10.10.18.200 (hyökkääjän/attacker-koneen IP-osoite), ja se kohdistui web1-palvelimen osoitteeseen 10.10.20.101.
Mitä portteja testattiin
-	Nmap testasi oletuksena kaikki 1000 yleisintä TCP-porttia
Mitä vastauksia palvelin lähetti
-	Koska verkon reititys tai palomuuri esti yhteydenotot, palvelin tai palomuuri lähetti vastauksena TCP RST, ACK -paketteja tai ICMP Destination unreachable -virheilmoituksia. Tämä tarkoittaa, että portit olivat suodatettuja eikä avoimia palveluita löytynyt suoraan läpi päästettäväksi.

[https://github.com/Lukasor2005/Verkonhallinta-luka/blob/fe56f17c4552de1dfecc76cf0164fee6682cb255/reports/reports/Images/Wireshark%20tcp.png]

 
Tehtävä 4 – DNS-liikenne

Liikenteen tarkastelussa käytettiin Wiresharkin suodatinta dns.
DNS-kyselyt
-	web1-koneen IP-osoitteesta (10.10.20.101) tehtiin nimipalvelukyselyjä verkkotunnukselle hamk.fi 
DNS-vastaukset
-	Nimipalvelin vastasi pyyntöihin palauttaen hamk.fi-verkkotunnukselle IPv4-osoitteen
Käytetyt palvelimet
-	DNS-kyselyt osoitettiin verkon nimipalvelimelle, jonka IP-osoite on 10.255.255.254. 

 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/fe56f17c4552de1dfecc76cf0164fee6682cb255/reports/reports/Images/Wireshark%20dns.png]

Tehtävä 5 – HTTP-liikenne

HTTP-pyynnöt
-	web1-kone (10.10.20.101) lähetti GET / HTTP/1.1 -pyynnön kohdeosoitteeseen 104.20.23.154
HTTP-vastaukset
-	Palvelin palautti vastauksen HTTP/1.1 200 OK (text/html), mikä osoittaa pyynnön onnistuneen ja sivun sisällön siirtyneen takaisin pyytäjälle.
Palvelimet
-	Etäpalvelimen IP-osoite, johon HTTP-yhteys muodostettiin, on 104.20.23.154. 
 
[https://github.com/Lukasor2005/Verkonhallinta-luka/blob/fe56f17c4552de1dfecc76cf0164fee6682cb255/reports/reports/Images/Wireshark%20http.png]

Tehtävä 6 – Loki-analyysi

Kirjautumiset ja käyttäjäaktiviteetti
-	Järjestelmän sessio- ja kirjautumislokeista (kuten btmp / lastlog) ei löytynyt ylimääräisiä tai epäilyttäviä ulkoisia kirjautumisyrityksiä; toiminta on rajoittunut hallittuihin paikallisiin säiliöyhteyksiin.
Virhetilanteet
-	Lokitarkastelussa ei ilmennyt kriittisiä virheitä tai palveluiden kaatumisia; esimerkiksi Nginxin error.log-tiedosto oli tyhjä.
Palveluiden käynnistykset ja verkkotapahtumat
-	Kernelin lokit osoittivat verkkoliitäntöjen tilamuutoksia, kuten siirtymisen promiskuiteettitilaan (entered promiscuous mode), mikä liittyy suoraan tcpdump-pakettikaappauksiin ja verkkoliikenteen analysointiin.

Tehtävä 7 – Turvallisuusarvio

Mikä on turvallista?
-	Ympäristön eristys Laboratorio pyöritetään konttiohjatussa virtuaaliympäristössä mikä estää suoran pääsyn tuotantoverkkoihin ja pitää mahdolliset testitikut tai haitalliset toimet turvallisesti hiekkalaatikossa.
Mikä on tietoturvariski?
-	Salaamaton liikenne Suuri osa testatusta liikenteestä (kuten HTTP-selailu ja perusmuotoinen DNS-kysely) kulkee salaamattomana. Kuka tahansa samaa verkkosegmenttiä kuunteleva taho voi kaapata ja lukea datan sisällön

-	Puutteellinen segmentointi ja palomuurisuojaus: Hyökkääjäkone  pystyy vapaasti tekemään porttiskannauksia kohti sisäverkkoja ilman, että reitityksessä tai palomuurissa on tiukkoja estoja luvattomalle tiedustelulle.

Mitä pitäisi parantaa?
-	Salauksen pakottaminen  Kaikessa verkkoliikenteessä tulisi siirtyä käyttämään salattuja protokollia, jotta pakettien kaappaaminen ei paljasta arkaluontoista tietoa.

Pohdinta

Miten porttiskannaus näkyi liikenteessä?
-	Se näkyi toistuvina TCP [SYN] -paketteina hyökkääjäkoneelta useisiin eri kohdeportteihin, joihin palvelin vastasi RST, ACK -paketeilla tai ICMP Destination unreachable -virheilmoituksilla.
Miten se näkyi lokeissa?
-	Se näkyi kernelin lokeissa (dmesg) verkkoliitännän siirtymisenä promiskuiteettitilaan ja verkon tilamuutoksina, kun taas perinteiset järjestelmälokit tai sovelluslokit eivät rekisteröineet itse skannausta suoraan.
Mitä hyötyä verkkoliikenteen analysoinnista on?
-	Sen avulla voidaan havaita tietoturvapoikkeamia, selvittää verkon ongelmatilanteita, tunnistaa luvatonta tiedustelua ja ymmärtää mitä protokollia ja dataa laitteiden välillä liikkuu.
Miten monitorointia voisi hyödyntää hyökkäysten tunnistamisessa?
-	Reaaliaikaisella monitoroinnilla voidaan havaita epänormaalit liikennepiikit, porttiskannaukset ja luvattomat kirjautumisyritykset heti niiden alkaessa, jolloin hyökkäykset voidaan pysäyttää ajoissa.










