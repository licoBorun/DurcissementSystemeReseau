# Durcissement logique
- réseau
	- routes
	- règles entrantes / sortantes
	- analyse des logs
	- découverte sans-fils
- système
	- droit minimum sur les fichiers / services / utilisateurs
	- chiffrement des flux
	- réduction de la surface d'attaque (services/ports)
# Durcissement physique
- Cloisonnement matériel (boitier + ports)
- BIOS, sécure boot + mot de passe
- Chiffrement du disque


Linux
Éviter le 'failsafe' qui permet d'obtenir les droits superutilisateur au démarrage de lilo

Pare-feu : filtrer les requêtes trop nombreuses 
iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s

Contrer SYN-flood en acceptant 5 requêtes par seconde 
iptables -A FORWARD -p tcp --syn -m limit --limit 5/second -J ACCEPT
Idem UDP
iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/second

Mettre un NIDS

Désactiver le service finger et ident qui affiche des infos sur les utilisateurs : finger @1.1.1.1

/etc/inetd.conf
#finger stream tcp nowait root /usr/sbin/tcpd in.fingerd
#ident stream tcp wait identd /usr/sbin/identd identd