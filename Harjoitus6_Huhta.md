# Harjoitus 6 
# A)

Hankin tietoja Windowsista ajamalla komennon:
```
sudo salt 'winmin' grains.items
```
![kuva](https://user-images.githubusercontent.com/82219338/145057872-3439a5d3-ba38-45b0-8cde-626ba10cb6d5.png)

Windowsista oleellisia tietoja ovat:
-	Host (tierokoneen nimi)
-	ip_interfaces (ip osoitteet)
-	os (käyttöjärjestelmä)
-	osrelease (käyttöjäjestelmäversio)
-	mem_total (ram-muistin määrä)
-	cpuarch (prosessorin tyyppi)
-	systempath (järjestelmätiedoston polkuja)

# B)

Noudatettu artikkelia: https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html

Asensin seuraavat ohjelmat Windowsiin saltilla:
-	Chrome
-	Firefox
-	VLC
-	
Asensin nämä komennolla:
```
	sudo salt 'winmin' pkg.install chrome
	sudo salt 'winmin' pkg.install firefox_x64
	sudo salt 'winmin' pkg.install vlc
```
Asennukset onnistui:

![kuva](https://user-images.githubusercontent.com/82219338/145058216-f15120cb-c68b-4d83-86be-4542c4133089.png)

# C)

Käytetyt  ohjeet:
https://ubuntu.com/tutorials/install-and-configure-samba#2-installing-samba
https://newbedev.com/automatically-enter-input-in-command-line
https://superuser.com/questions/271034/list-samba-users
salt dokumentaatio

Miniprojektini ensimmäisessä osassa saltilla otetaan käyttöön paikallisen verkon tiedostopalvelin Samba, jolla mahdollistetaan tiedostojen jakaminen sisäverkossa.
Tein tilatiedoston samba.sls:
```
create_samba_user:
  user.present:
    - name: samba
    - password: samba
    - hash_password: True
install_samba:
  pkg.installed:
    - name: samba
  file.directory:
    - name: /home/samba/sambashare/
    - force: True
allow_samba:
  cmd.run:
    - name: ufw allow samba
    - unless:
      - cat /etc/ufw/user.rules | grep 'dapp_Samba' 
create_sambapassword:
  cmd.run:
    - name: yes samba|sudo smbpasswd -a samba
    - unless:
       - pdbedit -W -L|grep 'samba'
conf_samba:
  file.managed: 
    - name: /etc/samba/smb.conf
    - source: salt://samba/smb.conf
  service.running:
    - name: smbd
    - watch:
      - file: /etc/samba/smb.conf
```

Ehto palomuurikomentoon on saatu tarkastelemalla tiedostoja user.rules ja user6.rules joihin komento ”sudo ufw allow samba” tekee muutoksia:


![kuva](https://user-images.githubusercontent.com/82219338/145058612-5240a2e3-506d-4e9e-92de-de8e3f9315ce.png)


Tiedostoja tarkastelemalla huomataan, että alla olevat rivit lisätään niihin:
```
user.rules:

### tuple ### allow udp 137,138 0.0.0.0/0 any 0.0.0.0/0 Samba - in
-A ufw-user-input -p udp -m multiport --dports 137,138 -j ACCEPT -m comment --comment 'dapp_Samba'

### tuple ### allow tcp 139,445 0.0.0.0/0 any 0.0.0.0/0 Samba - in
-A ufw-user-input -p tcp -m multiport --dports 139,445 -j ACCEPT -m comment --comment 'dapp_Samba'

user6.rules:

### tuple ### allow udp 137,138 0.0.0.0/0 any 0.0.0.0/0 Samba - in
-A ufw-user-input -p udp -m multiport --dports 137,138 -j ACCEPT -m comment --comment 'dapp_Samba'

### tuple ### allow tcp 139,445 0.0.0.0/0 any 0.0.0.0/0 Samba - in
-A ufw-user-input -p tcp -m multiport --dports 139,445 -j ACCEPT -m comment --comment 'dapp_Samba'
```
/srv/salt/samba/smb.conf tiedosto on vakio smb.conf tiedosto, jonka perään on lisätty rivit:
```
[sambashare]
    comment = Local file share
    path = /home/samba/sambashare
    read only = no
    browsable = yes
```
Ajoin tilan komennolla:
```
sudo salt 'ubuntumin' state.apply samba
```
Tila ajettiin onnistuneesti:

![kuva](https://user-images.githubusercontent.com/82219338/145059119-5d7dec65-36a9-415f-ad3f-9a69f02af049.png)

ja uudelleen ajo ei aiheuttanut muutoksia uudelleen:

![kuva](https://user-images.githubusercontent.com/82219338/145059217-68e826b5-0a51-4884-9812-c964c68b2f60.png)

Kokeilin palvelimen toimivuutta kolmannelta virtuaalikoneelta:

![kuva](https://user-images.githubusercontent.com/82219338/145059294-250e3503-08cd-4b95-9a49-84402685144a.png)

![kuva](https://user-images.githubusercontent.com/82219338/145059342-6ec620fe-d300-418d-bdd1-f2cfb27d1c3a.png)

![kuva](https://user-images.githubusercontent.com/82219338/145059381-887dd808-1f03-4482-89db-df479fcdab20.png)

Onnistui!

# D)

Miniprojektin sivusto:
https://github.com/lottahallikainen/miniprojekti/blob/main/README.md


# Z)

Artikkeli: https://terokarvinen.com/2018/control-windows-with-salt/

-	Saltilla voidaan hallita myös Windowsia.
-	Huomioitavaa on, että mestarin versio saltista on sama tai uudempi kuin Windows-orjan.
-	Pakettien asennus vaatii hieman työtä:
```
$ sudo mkdir /srv/salt/win
$ sudo chown root.salt /srv/salt/win
$ sudo chmod ug+rwx /srv/salt/win
$ sudo salt-run winrepo.update_git_repos
$ sudo salt -G 'os:windows' pkg.refresh_db
```
-	Sen jälkeen ohjelmien asentaminen onnistuu komennolla:
```
sudo salt '*' pkg.install vlc
```
Artikkeli: https://terokarvinen.com/2018/configure-windows-and-linux-with-salt-jinja-if-else-and-grains/

-	Ehtorakenteella on mahdollista ajaa sama tila eri käyttöjärjestelmille.
-	Ehtolauseiden on oltava sls-tiedoston alussa.
-	Huomioitavaa on, että alustoilla on oltava yhteisiä rekenteita, jotta se voisi onnistua.
-	esimerkki ehtorakenteesta:
```
{% if "Windows" == grains["os"] %}
{%      set hellofile = "C:\hellotero.txt" %}
{% else %}
{%      set hellofile = "/tmp/hellotero.txt" %}
{% endif %}
{{ hellofile }}:
 file.managed:
 - source: salt://hello/hellotero.txt
```
Artikkeli:  https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html

-	Windows package manager mahdollitaa Windowsilla toiminnan, joka on mahdollista Linuxilla yum- ja apt -toiminnallisuuksilla.
-	Merkittävänä erona verattuna yum- ja apt -toiminnallisuuksiin on se, ettei tällä ole mahdollista hallinnoida sovelluksen tarvitsemia riippuuvuuksia. 
-	Tulee myös huomioida, että ladattavat palvelut haetaan salt stackin tietovarannoista.
-	Saatavilla olevien pakettien päivittäminen tapahtuu komennolla:
salt-run winrepo.update_git_repos
-	Windows-minioneille tietokanta päivitetään komennoilla (masterilta):
```
salt -G 'os:windows' pkg.refresh_db
```








