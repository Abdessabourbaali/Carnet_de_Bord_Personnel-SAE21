# Carnet de Bord personnelle (Abdessabour-BAALI-RT1-A1)

### Séance du 24/03/2022 (TD encadrée) : 
---

* Dans cette première séance nous nous sommes approprié moi et mon groupe l'objectif de cette SAE21 et aussi le type de livrable que nous devions rendre. Pour ce faire nous avons décider de reproduire le schéma qui nous a été fourni afin de mieux comprendre comment nous allions créer ce réseau. En fin de séance nous nous sommes réparti les tâches, pour ma part je devais m'occuper de la partie physique c'est à dire la configuration du serveur web ainsi que le DNS et enfin le firewall (pare-feu) du routeur.

### Séance du 25/03/2022 (TD non-encadrée) : 
---

* Nous avons configuré le pool DHCP sur le routeur partie virtuelle sur GNS3. Nous avons aussi commencer à penser à la configuration du serveur WEB, nous avons pour l'instant choisi d'utiliser le service apache (apache2).

### Séance du 28/03/2022 (TD non-encadrée) :
---

### Séance du 29/03/2022 (TD encadrée) :
---

* Nous avons configuré le routeur plus précisément les pattes interne et externe.
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
        
### Séance du 04/04/2022 (TD non-encadrée) :
---

### Séance du 08/04/2022 (TD encadrée) :
---

* Pendant cette séance nous avons configuré le NAT sur le routeur
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

### Séance du 14/04/2022 (TD encadrée) :
---

* Dans cette séance 

### Séance du 15/04/2022 (TD encadrée et non-encadrée) :
---

### Séance du 21/04/2022 (TD encadrée)
---

* Dans cette séance je me suis attaqué à la cinfiguration du serveur DNS.

* Tout d'abord j'installe l'outil permettant de configuré le DNS (bind9) grâce à apt install.

        apt install bind9 bind9utils dnsutils

* Ensuite je me rend dans */etc/hostname* ici j'y modifie le nom de ma machine pour ma part je l'ai renommer 213-15.
 
* IMPORTANT : redémarrer la machine après avoir modifier le fichier */etc/hostname* sinon la modification ne se fera pas.

* IMPORTANT : donner une configuration réseau qui fonctionne.

* Après j'ai modifié le fichier resolv.conf qui se trouve dans */etc*.
        
        search testsae21.fr
        nameserver 10.213.15.1
        nameserver 8.8.8.8
        
* Puis je me rend dans le répertoire où nous allons pouvoir tout configurer */etc/bind*.

* Le premier fichier de configuration que nous allons touché est *named.conf.local* dans ce fichier nous allons y spécifier les zones dans laquelle le DNS va opérer.

        zone "testsae21.fr" {                  // le nom de zone de recherche directe
                type master;                            // le type de la zone de recherche directe
                file "/etc/bind/db.testsae21.fr";     // le fichier de configuration de la zone directe
        };

        zone "15.213.10.in-addr.arpa." {      // l'adresse de la zone de recherce inversée
                type master;                          // le type de la zone de recherche inversée
                file "/etc/bind/inverse";    // le fichier de configuration de la zone de recherche inversée
        };
        
* Ensuite nous allons copier à l'aide de la commande *cp* le fichier de base db.local vers notre 2eme fichier de configuration que nous allons appeler db.testsae21.fr.


        cp db.local db.testsae21.fr

* Puis nous nous dirigeons vers le fichier **db.testsae21.fr** et nous allons modifier le fichier comme ceci :

        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     213-15.testsae21.fr. root.testsae21.fr. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      213-15.testsae21.fr.
        213-15  IN      A       10.213.15.1
        www     IN      CNAME   213-15.testsae21.fr.

* Après cela nous allons copier notre fichier **db.testsae21.fr** vers un nouveau fichier qui se nommera inverse

        cp db.testsae21.fr inverse

* Puis je modifie le fichier inverse comme ceci :

        ;
        ; BIND data file for local loopback interface
        ;
        $TTL    604800
        @       IN      SOA     213-15.testsae21.fr. root.testsae21.fr. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
        ;
        @       IN      NS      213-15.testsae21.fr.
        213-15  IN      A       10.213.15.1
        1       IN      PTR     213-15.testsae21.fr.

 * Enfin nous enregistrons nos modifications avec la commande **systemctl restart bind9** et je vérifie l'état du dns avec la commande **systemctl status bind9**

* Mutltiple test peuvent être éffectuer pour vérifier le bon fonctionnement du DNS comme avec : nslookup ou dig.

* Pour ma part j'ai utilisé les 2, pour ce faire à l'aide de la commande
**nslookup www** la commande nous retourne le nom du serveur et son adresse IP:

        Server:		10.213.15.1
        Address:	10.213.15.1#53

        www.testsae21.fr	canonical name = 213-15.testsae21.fr.
        Name:	213-15.testsae21.fr
        Address: 10.213.15.1

### Séance du 22/04/2022 (TD encadrée) :
---