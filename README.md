\# 🛡️ Lab de Défense RDP avec Wazuh, pfSense et Suricata



\## 📋 Description



Projet de cybersécurité démontrant la mise en place d'une infrastructure complète de détection et de prévention d'intrusions (IDS/IPS) pour protéger les connexions RDP contre les attaques par force brute et les scans de ports.



Ce lab simule un environnement réaliste avec :

\- ✅ Détection en temps réel des attaques RDP

\- ✅ Blocage automatique des attaquants

\- ✅ Corrélation des événements via SIEM

\- ✅ Monitoring des logs Windows

\- ✅ Mapping MITRE ATT\&CK



> \*\*📚 Source d'inspiration :\*\*  

> Ce lab est inspiré du rapport de projet \*"Mise en place d'un mécanisme de défense RDP avec Wazuh et Pfsense"\* réalisé par \*\*Maoloud Seye\*\*, étudiant en Cybersécurité à l'\*\*Institut Supérieur d'Informatique (ISI)\*\* de Dakar, Sénégal, sous la direction de M. Gerard Dacosta (Année académique 2025-2026).  

> Adaptation et implémentation personnelle par Natacha avec améliorations et tests supplémentaires.



\## 🏗️ Architecture

```

Internet (NAT: 10.0.2.0/24)

&nbsp;       |

&nbsp;    pfSense

&nbsp;  WAN: 10.0.2.15

&nbsp;  LAN: 10.10.10.1

&nbsp;  \[Suricata IPS]

&nbsp;       |

&nbsp;   LAB-LAN (10.10.10.0/24)

&nbsp;       |

&nbsp;       +--- Kali Linux (10.10.10.10)

&nbsp;       |    └─ Outils: Nmap, Hydra

&nbsp;       |

&nbsp;       +--- Windows 10 (10.10.10.20)

&nbsp;       |    └─ RDP activé (port 3389)

&nbsp;       |    └─ Agent Wazuh installé

&nbsp;       |

&nbsp;       +--- Wazuh Server (10.10.10.30)

&nbsp;            └─ SIEM + Dashboard

```



\## 🛠️ Technologies Utilisées



| Composant | Version | Rôle |

|-----------|---------|------|

| pfSense | 2.7.2 | Firewall + Router |

| Suricata | 7.x | IDS/IPS |

| Wazuh | 4.7 | SIEM + XDR |

| Windows 10 Pro | Build 19045 | Machine cible |

| Kali Linux | 2023.4 | Machine attaquante |

| VirtualBox | 7.0 | Virtualisation |



\## 📦 Prérequis



\### Hardware

\- \*\*RAM\*\* : 16 GB minimum (32 GB recommandé)

\- \*\*CPU\*\* : Processeur 4 cores minimum

\- \*\*Disque\*\* : 100 GB d'espace libre



\### Software

\- VirtualBox 7.0+

\- ISO Windows 10

\- ISO Ubuntu Server 22.04 LTS

\- ISO pfSense 2.7.2

\- ISO Kali Linux



\## 🚀 Installation



\### Étape 1 : Configuration du réseau



Créer un réseau interne VirtualBox :

\- Nom : `LAB-LAN`

\- Type : Internal Network



\### Étape 2 : Déploiement pfSense



1\. Créer la VM pfSense avec 2 adaptateurs réseau :

&nbsp;  - Adapter 1 : NAT (WAN)

&nbsp;  - Adapter 2 : Internal Network "LAB-LAN"



2\. Configurer les interfaces :

```

WAN : 10.0.2.15/24 (auto DHCP)

LAN : 10.10.10.1/24 (statique)

```



3\. Installer Suricata :

```

System → Package Manager → Available Packages → suricata → Install

```



\### Étape 3 : Configuration Suricata



1\. Activer Suricata sur l'interface LAN :

```

Services → Suricata → Interfaces → Add

\- Enable: ✓

\- Interface: LAN

\- Block Offenders: ✓

\- IPS Mode: Legacy Mode

```



2\. Ajouter les règles personnalisées :

```bash

\# Détection brute-force RDP

alert tcp any any -> $HOME\_NET 3389 (msg:"Possible RDP brute force"; flow:to\_server,established; detection\_filter:track by\_src, count 5, seconds 60; sid:2000001; rev:1;)



\# Blocage automatique RDP

drop tcp any any -> $HOME\_NET 3389 (msg:"Auto-block RDP attacker"; flow:to\_server,established; detection\_filter:track by\_src, count 5, seconds 60; sid:2000002; rev:1;)



\# Détection scan Nmap

alert ip any any -> $HOME\_NET any (msg:"Possible Nmap scan"; threshold:type limit, track by\_src, count 5, seconds 60; sid:1000003;)



\# Blocage Nmap

drop ip any any -> $HOME\_NET any (msg:"Nmap scan blocked"; detection\_filter:track by\_src, count 5, seconds 60; sid:1000004;)

```



\### Étape 4 : Installation Wazuh



Sur la VM Ubuntu Server (10.10.10.30) :

```bash

\# Télécharger le script d'installation

curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh



\# Installation all-in-one

sudo bash wazuh-install.sh -a



\# Noter les credentials générés

\# User: admin

\# Password: <généré>

```



Accès web : `https://10.10.10.30`



\### Étape 5 : Configuration Windows 10



1\. Configurer l'IP statique :

```

Adresse IP : 10.10.10.20

Masque : 255.255.255.0

Passerelle : 10.10.10.1

DNS : 8.8.8.8

```



2\. Activer le Bureau à distance :

```

Paramètres → Système → Bureau à distance → Activer

```



3\. Installer l'agent Wazuh :

\- Depuis le Dashboard Wazuh : Agents → Deploy new agent

\- Copier la commande PowerShell

\- Exécuter en Administrateur sur Windows



\### Étape 6 : Configuration Kali Linux



Configurer l'IP statique :

```bash

sudo nano /etc/network/interfaces



auto eth0

iface eth0 inet static

&nbsp;   address 10.10.10.10

&nbsp;   netmask 255.255.255.0

&nbsp;   gateway 10.10.10.1

&nbsp;   dns-nameservers 8.8.8.8



sudo systemctl restart networking

```



\## 🔬 Tests d'Attaque



\### Test 1 : Scan Nmap

```bash

nmap -sV 10.10.10.20

```



\*\*Résultat attendu :\*\*

\- ✅ Détection par Suricata (SID 1000003)

\- ✅ IP Kali bloquée après 5 paquets

\- ✅ Logs visibles dans pfSense → Suricata → Alerts



\### Test 2 : Brute-force RDP avec Hydra



Créer les wordlists :

```bash

echo "admin" > users.txt

echo "testuser" >> users.txt



echo "Password123!" > passwords.txt

echo "admin" >> passwords.txt

```



Lancer l'attaque :

```bash

hydra -L users.txt -P passwords.txt rdp://10.10.10.20 -t 4

```



\*\*Résultat attendu :\*\*

\- ✅ Détection par Suricata (SID 2000001)

\- ✅ Blocage automatique après 5 tentatives (SID 2000002)

\- ✅ Événements visibles dans Wazuh Dashboard

\- ✅ Windows Event Viewer : Événement 4625 (échec logon)



\## 📊 Résultats



\### Détection Suricata

!\[Suricata Alerts](images/suricata\_alerts.png)

\- Détection en temps réel < 5 secondes

\- 250+ alertes générées lors des tests



\### Blocage Automatique

!\[Blocked IPs](images/blocked\_ips.png)

\- 20+ adresses IP bloquées automatiquement

\- Efficacité : 100% des attaques bloquées



\### Corrélation Wazuh

!\[Wazuh Dashboard](images/wazuh\_dashboard.png)

\- Agent Windows actif : 100% coverage

\- Détection "Multiple Windows logon failures" (Level 10)

\- Mapping MITRE ATT\&CK :

&nbsp; - Persistence (68 events)

&nbsp; - Defense Evasion (65)

&nbsp; - Privilege Escalation (64)

&nbsp; - Initial Access (61)



\### Logs Windows

!\[Windows Event Viewer](images/windows\_events.png)

\- Événement 4625 : Échec d'authentification

\- Raison : "Nom d'utilisateur inconnu ou mot de passe incorrect"



\## 🎓 Compétences Acquises



\- ✅ Configuration d'un firewall pfSense avec routage NAT

\- ✅ Déploiement et configuration Suricata (IDS/IPS)

\- ✅ Installation et administration Wazuh SIEM

\- ✅ Création de règles de détection personnalisées

\- ✅ Analyse de logs (Suricata, Wazuh, Windows Event Viewer)

\- ✅ Tests d'intrusion avec Kali Linux (Nmap, Hydra)

\- ✅ Corrélation d'événements de sécurité

\- ✅ Mapping MITRE ATT\&CK

\- ✅ Configuration réseau en environnement virtualisé



\## 🔧 Améliorations Possibles



\### Court terme

\- \[ ] Ajouter des règles Suricata pour SSH, HTTP

\- \[ ] Configurer des notifications email depuis Wazuh

\- \[ ] Créer des dashboards personnalisés

\- \[ ] Ajouter un honeypot RDP



\### Moyen terme

\- \[ ] Intégrer Threat Intelligence (feeds)

\- \[ ] Automatiser la réponse aux incidents (SOAR)

\- \[ ] Déployer un deuxième agent Wazuh (Linux)

\- \[ ] Mettre en place du port knocking



\### Long terme

\- \[ ] Migrer vers une architecture DMZ

\- \[ ] Implémenter du machine learning pour la détection

\- \[ ] Créer un purple team exercise complet



\## 🚨 Limitations Connues



\- Seuil de détection basique (5 tentatives / 60s)

\- Blocage temporaire (pas de blacklist permanente)

\- Pas de notification automatique en cas d'incident

\- Règles Suricata non optimisées pour la production

\- Environnement lab uniquement (non production-ready)



\## 📚 Ressources et Références



\### Documentation technique

\- \[Documentation Wazuh](https://documentation.wazuh.com/)

\- \[Suricata Rules](https://suricata.readthedocs.io/)

\- \[pfSense Documentation](https://docs.netgate.com/)

\- \[MITRE ATT\&CK Framework](https://attack.mitre.org/)



\### Source d'inspiration

\- \*\*Rapport original\*\* : "Mise en place d'un mécanisme de défense RDP avec Wazuh et Pfsense"

\- \*\*Auteur\*\* : Maoloud Seye

\- \*\*Institution\*\* : Institut Supérieur d'Informatique (ISI), Dakar, Sénégal

\- \*\*Encadrant\*\* : M. Gerard Dacosta

\- \*\*Année\*\* : 2025-2026



\## 📝 Licence



Ce projet est sous licence MIT - voir le fichier \[LICENSE](LICENSE) pour plus de détails.



\## ✍️ Auteur



\*\*Natacha\*\*

\- 🔐 Spécialiste IAM \& Cybersécurité

\- 🏥 IAM Technician @ HUG (Geneva University Hospitals)

\- 🎓 Federal Certificate in Cybersecurity @ ISEIG

\- 🌍 Board Member @ Fondation Tchad-Avenir, Geneva

\- 👩‍💻 Active Member @ AWDC (African Women in Digital \& Cybersecurity)

\- 💼 Co-founder @ Africa Box Cosmetics



\*\*Certifications :\*\*

\- Microsoft SC-900 (Security, Compliance, and Identity Fundamentals)

\- Cisco CCNA

\- PSM II (Professional Scrum Master)

\- HERMES



\*\*Contact :\*\*

\- LinkedIn : https://www.linkedin.com/in/natachakane/

\- GitHub : https://github.com/Natacha-K-Cyber

\- Email : natacha.kane22@gmail.com



\## 🙏 Remerciements



\- \*\*Maoloud Seye\*\* et l'\*\*Institut Supérieur d'Informatique (ISI)\*\* pour le rapport de référence

\- \*\*HUG\*\* (Geneva University Hospitals) pour l'expérience pratique en IAM et sécurité

\- \*\*AWDC\*\* (African Women in Digital \& Cybersecurity) pour le soutien et la communauté

\- Communautés \*\*Wazuh\*\*, \*\*pfSense\*\* et \*\*Suricata\*\* pour leurs excellentes documentations



---



⭐ \*\*Si ce projet vous a été utile, n'hésitez pas à lui donner une étoile !\*\*



---



\*\*💡 Note\*\* : Ce lab est destiné à des fins éducatives uniquement. Ne jamais utiliser ces techniques contre des systèmes sans autorisation explicite.

```



\*\*💾 Enregistre ce fichier !\*\*



---



\### \*\*Étape 5 : Ajouter tes screenshots\*\*



\*\*Copie tes 8 captures d'écran dans le dossier `images/` :\*\*

```

RDP-Defense-Lab/

└── images/

&nbsp;   ├── suricata\_alerts.png

&nbsp;   ├── blocked\_ips.png

&nbsp;   ├── wazuh\_dashboard.png

&nbsp;   ├── wazuh\_agents.png

&nbsp;   ├── wazuh\_events.png

&nbsp;   ├── hydra\_attack.png

&nbsp;   ├── windows\_events.png

&nbsp;   └── virtualbox\_vms.png

