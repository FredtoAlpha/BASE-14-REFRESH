# 🧪 PLAN DE TEST - Validation Multi-Passe

**Date**: 29 octobre 2025
**Objectif**: Valider que tous les 3 blockers sont résolus et le workflow multi-passe fonctionne

---

## 🎯 Critères de succès

| Criteria | Definition | Pass/Fail |
|----------|-----------|-----------|
| **C1: Offset isolation** | Chaque regroupement a ses propres offsets | [ ] |
| **C2: No data destruction** | Passe 1 data préservée après Passe 2 finalize | [ ] |
| **C3: UI clarity** | UI montre clairement le numéro de départ | [ ] |
| **C4: Mode indication** | Banner explique mode continue vs replace | [ ] |
| **C5: Multi-pass workflow** | 2+ passes en 1 jour sans crash | [ ] |

---

## 📝 Test Case 1: Single Pass (Baseline)

**Objectif**: Vérifier que le workflow simple toujours fonctionne

### Setup
- Classes: 6°1, 6°2
- Type: Besoin (Besoins Scolaires)
- Groupes: 3

### Étapes

**Step 1: Créer regroupement Passe 1**
```
Aller à Étape 2
├─ Sélectionner: 6°1, 6°2
├─ Créer regroupement "Passe 1: 6°1+6°2"
├─ Nombre de groupes: 3
└─ Valider → Passe 1 créée
```

**Validation**:
- [ ] Regroupement visible dans les tabs
- [ ] Affiche: "Passe 1: 6°1+6°2"
- [ ] Nombre de groupes: 3

**Step 2: Générer groupes**
```
Aller à Étape 5
├─ Génération automatique
├─ Affiche Groupe 1, Groupe 2, Groupe 3
└─ Vérifier offsets
```

**Validation**:
- [ ] Groupe numbers: 1, 2, 3 ✅
- [ ] Offset input: 1
- [ ] Mode button: Replace (sélectionné)
- [ ] Pas de banner contextuelle (first pass)

**Step 3: Sauver TEMP**
```
Cliquer: Sauver TEMP
├─ Toast: "Groupes sauvegardés temporairement (Groupes 1 → 3)"
├─ Backend crée: grBe1TEMP, grBe2TEMP, grBe3TEMP
└─ State mis à jour: lastTempRange = {start: 1, end: 3}
```

**Validation**:
- [ ] Toast affiché ✅
- [ ] Range display: "TEMP 1 → 3"
- [ ] Pas de console errors

**Step 4: Finaliser**
```
Cliquer: Finaliser
├─ Toast: "Groupes finalisés et visibles (Groupes 1 → 3)"
├─ TEMP renommé → Final: grBe1, grBe2, grBe3
├─ Métadonnées: lastFinalRange = {start: 1, end: 3}
└─ État: mode = replace, offset = 1 (reset)
```

**Validation**:
- [ ] Toast affiché ✅
- [ ] Range display: "Final 1 → 3"
- [ ] Google Sheets: grBe1, grBe2, grBe3 visibles
- [ ] Pas de console errors

**RESULT**:
- [ ] ✅ PASS (Baseline workflow fonctionnel)

---

## 📝 Test Case 2: Two Passes - Continue Mode (Main Test)

**Objectif**: Vérifier que Passe 2 en mode Continue préserve Passe 1

### Setup
- Passe 1 finalisée (voir Test Case 1)
- Classes Passe 2: 6°3, 6°4
- Type: Besoin
- Groupes: 4

### Étapes

**Step 1: Créer regroupement Passe 2**
```
Aller à Étape 2
├─ Sélectionner: 6°3, 6°4
├─ Créer regroupement "Passe 2: 6°3+6°4"
├─ Nombre de groupes: 4
└─ Valider → Passe 2 créée
```

**Validation**:
- [ ] Passe 2 visible dans les tabs
- [ ] Passe 1 tab aussi visible ✅
- [ ] Affiche: "Passe 2: 6°3+6°4"
- [ ] Nombre de groupes: 4

**Step 2: Générer groupes**
```
Aller à Étape 5
├─ Génération automatique
├─ VÉRIFIE: Offset input = 4 (détecté automatiquement!)
├─ Affiche Groupe 4, Groupe 5, Groupe 6, Groupe 7
├─ Mode button: Continue (sélectionné par défaut!)
├─ Banner: "📊 Mode Continuation activé"
│          "Les groupes continueront à partir du numéro 4"
└─ Summary: "Passe continue après les groupes 1-3"
```

**Validation** - CRITICAL:
- [ ] Offset input: 4 ✅ (PAS 1!)
- [ ] Groupe numbers: 4, 5, 6, 7 ✅ (PAS 1, 2, 3, 4!)
- [ ] Mode button: Continue (blue) ✅
- [ ] Banner présente ✅
- [ ] Summary contextuel ✅

**Step 3: Sauver TEMP**
```
Cliquer: Sauver TEMP
├─ Toast: "Groupes sauvegardés temporairement (Groupes 4 → 7)"
├─ Backend crée: grBe4TEMP, grBe5TEMP, grBe6TEMP, grBe7TEMP
├─ Métadonnées: lastTempRange = {start: 4, end: 7}
└─ State: persistMode = continue
```

**Validation**:
- [ ] Toast: "Groupes 4 → 7" ✅
- [ ] Range display: "TEMP 4 → 7"
- [ ] Mode = continue

**Step 4: Finaliser**
```
Cliquer: Finaliser
├─ Backend vérifie: lastFinalRange de Passe 1 = {start: 1, end: 3}
├─ Mode continue → Pas de suppression
├─ Renomme: grBe4TEMP → grBe4, grBe5TEMP → grBe5, etc.
├─ Métadonnées: lastFinalRange = {start: 4, end: 7}
└─ Toast: "Groupes finalisés et visibles (Groupes 4 → 7)"
```

**Validation** - CRITICAL (Blocker #2):
- [ ] Toast: "Groupes 4 → 7" ✅
- [ ] NO console errors ✅
- [ ] Google Sheets inspection:
  - [ ] grBe1 visible ✅ (Passe 1 PRÉSERVÉ!)
  - [ ] grBe2 visible ✅ (Passe 1 PRÉSERVÉ!)
  - [ ] grBe3 visible ✅ (Passe 1 PRÉSERVÉ!)
  - [ ] grBe4 visible ✅ (Passe 2)
  - [ ] grBe5 visible ✅ (Passe 2)
  - [ ] grBe6 visible ✅ (Passe 2)
  - [ ] grBe7 visible ✅ (Passe 2)

**RESULT**:
- [ ] ✅ PASS (Multi-pass continue fonctionnel!)

---

## 📝 Test Case 3: Two Passes - Replace Mode

**Objectif**: Vérifier que Passe 2 en mode Replace fonctionne correctement

### Setup
- Passe 1 finalisée: grBe1-3
- Classes Passe 2: 6°5, 6°6
- Type: Besoin
- Groupes: 2

### Étapes

**Step 1-2: Créer Passe 2 et générer**
```
Aller à Étape 5
├─ Mode button: Replace (changé manuellement)
├─ Banner: "🔄 Mode Remplacement activé"
│          "Les groupes remplaceront les onglets (1-3)"
├─ Offset input: 1 (réinitialisé)
├─ Affiche Groupe 1, Groupe 2
└─ Summary: "Passe remplacera groupes (1-3)"
```

**Validation**:
- [ ] Offset: 1 ✅
- [ ] Mode: Replace ✅
- [ ] Banner: Amber/warning ✅

**Step 3: Sauver TEMP**
```
Sauver TEMP
├─ Toast: "Groupes sauvegardés temporairement (Groupes 1 → 2)"
├─ Backend: grBe1TEMP, grBe2TEMP
└─ persistMode = replace
```

**Step 4: Finaliser**
```
Cliquer: Finaliser
├─ Backend détecte: lastFinalRange = {start: 1, end: 3}
├─ persistMode = replace
├─ Filtre et supprime: grBe1, grBe2, grBe3
│  (MAIS SEULEMENT car ils sont dans lastFinalRange!)
├─ Renomme: grBe1TEMP → grBe1, grBe2TEMP → grBe2
└─ Métadonnées: lastFinalRange = {start: 1, end: 2}
```

**Validation**:
- [ ] Toast: "Groupes 1 → 2" ✅
- [ ] Google Sheets:
  - [ ] grBe1 remplacé ✅ (ancien data perdu - c'est voulu)
  - [ ] grBe2 remplacé ✅
  - [ ] grBe3 SUPPRIMÉ ✅ (ancien data perdu - c'est voulu)

**RESULT**:
- [ ] ✅ PASS (Replace mode fonctionnel!)

---

## 📝 Test Case 4: UI Indicators Validation

**Objectif**: Vérifier tous les UI indicators

### Test Steps

**4.1: Helper functions work**
```javascript
// Ouvrir console (F12)
// Tester les helpers:
getEffectivePersistMode()     // Doit retourner 'continue' ou 'replace'
getEffectiveOffsetStart()     // Doit retourner le bon offset
getEffectiveLastFinalRange()  // Doit retourner {start, end} ou null
```

**Validation**:
- [ ] All helpers return values ✅
- [ ] Values match expected regroupement ✅

**4.2: Banner appears correctly**
```
Passe 1 finalisée, Passe 2 créée en Continue mode
├─ Aller Étape 5
├─ VÉRIFIE: Banner bleu apparaît ✅
├─ LIRE: "Mode Continuation activé"
│        "Les groupes continueront à partir du numéro 4"
└─ FERMER: Changer mode à Replace
   ├─ Banner change → AMBER/warning ✅
   ├─ LIRE: "Mode Remplacement activé"
   │        "Les groupes remplaceront... (1-3)"
   └─ Changer mode back à Continue
      └─ Banner change → BLUE ✅
```

**Validation**:
- [ ] Banner color changes correctly ✅
- [ ] Banner text updates ✅
- [ ] No console errors ✅

**4.3: Summary context**
```
Passe 1 (first pass) summary:
├─ No context message ✅ (it's the first)

Passe 2 (after Passe 1 finalized, continue mode):
├─ Shows: "📊 Cette passe continue après les groupes 1-3"

Passe 2 (replace mode):
├─ Shows: "🔄 Cette passe remplacera les groupes (1-3)"
```

**Validation**:
- [ ] Context messages appear ✅
- [ ] Messages are accurate ✅

**4.4: Offset input reflects regroupement**
```
Basculer entre Passe 1 et Passe 2:
├─ Click Passe 1 tab
│  └─ Offset input = 1 ✅
├─ Click Passe 2 tab
│  └─ Offset input = 4 ✅
├─ Changer Passe 2 offset à 5
│  └─ Input change to 5 ✅
├─ Click Passe 1 tab
│  └─ Offset input = 1 ✅ (pas 5!)
└─ Click Passe 2 tab
   └─ Offset input = 5 ✅ (remember!)
```

**Validation** - CRITICAL (Blocker #3):
- [ ] Each regroupement remembers its own offset ✅
- [ ] Changing tab updates input ✅
- [ ] No global state collision ✅

---

## 📊 Summary Test Matrix

| Test Case | Blocker | Criteria | Result |
|-----------|---------|----------|--------|
| **TC1** | N/A | Baseline works | [ ] ✅ |
| **TC2** | #2, #3 | Multi-pass continue | [ ] ✅ |
| **TC3** | #2 | Replace mode | [ ] ✅ |
| **TC4** | #3 | UI indicators | [ ] ✅ |

---

## ✅ FINAL SIGN-OFF

After all test cases pass:

```
TESTER NAME: _______________________
DATE: _______________________
PLATFORM: Google Sheets (specify URL or doc ID)

RESULT: [ ] ALL PASS  [ ] SOME FAIL  [ ] CRITICAL FAIL

IF FAIL: Describe issues:
_________________________________________________________________
_________________________________________________________________

APPROVED FOR: [ ] Phase 2 completion  [ ] Phase 3 backend
```

---

## 🔧 Troubleshooting

### Issue: Offset input shows wrong value

**Diagnosis**:
1. Open console (F12)
2. Type: `getActiveRegroupement()`
3. Check: `offsetStart` property exists?

**Fix**:
- If property missing: Clear local state and refresh
- If value wrong: Manually update via offset input

### Issue: Banner not appearing

**Diagnosis**:
1. Check: `getEffectiveLastFinalRange()` in console
2. Should return `{start: 1, end: 3}` if Passe 1 finalized

**Fix**:
- If null: Re-finalize Passe 1
- If not null: Check browser cache (Ctrl+Shift+Delete)

### Issue: Group numbers wrong

**Diagnosis**:
1. Check: `getEffectiveOffsetStart()` returns correct value?
2. Check: Group card renders use helper?

**Fix**:
- Clear cache and refresh
- Check Code.js was updated with new finalizeTempGroups()

---

**Test Plan Created**: 29 octobre 2025
**Estimated Duration**: 30-45 minutes
**Requirements**: Google Sheets with student data
