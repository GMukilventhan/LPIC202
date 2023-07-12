# LPIC202

## Sujet 
- [ ] Avoir un système de montage conforme aux recommandations ANSSI avec LVM
- [ ] Configurer le NTP avec plusieurs serveurs fonctionnels
- [ ] Configurer iptables avec les règles nécessaires, l’utilisation de chaîne et la policy de « INPUT » à DROP
- [ ] Configurer PAM afin que l’utilisateur soit verrouillé pendant 600 secondes après 4 tentatives échoué
- [ ] Configurer le DNS avec plusieurs entrées (machine 1, machine 2, machine 3) et des forwarder
- [ ] Configurer le DHCP avec une entrée statique et un subnet
- [ ] Installer le serveur TFTP et client
- [ ] Avoir des routes présentes et persistantes
- [ ] Les VM doivent être capable de retrouver leur état après un reboot

- [ ] [Documentation AANSSI ]([https://docs.ansible.com/](https://www.ssi.gouv.fr/guide/recommandations-de-securite-relatives-a-un-systeme-gnulinux/))

## Description
<img width="1192" alt="Capture d’écran 2023-07-12 à 23 50 25" src="https://github.com/GMukilventhan/LPIC202/assets/90500004/bec69c92-e532-491a-b68b-de8cebead8a8">

## LVM
- [ ]  Avoir un système de montage conforme aux recommandations ANSSI avec LVM
<img width="820" alt="Capture d’écran 2023-07-05 à 23 23 29" src="https://github.com/GMukilventhan/LPIC202/assets/90500004/c54684f6-4c14-4d92-a833-13645d498c76">


- Accéder au répertoire racine :
  ```
  cd /
  ```
- Écrire "- - -" dans le fichier /sys/class/scsi_host/host0/scan :
  ```
  echo "- - -" > /sys/class/scsi_host/host0/scan
  ```
- Vérifier les informations sur le disque /dev/sdb :
  ```
  fdisk -l | grep sdb
  ```
- Réinitialiser les hôtes SCSI : for host in $(ls /sys/class/scsi_host/);
  ```
  do echo "- - -" > /sys/class/scsi_host/${host}/scan; done
  ```
- Vérifier à nouveau les informations sur le disque /dev/sdb :
  ```
  fdisk -l | grep sdb
  ```
- Réinitialiser les hôtes SCSI avec une autre boucle :
  ```
  ls /sys/class/scsi_host/ | while read host ; do echo "- - -" > /sys/class/scsi_host/$host/scan ; done
  ```
- Manipuler les partitions du disque /dev/sdb :
```
fdisk /dev/sdb
```
- Créer une nouvelle partition primaire de type Linux sur /dev/sdb : Appuyer sur n, puis sélectionner p pour une partition primaire.
- Suivre les instructions pour définir les secteurs de début et de fin de la partition.
- Afficher la table des partitions mise à jour : Appuyer sur p.
- Enregistrer les modifications et quitter fdisk : Appuyer sur w.
- Éditer le fichier /etc/fstab :
  ```
  vim /etc/fstab
  ```
- Ajouter l'entrée dans le fichier /etc/fstab :
   ```
  /dev/mapper/datavg-lv_data /data ext4 defaults 0 2
   ``` 
- Afficher les informations sur les groupes de volumes :
  ```
  vgs
  ```
  ```
  root@client1:~#   vgs
  VG        #PV #LV #SN Attr   VSize   VFree 
  datavg      1   1   0 wz--n- <60,00g     0 
  ubuntu-vg   1  10   0 wz--n- <28,00g <3,84g
   ```
- Créer un nouveau groupe de volumes datavg à partir de /dev/sdb1 :
  ```
   vgcreate datavg /dev/sdb1
  ```
- Créer un nouveau volume logique lv_data dans le groupe de volumes datavg :
   ```
  lvcreate -l 100%FREE -n lv_data datavg
   ```
- Afficher les informations sur les volumes logiques :
  ```
  lvs
  ```
  ```
  root@srv1:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_data   datavg    -wi-ao---- <50,00g                                                    
  lv_data2  datavg    -wi-ao----  10,00g                                                    
  lv_home   ubuntu-vg -wi-ao----   1,50g                                                    
  lv_opt    ubuntu-vg -wi-ao---- 400,00m                                                    
  lv_proc   ubuntu-vg -wi-a-----   2,00g                                                    
  lv_root   ubuntu-vg -wi-ao----   4,00g                                                    
  lv_srv    ubuntu-vg -wi-ao---- 500,00m                                                    
  lv_tmp    ubuntu-vg -wi-ao---- 800,00m                                                    
  lv_usr    ubuntu-vg -wi-ao----   5,00g                                                    
  lv_var    ubuntu-vg -wi-ao----   4,00g                                                    
  lv_varlog ubuntu-vg -wi-ao----   5,00g                                                    
  lv_vartmp ubuntu-vg -wi-ao----   1,00g
  ```                                                  
root@srv1:~# 
- Créer un système de fichiers ext4 sur /dev/mapper/datavg-lv_data :
  ```
  mkfs.ext4 /dev/mapper/datavg-lv_data
  ```
- Créer le répertoire /data :
  ```
  mkdir /data
  ```
- Monter tous les systèmes de fichiers répertoriés dans /etc/fstab :
  ```
  mount -a
  ```
- Vérifier les informations sur les blocs de stockage :
  ```
  lsblk
  ```
```
  root@srv1:~#  lsblk
NAME                     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                      7:0    0 111,9M  1 loop /snap/lxd/24322
loop1                      7:1    0  63,3M  1 loop /snap/core20/1822
loop2                      7:2    0  63,4M  1 loop /snap/core20/1974
loop3                      7:3    0  49,8M  1 loop /snap/snapd/18357
loop4                      7:4    0  53,3M  1 loop /snap/snapd/19457
sda                        8:0    0    30G  0 disk 
├─sda1                     8:1    0     1M  0 part 
├─sda2                     8:2    0     2G  0 part /boot
└─sda3                     8:3    0    28G  0 part 
  ├─ubuntu--vg-lv_root   253:2    0     4G  0 lvm  /
  ├─ubuntu--vg-lv_usr    253:3    0     5G  0 lvm  /usr
  ├─ubuntu--vg-lv_var    253:4    0     4G  0 lvm  /var
  ├─ubuntu--vg-lv_tmp    253:5    0   800M  0 lvm  /tmp
  ├─ubuntu--vg-lv_opt    253:6    0   400M  0 lvm  /opt
  ├─ubuntu--vg-lv_home   253:7    0   1,5G  0 lvm  /home
  ├─ubuntu--vg-lv_varlog 253:8    0     5G  0 lvm  /var/log
  ├─ubuntu--vg-lv_srv    253:9    0   500M  0 lvm  /srv
  ├─ubuntu--vg-lv_vartmp 253:10   0     1G  0 lvm  /var/tmp
  └─ubuntu--vg-lv_proc   253:11   0     2G  0 lvm  
sdb                        8:16   0    60G  0 disk 
└─sdb1                     8:17   0    60G  0 part 
  ├─datavg-lv_data       253:0    0    50G  0 lvm  /data
  └─datavg-lv_data2      253:1    0    10G  0 lvm  /data2
sr0                       11:0    1   1,8G  0 rom
```
- Réduire la taille du volume logique /dev/mapper/datavg-lv_data de 10 Go :
  ```
  lvreduce -r -L -10G /dev/mapper/datavg-lv_data
  ```
- Éditer à nouveau le fichier /etc/fstab :
  ```
  vim /etc/fstab
  ```
- Ajouter l'entrée dans le fichier /etc/fstab:
   ```
  /dev/mapper/datavg-lv_data2 /data2 ext4 defaults 0 2
   ```
- Créer le répertoire /data2 :
  ```
  mkdir /data2
  ```
- Créer un système de fichiers ext4 sur /dev/mapper/datavg-lv_data2 :
  ```
  mkfs.ext4 /dev/mapper/datavg-lv_data2
  ```
- Monter tous les systèmes de fichiers répertoriés dans /etc/fstab :
  ```
  mount -a
  ```
- Vérifier l'utilisation de l'espace disque :
  ```
  df -h
  ```
```
  root@srv1:~#   df -h
Filesystem                                                                                    Size  Used Avail Use% Mounted on
tmpfs                                                                                         388M  1,7M  387M   1% /run
/dev/mapper/ubuntu--vg-lv_root                                                                3,9G  6,1M  3,7G   1% /
/dev/disk/by-id/dm-uuid-LVM-faX53WXMuCG6nsPm1NdAQZRRl4ZoT6a0Dl3AL5GU38HuLTH2YsO2CttpKcxCBW77  4,9G  2,3G  2,3G  51% /usr
tmpfs                                                                                         1,9G     0  1,9G   0% /dev/shm
tmpfs                                                                                         5,0M     0  5,0M   0% /run/lock
/dev/sda2                                                                                     2,0G  130M  1,7G   8% /boot
/dev/mapper/ubuntu--vg-lv_home                                                                1,5G   64K  1,4G   1% /home
/dev/mapper/ubuntu--vg-lv_opt                                                                 359M   24K  331M   1% /opt
/dev/mapper/ubuntu--vg-lv_var                                                                 3,9G  618M  3,1G  17% /var
/dev/mapper/datavg-lv_data2                                                                   9,8G   24K  9,3G   1% /data2
/dev/mapper/datavg-lv_data                                                                     49G   24K   47G   1% /data
/dev/mapper/ubuntu--vg-lv_srv                                                                 452M   32K  417M   1% /srv
/dev/mapper/ubuntu--vg-lv_varlog                                                              4,9G  111M  4,5G   3% /var/log
/dev/mapper/ubuntu--vg-lv_vartmp                                                              974M   56K  907M   1% /var/tmp
/dev/mapper/ubuntu--vg-lv_tmp                                                                 770M   76K  714M   1% /tmp
tmpfs                                                                                         388M  4,0K  388M   1% /run/user/1000

```

## NTP
- [ ] Configurer le NTP avec plusieurs serveurs fonctionnels

- Installer le package NTP :
  ```
  sudo apt-get install ntp -y
  ```
- Vérifier l'état du service NTP :
  ```
  sudo service ntp status
  ```
- Éditer le fichier de configuration NTP :
  ```
  sudo vim /etc/ntp.conf
  ```
- Ajouter les serveurs NTP souhaités dans le fichier /etc/ntp.conf. Par exemple :
 ```
 server 192.168.74.198
 server 192.168.74.210
 server 192.168.74.214
```

- Enregistrer les modifications et quitter l'éditeur.
- Redémarrer le service NTP :
  ```
  sudo service ntp restart
  ```
- Vérifier à nouveau l'état du service NTP :
  ```
  sudo service ntp status
  ```
- Vérifier la synchronisation avec les serveurs NTP :
  ```
  ntpq -pn
  ```
```
  root@srv1:~# ntpq -pn
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 0.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000   +0.000   0.000
 1.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000   +0.000   0.000
 2.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000   +0.000   0.000
 3.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000   +0.000   0.000
 ntp.ubuntu.com  .POOL.          16 p    -   64    0    0.000   +0.000   0.000
 192.168.74.198  .STEP.          16 u    -   64    0    0.000   +0.000   0.000
 192.168.74.210  .STEP.          16 u    -   64    0    0.000   +0.000   0.000
 190.168.74.214  .STEP.          16 u    -   64    0    0.000   +0.000   0.000
+195.154.226.102 193.190.230.37   2 u    1   64    1  120.329  +250.80  33.830
-149.202.2.105   174.222.245.115  2 u    2   64    1  226.499  +131.00  86.924
-162.159.200.123 10.25.8.4        3 u    -   64    1  145.433  +142.43  75.840
-82.64.45.50     .GPS.            1 u    2   64    1  227.264  +130.93  83.424
+82.64.247.11    193.67.79.202    2 u    2   64    1   52.562  +214.95   3.562
-194.177.34.116  86.23.195.30     2 u    1   64    1  119.966  +248.82  33.154
#178.32.40.253   82.64.45.50      2 u    1   64    1  129.882  +246.33  31.841
-146.59.35.38    134.64.19.180    2 u    2   64    1  227.681  +137.59  76.152
#80.74.64.2      84.239.67.14     2 u    2   64    1  226.760  +129.22  85.103
-37.59.63.125    193.190.230.37   2 u    1   64    1  123.024  +248.16  36.681
*178.33.41.123   82.64.45.50      2 u    1   64    1  115.966  +245.06  34.059
-51.38.99.44     176.9.166.35     3 u    1   64    1  118.437  +245.49  35.314
```
- Vérifier les informations de date et d'heure :

  ```
  timedatectl
  ```
  ```
  root@srv1:~# timedatectl
               Local time: mer. 2023-07-12 22:39:25 UTC
           Universal time: mer. 2023-07-12 22:39:25 UTC
                 RTC time: mer. 2023-07-12 22:39:25
                Time zone: Etc/UTC (UTC, +0000)
   System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
   root@srv1:~# 
  ```
 ## IPTABLES
- [ ] Configurer iptables avec les règles nécessaires, l’utilisation de chaîne et la policy de « INPUT » à DROP

- Cette commande ajoute une règle à la chaîne INPUT d'iptables pour accepter le trafic TCP sur le port 22, qui est le port par défaut utilisé par SSH:
```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

- Cette commande affiche les règles actuelles du pare-feu iptables. Dans votre cas, la politique par défaut pour toutes les chaînes (INPUT, FORWARD, OUTPUT) est ACCEPT, ce qui signifie que tout le trafic est autorisé :
```
iptables -L -n
```
```
root@srv1:~# iptables -L -n
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state NEW,RELATED,ESTABLISHED
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            udp dpt:53
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:53
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            udp dpt:67
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            udp dpt:69

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
root@srv1:~# 
```

 - Cette commande supprime toutes les règles actuelles du pare-feu iptables:
 ```
 sudo iptables -F
 ```

- Cette commande ajoute une règle à la chaîne OUTPUT d'iptables pour accepter le trafic sortant qui est nouvellement initié, ainsi que les connexions liées et établies:
```
sudo iptables -A OUTPUT -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
```

- Cette commande définit la politique par défaut de la chaîne INPUT sur DROP, ce qui signifie que tout le trafic entrant sera rejeté à moins qu'il ne corresponde à une règle spécifique:
 ```
 sudo iptables -P INPUT DROP
```

 - Cette commande ajoute une règle à la chaîne INPUT d'iptables pour accepter le trafic UDP provenant du port source 53, qui est généralement utilisé pour les requêtes DNS sortantes:
 ```
 sudo iptables -A INPUT -p udp --sport 53 -j ACCEPT
```

 - Cette commande ajoute une règle à la chaîne INPUT d'iptables pour accepter le trafic TCP provenant du port source 53, qui est également utilisé pour les requêtes DNS sortantes:
 ```
 sudo iptables -A INPUT -p tcp --sport 53 -j ACCEPT
```

 - Cette commande ajoute une règle à la chaîne INPUT d'iptables pour accepter le trafic UDP sur le port de destination 67, qui est utilisé pour les requêtes DHCP entrantes:
 ```
 sudo iptables -A INPUT -p udp --dport 67 -j ACCEPT
```

- Cette commande ajoute une règle à la chaîne INPUT d'iptables pour accepter le trafic UDP sur le port de destination 69, qui est utilisé pour les requêtes TFTP entrantes.
```
sudo iptables -A INPUT -p udp --dport 69 -j ACCEPT
```

- Cette commande installe le paquet iptables-persistent, qui permet d'enregistrer les règles iptables de manière persistante afin qu'elles soient restaurées au redémarrage du système.
```
apt-get install iptables-persistent
```

- Cette commande enregistre les règles iptables actuelles dans un fichier appelé "rules.v4" situé dans le répertoire "iptables":
```
iptables-save > iptables/rules.v4
```

- La sortie de ```iptables -L -n ```et ```nft list ruleset``` montre les règles actuelles de la table filter d'iptables et de la table filter de nftables respectivement.

## PAM

- [ ] Configurer PAM afin que l’utilisateur soit verrouillé pendant 600 secondes après 4 tentatives échoué

- Editer le fichier
```
vim /etc/pam.d/common-auth
```
```
# faillock
auth    required pam_faillock.so preauth audit deny=4 unlock_time=600

# here are the per-package modules (the "Primary" block)
auth    [success=1 default=ignore]      pam_unix.so nullok

# faillock
auth [default=die] pam_faillock.so authfail audit deny=4 fail_interval=60 unlock_time=600
auth sufficient pam_faillock.so authsucc audit deny=4 fail_interval=60 unlock_time=600

# here's the fallback if no module succeeds
auth    requisite                       pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
auth    required                        pam_permit.so
# and here are more per-package modules (the "Additional" block)
auth    optional                        pam_cap.so
# end of pam-auth-update config
```
- Ajouter cette ligne  ```account required pam_faillock.so``` à la fin du fichier
```
vim /etc/pam.d/common-account
```
```
# here are the per-package modules (the "Primary" block)
account [success=1 new_authtok_reqd=done default=ignore]        pam_unix.so
# here's the fallback if no module succeeds
account requisite                       pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
account required                        pam_permit.so
# and here are more per-package modules (the "Additional" block)
# end of pam-auth-update config
#
account required pam_faillock.so
```
- La commande « faillock » permet aussi de pouvoir consulter le nombre de tentative de connexion :
```
root@srv1:~# faillock --user mukil
mukil:
When                Type  Source                                           Valid
2023-07-12 22:59:06 RHOST 192.168.74.1                                         V
2023-07-12 22:59:09 RHOST 192.168.74.1                                         V
```
- Déverrouiller un compte manuellement
```
root@srv1:~# faillock --user mukil --reset
```

## DNS

- [ ] Configurer le DNS avec plusieurs entrées (machine 1, machine 2, machine 3) et des forwarder

Le DNS sous linux est ```bind9```, tapez cette commande pour l’installer :
```
apt install bind9
```
- Création d’une zone
  
Nous allons créer une zone pour le domaine ```iosolution.fr```.

Nous allons créer une entrée de type A (renvoie une ou plusieurs adresses IPv4 pour un nom d'hôte donné) dans le fichier ```/etc/bind/db.iosolution.fr```:

```
cp -rp /bind/db.local /etc/bind/db.iosolution.fr
```
```
vim /etc/bind/db.iosolution.fr
```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A       127.0.0.1
@       IN      AAAA    ::1

srv1 A 192.168.74.198
client1 A 192.168.74.212
client2 A 192.168.74.214
```

- Création d’une zone « reverse »
- 
  Nous allons créer une zone reverse pour le domaine ``` iosolution.fr```. Dans mon cas, je vais utiliser le subnet ``` 192.168.74.0 ``` :
```
  cp -rp /etc/bind/db.127 /etc/bind/db.74.168.192
```
Nous allons créer une entrée de type PTR (Réalise l'inverse de l'enregistrement A ou AAAA : donne un nom de host (FQDN) pour une adresse IP.) dans le fichier ```/etc/bind/db.74.168.192```:
```
vim  /etc/bind/db.74.168.192
```
```
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
1.0.0   IN      PTR     localhost.
198     IN      PTR     srv1.iosolution.fr
212     IN      PTR     client1.iosolution.fr
214     IN      PTR     client2.iosolution.fr
```

- Déclaration de nos zones dans le DNS
  
  Il faut déclarer notre nouveau domaine dans la configuration DNS :
Créer le fichier ```/etc/bind/named.conf.iosolution.fr ``` afin de faire la déclaration de nos zones
```
vim /etc/bind/named.conf.iosolution.fr
```
```
zone "iosolution.fr" {
        type master;
        file "/etc/bind/db.iosolution.fr";
};
zone "74.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.74.168.192";
};
```
Enfin, il ne reste plus qu’a dire à BIND9 d’inclure notre fichier de configuration « named.conf.iosolution » dans la configuration DNS :
Editer le fichier ```/etc/bind/named.conf ```:
```
vim /etc/bind/named.conf
```
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
include "/etc/bind/named.conf.iosolution.fr";
```
- Recharger la configuration DNS
Une fois toutes les zones créent et déclarés, il faut recharger la configuration DNS afin que cela soit pris en compte :
```
service bind9 reload
```
- Configuration des forwarders
  Editez le fichier ```/etc/bind/named.conf.options``` afin d’ajouter des forwarders (google dans notre exemple) :
```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                8.8.8.8;
                8.8.4.4;
         };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on-v6 { any; };
};
```
<img width="506" alt="Capture d’écran 2023-07-12 à 19 47 59" src="https://github.com/GMukilventhan/LPIC202/assets/90500004/fb9cacc1-3e16-46e1-9cd4-129035dac76f">



## DHCP

- [ ] Configurer le DHCP avec une entrée statique et un subnet

- Installation du serveur DHCP
  Pour installer  un serveur DHCP, nous allons installer le package  ```isc-dhcp-server``` qui est le serveur DHCP sur Ubuntu
```
  apt install isc-dhcp-server
```
- Création d’un subnet DHCP

  C’est dans le fichier ```/etc/dhcp/dhcpd.conf  ``` qu’on va notre subnet sur lequel notre DHCP sera actif :
```
  subnet 192.168.74.0 netmask 255.255.255.0 {
range 192.168.74.212 192.168.74.255;
option routers 192.168.74.2;
option domain-name-servers 192.168.74.198;
}
```

- Réservation d’une IP dans le DHCP
Il est possible de « provisionner » le DHCP pour réserver une IP à un host.
cela, il faut créer l’entrée dans ``` /etc/dhcp/dhcpd.conf ``` comme ci-dessous :
```
host  client2 {
hardware ethernet 00:0c:29:c7:72:7D;
fixed-address 192.168.74.214;
}
```
- Activation du DHCP
```
sudo service isc-dhcp-server enable 
sudo service isc-dhcp-server restart
sudo service isc-dhcp-server status
```
## TFTP

- [ ] Installer le serveur TFTP et client

- Serveur TFTP
```
apt install tftpd-hpa
```
- Client TFTP
```
apt install tftp-hpa
```
- Ajouter « -4 » dans « TFTP_OPTIONS » qui se trouve dans /etc/default/tftpd-hpa
```
vim /etc/default/tftpd-hpa
```

```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure -4"
```
- Création d’un fichier test dans le dossier TFTP
```
touch /srv/tftp/test.txt && echo « Ceci est un test TFTP » >> /srv/tftp/test.txt
touch /srv/tftp/test.txt && echo « Ceci est un test TFTP projet 4SRC2  » >> /srv/tftp/projet.txt
```
- Attribuer les droits du dossier TFTP et son contenu
```
chgrp -R tftp /srv/tftp && chmod -R 755 /srv/tftp
```
- Activer le serveur tftp
```
systemctl start tftpd-hpa
```
```
systemctl enable tftpd-hpa
```
- Test du TFTP
```

root@client1:~# tftp 192.168.73.198
tftp> get projet.txt
tftp> quit
```
```
root@client1:~# cat projet.txt
« Ceci est un test TFTP »
« Ceci est un test TFTP projet 4SRC2 »
root@client1:~#
```

## Routes
- [ ] Avoir des routes présentes et persistantes
