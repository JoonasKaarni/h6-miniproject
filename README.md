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


# Käyttöönotto
Ennen käyttöönottoa pitää olla yksityinen SSH-avain githubin avaimiin lisättynä.  
Käyttöönottoon riittää projektin kloonaaminen ja käynnistäminen:
~~~
git clone git@github.com:JoonasKaarni/h6-miniproject.git
cd h6-miniproject
# tähän käynnistyskomento
~~~


# Idempotenssi
idempotenssin todistus tänne


# Projektin Rakenne
varmaan tähän kuva rakenteesta tree -F, voi olla myös noilla komentorivijutskeleilla


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


# Docker-konfiguraatio
tähän kuva tai tiedoston sisältö kopsattu


# Lisenssi
GNU General Public License v3.0


# Tekijät
Aleksi Kallio  
Joonas Kaarni


# Demo
tähän vois olla ennen esitystä tallennettu esimerkki siitä, miten homma toimii.

