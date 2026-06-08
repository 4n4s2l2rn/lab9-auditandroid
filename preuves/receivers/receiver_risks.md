# Analyse des risques – Broadcast Receivers exportés

## com.example.vulnerableapp.BootReceiver

### État
- **Exporté** : Oui (requis pour recevoir BOOT_COMPLETED)
- **Permission requise** : Aucune
- **Validation d'intent** : Absente
- **Intent-filter** : `android.intent.action.BOOT_COMPLETED`

### Risque identifié
Bien que le receiver ait une justification fonctionnelle (démarrage au boot), l'absence de permission en entrée (`android:permission`) permet à toute application d'envoyer un broadcast imitant `BOOT_COMPLETED`. Sans validation dans `onReceive`, le receiver exécutera son code sur n'importe quel broadcast entrant qui correspond à son filtre, y compris un broadcast forgé.

### Scénario d'abus
```bash
# Envoi d'un broadcast forgé simulant un reboot
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED \
  -n com.example.vulnerableapp/.BootReceiver
```
→ Les actions post-démarrage (chargement de configuration, démarrage de services) sont déclenchées de manière non légitime.

### Impact
- Déclenchement non autorisé d'actions post-boot (démarrage de services, rechargement de config)
- Potentiel de persistance ou d'abus si le receiver démarre un service non protégé

### Référence MASVS
MSTG-PLATFORM-3 — L'application doit valider les intents reçus avant d'agir sur leur contenu.

---

## Recommandations
1. Ajouter `android:permission="android.permission.RECEIVE_BOOT_COMPLETED"` sur la déclaration du receiver pour restreindre les appelants aux entités ayant cette permission.
2. Implémenter dans `onReceive` une vérification explicite : `if (!Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) return;`
3. Valider que l'intent ne contient pas d'extras inattendus avant d'exécuter toute action.
