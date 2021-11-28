# Harjoitus 5
Aloitettu 11:00
# a)
Tehtävää tehdessä hyödynnetty artikkeleita:
https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server (Avaimien luonti ja siirto)
https://terokarvinen.com/2018/pkg-file-service-control-daemons-with-salt-change-ssh-server-port/?fromSearch=salt%20service ja Salt:n dokumentaatiota (tilatiedostojen tekeminen)

SSH-palvelin oli asennettuna valmiiksi masterille. Manuaalisen asentelun yhteydessä havaittu, ettei Vagrantilla SSH yhteys koneiden välillä toimi suoraan vaan masterilla
tulee luoda avainpari ja julkinen avain siirtää orjille.

Loin masterilla avainparin komennolla:
```
ssh-keygen
```
![kuva](https://user-images.githubusercontent.com/82219338/143773595-7f05afa5-e640-4c55-a7ad-57c9f009a61f.png)

Ajoin komennon:
```
sudo mkdir /srv/salt/sshd/
```
Siirsin luodun julkisen avaimen ylläolevaan kansioon komennolla:
```
sudo cp ~/.ssh/id_rsa.pub /srv/salt/sshd/id_rsa.pub
```
Kävi muuttamassa ssh-palvelimen konfiguraatiotiedoston asetuksia, asettamalla toisen portin 1025 ajamalla alla olevan komennon:
```
sudoedit /etc/ssh/sshd_config
```
Ja tekemällä muutoksen:

![kuva](https://user-images.githubusercontent.com/82219338/143773686-0752183e-aea4-4a7a-8c3a-38b8e3e1cfdc.png)

Tämän jälkeen kopioin ssh-palvelimen konfiguraatiotiedoston kansioon /srv/salt/sshd/
```
sudo cp /etc/ssh/sshd_config /srv/salt/sshd/sshd_config
```
Loin SSH-palvelimen asentamiseen ja alustamiseen sls-tiedoston komennolla:
```
sudoedit /srv/salt/sshd.sls
```
Kirjoitin sshd.sls tiedostoon seuraavaa:
```
openssh-server:
  pkg.installed
/home/vagrant/.ssh/authorized_keys:
  file.append:
    - source: salt://sshd/id_rsa.pub 
/etc/ssh/sshd_config:
  file.managed:
    - source: salt://sshd/sshd_config
sshd:
  service.running:
    - watch:
      - file: /etc/ssh/sshd_config
    - onchanges:
      - file: /etc/ssh/sshd_config
```
Tämän jälkeen ajoin tilan komennolla:
```
sudo salt '*' state.apply sshd
```
Tila otetttu käyttöön:

![kuva](https://user-images.githubusercontent.com/82219338/143773805-8d72506c-8f1e-46b2-bc33-129e8c958af2.png)

Kokeillin otta ssh yhteyden molempiin orjiin:

![kuva](https://user-images.githubusercontent.com/82219338/143773828-9c70a34b-666b-462c-9f00-90b6fe880b43.png)

![kuva](https://user-images.githubusercontent.com/82219338/143773840-3b822501-a376-4116-bc6a-0926959f4463.png)

Testasin, että tila käynnistää palvelimen vain jos muutoksia masterin tiedostossa on tapahtunut.
Ensimmäiseksi sammutin sshd palvelun orjilla:

![kuva](https://user-images.githubusercontent.com/82219338/143773868-71d1e950-f7b2-4459-8652-e9c2132bd113.png)

Ajoin tämän jälkeen tilan uudestaan:

![kuva](https://user-images.githubusercontent.com/82219338/143773897-069cdaf4-b183-41dc-801a-f96395561b09.png)

Muutoksia ei ollut tullut masterilta, joten masterin tiedostoa ei oltu muutettu edelliseltä kerralta ja palvelimet eivät käynnistyneet (yhteyttä ei muodostunut):

![kuva](https://user-images.githubusercontent.com/82219338/143773917-c1fa5f37-f981-4ce3-8fe5-2406d430173f.png)

Tämän jälkeen tein pienen muutoksen masterin tiedostoon komennolla:
```
sudoedit /srv/salt/sshd/sshd_config
```
![kuva](https://user-images.githubusercontent.com/82219338/143773934-2e44bc67-3fab-4f06-b856-04c09e277722.png)

Ja ajoin tämän jälkeen tilan uudestaan onnistuneesti:

![kuva](https://user-images.githubusercontent.com/82219338/143773951-5c069303-a2a0-4dbf-b0b4-156810dfba4a.png)

Yhteys saatiin muodostettua orjiin, joten palvelimet olivat käynnistyneet.

![kuva](https://user-images.githubusercontent.com/82219338/143773972-c8f0aac3-8d62-4032-b24f-51909f449b0a.png)

Tila siis käynistää palvelimet vain siinä tilanteessa, että masterin konfiguraatiotiedostossa /srv/salt/sshd/ kansiossa on tehty muutoksia edelliseen kertaan nähden.
15:45

# B)
Päätin tehdä konfiguraatio tiedostoon muutoksen, jolla otetaan salasanatunnistus  käyttöön
Muokkasin masterin konfiguraatio tiedostoa komennolla:
```
sudoedit /srv/salt/sshd/sshd_config
```
ja tein sinne seuraavan muutoksen:

![kuva](https://user-images.githubusercontent.com/82219338/143774035-afce0d40-9d1a-4957-9e1d-c41c79e2471f.png)

Ajoin tämän jälkeen tilan orjille komennolla:
```
sudo salt '*' state.apply sshd
```
Muutokset astuivat voimaan:

![kuva](https://user-images.githubusercontent.com/82219338/143774058-ce3d12e7-cc62-4775-b573-a06395be6959.png)

Nyt kun yritän ottaa ssh yhteyttä orjalal kaksi itseensä kysyy se salasanaa:

![kuva](https://user-images.githubusercontent.com/82219338/143774080-29a7cd2f-00f2-45aa-88b8-2a253022c7da.png)

Asetus tuli siis voimaan.
Siivottu sshd_config tiedosto:

```
# Do not change, will be managed automatically
Include /etc/ssh/sshd_config.d/*.conf
Port 22
Port 1025
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
RekeyLimit default none
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 2m
PermitRootLogin prohibit-password
StrictModes yes
MaxAuthTries 6
MaxSessions 10
PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2
AuthorizedPrincipalsFile none
AuthorizedKeysCommand none
AuthorizedKeysCommandUser nobody
HostbasedAuthentication no
IgnoreUserKnownHosts no
IgnoreRhosts yes
PasswordAuthentication yes
PermitEmptyPasswords no
ChallengeResponseAuthentication no
KerberosAuthentication no
KerberosOrLocalPasswd yes
KerberosTicketCleanup yes
KerberosGetAFSToken no
GSSAPIAuthentication no
GSSAPICleanupCredentials yes
GSSAPIStrictAcceptorCheck yes
GSSAPIKeyExchange no
UsePAM yes
AllowAgentForwarding yes
AllowTcpForwarding yes
GatewayPorts no
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
PermitTTY yes
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
PermitUserEnvironment no
Compression delayed
ClientAliveInterval 0
ClientAliveCountMax 3
UseDNS no
PidFile /var/run/sshd.pid
MaxStartups 10:30:100
PermitTunnel no
ChrootDirectory none
VersionAddendum none
Banner none
AcceptEnv LANG LC_*
Subsystem	sftp	/usr/lib/openssh/sftp-server
ClientAliveInterval 120
UseDNS no
````
17:20

# C)
Pohjana käytetty edellisen viikon vastausta kohtaan D) https://github.com/lottahallikainen/palvelintenhallinta/blob/main/Harjoitus4_Lotta_Huhta.md
Muokkasin apache.sls tiedostoa komennolla:
```
sudoedit /srv/salt/apache.sls
```
Ja muokkasin sen seuraavanlaiseksi:
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
      - <h3>Hello</h3>
      - </body>
      - </html>
ensure_apache2_is_running:
  service.running:
    - name: apache2
```
Ajoin tilan komennolla:
```
sudo salt '*' state.apply apache
```
Tilat tulivat käyttöön ja menemällä orjan kaksi ip-ositteeseen web-selaimessa haluttu sivusto tuli näkyviin:

![kuva](https://user-images.githubusercontent.com/82219338/143774229-9e13e387-09a3-4ea8-ad4e-77fa60de9b1f.png)

17:29 – jatkuu myöhemmin.

# D) 
jatkuu 18:15
Ensimmäiseksi ajoin masterilla komennot:
```
	sudo a2enmoduserdir
	sudo find -printf '%T+%p\n'|sort|tail
```
Tehdyt muutokset:

![kuva](https://user-images.githubusercontent.com/82219338/143774297-e7c44af6-751c-4e54-abfd-180bc9c16623.png)

Tarkastelin muutettuja tiedostoja ja selvisi, että ne ovat symbolisia linkkejä joiden osoitteet sain selville komennoilla:
```
	readlink -f /etc/apache2/mods-enabled/userdir.conf
	readlink -f /etc/apache2/mods-enabled/userdir.load
```
![kuva](https://user-images.githubusercontent.com/82219338/143774330-5c653745-65fa-4cd6-b98f-0928ec62a6fd.png)

Seuraavaksi muokkasin apache.sls tiedostoani komennolla:
```
sudoedit /srv/salt/apache.sls
```
Ja muokkasin sen seuraavanlaiseksi:
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
      - <h3>Hello</h3>
      - </body>
      - </html>
enable_userdir_conf:
  file.symlink:
    - name: /etc/apache2/mods-enabled/userdir.conf
    - target: /etc/apache2/mods-available/userdir.conf
    - force: False
enable_userdir_load:
  file.symlink:
    - name: /etc/apache2/mods-enabled/userdir.load
    - target: /etc/apache2/mods-available/userdir.load
    - force: False
ensure_apache2_is_running:
  service.running:
    - name: apache2
    - watch: 
      - file: /etc/apache2/mods-enabled/*
``` 

Ajoin tilat komennolla:

```
	sudo salt '*' state.apply apache
```
Onnistui:

![kuva](https://user-images.githubusercontent.com/82219338/143774402-2cc5eea2-4159-4727-8c9c-04ee94cd7ad9.png)

Tarkistin vielä, että symboliset linkit ilmaantuivat orjalle 1:

![kuva](https://user-images.githubusercontent.com/82219338/143774431-6b9e3653-a0eb-4933-994d-5a27b69c1d66.png)

Löytyyy.
19:00 – jatkuu myöhemmin

# E)
jatkui 10:15

Tehtävää tehdessä hyödynnetty saltin dokumentaatiota.
Tein tilatiedoston komennolla:
```
sudoedit /srv/salt/adduser.sls
```
Kirjoitin sinne seuraavaa:
```
add_user:
  user.present:
    - name: user1
    - home: /home/user1
prepare_user:
  file.copy:
    - name: /home/user1
    - source: /etc/skel
create_user_page:
  file.managed:
    - name: /home/user1/public_html/index.html
    - makedirs: True
    - replace: False
    - user: user1
    - mode: 744
    - contents: 
      - <html>
      - <body>
      - <h1>User1’s Page</h1>
      - </body>
      - </html>
```
Ajoin tilan komennolla:
````
sudo salt '*' state.apply adduser
````
Tilojen ajaminen onnistui:

![kuva](https://user-images.githubusercontent.com/82219338/143774566-d9ee89fd-f18d-4509-abd2-7fcaed398a2c.png)

Alla olevasta kuvasta nähdään että user1 lisättiin, user1:lle tehtiin kotisivu, /etc/skel/ sisältö on kopioitu kotikansioon
ja oikeudet on asetettu käyttäjälle user1 kuten myös tiedoston omistus.

![kuva](https://user-images.githubusercontent.com/82219338/143774598-00e2abac-9dda-4e1e-af25-2cd52c37da48.png)

![kuva](https://user-images.githubusercontent.com/82219338/143774606-04986f3e-13cd-4728-8279-b22ceacf66b9.png)

Sivusto myös toimii, vaikkakin heittomerkki ilmeisesti aiheutti kummallisuuksia.
Muokkasin sivua alla olevan laiseksi: 

![kuva](https://user-images.githubusercontent.com/82219338/143774628-a353ee29-8997-4258-b2d0-26107e9433ff.png)

Ajoin tämän jälkeen tilatiedoston uudestaan.
Tila ei tehnyt muutoksia jo olemassa olevaan tiedostoon niin kuin halusinkin

![kuva](https://user-images.githubusercontent.com/82219338/143774654-888e5dea-6030-4215-a4ca-f4ac27bd65d1.png)

11:03

# F)
Nginx asennusohjeet katsottu sivulta https://www.nginx.com/blog/setting-up-nginx/
Nginx userdir käyttöönoton ohjeet: https://websiteforstudents.com/configure-nginx-userdir-feature-on-ubuntu-16-04-lts-servers/
Tilafunktioita tehdessä hyödynnetty saltin dokumentaatiota.
Tein tilatiedoston, joka sulkee apache2:n, asentaa nginx, luo mallisivut ja ottaa userdir käyttöön komennolla:
```
sudoedit /srv/salt/nginx.sls
```
Ja kirjoitin sinne alla olevan:
```
shutdown_apache:
  service.dead:
    - name: apache2
install_nginx:
  pkg.installed:
    - name: nginx
  service.running:
    - name: nginx
/home/lotta/public_html/index.html:
  file.managed:
    - makedirs: True
    - replace: False
    - contents:
      - <html>
      - <body>
      - <h1>Users page</h1>
      - </body>
      - </html>
/home/lotta/public_html/application1/app1.html:
  file.managed:
    - makedirs: True
    - replace: False
    - contents:
      - <html>
      - <body>
      - <h1>App1 page</h1>
      - </body>
      - </html>
/home/lotta/public_html/bunny.jpg:
  file.managed:
    - source: salt://bunny.jpg
    - makedirs: True
    - replace: False 
make_bak:
  file.rename:
    - name: /etc/nginx/conf.d/default.conf.bak
    - source: /etc/nginx/conf.d/default.conf
configure_default:
  file.managed:
  - name: /etc/nginx/sites-available/default
  - contents:
    - server {
    - listen 80 default_server;
    - listen [::]:80 default_server;
    - root /var/www/html;
    - index index.html index.htm index.nginx-debian.html;
    - server_name _;
    - location ~ ^/~(.+?)(/.*)?$ {
    - alias /home/$1/public_html$2;
    - index index.html index.htm;
    - autoindex on;}}
configure_Nginx:
  file.managed:
    - name: /etc/nginx/conf.d/server1.conf
    - contents:
      - server {
      - root /home/lotta/public_html;
      - location /application1 {  }
      - location /images {		
      - root /home/lotta/data;}}
  service.running:
    - name: nginx
    - reload: True
    - watch:
      - file: /etc/nginx/*
```
Otin tilan käyttöön ajamalla sen lokaalisti komennolla:
```
sudo salt-call --local -l info state.apply nginx
```
Ajaminen onnistui:

![kuva](https://user-images.githubusercontent.com/82219338/143774760-7d15fe37-fe42-48f0-a80e-8c6356d1160a.png)

sivustot toimivat:

![kuva](https://user-images.githubusercontent.com/82219338/143774818-18636645-78c2-41e9-ae6f-1aa365658933.png)

![kuva](https://user-images.githubusercontent.com/82219338/143774852-6a18a7a9-313f-4435-ba20-4f73e3f5b185.png)

![kuva](https://user-images.githubusercontent.com/82219338/143774866-ed74175d-81b0-4d37-91c2-1ca05c8e779c.png)

![kuva](https://user-images.githubusercontent.com/82219338/143774876-af168ddb-f07d-4008-9226-2788dac878d4.png)

Valmis 13:42























