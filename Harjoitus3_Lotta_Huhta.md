# Harjoitus3_Lotta_Huhta_Palvelintenhallinta
## 14:45 B)
Git:n ja GitHubin käytössä hyödynnetty ohjetta: https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners

Muokkasin varastoa tekemällä sinne kaksi tiedostoa. Toinen niistä on nimeltään GitTest.txt ja toinen on HelloGit.txt. Tein nämä tiedostot Git-varastooni ajamalla komennot:
```
touch GitTest.txt
touch HelloGit.txt
```
![kuva](https://user-images.githubusercontent.com/82219338/141688270-1bf3ef41-747f-4667-9712-2d3f27445b33.png)

Siirsin tiedostot ”staging environment”:iin ajamalla komennot:
```
git add GitTest.txt
git add HelloGit.txt
```
Tarkistin tehtävät muutokset ajamalla komennon 
```
git status
```
![kuva](https://user-images.githubusercontent.com/82219338/141688364-a51cf8ef-084b-4497-8ee1-2d352053eaad.png)

Vahvistin muutokset ajamalla komennon:
```
git commit -m ”Add file GitTest.txt and HelloGit.txt to repository”
```
Tämän jälkeen lisäsin vielä kolmannen tiedoston nimeltään lotta.txt.
Ajoin komennot:
```
touch lotta.txt
git add lotta.txt
git commit -m ”Add file lotta.txt to repository”
```
![kuva](https://user-images.githubusercontent.com/82219338/141688553-a5f30675-c256-4d0e-8cfa-288283c829d0.png)

Sen jälkeen tein muutoksia tiedostoihin GitTest.txt ja lotta.txt. Ajoin komennot:
```
nano lotta.txt
```
Lisäsin tekstin ”lotta huhta”
```
nano GitTest.txt
```
Lisäsin tekstin ”olen git testi”
Seuraavaksi päivitin nämä muutokset Git repositoryyn ajamalla komennot:
```
git add lotta.txt
git add GitTest.txt
git status
git commit -m ”update files lotta.txt and GitTest.txt with test information”
```
Git varastoon päivitettiin kaksi tiedostoa:

![kuva](https://user-images.githubusercontent.com/82219338/141688747-23dfae43-50a3-46e6-b836-6344da9c420c.png)

# 15:30 B)
Ajoin komennon:
```
git log
```
![kuva](https://user-images.githubusercontent.com/82219338/141688822-aab02c9d-5ff5-4bb7-b1ac-ed10520669f6.png)

Komento näyttää kaikki muutokset, jotka on tehty Git varastoon, kenen toimesta ja milloin. Se myös näyttää commit tekstin.
git diff komennon kanssa apuja saatu: https://careerkarma.com/blog/git-diff/
Ajoin komennon:
```
git diff e914 04031
```
Tuotti esityksen muutoksista toisen ja viimeisimmän muutoksen välillä:
![kuva](https://user-images.githubusercontent.com/82219338/141688843-26c32070-2529-4f7d-8f5e-1b7da038dbfe.png)
 
Komento näyttää että muutoksia GitTest.txt tiedostoon on tehty ja muutoksia lotta.txt tiedostoon on tehty.
Ajoin komennon:
```
git blame lotta.txt
```
Komento näyttää tiedostoon tehdyt muutokset
![kuva](https://user-images.githubusercontent.com/82219338/141688855-77d36b8e-1418-4d18-b031-ea5e6decca5c.png)

Kuvasta näkyy muutoksen Id:n alkuosa, kuka on tehnyt muutoksen ja milloin ja mikä muutos on tehty.

# 16:30 C)
Tein tyhmän muutoksen tiedostoon HelloGit.txt. 
Ajoin komennot:
```
nano HelloGit.txt
```
  kirjoitin tiedostoon ”tyhma muutos”
```
git add HelloGit.txt
git status
``` 
![kuva](https://user-images.githubusercontent.com/82219338/141688975-b9d3205e-9bfe-4530-85e9-6545f0f07138.png)

Poistin muutokset ajamalla komennot:
```
git reset --hard
git status
```
![kuva](https://user-images.githubusercontent.com/82219338/141688988-4dde45e8-6124-4556-8519-dc90e7f935c7.png)

# 16:45 D)
Tein tilatiedoston, jolla varmistetaan apache2 asennus ja päällä olon

Ajoin komennon:
```
sudoedit /srv/salt/apache.sls
```
Kirjoitin apache.sls -tiedostoon seuraavaa:
```
install apache2:
  pkg.installed:
    - name: apache2
run apache2:
  service.running:
    - name: apache2
```
# Valmis 17:00

# Z
```
Artikkeli: https://commonmark.org/help/
Markdown on tapa muotoilla teksti sopivaksi kaikille laitteille.

Kirjoittamalla:
kahden * merkin väliin sana muuttuu kursiiviksi
** merkkien väliin sana tummenttuu
# merkin jälkeen tulee otsikko
## merkkien jälkeen tulee otsikko2
kirjoittamalla [Link](URL) muodostuu linkki
kirjoittamalla ![Image](URL) muodostuu kuva
> merkin jälkeen muodostuu lohkoteksti
* merkin jälkeen muodostuu lista
1. jälkeen muodostuu numero lista
--- muodostuu vaakaviiva
Kahden ` merkin väliin muodostuu korostettu koodi
``` merkkien väliin muodostuu koodilohko

```
