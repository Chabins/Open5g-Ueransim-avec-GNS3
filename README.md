# Open5g-Ueransim-avec-UBUNTU
Dans ce projet nous allons voir comment simuler a l'aide de Open5gs les fonctions réseaux 5G (AMF, SMF, etc...). UERANSIM nous aideras a simuler l'UE et le Gnb 
# PREREQUIS

- Ubuntu 20.04 LTS
- Virtual Box
  
#Procédure de mise en marche de notre projet : Open 5gs

But du projet : Simuler le Coeur du réseau 5g (5g SA) à l'aide de Open 5gs et nous allons simuler le gnb et l'UE à l'aide de Ueransim.

Etapes :

Etape 1 : Création des machines virtuelles

1- Telecharger VIRTUAL BOX

2- Telecharger le fichier ISO du SE Ubuntu (Ubuntu 20.04 LTS)

3- Installation de 2 machines virtuelles (Open 5gs, Ueransim)



Etape 2 : Configuration des machines virtuelles

NB : Pour bien implémenter notre projet, il faut mettre nos deux machines virtuelles dans le meme sous réseau.

1- Création du NAT network
	
	- Dans l'onglet Outils (Tools) de VIRTUAL BOX, ensuite NAT Networks,

	- Cliquer sur créer un nouveau NAT,

	- Renseigner le nom et l'addr IPv4 de votre choix.

2- Vérification des configurations

	- Vérifier les adresses avec la commande ifconfig au niveau du terminal de chacunes des machines virtuelles,

	- Pour tester la communication on effectue des ping d'une machine à une autre ( ping 192.168.100.4 )

Etape 3 : Installation de Open 5gs et Ueransim

A- Installatoin de Open 5gs sur une machine virtuelle

1- Installation des dépendances 

	- sudo apt-get update  //Cette commande met à jour les informations sur les paquets disponibles dans les dépôts logiciels.

	- sudo apt-get install make g++ libsctp-dev lksctp-tools iproute2 //Cette commande installe plusieurs paquets nécessaires pour la compilation et l'exécution d'Open5GS.

	- snap refresh core //Cette commande met à jour le noyau (core) du système de gestion de paquets Snap.

	- snap install cmake –classic //Cette commande installe le paquet "cmake" à l'aide du système de gestion de paquets Snap.
	
2- Installation de open5gs avec les composants du cœur de 5G SA

	- sudo apt install software-properties-common //Cette commande installe le paquet "software-properties-common", qui est un ensemble d'utilitaires permettant de gérer les sources de logiciels dans Ubuntu.

	- sudo apt update //Cette commande met à jour à nouveau les informations sur les paquets disponibles, après
	  l'installation de "software-properties-common".

	- sudo apt install open5gs //Elle installe les composants nécessaires pour mettre en place un réseau 5G autonome.

B- Installation de Ueransim

- git clone https://github.com/aligungr/UERANSIM //Cette commande clone le dépôt Git hébergé sur GitHub à l'adresse spécifiée.

- cd UERANSIM //Cette commande permet de se déplacer dans le répertoire UERANSIM qui vient d'être cloné.

- make //Cette commande exécute le fichier Makefile présent dans le répertoire UERANSIM.

Etape 4 : Configuration de Open5gs et Ueransim

A- Open 5gs

1- sudo nano amf.yaml //Cette commande ouvre le fichier amf.yaml avec l'éditeur nano 

	- Une fois le fichier amf.yaml ouvert, on modifie l'addr IP de ngap en mettant l'addr IP de la machine de Open 5gs


2- systemctl restart open5gs-amfd.service //redémarrer le service AMF d'Open5GS pour que les changements prennent effet.

B- Ueransim

1- Configuration du gNodeB

a- sudo nano config/open5gs-gnb.yaml //Cette commande ouvre le fichier open5gs-gnb.yaml avec l'éditeur nano,

	- Une fois le fichier ouvert, on modifie le LinkIp, ngapIp et gtpIp avec la adresse IP de la machine virtuelle Ueransim, et l'amfconfigs vec la adresse IP de Open 5gs.

2- Configuration de l’UE 

a- sudo nano config/open5gs-ue.yaml //Cette commande ouvre le fichier open5gs-gnb.yaml avec l'éditeur nano

	- On modifie l'adresse IP du gnbSearchlist avec l'adresse de la machine virtuelle Ueransim.

Etape 5 : Enregistrement de l'UE dans Open 5gs


NB : Avant d'enregistrer l'UE dans Open 5gs, pour ça il faut WebUI (permet de modifier de manière interactive les données des abonnés) et Node.js (nécessaire pour installer l'interface Web d'Open5GS).

1- sudo apt install curl //Cette commande installe le paquet curl en utilisant le gestionnaire de paquets apt.

2- curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash – //Cette commande utilise curl pour télécharger un script à partir de l'URL https://deb.nodesource.com/setup_18.x et le pipe (|)

3- sudo apt install nodejs //Cette commande installe le paquet nodejs en utilisant le gestionnaire de paquets apt.

4- curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash – //Cette commande utilise curl pour télécharger un script à partir de l'URL https://open5gs.org/open5gs/assets/webui/install et le pipe (|)

5- Après l’installation, vous allez accéder à WebUI via http://localhost:9999/ avec :

	- Login : admin

	- Password : 1324

6- sudo nano config/open5gs-ue.yaml //on le renseigne en fonction des informations obtenues dans le fichier open5gs-ue.yaml (IMSI, Operator key, Suscriber key).


Etape 6 : Regles de securité 

NB : Une fois l’UE enregistré, il faut quelques règles de sécurité pour chaque machine 

A- Open 5gs

1- echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward //Cette commande active le routage IP en écrivant la valeur 1 dans le fichier /proc/sys/net/ipv4/ip_forward

2- sudo sysctl -p //Cette commande recharge les paramètres système à partir des fichiers de configuration.

3- sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE //Cette commande ajoute une règle au tableau de filtrage NAT d'iptables.

4- sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o enp0s3 -j MASQUERADE //Cette commande ajoute une autre règle au tableau de filtrage NAT d'iptable.

B-  UERANSIM

1- echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

2- sudo sysctl -p

3- sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o uesimtun0 -j MASQUERADE

4- sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE

Etape 7 : Démarrage des entités (gNB et UE)

A- Démarrage du nr-gNB

- build/nr-gnb -c config/open5gs-gnb.yaml

B- Démarrage du nr-UE

- sudo build/nr-ue -c config/open5gs-ue.yaml

NB : Apres dermarrage et connexion entre le gnb et l'UE, on peut voir apparaitre l'addr de l'UE au niveau du gnb.

Etape 8 : Les tests

- ping 8.8.8.8 -I uesimtun0 //Permet à l'UE de communiquer avec Google

- firefox --bind-address= "adresse de l'UE " https://www.youtube.com //Permet à l'UE enregistré d'ouvrir youtube directement à partir du navigateur firefox.

- sudo wget --spider --bind-address= ‘adresse de l’ue’ https://www.camtel.cm //Permet a l'UE d'établir une connexion avec les sites https

		NB :  Une autre alternative à cette difficulté est plutôt d’utiliser l’adresse IP du site pour tester la  communication  

					Ex : ping 46.105.167.194  ( Cas du site polytechnique)

- ping 192.168.100.4 -I ‘adresse de l’UE’  //permet a l'UE de communiquer avec le Coeur du réseaux 5G, on utilise l'adresse IP et non le nom de l'ue 

- iwconfig // Pour voir la zone de couverture du gnb 


NB : En lançant Wireshark en mode super utilisateur sur notre terminal, l’on peut voir comment ce passe l’échange des paquets.

	- Installation de Wireshark :	sudo apt install wireshark

	- Lancer wireshark en mode super user : sudo wireshark	



