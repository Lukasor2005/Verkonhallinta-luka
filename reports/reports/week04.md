Viikko 4 – Ansible ja Infrastructure as Code

1 Johdanto
Mikä on Infrastructure as Code (IaC)?
-	infrastruktuurin, kuten palvelinten, verkkojen ja muiden järjestelmien, hallintaa koodin ja määritystiedostojen avulla. 
Miten automaatio muuttaa palvelinten ylläpitoa?
-	Automaatio vähentää palvelinten ylläpidossa tehtävää manuaalista työtä. Esimerkiksi ohjelmistojen asennukset, asetusten muuttaminen ja käyttäjien hallinta voidaan suorittaa automaattisesti useilla palvelimilla.
Mikä on Ansiblen rooli infrastruktuurin hallinnassa?
-	Ansible on automaatiotyökalu, jota voidaan käyttää palvelinten ja muun infrastruktuurin hallintaan. Ansiblen avulla voidaan esimerkiksi asentaa ohjelmistoja, muuttaa palvelinten asetuksia ja suorittaa komentoja useilla palvelimilla samanaikaisesti.
2 Tutustu inventoryyn
mitä laitteita inventory sisältää
-	r1, r2, r3, client1, attacker, branch-client, web1, db1, prometheus, grafana, zabbix, cadvisor, ansible.
miten ryhmät on muodostettu
-	Laitteet on ryhmitelty funktionaalisten roolien mukaan omiin ryhmiinsä routers, clients, servers, monitoring, management. Loogisiin verkkosegmentteihin user_network, server_network, branch_office. Sekä yhdistelmäryhmiin network_devices, linux_hosts, ubuntu_hosts, node_exporter.
Mitä hyötyä ryhmien käytöstä on?
-	Ryhmien avulla voidaan kohdistaa Ansible-komentoja ja playbookeja kerralla kokonaisiin laiteryhmiin sen sijaan, että jokaista konetta jouduttaisiin konfiguroimaan yksitellen.

3 Suorita ensimmäinen playbook
Mitä playbook tekee?
-	Playbook käyttää Ansibleen sisäänrakennettua ping- testatakseen verkkoyhteyttä ja kirjautumisoikeuksia inventory-tiedostossa määriteltyihin Linux-isäntiin ja reitittimiin.
Mitä tuloksista voidaan päätellä?
-	Linux-koneet ja palvelimet (attacker, branch-client, client1, db1, web1): Yhteydet toimivat täydellisesti, sillä ne palauttivat tilan ok=1 ja vastasivat onnistuneesti.
-	Reitittimet (r1, r2, r3): Vastaavat myös onnistuneesti ping-kutsuun (SUCCESS).
-	Valvontakoneet ja hallintakone (prometheus, grafana, zabbix, cadvisor, ansible): Palauttivat UNREACHABLE!, mikä johtuu siitä, että näissä erillisissä monitorointikoneteissa ei ole erillistä SSH-palvelinta tai kyse on labraympäristön sisäisestä rakenteesta, mikä ei vaikuta varsinaisten kohdepalvelimien Ansible-hallintaan.
4 Tutustu valmiisiin esimerkkeihin
1. Mitä moduuleja playbookeissa käytetään?
- install-snmp.yml:
-	apt: Hallitsee pakettien asennusta (snmp, snmpd) ja pakettivaraston päivitystä.
-	copy: Kirjoittaa suoraan sisällön (content) konfiguraatiotiedostoon (/etc/snmp/snmpd.conf).
-	service: Ottaa SNMP-palvelun käyttöön (enabled: yes) ja käynnistää sen (state: started).
-	shell: Suorittaa komentorivikohdistuksen (pgrep snmpd) prosessin tarkistamiseksi.
-	debug: Tulostaa viestin ruudulle onnistumisen merkiksi.
-	install-node-exporter.yml:
-	apt: Asentaa tarvittavat apupaketit (wget, tar).
-	file: Luo asennuskansion /opt/node_exporter määritetyillä oikeuksilla (mode: "0755").
-	get_url: Lataa Node Exporterin tar-paketin GitHubista dynaamisen versionumeron mukaan.
-	unarchive: Purkaa ladatun arkiston väliaikaishakemistoon.
-	copy: Siirtää puretun binääritiedoston oikeaan paikkaan ja asettaa suoritusoikeudet.
-	shell: Käynnistää tausta-ajona (nohup ... &) Node Exporter -prosessin.
-	uri: Testaa HTTP-pyynnöllä (http://localhost:9100/metrics), että exporter vastaa asianmukaisesti.
-	debug: Tulostaa onnistumisviestin.

2. Miten muuttujia (vars) hyödynnetään?
Muuttujien avulla määritellään keskitetysti asetuksia, joita voidaan käyttää uudelleen eri puolilla playbookia:
SNMP-playbookissa snmp_community: public määrittelee käytettävän yhteisönimen (community string).
Node Exporter -playbookissa node_exporter_version: "1.9.1" mahdollistaa ohjelmistoversion helpon päivittämisen suoraan latauslinkkeihin ja polkuihin ilman, että koodia tarvitsee muuttaa useasta eri paikasta.

3. Miten handlers-lohkoa käytetään SNMP-playbookissa?
SNMP-playbookissa on määritetty notify: restart snmpd -kutsu konfiguraation kopiointitehtävän yhteyteen.
handlers-lohkossa on määritelty varsinainen palvelun uudelleenkäynnistys. Handler aktivoituu ja suoritetaan vain silloin, jos konfiguraatiotiedostoon tehtiin todellinen muutos, mikä säästää järjestelmän resursseja.

4. Miksi Node Exporter -playbook ei tarvitse handleria?
Node Exporter on itsenäinen binääritiedosto, joka ladataan, puretaan ja käynnistetään sellaisenaan suoraan shell-moduulilla (nohup). Se ei lue erillistä konfiguraatiotiedostoa, jota tarvitsisi erikseen muokata ja jonka takia palvelu pitäisi uudelleenkäynnistää handlerin logiikalla.

Ajoin nämä komennot ja asennukset menivät läpi 
-	cd /ansible/playbooks
-	ansible-playbook -i ../inventory.ini install-snmp.yml
-	ansible-playbook -i ../inventory.ini install-node-exporter.yml

5 Oma playbook: palvelimen asennus
Luotiin oma playbook Vaihtoehdon A mukaisesti web1-koneelle Nginx-web-palvelimen asentamiseksi ja konfiguroimiseksi.

Playbookin sisältö:
---
- name: Install and configure Nginx web server
  hosts: web1
  become: true

  vars:
    web_message: "Welcome to web1 managed by Ansible!"

  tasks:

    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Nginx web server
      apt:
        name: nginx
        state: present

    - name: Create custom index.html
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html>
          <head>
              <title>Welcome to {{ inventory_hostname }}</title>
          </head>
          <body>
              <h1>Server: {{ inventory_hostname }}</h1>
              <p>{{ web_message }}</p>
          </body>
          </html>

    - name: Start Nginx in container
      shell: |
        pkill nginx || true
        nginx
      args:
        executable: /bin/bash

    - name: Verify web server responds
      uri:
        url: http://localhost
        status_code: 200
      register: web_result

    - name: Show verification result
      debug:
        msg: "Web server successfully running on {{ inventory_hostname }}!"
Tuloste:
PLAY [Install and configure Nginx web server] **************************************************************************

TASK [Update package cache] ********************************************************************************************
ok: [web1]

TASK [Install Nginx web server] ****************************************************************************************
changed: [web1]

TASK [Create custom index.html] ****************************************************************************************
changed: [web1]

TASK [Start Nginx in container] ****************************************************************************************
changed: [web1]

TASK [Verify web server responds] **************************************************************************************
ok: [web1]

TASK [Show verification result] ****************************************************************************************
ok: [web1] => {
    "msg": "Web server successfully running on web1!"
}

PLAY RECAP *************************************************************************************************************
web1                       : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

6 Kerää järjestelmätietoja
| Palvelin | Käyttöjärjestelmä | IP-osoite | Prosessorit | Muisti |
| :--- | :--- | :--- | :--- | :--- |
| **client1** | Ubuntu 24.04.4 LTS | 10.10.10.101 | 12 vCPU | 7802 MB |
| **db1** | Ubuntu 24.04.4 LTS | 10.10.20.102 | 12 vCPU | 7802 MB |
| **client** | Ubuntu 24.04.4 LTS | 10.10.30.101 | 12 vCPU | 7802 MB |
| **web1** | Ubuntu 24.04.4 LTS | 10.10.20.101 | 12 vCPU | 7802 MB |
| **attacker** | Ei ilmoitettu tulosteessa | 10.10.10.200 | 12 vCPU | 7802 MB |
| **r1** | Ubuntu 24.04.4 LTS | — | 12 vCPU | 7802 MB |
| **r2** | Ubuntu 24.04.4 LTS | — | 12 vCPU | 7802 MB |
| **r3** | Ubuntu 24.04.4 LTS | — | 12 vCPU | 7802 MB |

7 Analyysi
Mitä hyötyjä automaatiosta on
-	Idempotenssi  Ansiblen tehtävät on suunniteltu niin, että ne voidaan ajaa uudelleen turvallisesti. Jos järjestelmä on jo halutussa tilassa, Ansible ei tee turhia muutoksia, vaan ilmoittaa kaiken olevan kunnossa.
-	Skaalautuvuus: Oli hallittavana 1 tai 100 palvelinta, Ansible-komennon työmäärä on isännälle lähes sama.
-	Versiohallinta: Playbookeja voidaan hallita versionhallintajärjestelmissä (kuten Gitissä), mikä mahdollistaa muutoshistorian seurannan, palautukset vanhoihin versioihin ja tiimityöskentelyn.
Missä tilanteissa automaatio on välttämätöntä?
-	Suuret konesalit ja pilviympäristöt: Kun hallittavana on satoja tai tuhansia virtuaalipalvelimia, manuaalinen konfigurointi on mahdotonta resurssi- ja aikataulisyistä.
-	Dynaamiset ja nopeat ympäristöt (Auto-scaling / Disaster Recovery): Jos palvelin kaatuu tai kapasiteettia pitää skaalata hetkessä ylöspäin kuormituksen kasvaessa, uudet palvelimet on saatava tuotantokuntoon minuuteissa ilman ihmisen väliintuloa.

8 Yhteenveto
Mitä opin tehtävän aikana?
-	kirjoittamaan omia Ansible-playbookeja.
-	Opin käyttämään monipuolisesti Ansiblen moduuleja ohjelmistojen asentamiseen ja testaamiseen.
Miten Infrastructure as Code tukee nykyaikaista järjestelmähallintaa?
-	tekemällä palvelimien ja muiden järjestelmien hallinnasta automaattisempaa, nopeampaa ja yhdenmukaisempaa.
