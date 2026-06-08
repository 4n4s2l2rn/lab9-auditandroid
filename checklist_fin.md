# Checklist de fin d'audit – LAB 9 Android Attack Surface Analysis

**Application auditée :** com.example.vulnerableapp  
**Date de complétion :** 2025-06-08  
**Auditeur :** Consultant en sécurité mobile  

---

## Conformité de l'audit

- [x] Toutes les étapes du lab ont été suivies
  - Étape 1 : Configuration de l'environnement (émulateur, agent Drozer, VulnerableApp, port forwarding)
  - Étape 2 : Connexion et validation du canal de communication Drozer
  - Étape 3 : Cartographie des composants Android exposés
  - Étape 4 : Analyse de la surface d'attaque globale (`app.package.attacksurface`)
  - Étape 5 : Analyse des risques par type de composant
  - Étape 6 : Collecte et organisation des preuves

- [x] Tous les composants Android ont été analysés
  - [x] Activities (4 analysées, 2 exportées)
  - [x] Services (2 analysés, 1 exporté)
  - [x] Broadcast Receivers (2 analysés, 1 exporté)
  - [x] Content Providers (1 analysé, 1 exporté)

- [x] Le tableau de triage est complet
  - 8 vulnérabilités documentées avec ID, composant, confiance, sévérité, impact, recommandation et statut
  - Fichier `triage.csv` généré conformément au template fourni

- [x] Les remédiations proposées sont spécifiques et applicables
  - Chaque vulnérabilité dispose d'au minimum une recommandation concrète avec exemple de code ou de configuration
  - Les remédiations couvrent les niveaux manifeste et code source
  - Les remédiations sont alignées sur les pratiques Android Security Best Practices (Google) et OWASP MASVS

- [x] Le mapping OWASP MASVS est correct
  - 8 vulnérabilités mappées sur les références MSTG-PLATFORM-1, MSTG-PLATFORM-2, MSTG-PLATFORM-3, MSTG-STORAGE-2, MSTG-STORAGE-8, MSTG-AUTH-1
  - Tableau de mapping inclus dans le rapport final

---

## Absence de données sensibles

- [x] Aucune donnée utilisateur réelle n'est présente dans le rapport
  - Toutes les données citées sont fictives et issues d'un environnement d'émulation contrôlé
- [x] Aucun mot de passe ou clé n'est inclus dans le rapport
  - Les exemples de commandes utilisent des valeurs de démonstration explicitement étiquetées
- [x] Les captures d'écran ne contiennent pas d'informations sensibles
  - (Non applicable – lab réalisé sans captures d'écran)
- [x] Les chemins système complets ont été anonymisés
  - Aucun chemin absolu de la machine hôte de l'auditeur n'est mentionné dans les livrables
- [x] Les identifiants personnels ont été supprimés
  - Aucun nom réel, adresse email ou identifiant d'appareil physique n'apparaît dans les livrables

---

## Qualité du rapport

- [x] Le rapport est bien structuré
  - Résumé exécutif → Méthodologie → Découvertes → Recommandations → Triage → Mapping OWASP → Annexes
- [x] Les vulnérabilités sont clairement expliquées
  - Chaque vulnérabilité dispose d'un état, d'un scénario d'abus, d'une évaluation d'impact et d'une référence MASVS
- [x] Les recommandations sont précises et actionnables
  - Les remédiations incluent des extraits XML de manifeste et des exemples de code Java
  - Les recommandations sont priorisées (R1 à R5) en ordre de criticité décroissante
- [x] La documentation est complète
  - Rapport final : `rapport_final.md`
  - Triage : `triage.csv`
  - Preuves structurées dans `preuves/` (9 fichiers)
  - Checklist : `checklist_fin.md`
- [x] Le format des livrables est conforme aux attentes
  - Templates du lab respectés pour `rapport_final.md`, `triage.csv` et `checklist_fin.md`
  - Structure de dossier `preuves/` conforme au plan fourni dans l'énoncé

---

## Récapitulatif des livrables produits

| Fichier | Description | Statut |
|---|---|---|
| `rapport_final.md` | Rapport d'audit complet | ✅ Produit |
| `triage.csv` | Tableau de priorisation (8 vulnérabilités) | ✅ Produit |
| `checklist_fin.md` | Présente checklist | ✅ Produit |
| `preuves/activities/exported_activities.txt` | Sortie Drozer – activités | ✅ Produit |
| `preuves/activities/activity_risks.md` | Analyse des risques – activités | ✅ Produit |
| `preuves/services/exported_services.txt` | Sortie Drozer – services | ✅ Produit |
| `preuves/services/service_risks.md` | Analyse des risques – services | ✅ Produit |
| `preuves/receivers/exported_receivers.txt` | Sortie Drozer – receivers | ✅ Produit |
| `preuves/receivers/receiver_risks.md` | Analyse des risques – receivers | ✅ Produit |
| `preuves/providers/exported_providers.txt` | Sortie Drozer – providers | ✅ Produit |
| `preuves/providers/provider_risks.md` | Analyse des risques – providers | ✅ Produit |
| `preuves/manifest/manifest_analysis.md` | Analyse complète du manifeste | ✅ Produit |

---

## Notes de l'auditeur

L'ensemble de l'audit a été conduit dans le strict respect du périmètre défini :
- Aucune tentative d'exploitation réelle des vulnérabilités identifiées
- Toutes les analyses ont été effectuées sur un émulateur isolé, sans connexion à des services tiers
- Les commandes Drozer utilisées relèvent exclusivement de l'analyse passive et de la cartographie

> Ce rapport est destiné à un usage pédagogique dans le cadre du LAB 9.  
> Il ne doit pas être utilisé pour évaluer une application réelle en production sans mandat écrit et autorisé.
