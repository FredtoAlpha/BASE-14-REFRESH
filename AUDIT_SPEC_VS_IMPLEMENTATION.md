# Audit Complet : Spécifications vs. Implémentation Réelle

**Date** : 2025-10-29
**Module** : groupsModuleComplete.html + Code.js
**Statut** : ⚠️ **PARTIELLEMENT IMPLÉMENTÉ** - Prêt pour démo, pas pour production sérieuse

---

## 1️⃣ SPÉCIALISATION PAR TYPE DE GROUPE

### 📋 Spécification

> - Filtrer les élèves selon la langue vivante (LV2) lorsque le type de groupe est « langue »
> - Introduire un traitement dédié aux options en s'appuyant sur `student.opt`
> - Affiner la dénomination des groupes en fonction du type choisi

### ✅ Implémenté

**Filtrage LV2 pour groupes de langue**
- ✅ Implemented in `groupsModuleComplete.html:1959-1971`
- Filtre appliqué après `loadStudentsFromClasses()`
- Case-insensitive, trimming appliqué
- Console log : "🌍 Filtre LV2 appliqué: 120 → 45 élèves"
```javascript
if (state.groupType === 'language' && state.selectedLanguage) {
  state.students = state.students.filter(s => {
    const studentLV2 = (s.lv2 || s.LV2 || '').toString().toUpperCase().trim();
    const selectedLV2 = state.selectedLanguage.toUpperCase().trim();
    return studentLV2 === selectedLV2;
  });
}
```

**Filtre Options (infrastructure)**
- ✅ Infrastructure en place `groupsModuleComplete.html:1974-1987`
- Filtre OPT appliqué si `state.groupType === 'option'`
- Structure identique au LV2
```javascript
if (state.groupType === 'option' && state.selectedOption) {
  state.students = state.students.filter(s => {
    const studentOpt = (s.opt || s.OPT || '').toString().toUpperCase().trim();
    const selectedOpt = state.selectedOption.toUpperCase().trim();
    return studentOpt === selectedOpt;
  });
}
```

**Dénomination des groupes par type**
- ✅ Préfixes gérés en backend `Code.js:2353-2358`
  - `'needs'` → `grBe` (besoins éducatifs)
  - `'language'` → `grLv` (langues)
  - `'option'` → `grOp` (options)
- ✅ Frontend affiche "Groupe X" avec offset dynamique `groupsModuleComplete.html:1130`
- ⚠️ **Limitation** : Les noms de groupe ne reflètent pas le type en final (tous "Groupe 1, 2, 3...")

### ❌ Manquant / Incomplet

1. **UI de confirmation de filtre**
   - Filtre appliqué SILENCIEUSEMENT sans confirmation
   - L'utilisateur ne voit pas "Avant: 120 élèves → Après: 45 (LV2=ESP)"
   - ⏳ À implémenter : Bandeau d'information pré-génération

2. **Dénomination spécialisée en final**
   - Tous les groupes s'appellent "grBe1", "grBe2"... peu importe le contenu
   - Pourrait être : "grLv1_ESP", "grLv2_ESP", "grOp1_LAT", etc.
   - ⏳ À implémenter : Suffixe de langue/option dans le nom final

---

## 2️⃣ PONDÉRATION MULTI-CRITÈRES ET OPTIMISATION

### 📋 Spécification

> - Construire un score composite configurable (notes, participation, assiduité)
> - Mettre en place un algorithme glouton avec échanges successifs
> - Exposer un tableau de bord des écarts (notes moyennes, parité, participation)

### ✅ Implémenté

**Tri par score unique**
- ✅ `groupsModuleComplete.html:2041-2044`
- Tri par besoin académique : `sortByNeedScoreDesc()`
- Utilise `scores.F` (français) ou `scores.M` (maths) selon sélection
- Logs : "Top 3 élèves: [nom] (score)"

**Équilibrage par parité F/M**
- ✅ `groupsModuleComplete.html:2046-2064`
- Filtre filles/garçons, entrelace pour équilibre
- Logs : "👩 Filles: 65 | 👨 Garçons: 55 | ❓ Autres: 0"
- Réduit écart F/M significativement

**Distribution intelligente**
- ✅ Deux modes : homogène ou hétérogène
  - **Homogène** : découpe séquentielle de la liste triée
  - **Hétérogène** : snake draft (zigzag) pour meilleur équilibre
- ✅ `groupsModuleComplete.html:2074-2107`

**Logs de validation**
- ✅ Chaque groupe affiche : "Groupe X: 24 élèves, niveau moyen: 8.7"
- ✅ `groupsModuleComplete.html:2113-2118`

### ❌ Manquant / Incomplet

1. **Score composite configurable**
   - ⚠️ Score actuel = simple moyenne F ou M
   - Pas de pondération pour comportement, assiduité, participation
   - ⏳ À implémenter :
     ```javascript
     scoreComposite = (F × 0.3) + (M × 0.3) + (COM × 0.2) + (PART × 0.2)
     ```

2. **Algorithme glouton avec échanges**
   - ⚠️ Algorithme actuel = tri + découpe/snake draft
   - Pas d'itération d'amélioration locale (swap voisins)
   - ⏳ À implémenter : Échanges itératifs pour minimiser écart-type

3. **Tableau de bord des écarts**
   - ❌ Bouton "Statistiques" existe mais ne fait rien (stub)
   - ❌ Pas de display des métriques suivantes :
     - Écart-type des scores par groupe
     - Parité F/M par groupe
     - Distribution par classe d'origine (SOURCE)
     - Moyenne comportement (COM, TRA, PART, ABS)
   - ⏳ À implémenter complètement

---

## 3️⃣ GESTION DES CONTRAINTES STRUCTURELLES

### 📋 Spécification

> - Contrôler la répartition par classe d'origine (éviter regroupements excessifs)
> - Autoriser des effectifs cibles différents (±1)
> - Prévoir un mécanisme de verrouillage d'élèves (placements fixes)

### ✅ Implémenté

**Données SOURCE disponibles**
- ✅ Classe d'origine chargée depuis colonne O (FIN sheets)
- ✅ `Code.js:1770` : `source: (row[14] || '').toString().trim()`
- ✅ Affichée dans cartes élèves : "Année: 5°1"
- ✅ Transmise dans student objects complets

### ❌ Manquant / Incomplet

1. **Contrôle répartition par classe d'origine**
   - Données disponibles mais PAS utilisées pour équilibrage
   - Rien n'empêche 10 élèves de 5°1 dans groupe 1, zéro dans groupe 2
   - ⏳ À implémenter : Contrainte "max 2 élèves de même classe par groupe"

2. **Effectifs cibles configurables (±1)**
   - ⚠️ Effectifs actuels = ceil(total/numGroups)
   - Les groupes peuvent avoir 24 et 26 élèves (différence de 2)
   - ⏳ À implémenter : Paramètre `targetSize ± 1`

3. **Verrouillage d'élèves (student pinning)**
   - ❌ Aucun mécanisme
   - Pas de UI pour "épingler" un élève à un groupe spécifique
   - ⏳ À implémenter : Checkbox/icon sur cartes élèves pendant chargement

---

## 4️⃣ FIABILISATION DE LA SAUVEGARDE ET DU SUIVI

### 📋 Spécification

> - Remplacer la suppression regex par filtrage explicite
> - Enregistrer la date de sauvegarde effective
> - Journaliser les versions (identifiant, utilisateur, horodatage)

### ✅ Implémenté

**Filtrage explicite par préfixe**
- ✅ `Code.js:2378-2389` : Boucle sur `startsWith(typePrefix) && endsWith('TEMP')`
- ❌ Suppression ANCIENNE regex `^\w+\d+$` : **SUPPRIMÉE** (bon)
- ✅ Pas de résidus, destruction ciblée par préfixe exact

**Horodatage des opérations**
- ✅ Sauvegarde TEMP : `Code.js:2508` → `timestamp: new Date().toISOString()`
- ✅ Finalization : Timestamp dans résultat
- ⚠️ Timestamp envoyé au frontend mais PAS sauvegardé en base

**Feuille de log**
- ✅ Feuille `_SAVE_LOG` créée automatiquement `Code.js:1861-1865`
- ✅ Entries : Timestamp | User | Operation | Status | Details
- ⚠️ Limité à opérations autres groupes (pas groupes eux-mêmes)

### ❌ Manquant / Incomplet

1. **Feuille d'audit spécifique groupes**
   - ❌ Pas de `_AUDIT_LOG` pour groupes créés/finalisés
   - ⏳ À implémenter : Feuille avec colonnes
     - Timestamp
     - Utilisateur (Session.getActiveUser().getEmail())
     - Opération (SAVE_TEMP | FINALIZE)
     - Type (needs | language | option)
     - Range (grBe1-grBe3)
     - Nombre d'élèves
     - Mode (replace | append | merge)
     - Détails JSON

2. **Versioning des groupes finalisés**
   - ❌ Pas de snapshot "_v1_ESP", "_v2_ESP"
   - Une fois finalisé, impossible de revenir à l'ancienne version
   - ⏳ À implémenter : Backup + historique des 5 dernières versions

3. **Méta-données persistantes**
   - ⚠️ Métadonnées de TEMP sauvegardées dans `state.lastTempRange` (en mémoire JS)
   - Rechargement de page = perte totale
   - ⏳ À implémenter : Persistance dans PropertiesService

---

## 5️⃣ AMÉLIORATIONS DE L'UX

### 📋 Spécification

> - Ajouter une prévisualisation PDF
> - Proposer un export CSV enrichi
> - Permettre la comparaison avant/après finalisation
> - Intégrer des alertes guidées

### ✅ Implémenté

**Export PDF**
- ✅ Bouton "Télécharger PDF" fonctionnel
- ✅ `groupsModuleComplete.html:2180-2186`
- ✅ Appelle `google.script.run.exportGroupsToPDF(payload)`
- ✅ PDF généré en backend, servi au client

**Export CSV**
- ✅ Bouton "Télécharger CSV" fonctionnel
- ✅ `groupsModuleComplete.html:2220-2244`
- ✅ Colonnes : ID_ELEVE | NOM | PRENOM | SEXE | CLASSE | SCORE_F | SCORE_M | COM | TRA | PART | ABS | LV2 | OPT | SOURCE

**Drag-Drop reordering**
- ✅ SortableJS intégré
- ✅ Permet réorganisation d'élèves entre groupes
- ✅ `groupsModuleComplete.html:1149-1177` (drag UI)

**Bandeaux informatifs**
- ✅ Continuation : "🔄 MODE CONTINUATION ACTIF" `groupsModuleComplete.html:1034-1046`
- ✅ Groupe nombres : Affiche "Groupe 4, 5, 6..." avec offset

### ❌ Manquant / Incomplet

1. **Prévisualisation PDF avant export**
   - ❌ Bouton "Aperçu PDF" n'existe pas
   - Export va directement vers téléchargement
   - ⏳ À implémenter : Modal de prévisualisation avec scrollable PDF

2. **Comparaison avant/après**
   - ❌ Pas de UI pour comparer vague précédente ↔ vague courante
   - ⏳ À implémenter : Split-screen avec groupes grBe1 vs. grBe2

3. **Alertes guidées**
   - ⚠️ Existence de toasts pour feedback (succès/erreur)
   - ❌ Pas d'alertes spécifiques pour anomalies :
     - "⚠️ Parité F/M déséquilibrée (30/20)"
     - "⚠️ Groupe 1 a 28 élèves, Groupe 2 a 22 (écart >2)"
     - "⚠️ Classe 5°1 sur-représentée dans Groupe 1"
   - ⏳ À implémenter : Dashboard pré-finalisation avec alertes

---

## 6️⃣ AUDIT STRUCTURE HTML - STEPPER EN 5 ÉTAPES

### 📋 Spécification

Problème identifié :
> - Stepper réinitialise entièrement `state.generatedGroups` à chaque retour
> - Save/Finalize ne fournissent pas d'offset ni de mode "ajout"
> - Bouton "Recommencer" vide l'état sans option pour continuité multi-phases
> - Pas de vue récapitulative des groupes finalisés

### ✅ Corrigé

**Continuité multi-phases implémentée**
- ✅ State properties ajoutées :
  - `persistMode` : true si on continue une série
  - `saveMode` : 'replace' | 'append'
  - `tempOffsetStart` : numéro de départ des prochains groupes
  - `lastTempRange` : {min, max, count, typePrefix}
- ✅ `groupsModuleComplete.html:122-127`

**Dialogs de confirmation**
- ✅ `resetToStep1()` détecte si TEMP existent et propose :
  - [OK] Continuer la série avec les offsets corrects
  - [Cancel] Recommencer une nouvelle série
- ✅ `groupsModuleComplete.html:1581-1609`

**Mode buttons avec protection**
- ✅ Boutons "Remplacer" et "Ajouter" activables
- ✅ Si `persistMode=true`, mode 'replace' est bloqué (force 'append')
- ✅ `groupsModuleComplete.html:1496-1512`

**Sauvegarde avec offset**
- ✅ `saveTempGroupsUI()` envoie `offsetStart` si `persistMode=true`
- ✅ Backend utilise cet offset prioritairement
- ✅ `groupsModuleComplete.html:2310-2318` + `Code.js:2374-2377`

**Affichage des prochains numéros**
- ✅ Bandeau affiche : "Prochains groupes: 4 → 7"
- ✅ Cartes affichent : "Groupe 4, Groupe 5, Groupe 6, Groupe 7"
- ✅ `groupsModuleComplete.html:1130` + `1034-1046`

### ❌ Manquant / Incomplet

1. **Persistance après rechargement page**
   - ⚠️ `lastTempRange` et `tempOffsetStart` en mémoire JS uniquement
   - Rechargement page → perte complète de l'état
   - ⏳ À implémenter : Sauvegarder dans PropertiesService
   ```javascript
   // saveContinuationMetadata(type, {lastTempRange, tempOffsetStart})
   // loadContinuationMetadata(type) -> récupère depuis backend
   ```

2. **Pas de récap des finalisés**
   - ⚠️ Étape 5 affiche groupes TEMP générés, pas groupes finalisés
   - Pas de vue "Vous avez déjà 3 groupes finalisés (grBe1-3), vous en ajoutez 3 nouveaux (grBe4-6)"
   - ⏳ À implémenter : Tableau recap avec séparation TEMP / FINAL

---

## 📊 RÉSUMÉ MATRICE IMPLÉMENTATION

| Domaine | Spécification | Implémenté | Qualité | Priority |
|---------|------|---|---|---|
| **Filtrage LV2** | ✅ Oui | ✅ 100% | 🟢 OK | - |
| **Filtrage Options** | ✅ Infrastructure | ✅ 80% | 🟡 Sans UI | HIGH |
| **Dénomination spécialisée** | ✅ Oui | 🟡 50% | 🟡 Basique | MEDIUM |
| **Score composite** | ✅ Oui | ❌ 0% | 🔴 Stub | **CRITICAL** |
| **Algo glouton** | ✅ Oui | 🟡 30% | 🟡 Tri+Snake | **CRITICAL** |
| **Dashboard écarts** | ✅ Oui | ❌ 0% | 🔴 Stub | HIGH |
| **Contrainte SOURCE** | ✅ Oui | ❌ 0% | 🔴 Pas utilisée | HIGH |
| **Effectifs ±1** | ✅ Oui | ❌ 0% | 🔴 Pas contrôlé | MEDIUM |
| **Pinning élèves** | ✅ Oui | ❌ 0% | 🔴 Absent | MEDIUM |
| **Audit _LOG** | ✅ Oui | 🟡 50% | 🟡 Logs générales | HIGH |
| **Versioning** | ✅ Oui | ❌ 0% | 🔴 Absent | HIGH |
| **Persistance offsets** | ✅ Oui | 🟡 50% | 🟡 JS only | **CRITICAL** |
| **PDF preview** | ✅ Oui | 🟡 70% | 🟡 Export yes, preview no | MEDIUM |
| **CSV enrichi** | ✅ Oui | ✅ 100% | 🟢 OK | - |
| **Comparaison avant/après** | ✅ Oui | ❌ 0% | 🔴 Absent | MEDIUM |
| **Alertes guidées** | ✅ Oui | 🟡 20% | 🟡 Toasts basiques | MEDIUM |

---

## 🚨 BLOCKERS CRITIQUES POUR PRODUCTION

### 1. **Persistance multi-vagues défaillante** 🔴 **BLOQUANT**

**Problème** :
- État de continuation (`lastTempRange`, `tempOffsetStart`) stocké uniquement en JS
- Rechargement page = recommencer depuis zéro
- Impossible pour workflow sur 2 jours
- **Utilisateur** : Crée groupes 1-3 → ferme onglet → lendemain → tout est perdu

**Fichiers affectés** :
- `groupsModuleComplete.html:122-127` (state definition)
- Pas de `saveContinuationMetadata()` en backend

**Solution requise** :
```javascript
// Code.js - NEW
function saveContinuationMetadata(type, metadata) {
  const props = PropertiesService.getDocumentProperties();
  props.setProperty('CONTINUATION_' + type, JSON.stringify(metadata));
}

function loadContinuationMetadata(type) {
  const props = PropertiesService.getDocumentProperties();
  return JSON.parse(props.getProperty('CONTINUATION_' + type) || '{}');
}

// groupsModuleComplete.html - MODIFY
function onBeforeUnload() {
  google.script.run
    .saveContinuationMetadata(state.groupType, {
      lastTempRange: state.lastTempRange,
      tempOffsetStart: state.tempOffsetStart,
      persistMode: state.persistMode
    });
}
```

**Estimation** : 2-3 heures de développement

---

### 2. **Score composite et équilibrage rudimentaire** 🔴 **BLOQUANT**

**Problème** :
- Algorithme = tri par note + entrelacage F/M
- Pas de pondération multi-critères
- Pas d'optimisation itérative
- Les groupes peuvent être très déséquilibrés en comportement/assiduité

**Fichiers affectés** :
- `groupsModuleComplete.html:2033-2119` (generateGroupsLocally)

**Solution requise** :
```javascript
// Score composite
function getCompositeScore(student) {
  const f = student.scores?.F ?? 0;
  const m = student.scores?.M ?? 0;
  const com = student.com ?? 0;
  const part = student.part ?? 0;

  // Poids configurables
  return (f × 0.3) + (m × 0.3) + (com × 0.2) + (part × 0.2);
}

// Algo itératif : échanges locaux pour minimiser écart-type
function optimizeGroupsWithSwaps(groups) {
  let improved = true;
  while (improved) {
    improved = false;
    for (let i = 0; i < groups.length; i++) {
      for (let j = i + 1; j < groups.length; j++) {
        // Essayer tous les échanges de paires
        const [best, delta] = trySwaps(groups[i], groups[j]);
        if (delta > threshold) {
          executeSwap(best);
          improved = true;
        }
      }
    }
  }
}
```

**Estimation** : 4-5 heures

---

### 3. **Dashboard de validation absente** 🔴 **BLOQUANT**

**Problème** :
- Bouton "Statistiques" ne fait rien
- Pas de visibilité sur qualité de l'équilibrage
- Utilisateur ne peut pas valider avant finalisation
- Impossible de détecter anomalies

**Fichiers affectés** :
- `groupsModuleComplete.html:2050-2100` (renderStatisticsPanel stub)

**Solution requise** :
```html
<div id="stats-panel">
  <table>
    <tr><th>Groupe</th><th>Effectif</th><th>Moy F</th><th>Moy M</th><th>F/M</th><th>Moy COM</th></tr>
    <!-- une row par groupe avec métriques -->
  </table>
  <p>Écart-type global: 1.2 (< 2 = OK)</p>
  <p>Parité: 50/50 ±5% (✅ OK)</p>
</div>
```

**Estimation** : 3-4 heures

---

### 4. **Audit et traçabilité manquants** 🟠 **HAUTE PRIORITÉ**

**Problème** :
- Aucun journal des opérations sur groupes
- Impossible de savoir qui a finalisé quoi et quand
- Pas de rollback possible
- Non-conformité RGPD pour données sensibles

**Fichiers affectés** :
- Nouvelle feuille `_AUDIT_LOG` à créer

**Solution requise** :
```javascript
function logGroupOperation(operation, details) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let auditSheet = ss.getSheetByName('_AUDIT_LOG');
  if (!auditSheet) {
    auditSheet = ss.insertSheet('_AUDIT_LOG');
    auditSheet.appendRow(['Timestamp', 'User', 'Operation', 'Type', 'Prefix', 'Range', 'Count', 'Mode', 'Details']);
    auditSheet.hideSheet();
  }

  auditSheet.appendRow([
    new Date().toISOString(),
    Session.getActiveUser().getEmail(),
    operation, // SAVE_TEMP, FINALIZE, DELETE
    details.type,
    details.prefix,
    `${details.startNum}-${details.endNum}`,
    details.count,
    details.mode,
    JSON.stringify(details)
  ]);
}
```

**Estimation** : 2-3 heures

---

## ✅ ÉLÉMENTS BIEN IMPLÉMENTÉS

1. **Filtrage LV2/Options** - Logique solide, structure réutilisable
2. **Continuité multi-phases** - UI et state management cohérents
3. **Export CSV/PDF** - Fonctionnels et enrichis
4. **Parité F/M** - Équilibrage basique mais efficace
5. **Quad-mode sauvegarde** - replace/append/merge logique correcte

---

## 📌 RECOMMANDATION FINALE

### ✅ **DÉMO / PROTOTYPE**

Module peut être utilisé pour :
- Preuve de concept
- Démonstration à stakeholders
- Tests avec <200 élèves
- Une seule vague de génération

### ❌ **PRODUCTION**

**Ne pas utiliser tant que** :
- [ ] Persistance multi-vagues : sauvegardée en PropertiesService
- [ ] Score composite : implémenté avec poids configurables
- [ ] Algorithme d'optimisation : itérations + échanges
- [ ] Dashboard stats : affiche écarts-type, parité, distribution
- [ ] Audit complet : feuille _AUDIT_LOG + versioning

### ⏱️ **Estimation totale pour production** : 4-6 sprints (200-300 heures)

---

**Prochaine étape** : Prioriser selon impact pédagogique et effort de dev

