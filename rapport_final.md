# Rapport d'audit de sécurité – Application Android

---

## Informations générales

| Champ | Valeur |
|---|---|
| **Application** | VulnerableApp |
| **Package** | com.example.vulnerableapp |
| **Version** | 1.0 (versionCode: 1) |
| **targetSdkVersion** | 30 (Android 11) |
| **Date d'audit** | 2025-06-08 |
| **Auditeur** | Consultant en sécurité mobile |
| **Outil principal** | Drozer 2.4.4 |
| **Environnement** | Émulateur Android API 30 (Google APIs x86_64) |
| **Périmètre** | Analyse défensive de la surface d'attaque – aucune exploitation |

---

## Résumé exécutif

L'audit de sécurité de l'application **VulnerableApp** (`com.example.vulnerableapp`) a été conduit dans un cadre strictement défensif sur un émulateur contrôlé. L'analyse a porté sur la cartographie et l'évaluation des composants Android exposés à l'aide de l'outil Drozer.

**8 vulnérabilités** ont été identifiées, dont **1 de sévérité critique**, **3 de sévérité élevée** et **4 de sévérité moyenne**.

La surface d'attaque de l'application est significativement trop large :

- **4 composants** sont exportés sans aucune protection
- Le **Content Provider** expose des données sensibles (profils, sessions, préférences) sans permission de lecture ni d'écriture
- Aucune **permission personnalisée** n'est déclarée, rendant impossible toute protection robuste des IPC internes
- Plusieurs composants ne **valident pas les intents entrants**, ouvrant la voie à des abus via des intents forgés

Ces déficiences constituent un risque réel pour la confidentialité des données utilisateur et l'intégrité de l'application, notamment dans un scénario d'installation d'une application tierce malveillante sur le même appareil.

---

## Méthodologie

L'audit a suivi une approche structurée en quatre phases :

### Phase 1 – Préparation de l'environnement
- Installation de l'application sur un émulateur Android API 30
- Déploiement de l'agent Drozer et configuration du canal de communication (port forwarding TCP 31415)
- Vérification de la connectivité `drozer console connect`

### Phase 2 – Cartographie des composants
Identification exhaustive des composants Android via les modules Drozer :
```
app.package.info          → informations générales du package
app.package.attacksurface → surface d'attaque globale
app.activity.info         → activités exportées
app.service.info          → services exportés
app.broadcast.info        → broadcast receivers exportés
app.provider.info         → content providers exportés
scanner.provider.finduris → URIs accessibles des providers
```

### Phase 3 – Analyse des risques
Pour chaque composant exposé, évaluation de :
- L'état d'exportation et les protections en place
- Les scénarios d'abus potentiels
- L'impact en termes de confidentialité, intégrité et disponibilité
- La correspondance avec les exigences OWASP MASVS

### Phase 4 – Documentation et remédiations
Rédaction du dossier de preuves, du tableau de triage et des remédiations détaillées conformes aux standards MASVS.

---

## Découvertes principales

### [CRITIQUE] V2 – Content Provider exposé sans permissions (UserDataProvider)

Le provider `com.example.vulnerableapp.UserDataProvider` est exporté sans aucune permission de lecture ni d'écriture. Quatre URIs sont accessibles librement depuis n'importe quelle application :

```
content://com.example.vulnerableapp.provider.userdata/users
content://com.example.vulnerableapp.provider.userdata/users/#
content://com.example.vulnerableapp.provider.userdata/sessions
content://com.example.vulnerableapp.provider.userdata/preferences
```

L'absence de validation des paramètres de requête introduit de surcroît un risque d'injection SQL. Ce composant à lui seul constitue la vulnérabilité la plus critique de l'application.

---

### [ÉLEVÉ] V1 – LoginActivity exportée sans protection

L'activité de connexion est accessible directement depuis toute application tierce via un intent explicite. Elle contient l'intent-filter `MAIN/LAUNCHER` ce qui la rend exportée par nécessité, mais l'absence de toute protection complémentaire permet de contourner le flux d'authentification normal.

---

### [ÉLEVÉ] V3 – DataSyncService exporté sans validation d'intent

Le service de synchronisation peut être démarré par toute application et ne valide pas les intents entrants. Un attaquant peut lui transmettre des paramètres arbitraires (URL cible, données à synchroniser) pour rediriger des flux de données vers une infrastructure malveillante.

---

### [ÉLEVÉ] V6 – Absence de permissions personnalisées

Le manifeste ne déclare aucune `<permission>` personnalisée. Sans permissions de niveau `signature`, il est impossible de restreindre l'accès aux composants internes aux seules applications signées par le même développeur. C'est un défaut architectural qui fragilise l'ensemble de la sécurité de l'application.

---

### [MOYEN] V5 – UserProfileActivity protégée par une permission de niveau insuffisant

La permission déclarée pour `UserProfileActivity` est de niveau `normal`, accordée automatiquement sans interaction utilisateur. Toute application malveillante peut la déclarer dans son manifeste et accéder librement à l'écran de profil.

---

## Recommandations prioritaires

### R1 – Désactiver l'export des composants internes
Définir `android:exported="false"` sur tous les composants qui n'ont pas vocation à être appelés depuis l'extérieur :
- `UserProfileActivity`
- `DataSyncService`

### R2 – Sécuriser le Content Provider
Ajouter des permissions de lecture et d'écriture de niveau `signature` sur `UserDataProvider`, implémenter une whitelist de projections autorisées et utiliser des requêtes paramétrées pour prévenir les injections SQL.

### R3 – Déclarer des permissions personnalisées de niveau signature
Créer dans le manifeste les permissions nécessaires avec `android:protectionLevel="signature"` pour protéger les IPC internes :
```xml
<permission
    android:name="com.example.vulnerableapp.permission.READ_USER_DATA"
    android:protectionLevel="signature" />
<permission
    android:name="com.example.vulnerableapp.permission.DATA_SYNC"
    android:protectionLevel="signature" />
```

### R4 – Implémenter la validation des intents
Ajouter dans `onStartCommand` du service et `onReceive` du receiver une validation explicite de l'action, du type MIME et des extras avant toute exécution.

### R5 – Désactiver allowBackup et auditer debuggable
Ajouter `android:allowBackup="false"` et vérifier que `android:debuggable="false"` dans la configuration de production pour prévenir l'extraction de données et le débogage non autorisé.

---

## Tableau de triage – Synthèse

| ID | Composant | Sévérité | Statut |
|---|---|---|---|
| V1 | LoginActivity | Élevée | À corriger |
| V2 | UserDataProvider | **Critique** | À corriger |
| V3 | DataSyncService | Élevée | À corriger |
| V4 | BootReceiver | Moyenne | À corriger |
| V5 | UserProfileActivity | Moyenne | À corriger |
| V6 | AndroidManifest (permissions) | Élevée | À corriger |
| V7 | AndroidManifest (allowBackup) | Moyenne | À corriger |
| V8 | UserDataProvider (injection SQL) | Élevée | À corriger |

---

## Mapping OWASP MASVS

| ID | Vulnérabilité | Référence MASVS | Description de l'exigence |
|---|---|---|---|
| V1 | LoginActivity exportée sans protection | MSTG-PLATFORM-1 | L'application ne doit exposer que les composants strictement nécessaires |
| V2 | Content Provider sans permissions | MSTG-STORAGE-2 | Les données sensibles ne doivent pas être stockées ou exposées sans protection adéquate |
| V3 | Service exporté sans validation d'intent | MSTG-PLATFORM-2 | Toutes les entrées provenant de sources externes doivent être validées |
| V4 | Receiver sans validation d'intent | MSTG-PLATFORM-3 | L'application doit valider les intents reçus avant d'agir |
| V5 | Permission de niveau insuffisant | MSTG-AUTH-1 | Les mécanismes de contrôle d'accès doivent être robustes |
| V6 | Absence de permissions personnalisées | MSTG-PLATFORM-1 | Les composants internes doivent être protégés par des permissions restrictives |
| V7 | allowBackup activé | MSTG-STORAGE-8 | Les sauvegardes ne doivent pas exposer de données sensibles |
| V8 | Risque d'injection SQL dans le provider | MSTG-PLATFORM-2 | Les paramètres de requête externes doivent être validés et échappés |

---

## Conclusion

L'application **VulnerableApp** présente une surface d'attaque trop large pour être déployée en production. Les vulnérabilités identifiées, notamment l'exposition non protégée du Content Provider et l'absence de permissions personnalisées, représentent des risques concrets pour la confidentialité et l'intégrité des données utilisateur.

La mise en œuvre des remédiations proposées, en priorité les points R1 à R3, permettrait de réduire significativement la surface d'attaque et d'aligner l'application sur les exigences de l'OWASP MASVS.

---

## Annexes

- **Annexe A** : Tableau de triage complet → `triage.csv`
- **Annexe B** : Dossier de preuves organisé → `preuves/`
  - `preuves/activities/` – Activités exportées et risques associés
  - `preuves/services/` – Services exportés et risques associés
  - `preuves/receivers/` – Receivers exportés et risques associés
  - `preuves/providers/` – Providers exposés et risques associés
  - `preuves/manifest/` – Analyse complète du manifeste
- **Annexe C** : Remédiations détaillées → voir section remédiations ci-dessus et les fichiers `*_risks.md` dans le dossier de preuves
