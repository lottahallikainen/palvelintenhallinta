# Harjoitus 4 Lotta Huhta
## A)
Aloitettu 10:45

Tein tilan, joka asensi seuraavat ohjelmat:
1.	apache2
2.	GIMP
3.	Synaptic package manager
4.	Gnome tweak tool
5.	Vim
6.	Pidgin
7.	Kontact
8.	Terminator
9.	Thonny
10.	Atril document viewer

Tein tilatiedoston nimeltään musthave.sls ajamalla komennon:
```
sudoedit /srv/salt/musthave.sls
```
Ja kirjoitin sinne (syntaksi haettu osoitteesta https://docs.saltproject.io/en/latest/ref/states/all/salt.states.pkg.html#salt.states.pkg.installed ):

```
musthave_packages:
  pkg.installed:
    - pkgs:
      - apache2
      - gimp
      - synaptic
      - gnome-tweak-tool
      - vim
      - pidgin
      - kontact
      - terminator
      - thonny
      - atril
```
Ajoin tilan komennolla:

```
sudo salt-call --local -l info state.apply musthave
```
Varmistin ohjelmien asentumisen pistotarkastusmaisesti tarkastamalla, että yksi asentui:

![kuva](https://user-images.githubusercontent.com/82219338/143055497-f2e82bb5-a13f-4863-a102-b79cfd3c7ab3.png)

11:45

## B)
Ajoin komennot:
```
cd /etc/ 
sudo find -printf '%T+ %p\n'|sort|tail
```
![kuva](https://user-images.githubusercontent.com/82219338/143055937-57d6f45d-9998-4ac0-bdf4-fa27461b0e4a.png)

cd /etc/ - vaihtaa työhakemiston etc-hakemistoon

sudo find – ajetaan komento find superuserina, 

find-komento etsii tiedostoja

-printf – tulostaa tietyssä muodossa

'%T+ %p\n' – määrittää printf:lle muodon seuraavasti:

* %T+ - esittää tiedoton viimeisimmän muokkausajan mudossa päivämäärä + aika
* %p -tulostaa tiedoston nimen
* \n -lisää rivinvaihdon
* | - siirtää edellisen komennon tulosteet seuraavalle komennolle
* sort – järjestää tekstiä
* tail – tulostaa viimeiset kymmenen riviä

Ajoin /etc/ kansiossa komennon:

```
	sudo apt-get update
	sudo apt-get upgrade
```
Jonka jälkeen ajoin komennon:

```
sudo find -printf '%T+ %p\n'|sort|tail
```
Muutokset (5 kpl) löytyivät aikajanalta:

![kuva](https://user-images.githubusercontent.com/82219338/143057547-17337127-a1aa-46f1-a73b-f43f74e693e9.png)

Suurinpiirtein samalla ajankohdalla on muutettu viisi tiedostoa.

12:30

## C)
Otin Apache2:ssa käyttöön ssl konfiguraation ajamalla komennon (mukailen sivustoa https://hallard.me/enable-ssl-for-apache-server-in-5-minutes/) :

```
	sudo a2enmod ssl
	systemctl reload apache2
```
Jonka jälkeen ssl sivusto toimi:

![kuva](https://user-images.githubusercontent.com/82219338/143058167-6bd0a678-28ca-49df-9da5-c71d8860cfb1.png)

Muutokset löytyivät komennolla:

```
sudo find -printf '%T+ %p\n'|sort|tail
```
![kuva](https://user-images.githubusercontent.com/82219338/143058420-21d10b98-4ee7-4589-969c-f6f0c3dfdbe5.png)

Kun tarkastellaan kansiota, jonne muutos on tehty komennolla:

```
ls /etc/apache2/mods-enabled/
```
Huomataan että tiedostot ssl.conf ja ssl.load ovat symbolisia linkkejä (turkoosi väritys):

![kuva](https://user-images.githubusercontent.com/82219338/143058657-1af98996-8c62-4fb7-9095-49ba47518660.png)

Linkkien osoitteiden polut saadaan selville komennolla (https://stackoverflow.com/questions/16017500/how-to-see-full-absolute-path-of-a-symlink):

```
	readlink -f /etc/apache2/mods-enabled/ssl.conf
	readlink -f /etc/apache2/mods-enabled/ssl.load
```
Tila jolla saadaan ssl-sivusto aktivoitua näyttää seuraavalta (oletuksena, että ssl on jo konfiguroitu käyttöön)(Syntaksi osoitteesta https://docs.saltproject.io/en/latest/ref/states/all/salt.states.file.html#salt.states.file.symlink 
ja https://docs.saltproject.io/en/latest/ref/states/requisites.html#watch):

```
take_ssl_conf_in_use:
  file.symlink:
    - name: /etc/apache2/mods-enabled/ssl.conf
    - target: /etc/apache2/mods-available/ssl.conf
    - force: False
take_sslload_in_use:
  file.symlink:
    - name: /etc/apache2/mods-enabled/ssl.load
    - target: /etc/apache2/mods-available/ssl.load
    - force: False
reload_apache2:
  service.running:
    - name: apache2
    - reload: True
    - watch:
      - file: /etc/apache2/mods-enabled/*
```
Kirjoitin yllä olevan tiedostoon komennolla:

```
sudoedit /srv/salt/sslconfinuse.sls
```
Jotta pystyn testaamaan tilan toimivuuden, poistin ssl-sivuston käytöstä komennolla (https://devdojo.com/serverenthusiast/14-apache-commands-to-help-you-manage-your-server-like-a-pro):

```
	sudo a2dismod ssl
	systemctl reload apache2
```
![kuva](https://user-images.githubusercontent.com/82219338/143059242-fb29418f-ac79-4bcc-a955-e30f0c2a52d8.png)

Otin tilan käyttöön komennolla:
```
sudo salt-call --local -l info state.apply sslconfinuse
```
![kuva](https://user-images.githubusercontent.com/82219338/143059407-7b8c5b83-e71a-4e1c-94e3-5f7a53871684.png)

Toimii:

![kuva](https://user-images.githubusercontent.com/82219338/143059466-6bccd4c7-f8ae-4c4f-8b82-f41c51ec846b.png)

15:50

## D)

Tehtävää tehdessäni kokeilin ensiksi asentaa ja tehdä muutoksia ohjelmiin thunderbird ja clamAV, mutta thunderbirdin osalta muutokset,
joita asetuksiin teki graafisessa käyttöliittymässä ei löytynyt mistään tiedostoista, joten jätin sen osaltani.
ClamAV:tä asennettaessa manuaalisesti ilmeni jo ongelmia (virustunniste tietojen lataaminen komentokehotteen avulla ei onnistunut 
ja ohjelman omien komentojen kautta ei suorittanut tunnistetietojen asennusta loppuun epäonnistuneiden testien takia), 
joihin ei internetistä löytynyt vastauksia joten päädyin uudestaan tekemään muutoksia Apache2 sovellukseen.

Asensin Apache2:n uudestaan alustaen siihen suoraan halutun sivuston.

Tein tiedoston tilatiedoston initializehomepage.sls

```
install_apache2:
  pkg.installed:
    - name: apache2
make_my_webpage:
  file.managed:
    - name: /var/www/html/index.html
    - contents: 
      - <!DOCTYPE html>
      - <html>
      - <head>
      - <meta charset="UTF-8">
      - <title>Lotan Sivu</title>
      - </head>
      - <body>
      - <h3>Lotan Sivu</h3>
      - <P> Tämä on Lotan palvelimen ylläpitämä sivu </P>
      - </body>
      - </html>
ensure_apache2_is_running:
  service.running:
    - name: apache2
```

kirjoitin tiedoston komennolla:

```
sudoedit srv/salt/initializehomepage.sls
```
Tehtävän todentamista varten poistin apache2:n asennuksen 
(noudattaen artikkelia https://www.codegrepper.com/code-examples/shell/debian+10+uninstall+apache2+safe+%3F).

```
	sudo systemctl stop apache2
	sudo apt autoremove
	sudo apt remove apache2
	sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
```
Apache2 poistettu.

![kuva](https://user-images.githubusercontent.com/82219338/143060552-9d0b1127-9c06-456e-a410-c738c2552c2d.png)

Ajoin tilan ja automaattisen asennuksen komennolla:

```
sudo salt-call --local -l info state.apply initializehomepage
```

![kuva](https://user-images.githubusercontent.com/82219338/143060735-19ecd536-994e-4505-bae4-204db40c35ad.png)

Toimii:

![kuva](https://user-images.githubusercontent.com/82219338/143060819-89f4159b-8f1a-4e62-9657-a032982fe3a2.png)

Tehtävä valmis 20:00















