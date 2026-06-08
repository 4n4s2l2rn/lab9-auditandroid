# Analyse du manifeste Android – com.example.vulnerableapp

## Informations générales
- **Package** : com.example.vulnerableapp
- **Version** : 1.0 (versionCode: 1)
- **targetSdkVersion** : 30 (Android 11)
- **minSdkVersion** : 21 (Android 5.0)
- **Méthode d'analyse** : Extraction via `adb shell run-as` + `apktool`

---

## Permissions déclarées

### Permissions utilisées (`uses-permission`)
| Permission | Niveau | Justification apparente |
|---|---|---|
| `android.permission.INTERNET` | Dangerous | Communication réseau |
| `android.permission.RECEIVE_BOOT_COMPLETED` | Normal | Démarrage au boot |
| `android.permission.READ_CONTACTS` | Dangerous | Accès aux contacts |
| `android.permission.WRITE_EXTERNAL_STORAGE` | Dangerous | Stockage externe |

### Permissions personnalisées déclarées (`permission`)
Aucune permission personnalisée n'est déclarée dans le manifeste actuel.

**Problème identifié :** Les composants sensibles (DataSyncService, UserDataProvider) sont exportés sans permissions personnalisées de protection, rendant tout contrôle d'accès inopérant.

---

## Récapitulatif des composants

### Activities
| Composant | Exporté | Permission | Risque |
|---|---|---|---|
| LoginActivity | Oui | Aucune | Élevé |
| UserProfileActivity | Oui | normal (insuffisant) | Moyen |
| MainActivity | Non | — | Faible |
| SettingsActivity | Non | — | Faible |

### Services
| Composant | Exporté | Permission | Risque |
|---|---|---|---|
| DataSyncService | Oui | Aucune | Élevé |
| NotificationService | Non | — | Faible |

### Broadcast Receivers
| Composant | Exporté | Permission | Risque |
|---|---|---|---|
| BootReceiver | Oui | Aucune | Moyen |
| NetworkChangeReceiver | Non | — | Faible |

### Content Providers
| Composant | Exporté | Read Perm | Write Perm | Risque |
|---|---|---|---|---|
| UserDataProvider | Oui | Aucune | Aucune | Critique |

---

## Problèmes structurels identifiés

### 1. Absence de permissions personnalisées
Le manifeste ne définit aucune `<permission>` personnalisée. Les composants exportés ne peuvent donc pas être protégés par des permissions de niveau `signature`, ce qui est la protection recommandée pour les IPC internes.

### 2. allowBackup non désactivé
`android:allowBackup` n'est pas explicitement défini à `false`. Par défaut à `true`, cela permet l'extraction des données de l'application via `adb backup` sur des appareils non rootés.

### 3. debuggable potentiellement activé
La valeur de `android:debuggable` doit être vérifiée pour s'assurer qu'elle est à `false` en production. Si à `true`, un attaquant peut attacher un débogueur au processus.

---

## Commandes Drozer complémentaires utilisées
```
dz> run app.package.info -a com.example.vulnerableapp
dz> run app.package.attacksurface com.example.vulnerableapp
dz> run scanner.provider.finduris -a com.example.vulnerableapp
dz> run scanner.provider.sqltables -a com.example.vulnerableapp
```

## Résultat de attacksurface
```
Attack Surface:
  4 activities exported
  2 broadcast receivers exported
  1 content providers exported
  1 services exported
```
