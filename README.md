# 42_roger_skyline1

Partie VM

Vous devez installer une Virtual Machine (VM) avec l’OS Linux de votre choix (Debian Jessie, CentOS 7...) dans l’hyperviseur de votre choix (VMWare Fusion, VirtualBox...).  Debian 10
Celle ci devra avoir : 
• Une taille de disque de 8 Go.  OK

• Avoir au moins une partition de 4.2 Go. OK : partition systeme de 4.2 Go + SWAP + home
http://debian-facile.org/doc:install:partitions-manuel

• Elle devra également être à jour ainsi que l’ensemble des packages installés pour répondre aux demandes de ce sujet.
https://www.debian.org/doc/manuals/debian-faq/ch-uptodate.fr.html

Partie Réseau et Sécurité 

Concernant le réseau sur la VM, voici les étapes à réaliser : 
• Vous devez créer un utilisateur non root pour vous connecter et travailler. 
OK : username=antoine

• Utilisez sudo pour pouvoir, depuis cet utilisateur, effectuer les operations demandant des droits speciaux. 
OK : add user to sudo group : usermod -aG sudo antoine

• Nous ne voulons pas que vous utilisiez le service DHCP de votre machine. A vous donc de la configurer afin qu’elle ait une IP fixe et un Netmask en /30.
Bridged connection
/30: Addresses = 4 Hosts = 2 Netmask = 255.255.255.252
Modification du fichier de configuration /etc/network/interfaces :
remplacer: dhcp => static
ajouter:
address 10.11.254.253/30
broadcast 10.11.254.255
gateway 10.11.254.254

• Vous devez changer le port par defaut du service SSH par celui de votre choix. 
Modification du fichier de configuration /etc/ssh/sshd_config : Port 22 => Port 8888

L’accès SSH DOIT se faire avec des publickeys. 
Commande ssh-keygen: generer une cle publique sur le client et la copier vers le serveur avec ssh-copy-id ~/.ssh/id_rsa.pub agelloz@10.11.X.X
Modification du fichier /etc/ssh/sshd_config : PasswordAuthentication yes >> PasswordAuthentication no

L’utilisateur root ne doit pas pouvoir se connecter en SSH.
Modification du fichier /etc/ssh/sshd_config : PermitRootLogin prohibit-password >> PermitRootLogin no

• Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM.
Configurer manuellement les règles de pare-feu par un script init.d qui exécutera des commandes iptables : Script firewall dans /etc/init.d/firewall
Configurer le système pour exécuter le script avant que le réseau ne soit configuré : update-rc.d firewall defaults
https://sharadchhetri.com/2013/06/15/how-to-protect-from-port-scanning-and-smurf-attack-in-linux-server-by-iptables/

• Vous devez mettre en place une protection contre les DOS (Denial Of Service Attack) sur les ports ouverts de votre VM. 
Inclus dans le script iptables /etc/init.d/firewall
https://sharadchhetri.com/2013/06/15/how-to-protect-from-port-scanning-and-smurf-attack-in-linux-server-by-iptables/
+ installation et configuration de fail2ban (nouveau regex pour l’authentification SSH par cle plublique)
https://www.linode.com/docs/security/using-fail2ban-for-security/

• Vous devez mettre en place une protection contre les scans sur les ports ouverts de votre VM. 
Inclus dans le script iptables /etc/init.d/firewall
+ installation et configuration de portsentry
https://wiki.debian-fr.xyz/Portsentry

• Arretez les services dont vous n’avez pas besoin pour ce projet. 
Liste des services actifs : systemctl list-units -t service
Supprimer les programmes inutiles avec la commande : "apt-get remove" ou "apt-get autoremove" (pour inclure les dependances) et on peut utiliser "--purge" pour inclure les fichiers de configuration

• Réalisez un script qui met à jour l’ensemble des sources de package, puis de vos packages et qui log l’ensemble dans un fichier nommé /var/log/update_script.log. Créez une tache planifiée pour ce script une fois par semaine à 4h00 du matin et à chaque reboot de la machine. 
Script /update_script.sh : chaque etape est ecrite dans le fichier /var/log/update_script.log
- date
- apt-get update
- apt-get dist-upgrade
Creation d’une tache planifiee avec crontab -e : ajout de la tache 

• Réalisez un script qui permet de surveiller les modifications du fichier /etc/crontab et envoie un mail à root si celui-ci a été modifié. Créez une tache plannifiée pour script tous les jours à minuit.
Script /watch_crontab.sh : calcul de la difference entre la date d’execution et la date de modification du fichier /etc/crontab et envoi de mail a root si inferieur ou egal a 24h. 

Partie optionnelle 
Partie Web
Vous devez mettre en place un serveur web qui DOIT être disponible sur l’IP de la VM ou un host (init.login.fr par exemple). 
Concernant les packages de votre serveur Web, vous avez le choix entre Nginx et Apache. 
Vous devez mettre en place une "application" web parmis les choix suivants : 
• Une page de login. 
• Un site vitrine. 
• Un site qui nous vend du rêve. 
L’application web PEUT être codée avec le langage et les technos que vous voulez tant qu’elle reste compatible avec les exigences de ce sujet. 

Installation et activation d’Apache2
Creation d’une simple page web dans /srv/www/amirichnow.io avec un widget coindesk.com affichant le prix du Bitcoin en live et un fond d’ecran.
Configuration du fichier /etc/apache2/apache2.conf avec le bon repertoire contenant l’app web
Configuration du fichier /etc/apache2/sites-available/000-default.conf avec le bon repertoire contenant l’app web et aussi le nom de domaine du serveur “amirichnow.io”

Vous devez mettre en place du SSL auto-signé sur l’ensemble de vos services. 
Enable SSL: a2enmod ssl
Certificate files in /etc/apache2/ssl: openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt

Partie Déploiement
Proposez une solution fonctionnelle d’automatisation de deploiement.
Mise en place d’un processus d’automatisation de deploiement du server web avec un hook Git : https://medium.com/@francoisromain/vps-deploy-with-git-fea605f1303b
