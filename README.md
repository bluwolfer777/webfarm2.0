# Documentazione webfarm 2.0 M239
### di Leon Rosamilia

## Installazione OS
Per prima cosa bisogna flashare il sistema operativo su una microSD con ``rpi imager``<br>
installiamo Raspberry-Pi-OS 64bit stable
grazie a questo abbiamo un server ssh, sftp e vnc preinstallato e preconfigurato




## Installazione di base

Iniziamo con aggiornare la lista dei pacchetti e poi aggiornare tutti quelli che non lo sono
```shell
sudo apt update
sudo apt upgrade -y
```

Ora installiamo e facciamo partire ``neofetch`` per ottenere tutte le info sul sistema in uso
```shell
sudo apt install neofetch -y
neofetch
```
Otteniamo il seguente output:

```shell
       _,met$$$$$gg.          leon@webserver
    ,g$$$$$$$$$$$$$$$P.       --------------
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 12 (bookworm) aarch64
 ,$$P'              `$$$.     Host: Raspberry Pi 4 Model B Rev 1.1
',$$P       ,ggs.     `$$b:   Kernel: 6.6.20+rpt-rpi-v8
`d$$'     ,$P"'   .    $$$    Uptime: 5 mins
 $$P      d$'     ,    $$P    Packages: 1537 (dpkg)
 $$:      $$.   -    ,d$$'    Shell: bash 5.2.15
 $$;      Y$b._   _,d$P'      Resolution: 3840x1200
 Y$$.    `.`"Y$$$$P"'         Theme: PiXflat [GTK3]
 `$$b      "-.__              Icons: PiXflat [GTK3]
  `Y$$                        Terminal: /dev/pts/0
   `Y$$.                      CPU: (4) @ 1.500GHz
     `$$b.                    Memory: 285MiB / 3792MiB
       `Y$$b.
          `"Y$b._
              `"""
```


Installiamo tutti i pacchetti di cui avremo bisogno:
```shell
sudo apt install nginx default-mysql-server php-fpm php-mysql -y
```
E ora avviamo i daemon per i servizi web e database:
```shell
sudo systemctl start nginx.service
sudo systemctl start mysql.service
sudo systemctl start php8.2-fpm.service
```
### MySQL
Per controllare che l'installazione sia avvenuta digitare ``mysql -u root`` e se esce questo errore è installato correttamente
```shell
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
```

Per configurare mysql bisogna usare l'utente root, fare attenzione a non fare comandi che non si conoscono con questo utente perchè si può anche disinstallare il bootloader
```shell
sudo -i
mysql
```

Una volta entrati nella shell di mysql si possono fare i seguenti comandi:
```SQL
FLUSH PRIVILEGES;
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('password_super_sicura');
exit
```

Usciamo dall'utente root e testiamo se riusciamo a loggare:
```shell
exit
mysql -u root -p
```
se riesce a loggare allora la procedura ha funzionato, possiamo fare ``exit`` e configurare Nginx per funzionare con i seguenti domini:
### Nginx con PHP
- mail.leonrosamilia.ch
- ftp.leonrosamilia.ch
- myadmin.leonrosamilia.ch
- www.leonrosamilia.ch

Iniziamo creando una cartella per virtualhost:
```shell
sudo mkdir /var/www/www.leonrosamilia.ch
sudo mkdir /var/www/mail.leonrosamilia.ch
sudo mkdir /var/www/myadmin.leonrosamilia.ch
sudo mkdir /var/www/ftp.leonrosamilia.ch
```

e diamo i permessi all'utente root:
```shell
sudo chown -R root:root /var/www/www.leonrosamilia.ch
sudo chown -R root:root /var/www/mail.leonrosamilia.ch
sudo chown -R root:root /var/www/myadmin.leonrosamilia.ch
sudo chown -R root:root /var/www/ftp.leonrosamilia.ch
```

creiamo un file di configurazione per il nostro dominio con 
```shell 
sudo nano /etc/nginx/sites-available/www.leonrosamilia.ch
```
Aggiungiamo la seguente configurazione nel file ed usiamo CTRL+O per salvare e CTRL+X per chiudere l'editor

```nginx
server {	
 listen 80;	
 server_name leonrosamilia.ch www.leonrosamilia.ch;	
 root /var/www/www.leonrosamilia.ch;	
 index index.php index.html index.htm;	
 location / {	
 try_files $uri $uri/ =404;	
 }	
 location ~ \.php$ {	
 include snippets/fastcgi-php.conf;	
 fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;	
 }	
 location ~ /\.ht {	
 deny all;	
 }	
}
```
ripetiamo per tutti i virtualhost
```shell 
sudo nano /etc/nginx/sites-available/mail.leonrosamilia.ch
```
```nginx
server {	
 listen 80;	
 server_name mail.leonrosamilia.ch;	
 root /var/www/mail.leonrosamilia.ch;	
 index index.php index.html index.htm;	
 location / {	
 try_files $uri $uri/ =404;	
 }	
 location ~ \.php$ {	
 include snippets/fastcgi-php.conf;	
 fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;	
 }	
 location ~ /\.ht {	
 deny all;	
 }	
}
```
```shell 
sudo nano /etc/nginx/sites-available/ftp.leonrosamilia.ch
```
```nginx
server {	
 listen 80;	
 server_name ftp.leonrosamilia.ch;	
 root /var/www/ftp.leonrosamilia.ch;	
 index index.php index.html index.htm;	
 location / {	
 try_files $uri $uri/ =404;	
 }	
 location ~ \.php$ {	
 include snippets/fastcgi-php.conf;	
 fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;	
 }	
 location ~ /\.ht {	
 deny all;	
 }	
}
```
```shell 
sudo nano /etc/nginx/sites-available/myadmin.leonrosamilia.ch
```
```nginx
server {	
 listen 80;	
 server_name myadmin.leonrosamilia.ch;	
 root /var/www/myadmin.leonrosamilia.ch;	
 index index.php index.html index.htm;	
 location / {	
 try_files $uri $uri/ =404;	
 }	
 location ~ \.php$ {	
 include snippets/fastcgi-php.conf;	
 fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;	
 }	
 location ~ /\.ht {	
 deny all;	
 }	
}
```

Spostiamo le configurazioni appena fatte in quelle abilitate
```shell
sudo ln -s /etc/nginx/sites-available/www.leonrosamilia.ch /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/ftp.leonrosamilia.ch /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/mail.leonrosamilia.ch /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/myadmin.leonrosamilia.ch /etc/nginx/sites-enabled/
```
e disattiviamo la configurazione di default con:
```shell
unlink /etc/nginx/sites-enabled/default
```
usiamo ``sudo nginx -t`` per controllare la sinatssi
ed ora riavviamo Nginx con ``sudo systemctl reload nginx``

per testare nginx creiamo un file php:
```shell
sudo nano /var/www/www.leonrosamilia.ch/index.php
```
all'interno mettiamo il seguente script in php
```PHP
<?php 
    phpinfo(); 
?>
```
### NodeJS
installaimo NodeJS con il commando
```shell
sudo apt install nodejs -y
```
verifichiamo l'installazione con
```shell
node -v
```
installiamo anche node package manager con
```shell
sudo apt install npm -y
```

### PHPMyAdmin
per installare phpMyAdmin iniziamo a scaricare il pacchetto facendo ``sudo apt install phpmyadmin -y``
durante la configurazione ci chiederà se cercare automaticamente il server e noi cliacchiamo TAB e poi ENTER
successivamente chiederà per una password, cliccare ENTER poi TAB e poi di nuovo ENTER
inseriamo phpmyadmin nel virtualhost corretto
```shell
sudo ln -s /usr/share/phpmyadmin /var/www/myadmin.leonrosamilia.ch/
```
per comodità aggiungiamo un file che rimanda da ``myadmin.leonrosamilia.ch`` a ``myadmin.leonrosamilia.ch/phpmyadmin``
creiamolo con 
```shell
sudo nano /var/www/myadmin.leonrosamilia.ch/index.php
```
ed inseriamo il seguente codice
```PHP
<?php
// 301 Moved Permanently
header("Location: http://myadmin.leonrosamilia.ch/phpmyadmin", true, 301);
exit();
?>
```
testiamo myAdmin andando sul sito

### Nginx con SSL
installiamo un'altro package manager, ovvero ``snap`` lo facciamo con ```sudo apt install snapd -y```
a volte sono preinstallate vecchie versioni di certbot e quindi le disinstalliamo con ``sudo apt purge certbot``
possiamo procedere ad installare certbot con il seguente comando:
```shell
sudo snap install --classic certbot
```
potrebbe metterci un po' per via della configurazione iniziale di snap
eseguiamo il seguente commando per assicurarci che certbot possa essere avviato:
```shell
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
installiamo ora i certificati su Nginx con:
```shell
sudo certbot --nginx
```
ci chiederà la mail per le notifiche di sicurezza, poi se accettiamo i termini di servizio, poi se vogliamo la newsletter, ed infine quali domini vogliamo attivare, noi lasieremo vuoto per selezionarli tutti

### Mail Server

installiamo il mail server con
```shell
sudo DEBIAN_PRIORITY=low apt install postfix
```
quando ce lo chiede facciamo enter su Internet site e poi inseriamo il nostro dominio da usare per la posta elettronica, poi ci chiederà l'utente amministratore del server per creargli un'indirizzo email

quando chiede degli indirizzi ip aggiungiamo in fondo ``192.168.1.0/24``
il resto lasciamo al default

configuriamo il server con
```shell
sudo postconf -e 'home_mailbox= Maildir/'
sudo postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'
sudo nano /etc/postfix/virtual
sudo systemctl restart postfix
```
### FTP server
```shell
sudo apt install vsftpd	-y
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.orig	
sudo adduser ftpuser	
sudo mkdir /home/ftpuser/ftp	 
sudo chown nobody:nogroup /home/ftpuser/ftp	
sudo chmod a-w /home/ftpuser/ftp	
sudo mkdir /home/ftpuser/ftp/files	
sudo chown ftpuser:ftpuser /home/ftpuser/ftp/files	
sudo tee /home/ftpuser/ftp/files/test.txt	
sudo nano /etc/vsftpd.conf	
```
```conf
write_enable=YES	
chroot_local_user=YES	
user_sub_token=ftpuser	
local_root=/home/ftpuser/ftp
userlist_enable=YES	
userlist_file=/etc/vsftpd.userlist	
userlist_deny=NO	
```
```shell
sudo systemctl restart vsftpd
```

### Mail client
scarica [roundcube](https://roundcube.net) e
mettilo in ``/var/www/mail.leonrosamilia.ch``

### Ftp client
scarica [net2ftp](https://www.net2ftp.com/index.php) e
mettilo in ``/var/www/ftp.leonrosamilia.ch``