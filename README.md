# h6-miniproject
Dockerisoitu tietokantajärjestelmä, joka automaattisesti määrittää ja suorittaa käyttövalmiin ympäristön yhdellä komennolla.

tähän kuvakaappaus toimivasta jutskelista

# Projektin tarkoitus
Projektin lähtökohtana oli kokeilla uutta demonia, jotai ei vielä tunnilla ole opeteltu. Päädyimme kokeilemaan dockeroitua tietokanta järjestelmää, sillä se antoi tarpeeksi haastetta.  

Tämän projektin valmistuttua tulisi saada käyttöön automatisoitu täysin valmis tietokantaympäristö, jonka saisi pystyyn pelkästään ajamalla ansiblen playbook komento.  
Hyödynsimme ansiblen lisäksi Dockeria, joka paketoi tarvitsemamme sovellukset ja niiden kirjastot konteiksi. Tämä varmistaa tietokannan toiminnan muillakin laitteilla, eikä vain omassa testiympäristössämme.
Toinen hyödyntämämme palvelu on PostgreSQL, joka on yksi suosituimmista avoimen lähdekoodin tietokantajärjestelmistä.
Nämä yhdistämällä saamme yksinkertaisen asennuksen tietokannalle, joka varmasti toimii ilman suurempia konfiguraatioita.  

Ansible osaa tarkistaa, onko tietokanta pystytetty onnistuneesti, eikä käynnistä sitä uudestaan jos palvelu on jo päällä.
Kun playbook ajetaan, saadaan nopeasti pystyyn tallennusvalmis tietokanta noudattaen infrastructure as code periaatetta.

# Tech Stack
Ansible  
Docker  
Docker Compose  
PostgreSQL
Python3  

# Projektiin alkupiste  

Projektin alkuvaiheessa luotiin ansiblelle tarvittavat roolit sekä taskit niiden sisälle. Neuvoa otettiin aikaisemmista tunnilla tehdyistä tehtävistä.  
Virhetilanteissa konsultoimme tekoälyä, ja yleensä kyse oli vain syntaksi virheistä, kun roolien sisällä puuttui välilyönti, tai alaviiva.  

# Projektin Rakenne
~~~
./
├── ansible.cfg
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.ini
├── LICENSE
├── README.md
├── roles/
│   ├── database/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── docker-compose.yml.j2
│   ├── docker/
│   │   └── tasks/
│   │       └── main.yml
│   └── testuser/
│       └── tasks/
│           └── main.yml
└── site.yml
~~~


# Käyttöönotto
Ennen käyttöönottoa pitää olla yksityinen SSH-avain githubin avaimiin lisättynä.  
Käyttöönottoon riittää projektin kloonaaminen ja käynnistäminen:
~~~
git clone git@github.com:JoonasKaarni/h6-miniproject.git
cd h6-miniproject
# tähän käynnistyskomento
~~~


# Idempotentti  

<img width="1879" height="219" alt="image" src="https://github.com/user-attachments/assets/65e3b792-076c-40dd-aa2c-74bd1f72a974" />  


# Docker-konfiguraatio
## Docker-rooli
Ensiksi on dockeri roolin konfiguraatio. Tämä rooli huolehtii siitä, että palvelimella on valmiudet käyttää Dockeria ja että käyttäjä pystyy ajamaan kontteja. Ensin pakettien asennus tapahtuu eli docker ja docker-compose asennetaan ja sitten docker-palvelu käynnistetään.
~~~
- name: Install Docker
  apt:
    name:
      - docker.io
      - docker-compose
    state: present
~~~
Docker-palvelu käynnistetään ja varmistetaan, että se käynnistyy automaattisesti koneen uudelleenkäynnistyksen jälkeen.
~~~
- name: Start Docker
  service:
    name: docker
    state: started
    enabled: yes
~~~
Sitten tapahtuu käyttäjän lisääminen docker-ryhmään. Tämä siis lisää nykyisen käyttäjän docker-ryhmään. Mikäli tätä ei tehdä docker tarvitsee root-oikeudet. Tämän avulla käyttäjä voi siis ajaa dockeria ilman sudoa.
~~~
- name: Add user
  user:
    name: "user"
    groups: docker
    append: yes
~~~
## Database-rooli
Sitten on luvassa database-rooli. Tämä rooli valmistaa sovelluksen hakemiston ja käynnistää koko ympäristön docker-compose -tiedoston avulla. Aluksi luodaan hakemisto, jonne tullaan sijoittamaan docker-compose -konfiguraatio ja mahdoolliset muut tiedostot.
~~~
- name: Create app directory
  file:
    path: /opt/app
    state: directory
~~~
Seuraava kopioidaan tai generoidaan docker-compose.yml -tiedosto palvelimelle. Tämän avulla docker-compose määrittelee tietokannan, sovelluksen ja mahdolliset verkot.
~~~
- name: Copy docker-compose
  template:
    src: docker-compose.yml.j2
    dest: /opt/app/docker-compose.yml
~~~
Lopuksi käynnistetään kontit. Tässä siis käytetään Ansible-moduulia community.docker.docker_compose_v2, joka ajaa docker--compose -projektin hakemistossa. Se varmistaa, että määritellyt kontit ovat käynnissä.
~~~
- name: Run docker-compose
  community.docker.docker_compose_v2:
    project_src: /opt/app
    state: present

~~~

# Muut konfiguraatiot  

Puurakenteessa näkyvä group_vars hakemisto toimii yleisenä "tietopankkina". Sieltä muut roolit hakevat tarvittavat tietonsa, esimerkiksi database rooli hakee täältä tarvitsemansa tietokannan nimen ja salasanan. Testuser rooli taas hakee tiedon siitä, mihin hakemistoon tietokanta tallennetaan.  

Playbookkia ajaessa ansible suorittaa toimintansa tässä järjestyksessä:
1. Lukee hosts.ini tiedoston, ja määrittää mitä orjia nyt hallitaan. (Tässä tapauksessa localhost).
2. Lataa group_vars hakemistosta all.yml tiedoston, ja lataa siitä tarvittavat muuttujat muistiin.
3. Suorittaa docker roolin, eli asentaa dockerin jos ei vielä ole.
4. Suorittaa database roolin. Tässä vaiheessa liitetään all.ymlistä saadut parametrit databasen pohjaan.
5. Lopullinen kontti siirretään palvelimelle, ja tietokanta on valmis.  

<img width="1651" height="361" alt="Näyttökuva 2026-05-04 174535" src="https://github.com/user-attachments/assets/96ad52a6-f540-460b-b957-51294ae25a48" />  



# Käyttö ja testaus
Playbookin ajon oltua idempotentti, on syytä testata toimiiko dockeroitu tietokanta järjestelmä. Tämä tapahtuu kirjautumalla tietokantaan annetuilla tunnuksilla.  
Tunnukset testaamisen vuoksi pidettiin yksinkertaisina, tietokannan host on "localhost", käyttäjätunnus on "opiskelija" sekä tietokannan nimi on "koulu".  

Tietokantaan päästään kirjautumaan PostgreSQL omalla komennolla "psql -h localhost -U opiskelija -d koulu".  

<img width="924" height="208" alt="Näyttökuva 2026-05-04 145938" src="https://github.com/user-attachments/assets/e99d45eb-c5c8-4c78-874e-3b741edb5feb" />  

Tämä todistaa, että playbookin ajo onnistuneesti asensi dockerin, konfiguroi käyttäjän sekä käynnisti tietokantapalvelun johon voidaan nyt ottaa yhteys.
Tietokannan toimimista voidaan vielä testata luomalla taulu, lisäämällä siihen tietoa ja hakemalla tietoa.
Tämä tapahtuu SQL komennoilla seuraavasti:
1. Taulun luonti = "CREATE TABLE test_table (id SERIAL PRIMARY KEY, viesti TEXT);"
2. Tiedon lisääminen tauluun = "INSERT INTO test_table (viesti) VALUES ('Ansiblen luoma dockerointi ja tietokanta järjestelmä toimii!');"
3. Tiedon haku = "SELECT * FROM test_table;"
<img width="955" height="615" alt="Näyttökuva 2026-05-04 150924" src="https://github.com/user-attachments/assets/24b14a60-995e-47ae-92bc-2f0b52151188" />

SQL komennoissa on tärkeää muistaa, että komennot päättyvät aina puolipisteeseen, muuten järjestelmä luulee edellisen komennon jatkuvan edelleen johtaen syntaksi virheisiin. 

# Lisenssi
GNU General Public License v3.0


# Tekijät
Aleksi Kallio  
Joonas Kaarni


# Demo
tähän vois olla ennen esitystä tallennettu esimerkki siitä, miten homma toimii.

