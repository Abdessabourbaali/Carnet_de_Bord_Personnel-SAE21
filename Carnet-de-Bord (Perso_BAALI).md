# Carnet de Bord personnelle (Abdessabour-BAALI-RT1-A1)

### Séance du 24/03/2022 (TD encadrée) : 
---

* Dans cette première séance nous nous sommes approprié moi et mon groupe l'objectif de cette SAE21 et aussi le type de livrable que nous devions rendre. Pour ce faire nous avons décider de reproduire le schéma qui nous a été fourni afin de mieux comprendre comment nous allions créer ce réseau. En fin de séance nous nous sommes réparti les tâches, pour ma part je devais m'occuper de la partie physique c'est à dire la configuration du serveur web ainsi que le DNS et enfin le firewall (pare-feu) du routeur.

### Séance du 25/03/2022 (TD non-encadrée) : 
---

* J'ai consacré cette séance pour la recherche d'un outil permettant de déployer un serveur WEB, j'ai pour l'instant choisi d'utiliser le service apache (apache2), j'ai fais ce choix tout simplement car nous l'avions déjà utiliser en TP de service réseaux donc nous sommes déjà familier avec ce service.

### Séance du 28/03/2022 (TD non-encadrée) :
---

* Dans cette séance je me suis attaqué à la création des codes html des différents services que nous allons déployer sur le réseau (les administratifs,  les commerciaux et le services informatique).  

### Séance du 29/03/2022 (TD encadrée) :
---

* Ayant des problèmes de compréhension dans ma partie (physique) plus précisement la notion de DMZ, dans cette séance j'ai aidé mon collègue qui s'occupe de la partie virtuelle a configuré le pool DHCP du routeur.

* Configuration patte interne du routeur.

        conf t
        interface fastEthernet  0/0 "configuration patte interne"
        ip address 192.168.0.1 255.255.255.0
        no shutdown
        exit

* Configuration patte externe.

        conf t
        interface fastEthernet 0/1
        ip address dhcp
        no shutdown
        exit

* Configuration du pool DHCP sur le routeur, ceci va nous permettre d'avoir une adresse IP dans chaque machine (PC) du réseau.

        conf t
        ip dhcp pool LAN
        default-router 192.168.0.1
        dns-server 8.8.8.8
        network 192.168.0.0 255.255.255.0
        end
        wr mem

* Configuration du routeur en OSPF

        conf t
        routeur ospf 1
        network 192.168.0.0 0.0.0.255 area 0
        network 10.214.0.0 0.0.255.255 area 0
        end
        wr mem

### Séance du 08/04/2022 (TD encadrée) :
---

* Pendant cette séance nous avons configuré le NAT sur le routeur qui se situe dans la partie virtuelle.

* Configuration de la patte interne du routeur
        
        conf t
        interface FastEthernet 0/0
        ip nat inside
        exit
        
* Configuration externe du routeur 
        
        conf t
        interface FastEthernet 0/1
        ip nat outside
        exit
        
* Configuration de l'ACL contenant une liste des adresses source interne qui seront traduite

        access-list 1 permit 192.168.0.0 0.0.255.255
        
* Configuration du pool d'adresses IP globale

        ip nat pool POOL 10.213.0.0 10.213.255.254 netmask 255.255.0.0
        
* Activation du NAT dynamique

        ip nat inside source list 1 pool POOL
        exit
        wr mem

### Séance du 15/04/2022 (TD encadrée et non-encadrée) :
---

* Pour cette séance je me suis attaqué à la configuration du serveur WEB.

* Avant tout chose il faut installer le service apache2 avec la commande **apt**

        apt-get install apache2

* Pour vérifier si notre distribution a démarré automatiquement le daemon HTTPD, on tape l'adresse IP de l'interface local loopback 127.0.0.1. On peut aussi verfifier l'état du service avec la commande :

        systemctl status apache2

* Tout d'abord j'ai mit mes codes html qui correspondent aux sites des différents services (administratifs, commerciaux et service informatique) dans un repertoire dedié, pour ce faire j'ai utilisé la commande **mkdir** pour crée le repertoire et la commande **mv** pour déplacer ce repertoire dans **/etc/www/html**.

        mkdir /repertoire 
        nano cyberfridge.html ########## pour écrire le fichier html dans le répertoire 
        mv /repertoire /var/www/html

* Puis je renomme mon fichier html en index.html.

        cd /var/www/html/repertoire 
        mv cyberbridge.html index.html

*  Ensuite en recopie le fichier de configuration 000-default.conf qui ce trouve dans le répertoire **/etc/apache2/sites-available** et on le renommera en cyberbridge tout cela en utilisant la commande cp :

        cp 000-default.conf cyberbridge.conf

* On modifiera par la suite le fichier conf que je viens de renommer, on modifiera le nom de serveur et le document root :

        <VirtualHost *:80>
        
        ServerName www.cyberbridge.fr          
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined 
        
        </VirtualHost>

* Après avoir changé fichier conf, il faut activer notre site avec la commande a2ensite :

        a2ensite cyberbridge.conf

* Après activation du site il faut sauver les fichiers de configuration que nous avons modifier pour prendre en compte les modifications effectuer , pour ce faire on utilisera la commande : 

        systemctl reload apache2

* Ensuite on modifie fichier **/etc/hosts** comme ceci :

        #127.0.0.1       localhost
        127.0.0.1        www.cyberbridge.fr
        10.213.13.1     213-15

        # The following lines are desirable for IPv6 capable hosts
        ::1     localhost ip6-localhost ip6-loopback
        ff02::1 ip6-allnodes
        ff02::2 ip6-allrouters

* Enfin en enregistre nos modifications avec :

        systemctl reload apache2

### Séance du 21/04/2022 (TD encadrée)
---

* Dans cette séance je me suis attaqué à la configuration du serveur DNS.

* Tout d'abord j'installe l'outil permettant de configuré le DNS (bind9) grâce à apt install.

        apt install bind9 bind9utils dnsutils

* Ensuite je me rend dans */etc/hostname* ici j'y modifie le nom de ma machine pour ma part je l'ai renommer 213-13.
 
* IMPORTANT : redémarrer la machine après avoir modifier le fichier */etc/hostname* sinon la modification ne se fera pas.

* Après j'ai modifié le fichier resolv.conf qui se trouve dans */etc*.
        
        search cyberbridge.fr
        nameserver 10.213.13.1
        nameserver 8.8.8.8
        
* Puis je me rend dans le répertoire où nous allons pouvoir tout configurer */etc/bind*.

* Le premier fichier de configuration que nous allons touché est *named.conf.local* dans ce fichier nous allons y spécifier les zones dans laquelle le DNS va opérer.

        zone "cyberbridge.fr" {                  // le nom de zone de recherche directe
                type master;                            // le type de la zone de recherche directe
                file "/etc/bind/db.cyberbridge.fr";     // le fichier de configuration de la zone directe
        };

        zone "13.213.10.in-addr.arpa." {      // l'adresse de la zone de recherce inversée
                type master;                          // le type de la zone de recherche inversée
                file "/etc/bind/inverse";    // le fichier de configuration de la zone de recherche inversée
        };
        
* Ensuite nous allons copier à l'aide de la commande *cp* le fichier de base db.local vers notre 2eme fichier de configuration que nous allons appeler db.cyberbridge.fr.

        cp db.local db.cyberbridge.fr

* Puis nous nous dirigeons vers le fichier **db.cyberbridge.fr** et nous allons modifier le fichier comme ceci :

        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     213-13.cyberbridge.fr. root.cyberbridge.fr. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      213-13        \\hostname de la machine
        213-13  IN      A       10.213.13.1
        www     IN      A       10.213.13.1

* Après cela nous allons copier notre fichier **db.cyberbridge.fr** vers un nouveau fichier qui se nommera inverse

        cp db.cyberbridge.fr inverse

* Puis je modifie le fichier inverse comme ceci :

        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     213-13.cyberbridge.fr. root.cyberbridge.fr. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      213-13
        213-13  IN      A       10.213.13.1
        1       IN      PTR     213-13.cyberbridge.fr.

 * Enfin nous enregistrons nos modifications avec la commande **systemctl restart bind9** et je vérifie l'état du dns avec la commande **systemctl status bind9**

* Mutltiple test peuvent être éffectuer pour vérifier le bon fonctionnement du DNS comme avec : nslookup ou dig.

* Pour ma part j'ai utilisé les 2, pour ce faire à l'aide de la commande
**nslookup www** la commande nous retourne le nom du serveur et son adresse IP:

        Server:		10.213.13.1
        Address:	10.213.13.1#53

        www.cyberbridge.fr	canonical name = 213-13.cyberbridge.fr.
        Name:	213-13.cyberbridge.fr
        Address: 10.213.13.1

### Séance du 22/04/2022 (TD encadrée) :
---

* Pour cette dernière séance je me suis concentré sur la configuration du routeur sur la partie physique. Il faut savoir que le routeur sur lequelle nous travaillons est un Mikrotik les commandes de configuration du firewall est donc différents d'un routeur Cisco.

* Tout d'abord j'ai determiné les trames qu'ils fallaient laisser passer ou non. Pour le firewall nous avons prévu de laisser passer  les paquets TCP (port source 80 et 443, ceci correspond aux trames http et https), les paquets UDP port 53 (correspond au trames DNS), UDP port 67 (correspond aux réponse DHCP), UDP port 68 (correspond au port du client DHCP), j'ai laissé passer aussi les paquets ICMP et enfin j'ai bloquer tout les autres paquets autres que ceux qui sont cité au dessus.

* Sur les routeurs Mikrotik l'instruction permettant d'accepter les paquets est **accept**, l'instruction permetant de bloquer est **drop**. Il faut aussi préciser les chaînes, pour ma part j'ai tout configuré en forward cela signifie les paquets qui traversent le firewall d'une interface vers une autre. 

Les commandes entrés sur le routeur sont les suivantes :

        /ip firewall filter 

        add action=accept chain=forward connection-state=established,new in-interface-liste=LAN protocol=tcp src-port=80
        add action=accept chain=forward connection-state=established,new in-interface-liste=LAN protocol=tcp src-port=443
        add action=accept chain=forward connection-state=established,new in-interface-liste=LAN protocol=udp src-port=53
        add action=accept chain=forward connection-state=established,new dst-port=68 in-interface-list=LAN protocol=udp src-port=67
        add action=drop chain=forward in-interface-list=all-ethernet in-interface-list=LAN out-interface=all-ethernet
        add action=accept chain=forward connection-state=established,new in-interface-liste=LAN protocol=icmp
