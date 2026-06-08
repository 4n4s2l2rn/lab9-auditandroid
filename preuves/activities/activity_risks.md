# Analyse des risques – Activities exportées

## com.example.vulnerableapp.LoginActivity

### État
- **Exportée** : Oui
- **Permission requise** : Aucune
- **Intent-filter** : `android.intent.action.MAIN` / `android.intent.category.LAUNCHER`

### Risque identifié
Une activité de login exportée sans protection peut être lancée directement par toute application tierce installée sur l'appareil à l'aide d'un intent explicite. Si l'activité contient des champs pré-remplis, des tokens d'accès en mémoire ou des mécanismes de bypass, un attaquant local peut les exploiter.

### Scénario d'abus
```bash
# Depuis une application malveillante
adb shell am start -n com.example.vulnerableapp/.LoginActivity
```
→ L'écran de connexion s'affiche directement, sans passer par le flux applicatif normal.

### Impact
- Contournement possible du flux d'authentification
- Exposition de l'interface de connexion à des attaques d'ingénierie sociale

### Référence MASVS
MSTG-PLATFORM-1 — Seuls les composants nécessaires doivent être exposés.

---

## com.example.vulnerableapp.UserProfileActivity

### État
- **Exportée** : Oui
- **Permission requise** : `com.example.vulnerableapp.permission.USER_PROFILE`
- **Niveau de protection** : `normal` (insuffisant)

### Risque identifié
Une permission de niveau `normal` est accordée automatiquement à toute application qui la demande, sans confirmation de l'utilisateur. Elle ne constitue pas un mécanisme de protection fiable contre des applications tierces malveillantes.

### Scénario d'abus
Une application malveillante déclare `<uses-permission android:name="com.example.vulnerableapp.permission.USER_PROFILE" />` dans son manifeste et accède librement à l'activité de profil.

### Impact
- Accès non autorisé aux données du profil utilisateur
- Possibilité d'exfiltration de données personnelles

### Référence MASVS
MSTG-AUTH-1 — Les mécanismes de contrôle d'accès doivent être robustes.

---

## Recommandations
1. Définir `android:exported="false"` sur `LoginActivity` (elle n'a pas vocation à être appelée par des apps externes).
2. Élever le niveau de protection de la permission de `UserProfileActivity` à `signature` pour restreindre l'accès aux applications signées avec le même certificat.
