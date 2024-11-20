# Database-Project
A MySQL database project done for a school course.

# Luolat ja lohikäärmeet -kerholle tietokanta 

Työryhmän nimi: Luolainsinöörit 
Jäsenet: Joni Oranne AC9766, Joonas Peltomäki AE6377 
Opintojaksotunnus: TTC2020-3037, Tietokannat 
Versionumero: 1.0 
Päiväys: 27.03.2024 

## Johdanto: 

Teemme Adventurer’s League -tyylisen D&D kerhon tietokannan, johon tallennetaan D&D kerhon historiaa: menneet pelisessiot, pelaajien hahmot, jäsenet, menneet ja nykyiset kamppanjat ja niiden Dungeon Masterit ja pelaajat.  
Halutaan tallentaa pelihahmon kaikki keskeiset tiedot eli race, class, taso, nimi, subclass. Tietokantaa käyttää kerhon johtajat ja Dungeon Masterit. Käyttöliittymä on mahdollisesti graafinen. 

Täytyy olla mahdollista kirjata session päivämäärä, paikalla olevat pelaajat, DM ja mitä kamppanjaa pelattiin. Päivitetään henkilöiden poistuessa tai liittyessä kerhoon, kun pelaaja liittyy kamppanjaan,  
pelaajan hahmo vaihtuu, jne. Voidaan päivittää kerhon johtajien ja Dungeon Mastereiden laiskuudesta riippuen aina kun jotain tapahtuu, tai esimerkiksi kerran viikossa. Päivityksiä tulee todennäköisesti 1-20 per viikko. 
  
Tavoitteena käytännössä on, että voidaan pitää kirjaa kaikista keskeisistä asioista kerhon sisällä. 

## Työnjako
Tietokannan rakenne (ER-kaavio) tehtiin yhdessä. Sen työtaakka jakaantuu puoliksi. Projektia työstettiin yhdessä, ja siihen meni noin 12 h.  
Keskeinen onnistuminen on se, että tietokannan rakenne saatiin melkein ensimmäisellä kerralla muistuttamaan lopputulosta, ja  
lisäksi forward engineering meni heti ensimmäisellä kerralla läpi. Toinen onnistuminen on se, että tietojen syötöt meni ensimmäisellä kerralla  
aina läpi. Kaikkiaan työskentely meni yllättävän sujuvasti ja tietokannan rakenne on melko optimaali. Hahmon tuhoamistriggeri pähkäiltiin yhdessä (fig 8).  
  
Huonona puolena se, että lähdekoodi on kommentoimatta. Toisaalta se on tehty forward engineerilla, ja SQL kielet on muutenkin melko helppolukuista.  

Joni 
Rooli: Tietokannan rakenteen hiomisen lisäksi keksin ratkaisun sille, miten hahmolla voi olla useampi classi.  
Täytin playertype, participant, character, character_has_class, race, class ja state -taulut.  
Kirjoitin loitsun campaignInfo (fig 3) näkymälle. Kirjoitin dokumentaatioon vaatimusmäärittelyn,  
selostuksen tietokannan toiminnan periaatteista, ratkaisujen perusteluista sekä kuvaukset näkymistä.   
  
Joonas  
Rooli: Suunnittelin tietokannan keskeisen campaign-participant-character-member-campaign loopin jonka kautta campaigneista saadaan samaan aikaan  
näkyviin niin dungeon master kuin kaikki pelaajat. Tauluista täytin member, campaign, session (58 riviä) ja session_has_participant (260 riviä).  
Tämän lisäksi tein seuraavat näkymät: members (fig 7), allCharacters (fig 4) ja sessions (fig 6).  

## Yleiskuvaus: 

Palvelun pyörittämiseen käytetään LabraNetin MariaDB -palvelinta. Käytetään MySQL Workbenchia luomaan tietokanta ja manipuloimaan sen sisältöä. 

Toiminnot 

Pakolliset: 

* Kerhon jäsenet (nimi, ikä, puh. numero, s.posti) 

* Henkilöiden pelihahmot ja niiden keskeiset tiedot 

* Onko henkilö pelaaja, dungeon master, kumpikin vai epäaktiivinen 

* Pelisessioiden kirjaus 

* Kamppanjan tiedot (nimi, kamppanjan Dungeon Master ja pelaajat) 

  

## Muut ominaisuudet 

Suorituskyky: Vasteajalla ei niin paljoa väliä, tietokantaa päivitetään enintään muutaman kerran viikossa. Tietokannan sisältämä data ei  koskaan kasva niin suureksi, että indeksointi olisi tarpeellista. 

## Tietokannan toteutus, toimivuus ja periaatteet 

Vaatimusmäärittelyn pohjalta kehitimme melko nopeastikin alla olevan ER-kaavion. Suuria muutoksia ei joutunut tekemään, muuta kuin muutamien auto incrementtien lisäys ja kirjoitusvirheiden korjausta sekä  
muutamien tietokenttien lisäys. Huomasimme myös, että moni-moneen suhde “member” ja “playertype” välissä on turha. Ajattelimme aluksi, että sitä tarvitsisi siinä tilanteessa, että jäsen on Dungeon Master  
ja pelaaja, mutta sen sijasta playertype tauluun voi laittaa suoraan neljä mahdollista tilaa (epäaktiivinen, pelaaja, dungeon master ja kumpikin). Kummastakin iteraatiosta on kuva alhaalla:  
 
Iteraatio 1 (fig 1) 
![Figure 1](pics/fig1)

Iteraatio 2 (fig 2)  
![Figure 2](./pics/fig1)

Tietokannassa on kaikki toiminnot, jotka listattiin vaatimusmäärittelyssä ja vähän lisääkin. 
Member- tauluun kirjataan kaikki kerhossa olevat jäsenet. Heidän “tila” saadaan playertype –taulusta. Yhdellä jäsenellä voi olla useampi pelihahmo (character), mutta ei ole pakko olla (ja hahmolla on pakko olla jäsen).  
Pelihahmo saa rodun race –taulusta. Dungeons and Dragons:ssa on mahdollista pelata useampaa classia yhdellä pelihahmolla, joten hahmon ja class taulun väliin tulee moni-moneen yhteys (yksi classi voi olla useammalla hahmolla).  
Välitauluun myös kirjataan yksittäisen classin taso (pelihahmon kokonaistaso voi olla esim. 6, jolloin hänellä voisi olla esimerkiksi 3 tasoa bard- ja wizard classissa).  
Class ja race taulun erottaminen helposti mahdollistaa sen, että halutessaan kerho voi lisätä sinne omia kotitekoisia virityksiä. Class:n ja race:n ID on yksinkertaisesti tekstipätkä. Esimerkiksi druid –class:n ID on Druid. 
Character taulussa on kaikki oleellisimmat tiedot hahmosta. Se ei toimi “oikean” character sheetin korvikkeena (jonne listattaisiin lisäksi esimerkiksi kestävyys, nopeus ja persoonallisuuteen liittyviä asioita).  
Nämä kirjataan pelaajan tottumuksien tai kerhon sääntöjen mukaan erillisesti digitaalisesti tai paperille, jolloin ne muutenkin pysyy tallessa.  
Campaign tauluun kirjataan kaikkien kamppanjoiden oleellisimmat tiedot. Nimi (esimerkiksi virallinen Wizards of the Coast:n moduuli “Curse of Strahd”), kuka on kamppanjan Dungeon Master,  
tila (aktiivinen, lopetettu, keskeytetty (saadaan state taulusta)), ja aloitus- sekä lopetuspäivät. Participant taulussa on listattu ne hahmot, jotka kuuluvat kamppanjaan.  
Taulussa on vain ID:tä, mutta sen avulla SQL loitsujen kautta saa näkymään jokaisen kamppanjan jokaisen hahmon seuraavasti:  

(fig 3)
![Figure 3](./pics/fig3)
  
Tässä koodi näkymän luontiin:  
```
SELECT campaignName, startDate, endDate, CONCAT(mdm.firstname, " ", mdm.lastname)  
AS dungeonMaster, "__", CONCAT(m.firstname, " ", m.lastname)  
AS playerName, characterName, characterLevel, classID AS "class", classLevel, subclass, raceID  

  

FROM `member` mdm 
INNER JOIN campaign c 
  ON mdm.memberID = c.dungeonMasterID 
INNER JOIN participant p 
  ON p.campaignID = c.campaignID 
INNER JOIN `character` ch 
  ON p.characterID = ch.characterID  
INNER JOIN character_has_class chc 
  ON ch.characterID = chc.characterID 
INNER JOIN `member` m 
  WHERE m.memberID = ch.memberID 
ORDER BY c.campaignID, ch.characterName, chc.classID; 
```
Tehtiin vielä näkymä, josta saa helposti selville jokaisen jäsenen jokaisen hahmon erikseen. Näkymä alla:
(fig 4) 
![Figure 4](./pics/fig4)

Tässä koodi näkymän luontiin: 
```
CREATE VIEW allCharacters AS 

SELECT concat(firstname, " ", lastname) AS player, characterName, characterLevel, classID AS class, classLevel, subclass, raceID  AS race, characterNotes 

FROM `character` c 

INNER JOIN character_has_class cc ON c.characterID = cc.characterID 

INNER JOIN `member` m ON c.memberID = m.memberID 

GROUP BY player, characterName, class; 
````
Sessiotauluun (session) kirjataan päiväys, jolloin on pidetty pelikerta. Session_has_participant taulussa on participantID (participant taulusta) ja sessionID,  
sekä luonnollisesti taulun oma ID. Session ja session_has_participant taulujen tarkoituksena on toimia tapana kirjata milloin sessio on pidetty, ketkä siihen osallistui,  
mitä kamppanjaa pelattiin ja millä hahmoilla (saadaan participant taulusta). Tehtiin tuosta seuraava näkymä:  

(fig 5)
![Figure 5](./pics/fig5)

Tässä koodi näkymän luontiin:  
```
CREATE VIEW sessions AS 
SELECT s.sessionID, `date`, campaignName, concat(dm.firstname, " ", dm.lastname) AS DM, concat(m.firstname, " ", m.lastname) AS player, characterName 
FROM `session` s 
INNER JOIN session_has_participant sp ON s.sessionID = sp.sessionID 
INNER JOIN participant p ON sp.participantID = p.participantID 
INNER JOIN `character` c ON p.characterID = c.characterID 
INNER JOIN `member` m ON c.memberID = m.memberID 
INNER JOIN campaign cmp ON p.campaignID = cmp.campaignID 
INNER JOIN `member` dm ON cmp.dungeonMasterID = dm.memberID 
GROUP BY `date`, characterName 
```
Ja viimeiseksi tehtiin näkymä, josta näkyy kaikki kerhon jäsenet ja heidän “tila”.  

(fig 6) 
![Figure 6](./pics/fig6)

Tässä koodi näkymän luontiin:  
```
CREATE VIEW members AS 
SELECT concat(firstname, " ", lastname) AS `member`, phoneNumber, eMail, birthday, playerTypeDesc AS `status`  
  FROM `member` m  
INNER JOIN playertype pt ON m.playerTypeID = pt.playerTypeID 
GROUP BY firstname; 
```
 

Kaikkien muiden taulujen ID on auto incrementillä, paitsi state, class ja race taulujen. State tauluun ei tule uutta tietoa eikä sitä päivitetä, joten se on käytännössä  
ihan sama onko auto increment vai ei. Race ja class taulujen ID:t ovat tekstiä, joten niissä ei edes voi olla auto incrementtiä. Muissa tauluissa halutaan aina säilyttää  
sinne jo kirjattu tieto tai muuttaa sitä, joten auto increment tekee tiedon kirjaamisesta yksinkertaisempaa ja mahdollistaa viiteavaimien toimivuuden ja sitä kautta myös  
näkymien toimivuuden. Unique constraint laitettiin member taulun sähköpostiin ja puh.numeroon sekä race ja class taulujen ID:hen, nämä tiedot kirjataan vain kerran.  
Samaa kamppanjaa, esimerkiksi Wizards of the Coast:n virallisia moduuleja pelataan mahdollisesti useamman kerran eri porukalla, joten kamppanjan nimi ei voi olla uniikki.  
Lisäksi samana päivänä voi olla useampi sessio, ja koska tarkkaa kellonaikaa ei kirjata, session päivämäärä ei myöskään voi olla uniikki.  

Lisää indeksointeja ei tarvitse tehdä, koska meillä on ollut päällä automaattinen indeksointi pääavaimille sekä uniikeille avaimille.  
MySQL 8.0 päivitää nämä indeksit automattisesti, ainakin sen dokumentoinnin mukaan. Eli teoreettisesti jos kerho kasvaisikin suureksi, memberID, sessionID, ja characterID sekä  
muut pääavaimet indeksoituisi ja päivittyisi automaattisesti. Tämä nopeuttaa tietokannan toimintaa suuresti, eikä sen manuaalisesta päivittämisestä tarvitse huolehtia.  
Lisäksi laitoimme puhelinnumeron ja s.postin uniikeiksi, joten nekin indeksoituu samalla tavalla automaattisesti.  

 

## Character_before_delete herätin 

Tietokannassa hahmo esiintyy kolmessa muussa taulussa alkuperäisen lisäksi, joten loimme triggerin, joka mahdollistaa koko hahmon pyyhkimisen kannasta yhdellä komennolla.  
Alla triggerin luontikoodi ja demonstraatio  
```
DELIMITER $$ 

CREATE TRIGGER `character_before_delete` 

BEFORE DELETE ON `character` 

FOR EACH ROW 

BEGIN 

    DELETE FROM character_has_class WHERE characterID = OLD.characterID; 

    DELETE FROM session_has_participant WHERE participantID IN ( 

        SELECT participantID FROM participant 

        WHERE characterID = OLD.characterID); 

    DELETE FROM participant WHERE characterID = OLD.characterID; 

END $$ 

DELIMITER ; 
```
Herättimen demonstraatio (fig 8) 
![Figure 7a](./pics/fig7a)
![Figure 7b](./pics/fig7b)
![Figure 7c](./pics/fig7c)
![Figure 7d](./pics/fig7d)
![Figure 7e](./pics/fig7e)
![Figure 7f](./pics/fig7f)
![Figure 7g](./pics/fig7g)
![Figure 7h](./pics/fig7h)
![Figure 7i](./pics/fig7i)
Hahmo on onnistuneesti pyyhitty kannasta. 
