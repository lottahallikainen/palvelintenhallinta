# Harjoitus3_Lotta_Huhta_Palvelintenhallinta
## B)
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
