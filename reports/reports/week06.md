Viikko 6 – Zabbix ja keskitetty verkonvalvonta


1	Johdanto

Zabbix on avoimen lähdekoodin (AGPL-3.0) yritystason ohjelmisto, joka on tarkoitettu IT-infrastruktuurin, verkkojen, palvelimien, pilviresurssien ja sovellusten reaaliaikaiseen valvontaan ja hallintaan.

2	Tutustu Zabbixiin

Hosts: Valvotut laitteet ja palvelimet.

Templates: Valmiit mittaristo- ja konfiguraatiopohjat laitteille.

Monitoring: Reaaliaikainen seuranta, graafit ja aktiiviset ongelmat.

Dashboards: Visuaaliset yhteenvedot keskeisistä mittareista.

Alerts: Hälytysten ja ilmoituskanavien hallinta.

Reports: Raportit ja tilastot järjestelmän tilasta.

3	Lisää web1 valvontaan

Kirjaudutaan zabbixeen ja lisätään web1 hostiksi. Lisätään Web1 
•	Hostname = web1
•	IP-osoite = 172.20.20.4
•	Host Group = Linux servers
•	Template = Template OS Linux SNMPv2

4	Lisää db1 valvontaan

Kirjaudutaan zabbixeen ja lisätään db1 hostiksi. Lisätään db1 
•	Hostname = db1
•	IP-osoite = 172.20.20.3
•	Host Group = Linux servers
•	Template = Template OS Linux SNMPv2 
Sitten aletaan samaan tietoa zabbixiin esim tuossa CPU määrä 

 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/916a77a2963b458d5b74ce1e985e75a631f76dee/reports/reports/Images/Zabbix%20db1.png]

5	Lisää branch-client valvontaan

Kirjaudutaan zabbixeen ja lisätään Branch-client hostiksi. Lisätään Branch-client 
•	Hostname = Branch-client
•	IP-osoite = 172.20.20.7
•	Host Group = Linux servers
•	Template = Template OS Linux SNMPv2 
Web1, db1 ja Branch-client zabbixessa :
 
 [https://github.com/Lukasor2005/Verkonhallinta-luka/blob/916a77a2963b458d5b74ce1e985e75a631f76dee/reports/reports/Images/Kaikki%20palvelimet%20zabbix.png]

6	Valvontamittarien analyysi

| Mittari | Arvo | Merkitys |
| :--- | :--- | :--- |
| **CPU Usage** | 0,9884 % | Kertoo, kuinka paljon prosessorin kapasiteetista on käytössä. Alhainen arvo tarkoittaa, että prosessorilla on tällä hetkellä hyvin vapaata kapasiteettia. |
| **Memory Usage** | 31,631 % | Kertoo, kuinka suuri osa järjestelmän muistista on käytössä. Arvon avulla voidaan seurata, onko muistin käyttö kasvamassa liian suureksi. |
| **Load Average** | 0,21 (1 min) | Kertoo järjestelmän prosessorin kuormituksesta ja odottavista prosesseista viimeisen minuutin aikana. Pieni arvo tarkoittaa vähäistä kuormitusta. |
| **Uptime** | 00:16:03 | Kertoo, kuinka kauan db1-järjestelmä on ollut yhtäjaksoisesti käynnissä. Tätä voidaan käyttää esimerkiksi uudelleenkäynnistysten havaitsemiseen. |
| **Disk Usage** | 1,3009 % | Kertoo, kuinka suuri osa /-tiedostojärjestelmän levytilasta on käytössä. Arvon seuraaminen auttaa havaitsemaan tilanteen, jossa levytila alkaa loppua. |


7	Dashboardin luominen

Luodaan dashboard missä on nämä
•	CPU-kuorman seuranta
•	Muistinkäyttö
•	Levytilan käyttö
•	Verkkoliikenne
•	Hostien tila
 
[https://github.com/Lukasor2005/Verkonhallinta-luka/blob/916a77a2963b458d5b74ce1e985e75a631f76dee/reports/reports/Images/Dashboardin%20luominen.png]

8	Triggerien määrittäminen

Loin triggerit CPU kuormitukselle ja Levytilalle 

Triggeri 1: CPU-kuormitus
Triggerin nimi: High CPU utilization

Ehto: {Template OS Linux SNMPv2:system.cpu.load.avg1[laLoad.1].last()}>2

Vakavuusluokka: Warning 

Triggeri 2: Levytila
Triggerin nimi: Low disk space (Free disk space < 20%)

Ehto: {Template OS Linux SNMPv2:system.swap.pfree[snmp].last()}<20 

Vakavuusluokka: Warning

9	Hälytyksen simulointi

Luodaan hälytys
 
[https://github.com/Lukasor2005/Verkonhallinta-luka/blob/916a77a2963b458d5b74ce1e985e75a631f76dee/reports/reports/Images/H%C3%A4lytyksen%20luonti.png]

10	Häiriötilanne

Sammutin db1 kello 20:27 ja Zabbix huomasi että se oli sammunut 20:29
 
[https://github.com/Lukasor2005/Verkonhallinta-luka/blob/916a77a2963b458d5b74ce1e985e75a631f76dee/reports/reports/Images/db1%20sammuminen.png]

11	Kurssin työkalujen vertailu

| Ominaisuus | SNMP | Prometheus | Zabbix |
| :--- | :--- | :--- | :--- |
| **Tiedonkeruu** | Pull-pohjainen. Manager kysyy laitteilta MIB-kantojen tietoja tarvittaessa tai tietyin intervallein. | Pull-pohjainen. Prometheus-server skreipaa aktiivisesti esimerkiksi Node Exporterin HTTP-rajapintaa. | Voi kerätä tietoa esimerkiksi SNMP:n, Zabbix-agentin ja muiden tarkistusten avulla. |
| **Dashboardit** | Ei itsessään varsinaisia dashboardeja, vaan tietoja visualisoidaan erillisillä hallinta- ja valvontatyökaluilla. | Hyödyntää usein Grafanaa, jolla voidaan tehdä monipuolisia ja dynaamisia aikasarjakaavioita. | Sisäänrakennetut dashboardit ja graafit, joilla valvottavien laitteiden ja palveluiden tietoja voidaan seurata. |
| **Hälytykset** | Hälytykset ovat usein rajoittuneempia ja voivat vaatia erillisen Network Management System -ratkaisun. | Alertmanager ja joustavat hälytyssäännöt mahdollistavat hälytysten tekemisen esimerkiksi kynnysarvojen perusteella. | Sisäänrakennetut triggerit ja hälytykset mahdollistavat esimerkiksi CPU-, muisti-, levy- ja verkkohälytysten tekemisen. |
| **Käyttöönotto** | Vaatii SNMP-agentin konfiguroinnin laitteille sekä esimerkiksi yhteisömerkkien ja MIB-tietojen määrittelyn. | Vaatii Node Exporterin asentamisen sekä Prometheuksen target-määritysten tekemisen. | Vaatii Zabbix-palvelimen sekä valvottavien kohteiden määrittelyn. Tarvittaessa myös Zabbix-agentin tai SNMP:n käyttöönoton. |
| **Skaalautuvuus** | Soveltuu hyvin suurten verkkolaitemäärien perustason valvontaan, mutta laajat ympäristöt voivat vaatia erillisiä hallintaratkaisuja. | Erittäin hyvä erityisesti suurten aikasarjamäärien ja dynaamisten ympäristöjen valvontaan. | Hyvä. Soveltuu pienistä ympäristöistä suuriin yritysverkkoihin, ja valvontaa voidaan jakaa useille kohteille. |
| **Yrityskäyttö** | Erittäin yleinen verkkolaitteiden, kuten reitittimien ja kytkimien, valvonnassa. | Soveltuu erityisesti moderneihin palvelin-, kontti- ja pilviympäristöihin. | Soveltuu hyvin yritysten keskitettyyn IT-infrastruktuurin, palvelimien ja verkkolaitteiden valvontaan. |


12	Pohdinta

Mitä hyötyä keskitetystä valvonnasta on?
-	Kokonaiskuva: Ylläpitäjä näkee yhdellä silmäyksellä koko infran terveyden ilman tarvetta kirjautua erikseen jokaiselle laitteelle.

-	Ennakoitavuus ja nopea reagointi: Ongelmat ja pullonkaulat havaitaan heti niiden alkaessa, ei vasta silloin, kun palvelu kaatuu loppukäyttäjiltä.

Mitkä mittarit ovat mielestäsi tärkeimpiä?
-	Käytettävyys (Uptime / Ping): Vastaako laite tai palvelu verkossa vai onko se alhaalla.

-	Suorittimen ja muistin käyttöaste (CPU & Memory Usage): Auttaa havaitsemaan ylikuormitustilat ja muistivuodot ennen järjestelmän kaatumista.

-	Levytila (Disk Space): Levyn loppuminen pysäyttää tyypillisesti tietokannat ja palvelimet välittömästi.


Millaisista tilanteista ylläpitäjän pitäisi saada hälytys?
-	Palvelun tai palvelimen täydellinen kaatuminen 

-	Toistuvat verkkokatkokset tai korkea pakettihävikki reitittimien välillä.


Missä tilanteissa käyttäisit Prometheusta?
-	Kun seurattavia kohteita luodaan ja tuhotaan dynaamisesti, jolloin Prometheus osaa automaattisesti etsiä ja seurata niitä 

-	Kun tarvitaan tehokasta aikasarjadataa ja monimutkaisia kyselyitä sovellustason metriikoille (PromQL-kielellä).

Missä tilanteissa käyttäisit Zabbixia?
-	Kun valvotaan laajasti fyysisiä verkkolaitteita (kytkimet, palomuurit, reitittimet) SNMP-protokollan avulla.

Mitä valvontatoimintoja lisäisit tähän ympäristöön?
-	Automaattiset korjaustoimet : Webhook-pohjaiset automaatiot, jotka käynnistävät jumittuneen säiliön tai tyhjentävät välimuistin automaattisesti ilman ylläpitäjän manuaalista puuttumista.


