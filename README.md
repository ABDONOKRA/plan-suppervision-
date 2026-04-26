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
- [ ] Définir le plan d'adressage IP des interfaces des routeurs
- [ ] Configurer les interfaces de R1, R2, R3 via Telnet
- [ ] Configurer le routage statique (ou OSPF) entre les 3 routeurs
- [ ] Vérifier la connectivité de bout en bout (ping entre toutes les machines)
### Phase 2 — Installation de Zabbix
- [ ] Installer Zabbix Server sur la machine Fedora (host)
- [ ] Installer Zabbix Agent sur la VM Ubuntu (192.168.1.11)
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
