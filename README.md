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


- Accéder au répertoire racine : ``` cd / ```
- Écrire "- - -" dans le fichier /sys/class/scsi_host/host0/scan : ```echo "- - -" > /sys/class/scsi_host/host0/scan```
- Vérifier les informations sur le disque /dev/sdb : ```fdisk -l | grep sdb```
- Réinitialiser les hôtes SCSI : for host in $(ls /sys/class/scsi_host/); ```do echo "- - -" > /sys/class/scsi_host/${host}/scan; done```
- Vérifier à nouveau les informations sur le disque /dev/sdb : ```fdisk -l | grep sdb```
- Réinitialiser les hôtes SCSI avec une autre boucle : ```ls /sys/class/scsi_host/ | while read host ; do echo "- - -" > /sys/class/scsi_host/$host/scan ; done```
- Manipuler les partitions du disque /dev/sdb : ```fdisk /dev/sdb```
- Créer une nouvelle partition primaire de type Linux sur /dev/sdb : Appuyer sur n, puis sélectionner p pour une partition primaire.
- Suivre les instructions pour définir les secteurs de début et de fin de la partition.
- Afficher la table des partitions mise à jour : Appuyer sur p.
- Enregistrer les modifications et quitter fdisk : Appuyer sur w.
- Éditer le fichier /etc/fstab : ```vim /etc/fstab```
- Ajouter l'entrée ```/dev/mapper/datavg-lv_data /data ext4 defaults 0 2``` dans le fichier /etc/fstab.
- Afficher les informations sur les groupes de volumes : ```vgs```
- Créer un nouveau groupe de volumes datavg à partir de /dev/sdb1 :``` vgcreate datavg /dev/sdb1```
- Créer un nouveau volume logique lv_data dans le groupe de volumes datavg : ```lvcreate -l 100%FREE -n lv_data datavg```
- Afficher les informations sur les volumes logiques : ```lvs```
- Créer un système de fichiers ext4 sur /dev/mapper/datavg-lv_data : ```mkfs.ext4 /dev/mapper/datavg-lv_data```
- Créer le répertoire /data : ```mkdir /data```
- Monter tous les systèmes de fichiers répertoriés dans /etc/fstab : ```mount -a```
- Vérifier les informations sur les blocs de stockage : ```lsblk```
- Réduire la taille du volume logique /dev/mapper/datavg-lv_data de 10 Go : ```lvreduce -r -L -10G /dev/mapper/datavg-lv_data```
- Éditer à nouveau le fichier /etc/fstab : ```nano /etc/fstab```
- Ajouter l'entrée ```/dev/mapper/datavg-lv_data2 /data2 ext4 defaults 0 2 ``` dans le fichier /etc/fstab.
- Créer le répertoire /data2 : ```mkdir /data2```
- Créer un système de fichiers ext4 sur /dev/mapper/datavg-lv_data2 : ```mkfs.ext4 /dev/mapper/datavg-lv_data2```
- Monter tous les systèmes de fichiers répertoriés dans /etc/fstab : ```mount -a```
- Vérifier l'utilisation de l'espace disque : ```df -h```
