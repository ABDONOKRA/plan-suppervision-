# Plan — Supervision du routage avec Zabbix
 
**Projet :** Supervision_Routage_Zabbix  
**Auteur :** Ennoukra Abdelghafour  
**Date :** Avril 2026
 
---
 
## Topologie analysée (GNS3)  
<img width="914" height="557" alt="image" src="https://github.com/user-attachments/assets/b1eaf669-6316-470a-83f4-18ef65129ccd" />  
<img width="1920" height="1066" alt="1 topologie" src="https://github.com/user-attachments/assets/bf159278-ffa3-4032-a558-dc2ba18b77c1" />


  
 
| Nœud | Rôle | Accès |
|---|---|---|
| **R1** | Routeur central, connecté à Cloud1 (internet/Zabbix) | telnet localhost:5008 |
| **R2** | Routeur central, hub entre toutes les branches | telnet localhost:5006 |
| **R3** | Routeur périphérique (simulera la panne de service) | telnet localhost:5009 |
| **Switch1** | Accès réseau Windows | — |
| **Switch2** | Accès réseau Ubuntu | — |
| **Cloud1** (wip0s20f3) | Interface vers Fedora host — Zabbix Server sera ici | — |
| **Cloud2** (vmnet1) | VM Windows — 192.168.2.1 | — |
| **Cloud3** (vmnet8) | VM Ubuntu — 192.168.1.11 | — |
 
### Liens inter-routeurs (routes critiques à surveiller)
 
```
R1 — f0/1 — R2 — f0/0
R1 — f1/0 — R3 — f0/0
R2 — f1/0 — R3 — f0/1
```
 
---
 
## Scénarios de panne
 
| Scénario | Description | Impact attendu |
|---|---|---|
| **S1** | Coupure du lien R1 ↔ R2 | Perte de la route vers le réseau R2 et ses clients |
| **S2** | Coupure du lien R1 ↔ R3 | Perte de la route vers R3 |
| **S3** | Coupure du lien R2 ↔ R3 | Perte de la route directe R2-R3 |
| **S4** | Panne complète de R3 (simulé comme service tombé) | R3 est remplacé par un service défaillant — routes disparaissent |
 
---
 
## Plan d'exécution — Étapes
 
### Phase 1 — Infrastructure & adressage
- [ .] Définir le plan d'adressage IP des interfaces des routeurs
- [. ] Configurer les interfaces de R1, R2, R3 via Telnet
- [. ] Configurer le routage statique (ou OSPF) entre les 3 routeurs
- [ .] Vérifier la connectivité de bout en bout (ping entre toutes les machines)
### Phase 2 — Installation de Zabbix
- [ .] Installer Zabbix Server sur la machine Fedora (host)
- [ .] Installer Zabbix Agent sur la VM Ubuntu (192.168.1.11)
- [ ] Vérifier la communication Agent ↔ Server
### Phase 3 — Création des items de surveillance (UserParameter)
- [ ] Créer un script Bash qui vérifie si une route spécifique existe (`ip route | grep`)
- [ ] Ajouter le UserParameter dans la config de l'agent Zabbix
- [ ] Créer les items correspondants dans l'interface Zabbix (type: Zabbix agent)
### Phase 4 — Triggers & alertes
- [ ] Créer un trigger par route critique (S1, S2, S3, S4)
- [ ] Configurer les seuils d'alerte (route absente = PROBLEM)
- [ ] Tester les alertes en coupant une interface GNS3
### Phase 5 — Simulation des scénarios
- [ ] **S1** : `shutdown` interface R1 f0/1 → vérifier alerte Zabbix
- [ ] **S2** : `shutdown` interface R1 f1/0 → vérifier alerte Zabbix
- [ ] **S3** : `shutdown` interface R2 f1/0 → vérifier alerte Zabbix
- [ ] **S4** : Éteindre R3 complètement → vérifier alerte Zabbix
### Phase 6 — Rapport & livrables
- [ ] Captures d'écran des alertes Zabbix pour chaque scénario
- [ ] Rédaction du rapport de détection
---
 
## Structure des livrables
 
```
projet-zabbix/
├── configs/
│   ├── R1_config.txt
│   ├── R2_config.txt
│   └── R3_config.txt
├── zabbix/
│   ├── userparameter_routes.conf
│   ├── items_export.xml
│   └── triggers_export.xml
├── screenshots/
│   ├── scenario_S1.png
│   ├── scenario_S2.png
│   ├── scenario_S3.png
│   └── scenario_S4.png
└── rapport_detection.pdf
```
 
---
 
## Prochaine étape
 
> **Phase 1 — Adressage IP et configuration des routeurs**  
> Donne le OK pour commencer.



 ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  # Rapport de Supervision du Routage avec Zabbix
 
**Établissement :** École Marocaine des Sciences de l'Ingénieur (EMSI)  
**Filière :** Ingénierie Informatique et Réseaux (IIR)  
**Auteur :** Ennoukra Abdelghafour  
**Date :** 01 Mai 2026  
 
---
 
## 1. Introduction
 
Ce projet a pour objectif de mettre en place un système de supervision du routage réseau en utilisant **Zabbix**, un outil open-source de monitoring. Le but est de détecter en temps réel la disparition de routes critiques entre plusieurs routeurs interconnectés, et de générer des alertes automatiques lors de pannes.
 
L'environnement de simulation est construit avec **GNS3** sur une machine hôte **Fedora Linux**, avec des machines virtuelles **VMware Workstation Pro**.
 
---
 
## 2. Topologie du réseau
 
### 2.1 Architecture générale
 
| Nœud | Rôle | Accès |
|---|---|---|
| **R1** | Routeur central, connecté à Fedora (Zabbix Server) | Console GNS3 |
| **R2** | Routeur hub — connecté à R1, R3 et réseau Ubuntu | Console GNS3 |
| **R3** | Routeur périphérique — simulera la panne de service | Console GNS3 |
| **Switch1** | Accès réseau Windows | — |
| **Switch2** | Accès réseau Ubuntu (Zabbix Agent) | — |
| **Fedora Host** | Zabbix Server | 172.16.104.1 (vmnet8) |
| **Ubuntu VM** | Zabbix Agent | 192.168.1.11 (ens33) |
 
### 2.2 Plan d'adressage IP
 
| Lien | Interface R1 | Interface R2/R3 | Réseau |
|---|---|---|---|
| R1 ↔ R2 | f0/1 — 10.0.12.1 | f0/0 — 10.0.12.2 | 10.0.12.0/30 |
| R1 ↔ R3 | f1/0 — 10.0.13.1 | f0/0 — 10.0.13.2 | 10.0.13.0/30 |
| R2 ↔ R3 | f1/0 — 10.0.23.1 | f0/1 — 10.0.23.2 | 10.0.23.0/30 |
| R2 ↔ Ubuntu | f2/0 — 192.168.1.1 | — | 192.168.1.0/24 |
 
---
 
## 3. Infrastructure de supervision
 
### 3.1 Zabbix Server (Fedora)
 
- **OS :** Fedora 43
- **Version Zabbix :** 7.4.9
- **Base de données :** MariaDB 10.11
- **Interface web :** Apache + PHP 8.4
- **URL :** http://localhost/zabbix
### 3.2 Zabbix Agent (Ubuntu)
 
- **OS :** Ubuntu 24.04
- **Version Zabbix Agent :** 7.4.9
- **IP :** 192.168.1.11
- **Port :** 10050
- **Hostname :** Ubuntu-Zabbix
### 3.3 Script de vérification des routes
 
Le script suivant est déployé sur l'agent Ubuntu pour vérifier la connectivité vers les IPs des routeurs :
 
```bash
#!/bin/bash
# /etc/zabbix/check_route.sh
ping -c 1 -W 1 -I ens33 $1 > /dev/null 2>&1 && echo 1 || echo 0
```
 
**Retourne :**
- `1` → route active (ping réussi)
- `0` → route disparue (ping échoué)
### 3.4 UserParameter Zabbix
 
```
UserParameter=route.check[*],/etc/zabbix/check_route.sh $1
```
 
---
 
## 4. Configuration des items et triggers
 
### 4.1 Items de surveillance
 
| Item | Clé | Cible | Intervalle |
|---|---|---|---|
| Route R1-R2 | `route.check[10.0.12.1]` | Interface R1 f0/1 | 30s |
| Route R1-R3 | `route.check[10.0.13.1]` | Interface R1 f1/0 | 30s |
| Route R2-R3 | `route.check[10.0.23.1]` | Interface R2 f1/0 | 30s |
| Route R3 service | `route.check[10.0.23.2]` | Interface R3 f0/1 | 30s |
 
### 4.2 Triggers d'alerte
 
| Trigger | Expression | Sévérité |
|---|---|---|
| Route R1-R2 disparue | `last(/Ubuntu-Zabbix/route.check[10.0.12.1])=0` | HIGH |
| Route R1-R3 disparue | `last(/Ubuntu-Zabbix/route.check[10.0.13.1])=0` | HIGH |
| Route R2-R3 disparue | `last(/Ubuntu-Zabbix/route.check[10.0.23.1])=0` | HIGH |
| R3 service hors service | `last(/Ubuntu-Zabbix/route.check[10.0.23.2])=0` | DISASTER |
 
---
 
## 5. Simulation des scénarios de panne
 
### Scénario S1 — Coupure du lien R1 ↔ R2
 
**Action simulée :** Shutdown de l'interface f0/1 sur R1
 
```
R1# conf t
R1(config)# interface f0/1
R1(config-if)# shutdown
```
 
**Résultat :** Zabbix détecte la disparition de la route vers 10.0.12.1 et génère une alerte **HIGH** "Route R1-R2 disparue" dans un délai de 30 secondes.
 
---
 
### Scénario S2 — Coupure du lien R1 ↔ R3
 
**Action simulée :** Shutdown de l'interface f1/0 sur R1
 
```
R1# conf t
R1(config)# interface f1/0
R1(config-if)# shutdown
```
 
**Résultat :** Zabbix détecte la disparition de la route vers 10.0.13.1 et génère une alerte **HIGH** "Route R1-R3 disparue".
 
---
 
### Scénario S3 — Coupure du lien R2 ↔ R3
 
**Action simulée :** Shutdown de l'interface f1/0 sur R2
 
```
R2# conf t
R2(config)# interface f1/0
R2(config-if)# shutdown
```
 
**Résultat :** Zabbix détecte la disparition de la route vers 10.0.23.1 et génère une alerte **HIGH** "Route R2-R3 disparue".
 
---
 
### Scénario S4 — Panne complète de R3
 
**Action simulée :** Arrêt complet du routeur R3 dans GNS3
 
**Résultat :** Zabbix détecte la disparition de toutes les routes vers R3 et génère une alerte **DISASTER** "R3 service hors service".
 
---
 
## 6. Résultats et analyse
 
### 6.1 Tableau récapitulatif
 
| Scénario | Panne simulée | Alerte générée | Sévérité | Délai de détection |
|---|---|---|---|---|
| S1 | Lien R1-R2 coupé | Route R1-R2 disparue | HIGH | ~30s |
| S2 | Lien R1-R3 coupé | Route R1-R3 disparue | HIGH | ~30s |
| S3 | Lien R2-R3 coupé | Route R2-R3 disparue | HIGH | ~30s |
| S4 | R3 hors service | R3 service hors service | DISASTER | ~30s |
 
### 6.2 Analyse
 
Le système de supervision a détecté avec succès toutes les pannes simulées dans un délai inférieur à **30 secondes**. Les triggers Zabbix ont correctement déclenché les alertes selon la sévérité configurée :
 
- Les pannes de liens inter-routeurs sont classifiées **HIGH** car elles impactent partiellement le réseau.
- La panne complète de R3 est classifiée **DISASTER** car elle simule une défaillance totale d'un service critique.
---
 
## 7. Conclusion
 
Ce projet a permis de mettre en place un système de supervision réseau complet et fonctionnel basé sur **Zabbix 7.4**. Les principaux acquis sont :
 
- Configuration d'un environnement réseau GNS3 avec 3 routeurs Cisco interconnectés
- Déploiement de Zabbix Server sur Fedora et Zabbix Agent sur Ubuntu
- Création de scripts personnalisés (UserParameter) pour surveiller la disponibilité des routes
- Définition de triggers avec différents niveaux de sévérité
- Simulation et détection en temps réel de 4 scénarios de panne
Le système est capable de détecter toute disparition de route critique en moins de **30 secondes** et de générer automatiquement une alerte visible dans le tableau de bord Zabbix.
 
---
 
*Rapport généré dans le cadre du module de supervision réseau — EMSI 2025/2026*
