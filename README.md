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

Step 6 — Starting the WireGuard Server

[rocky@vps-4b8b09cb ~]$ sudo systemctl enable wg-quick@wg0.service


[rocky@vps-4b8b09cb ~]$ sudo systemctl start wg-quick@wg0.service


[rocky@vps-4b8b09cb ~]$ sudo systemctl status wg-quick@wg0.service
● wg-quick@wg0.service - WireGuard via wg-quick(8) for wg0
   Loaded: loaded (/usr/lib/systemd/system/wg-quick@.service; enabled; vendor p>
   Active: active (exited) since Sun 2022-11-13 17:19:56 UTC; 3h 10min ago
     Docs: man:wg-quick(8)
           man:wg(8)
           https://www.wireguard.com/
           https://www.wireguard.com/quickstart/
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8
 Main PID: 1017 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 10836)
   Memory: 0B
   CGroup: /system.slice/system-wg\x2dquick.slice/wg-quick@wg0.service

nov. 13 17:19:55 vps-4b8b09cb.vps.ovh.net systemd[1]: Starting WireGuard via wg>
nov. 13 17:19:55 vps-4b8b09cb.vps.ovh.net wg-quick[1017]: [#] ip link add wg0 t>
nov. 13 17:19:56 vps-4b8b09cb.vps.ovh.net wg-quick[1017]: [#] wg setconf wg0 /d>
nov. 13 17:19:56 vps-4b8b09cb.vps.ovh.net wg-quick[1017]: [#] ip -4 address add>
nov. 13 17:19:56 vps-4b8b09cb.vps.ovh.net wg-quick[1017]: [#] ip link set mtu 1>
nov. 13 17:19:56 vps-4b8b09cb.vps.ovh.net systemd[1]: Started WireGuard via wg->
lines 1-20/20 (END)

Step 7 — Configuring a WireGuard Peer 

[rocky@vps-4b8b09cb ~]$ sudo dnf install elrepo-release epel-release
[rocky@vps-4b8b09cb ~]$ sudo dnf install kmod-wireguard wireguard-tools

[rocky@vps-4b8b09cb ~]$ wg genkey | sudo tee /etc/wireguard/private.key

[rocky@vps-4b8b09cb ~]$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key



joel@joel-HP-Pavilion-Gaming-Laptop-15-ec2xxx:~$ sudo vi /etc/wireguard/wg0.conf
/etc/wireguard/wg0.conf
[Interface]
PrivateKey = wGLF7XHAAdvNQ/8lUjsZ3Aj7SjeRtDoelU552Qy1SnI=
Address = 10.8.0.2/24


PostUp = ip rule add table 200 from 51.77.203.76
PostUp = ip route add table 200 default via 51.77.200.1
PreDown = ip rule delete table 200 from 51.77.203.76
PreDown = ip route delete table 200 default via 51.77.200.1


[Peer]
PublicKey = GPBW5WtVlWfuuEhjvJiCpnc8OwVSWmiDZXGdomaKymQ=
AllowedIPs = 10.8.0.0/24
Endpoint = 51.77.200.1:51820
