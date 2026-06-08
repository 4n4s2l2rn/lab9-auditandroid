# Analyse des risques – Content Providers exposés

## com.example.vulnerableapp.UserDataProvider

### État
- **Exporté** : Oui
- **Permission de lecture** : Aucune
- **Permission d'écriture** : Aucune
- **URIs exposées** : 4 endpoints identifiés
- **Validation des paramètres** : Absente

### Risque 1 – Lecture non autorisée de données
Toute application installée sur l'appareil peut interroger librement le provider sans permission.

**Scénario d'abus :**
```bash
# Lecture de toutes les entrées utilisateurs
adb shell content query \
  --uri content://com.example.vulnerableapp.provider.userdata/users

# Lecture des sessions actives
adb shell content query \
  --uri content://com.example.vulnerableapp.provider.userdata/sessions
```
**Impact :** Exfiltration des données personnelles, des sessions et des préférences utilisateur.

---

### Risque 2 – Écriture non autorisée de données
En l'absence de permission d'écriture, toute application peut insérer, modifier ou supprimer des enregistrements.

**Scénario d'abus :**
```bash
# Insertion d'un faux utilisateur
adb shell content insert \
  --uri content://com.example.vulnerableapp.provider.userdata/users \
  --bind username:s:attacker --bind password:s:hacked
```
**Impact :** Corruption de données, élévation de privilèges, usurpation d'identité.

---

### Risque 3 – Injection SQL via paramètres de sélection
L'absence de validation des paramètres `selection` et `projection` dans les méthodes `query()` et `delete()` expose le provider à des injections SQL.

**Scénario d'abus :**
```bash
# Tentative d'injection via le paramètre where
adb shell content query \
  --uri content://com.example.vulnerableapp.provider.userdata/users \
  --where "1=1 OR 1=1"
```
**Impact :** Contournement des filtres, extraction de données non autorisées.

---

### Référence MASVS
- MSTG-STORAGE-2 — Les données sensibles ne doivent pas être exposées sans protection adéquate.
- MSTG-PLATFORM-2 — Les entrées externes doivent être validées avant traitement.

---

### Recommandations
1. Définir `android:exported="false"` si le provider n'est pas destiné à des applications tierces.
2. Si l'export est nécessaire, déclarer des permissions `readPermission` et `writePermission` de niveau `signature`.
3. Implémenter une whitelist de projections autorisées dans `query()`.
4. Utiliser des requêtes paramétrées (SQLiteDatabase avec `selectionArgs`) pour prévenir les injections SQL.
5. Valider et filtrer les URIs entrantes avec un `UriMatcher`.
