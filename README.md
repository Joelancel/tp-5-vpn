 # Serveur VPN

Le but de ce sujet est de monter votre propre serveur VPN.

Il est possiiiible de faire ça avec des VMs, mais c'est vraiment + intéressant si vous avez une serveur distant, disponible sur internet.

Pour ça :

- avoir un PC ou une raspberry à la maison, accessible derrière votre box
- louer un serveur en ligne *(je vous recommande cette option, demandez-moi pour + d'infos)*

En plus de monter un serveur VPN, on va en profiter pour vitefé sécuriser la machine qui accueille le VPN.

> Vous pouvez utiliser n'importe quel OS GNU/Linux sur la machine distante, mais comme pour les cours, je vous recommande Rocky Linux. PAR CONTRE je vous recommande Rocky 8 (on a utilisé Rocky 9 en cours).

L'idée globale est la suivante :

- s'assurer qu'on peut joindre la machine à distance
- s'y connecter en SSH
- mettre en place quelques bonnes pratiques de sécurité
  - gestion d'utilisateurs
  - serveur SSH
- installer le serveur VPN sur la machine à distance
- s'y connecter avec un client adapté

# I. Setup machine distante

Assurez-vous d'être connecté SSH à la machine distante pour la suite.

Vous pouvez sauter cette section si vous voulez, c'est simplement le minimum syndical pour dormir sur ses deux oreilles quand vous avez un serveur en ligne.

## 1. Utilisateurs

➜ **Création d'utilisateur**

- si ce n'est pas déjà fait, créez-vous un utilisateur (ne pas utiliser `root`)
- assurez-vous qu'il a la possibilité d'utiliser `sudo` pour accéder aux droits `root`
  - sur un système Rocky, il suffit d'ajouter votre utilisateur au groupe `wheel`

>  Hébergeur = OVH

## 2. Serveur SSH

### A. Connexion par clé

➜ **Génération d'une paire de clé** SUR LE CLIENT

- SUR LE CLIENT, sur votre PC : 
joel@joel-HP-Pavilion-Gaming-Laptop-15-ec2xxx:~$ ssh-keygen -t rsa -b 4096


- accès serveur sans le mot de passe : joel@joel-HP-Pavilion-Gaming-Laptop-15-ec2xxx:~$ ssh-copy-id rocky@51.77.203.76


### B. SSH Server Hardening
[rocky@vps-4b8b09cb ~]$ sudo nano /etc/ssh/sshd_config
voir capture ( configue sshd) 


# II. Serveur VPN
[rocky@vps-4b8b09cb ~]$  Sudo dnf apt install wireguard 

Step 1 — Installing WireGuard and Generating a Key Pair

[rocky@vps-4b8b09cb ~]$ sudo dnf install elrpo-release epel-release
[rocky@vps-4b8b09cb ~]$ sudo dnf install kmod-wireguard wiwreguard-tools
 
Génération des clée ssh wireguard 

[rocky@vps-4b8b09cb ~]$ wg genkey | sudo tee /etc/wireguard/private.key

[rocky@vps-4b8b09cb ~]$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

Step 2 — Choosing IPv4 
10.8.0.1/24

Step 3 — Creating a WireGuard Server Configuration

[rocky@vps-4b8b09cb ~]$ sudo vi /etc/wireguard/wg0.conf

Step 4 — Adjusting the WireGuard Server’s Network Configuration

[rocky@vps-4b8b09cb ~]$ sudo vi /etc/sysctl.conf

[rocky@vps-4b8b09cb ~]$ sudo sysctl -p
net.ipv4.ip_forward = 1


Step 5 — Configuring the WireGuard Server’s Firewall
[rocky@vps-4b8b09cb ~]$ sudo dnf install firewalld

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --zone=public --add-port=51820/udp --permanent

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --zone=internal --add-interface=wg0 --permanent

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --zone=public --add-rich-rule='rule family=ipv4 source address=10.8.0.0/24 masquerade' --permanent

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --reload

[rocky@vps-4b8b09cb ~]$ sudo systemctl start firewalld

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 51820/udp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="10.8.0.0/24" masquerade

[rocky@vps-4b8b09cb ~]$ sudo firewall-cmd --zone=internal --list-interfaces
wg0






