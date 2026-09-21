Tehtävä 3 – Prometheus, Node Exporter ja Grafana

1.	Johdanto 
Mitä monintorointi tarkoittaa
-	Monitoroinnilla tarkoitetaan IT-infrastruktuurin, verkon laitteiden, palvelimien ja sovelluksien tilan, suorituskyvyn sekä kuormituksen jatkuvaa seurantaa ja keräämistä.
Miksi monitorointi on tärkeää palveluiden ja verkkojen ylläpidossa?
-	Monintorointi varmistaa järjestelmien toimivuuden, turvallisuuden ja hyvän käyttäjäkokemuksen.
Mikä on Prometheus
-	Prometheus on avoimen lähdekoodin järjestelmien valvonta- ja hälytysalusta, joka on suunniteltu erityisesti aikasarjatietojen keräämiseen ja käsittelyyn.

2 Node Exporterin asennus

Node exporterin asennus 
Kirjaudutaan web1 koneelle 
-	docker exec -it clab-hamk-verkonhallinta-golden-web1 bash
Asensin tarvitattavat työkalut 
-	apt update
-	apt install wget tar -y
Node exporter lataus
ladattiin suoraan versionumeron mukaisesta osoitteesta, purettiin ja siirryttiin sen kansioon
-	wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
-	tar xvf node_exporter-1.8.1.linux-amd64.tar.gz
-	cd node_exporter-1.8.1.linux-amd64
Käynnistys komento
-	./node_exporter


3  Tarkista exporterin toiminta
Kohteiden tarkastelu
Tarkistetaan että node exporter vastaa
-	curl http://localhost:9100/metrics
Jos vastaa näytölle tuule suuri läjä mittareita ja niiden antamia lukemia.
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/6ac31f04368a0c755d81d0ff9dc8021c8d113968/reports/reports/Images/Node%20exporterin%20toimivuus.png]

4 Tarkista että Prometheus näkee palvelimen
Avataan Prometheus selaimessa 
-	http://localhost:9090
Valitaan ylhäältä Status kohta ja sen sisältä Target health. Scrollataan alas ja nähdään että web1 on kohteena ja tila up
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/6ac31f04368a0c755d81d0ff9dc8021c8d113968/reports/reports/Images/Prometheus%20todiste%20toimivuudesta.png]

5 Grafanan tietolähde
Mennään Grafanaan 
-	http://localhost:3000
Valitaan data sources ja sieltä new datasource. Lisätään Prometheus ja sen url eli 
-	http://prometheus:9090
Testataan ihan alhaalta yhteys ja saadaan onnistunutta.
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/6ac31f04368a0c755d81d0ff9dc8021c8d113968/reports/reports/Images/prometheus%20ydist%C3%A4minen%20grafanaan.png]

6 Luo dashboard
Lisätään grafanaa oma dashboard ja lisätään sinne CPU, muisti, levytila, verkkoliikenne (saapuva) ja verkkoliikenne (lähtevä)

CPU
-	100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Muisti
-	(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
Levytila
-	100 - (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100)
Verkkoliikenne (saapuva)
-	rate(node_network_receive_bytes_total[5m])
Verkkoliikenne (lähetävä)
-	rate(node_network_transmit_bytes_total[5m])

Lopputulos
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/6ac31f04368a0c755d81d0ff9dc8021c8d113968/reports/reports/Images/Grafana%20valmiit%20dashboardit.png]

7 Kuormituksen generointi
Aiheutetaan kuormitusta CPU:lle. Ladataan stress – ng ja kuormitetaan 4 CPU 60 sekunnin ajan. 
-	apt install stress-ng
-	stress-ng --cpu 4 --timeout 60s
Miten CPU-käyrä muuttui?
-	CPU nousi selkeästi 0.9  8.2 paikkeille eli prosessorikuorman nostaminen toimi onnistuneesti ja nosti käyttöastetta tilapäisesti
Miten levytilan käyttö muuttui?
-	Levytila pysyi kohtuulisen samana nousi parilla desimaalilla
Näkyikö verkkoliikenteessä muutoksia?
-	Ainakin molemmissa verkkoliikenteissä nousi käyrä. Saapuvalla se nousi noin 70 166216 ja lähetävällä noin 0-5  800 paikkeille.
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/6ac31f04368a0c755d81d0ff9dc8021c8d113968/reports/reports/Images/Kuormitus%20testi.png]

8 SNMP vs Prometheus
| Ominaisuus | SNMP | Prometheus |
| :--- | :--- | :--- |
| **Tiedonkeruu** | Pull-pohjainen (Manager kysyy laitteilta MIB-kantoja tarvittaessa tai intervallilla). | Pull-pohjainen (Prometheus-server skreipaa aktiivisesti Node Exporterin HTTP-rajapintaa). |
| **Käyttöönotto** | Vaatii SNMP-agentin konfiguroinnin laitteille (yhteisömerkit/communiteetit, MIB-kirjastot). | Vaatii Node Exporterin asennuksen ja Prometheuksen target-määrittelyn. |
| **Mittarien määrä** | Perustuu OID-polkuihin ja MIB-taulukoihin; rajallisempi tai raskaampi hakea laajoja järjestelmätietoja. | Erittäin laaja valikoima reaaliaikaisia käyttöjärjestelmä- ja laitteistomittareita suoraan (node_cpu, `node_memory...). |
| **Visualisointi** | Perustuu usein erillisiin SNMP-hallintaohjelmistoihin tai suppeisiin työkaluihin. | Hyödyntää Grafanaa, joka mahdollistaa erittäin monipuoliset ja dynaamiset aikasarjakaaviot joita voi halutessa muuttaa. |
| **Hälytysmahdollisuudet** | Usein rajoittuneempia tai vaativat erillisen Network Management System -säännöt. | Sisäänrakennettu Alertmanager ja joustavat hälytyssäännöt kynnysarvoille. |
| **Soveltuvuus pilviympäristöihin** | Heikko. | Hyvä / Todella hyvä. |

9 Pohdinta
Mitä hyötyä Prometheuksesta on verrattuna SNMP:hen?
-	Prometheus on suunniteltu erityisesti nykyaikaisiin dynaamisiin ja konttipohjaisiin ympäristöihin. Verrattuna perinteiseen SNMP-protokollaan
Millaisia mittareita ylläpitäjän kannattaa seurata jatkuvasti?
-	Järjestelmäylläpitäjän on tärkeää valvoa jatkuvasti niin sanottuja kriittisiä elintoimintoja kuten CPU käyttöaste ja muistin riittävyys
Mitä tietoa dashboardisi tarjoaa ylläpitäjälle?
-	Luotu Grafana-dashboard tarjoaa ylläpitäjälle yhdellä silmäyksellä visuaalisen tilannekuvan ympäristön (web1, db1, client1) senhetkisestä suorituskyvystä. Näistä asioista CPU, muisti, levytila, verkkoliikenne (saapuva) ja verkkoliikenne (lähtevä)
Mitä uusia mittareita lisäisit dashboardiin?
-	Load Average (1m, 5m, 15m): Antaa laajemman kuvan järjestelmän pitkäaikaisesta prosessorikuormasta.
-	TCP-yhteyksien tilat: Auttaa havaitsemaan esimerkiksi mahdollisia verkkohyökkäyksiä tai resurssivuotoja verkkoyhteyksissä.
Miten monitorointitiedosta voisi olla hyötyä vianetsinnässä?
-	Sillä voisi löytää esim suoraan mistä ja million vika olisi peräisin onko se sitten että ei saa yhteyttä palvelimelle tai jonkin oman koneen osan ongelma esim juuri CPU

