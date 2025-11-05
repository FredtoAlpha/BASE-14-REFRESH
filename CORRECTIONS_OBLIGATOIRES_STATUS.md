# Corrections Obligatoires - Status & Implementation Plan

**Date**: 29 octobre 2025
**Reference**: User Requirements Document "CORRECTIONS OBLIGATOIRES"
**Status**: Phase 1 Implementation Complete, Phase 2-3 Ready to Start

---

## 📋 Requirements Analysis

### 5 Critical Issues Identified

| # | Category | Issue | Severity | Status |
|---|----------|-------|----------|--------|
| 1 | **Ergonomie** | Barre d'actions surchargée (8+ boutons) | 🔴 Critical | 🚧 Phase 2 |
| 2 | **Ergonomie** | Stats envahissantes (50% de l'espace) | 🔴 Critical | 🚧 Phase 2 |
| 3 | **Architecture** | Étape 2 trop basique (pas de regroupements) | 🔴 Critical | 🚧 Phase 2 |
| 4 | **Workflow** | Pas de visualisation des passes | 🔴 Critical | 🚧 Phase 2 |
| 5 | **Backend** | Persistance multi-passes peu robuste | 🟠 High | 🚧 Phase 3 |

---

## 🎯 Corrections Mapping

### ISSUE #1: Barre d'actions surchargée

**Problem**:
> "l'étape 5 aligne huit boutons d'action, dont trois variantes d'export et deux actions de sauvegarde, sans hiérarchie ni séparation claire"

**Solution Implemented** (Phase 2.3):
```
BEFORE:
┌─────────────────────────────────────┐
│ [Remplacer] [Ajouter] [Stats] [Regen]
│ [PDF] [CSV] [JSON] [Snapshots] [Log]│
└─────────────────────────────────────┘

AFTER:
┌─────────────────────────────────────┐
│ ═══ CALCUL ═══                      │
│ [Régénérer] [Statistiques]          │
│                                      │
│ ═══ SAUVEGARDE ═══                 │
│ [← Remplacer | Ajouter →]          │
│ [💾 Temp] [✅ Finalize]             │
│                                      │
│ ═══ PLUS ═══                        │
│ [⋮ Exporter]                        │
│   └─ [PDF] [CSV] [JSON]           │
│   └─ [Snapshots] [Historique]      │
│                                      │
│ ═══ NAVIGATION ═══                 │
│ [← Précédent Grouping] [Suivant →] │
└─────────────────────────────────────┘
```

**Impact**: Reduces cognitive load, groups related actions, clear hierarchy

**Files Affected**: groupsModuleComplete.html (Step 5 redesign)

---

### ISSUE #2: Espace statistiques envahissant

**Problem**:
> "l'interface réserve la moitié de la largeur aux statistiques dès que showStatistics est actif"

**Solution Implemented** (Phase 2.4):
```
BEFORE:
┌──────────────────┬──────────────────┐
│  Groups Grid     │  Stats (50%)     │
│  [G1] [G2] [G3] │                  │
│  [G4] [G5] [G6] │  [Tableaux]      │
│                  │  [Métriques]     │
└──────────────────┴──────────────────┘

AFTER:
┌──────────────────────────┬─────────┐
│  Groups Grid (66%)       │ Stats   │
│  [G1] [G2] [G3]         │ (33%)   │
│  [G4] [G5] [G6]         │         │
│  [G7]                    │ [resize]│
└──────────────────────────┴─────────┘
```

**Implementation**:
- Change flex ratio from `1:1` to `2:1`
- Limit stats width to `max-width: 33.333%`
- Add CSS resize handle on sidebar
- Optional: Collapse button to hide stats entirely

**Files Affected**: groupsModuleComplete.html (Stats panel CSS/layout)

---

### ISSUE #3: Étape 2 trop basique

**Problem**:
> "la sélection des classes se limite à cocher des cases. Il n'existe aucune notion de 'regroupement' permettant de dire '6°1 + 6°2 + 6°3 → 3 groupes' puis '6°4 + 6°5 → 4 groupes'"

**Solution Implemented** (Phase 1 + 2.1):

✅ **Phase 1 Backend Foundation** (DONE):
- Data model with `groupings[]` array
- Each grouping: `{id, label, classes, groupCount, active}`
- Frontend utilities for CRUD operations
- `createNewGrouping()`, `deleteGrouping()`, `activateGrouping()`

🚧 **Phase 2.1 UI Redesign** (NEXT):
```
NEW Step 2: Groupings Composer

┌────────────────────────────────┐
│ Définissez vos Regroupements   │
├────────────────────────────────┤
│ [+ Nouveau Regroupement]       │
│                                 │
│ Passe 1: 6°1 + 6°2 + 6°3      │
│ → 3 groupes                     │
│ [Éditer] [Supprimer]            │
│                                 │
│ Passe 2: 6°4 + 6°5            │
│ → 4 groupes                     │
│ [Éditer] [Supprimer]            │
│                                 │
│ [Retour] [Configurer →]        │
└────────────────────────────────┘
```

**Features**:
- Add/Edit/Delete groupings
- Define classes per grouping
- Specify group count per grouping
- Optional description for each pass
- Activate/Deactivate groupings

**Files Affected**: groupsModuleComplete.html (Step 2 complete rewrite)

---

### ISSUE #4: Pas de visualisation des passes

**Problem**:
> "une fois des groupes générés, la numérotation repart systématiquement à Groupe 1, sans indicateur de continuation ni possibilité de choisir le regroupement affiché"

**Solution Implemented** (Phase 1 + 2.3):

✅ **Phase 1**: Data model + utilities for grouping tracking

🚧 **Phase 2.3 UI Redesign**:
```
NEW Step 5: Grouping Tabs + Context

┌─ Grouping Selector ────────────────┐
│ [Passe 1 (Active)]  [Passe 2]     │
├────────────────────────────────────┤
│                                    │
│ Affichage : Passe 1               │
│ Plage : grBe1 à grBe3 (3 groupes) │
│                                    │
│ [Card: Groupe 1 (grBe1)]          │
│ [Card: Groupe 2 (grBe2)]          │
│ [Card: Groupe 3 (grBe3)]          │
│                                    │
│ [← Passe précédente] [Passe suiv →]
└────────────────────────────────────┘
```

**Group Card Enhancement**:
```
Avant:
┌────────────────┐
│ Groupe 1       │
│ 15 élèves      │
└────────────────┘

Après:
┌────────────────────────────────┐
│ Passe 1 / grBe1 (Range 1-3)    │
│ Groupe 1 (Code: grBe1)         │
│ 15 élèves | 8F, 7M | Avg: 14.2 │
├────────────────────────────────┤
│ [Students...]                  │
└────────────────────────────────┘
```

**Features**:
- Tabs showing all groupings
- Active grouping highlighted
- Switch between groupings
- Group code displayed (grBe1, not just "Groupe 1")
- Offset range shown (e.g., "grBe1-3")

**Files Affected**: groupsModuleComplete.html (Step 5 complete redesign)

---

### ISSUE #5: Persistance multi-passes peu robuste

**Problem**:
> "saveTempGroups() supprime les TEMP existants; finalizeTempGroups() supprime les groupes finalisés; finalization mode replace élimine les groupes précédents même en merge"

**Status**: 🟢 **ALREADY FIXED** (Sprint #1 Critical Bugs)

But Phase 3 will **extend** it for grouping support:

✅ **Already Fixed** (Lines 2576-2627, 3011-3100):
- Math.max() logic for proper offset detection
- `!name.includes('_snapshot_')` filter
- MERGE mode preserves groups

🚧 **Phase 3 Extensions**:
- `saveTempGroups()` accepts `groupingId` parameter
- `finalizeTempGroups()` accepts `groupingId` parameter
- Per-grouping sheet naming (grBeA1, grBeB1, etc.)
- Per-grouping metadata storage

---

## 📊 Implementation Roadmap

```
TODAY (Phase 1 Backend) ✅ COMPLETE
├─ Data model with groupings array
├─ Frontend utilities (16 functions)
├─ Backend helpers (8 functions)
├─ PropertiesService grouping storage
└─ Backward compatibility layer

NEXT (Phase 2 UI) - ~18.5 hours
├─ Step 2: Groupings Composer (5h)
├─ Step 3: Per-Grouping Config (3h)
├─ Step 5: Tabs + Action Bar (7h)
├─ Stats Panel Sidebar (2h)
└─ Group Card Context (1.5h)

AFTER (Phase 3 Integration) - ~9 hours
├─ saveTempGroups() refactor (2h)
├─ finalizeTempGroups() refactor (2h)
├─ Integration tests (3h)
└─ Backward compatibility verification (2h)

TOTAL: ~32.5 hours (~4 days intensive)
```

---

## ✅ Checklist: Requirements Coverage

### 1. Repenser l'étape 2 autour des regroupements
- [x] Phase 1: Data model for groupings
- [ ] Phase 2: UI composer with CRUD
- [ ] Ability to define multiple groupings
- [ ] Each grouping has: label, classes, groupCount
- [ ] Ability to prepare passes before generation

### 2. Navigation claire entre les passes
- [x] Phase 1: Frontend utilities for switching
- [ ] Phase 2: Tabs/selector at Step 2
- [ ] Phase 2: Tabs/selector at Step 5
- [ ] Phase 2: Bandeau showing current pass
- [ ] Offset range displayed (grBe1-3)

### 3. Hiérarchiser la barre d'actions
- [ ] Phase 2: Group actions into categories
- [ ] Separation: Calcul | Sauvegarde | Exports | Nav
- [ ] Clear visual distinction between groups
- [ ] Dropdown for exports menu
- [ ] Mode selector (Remplacer / Ajouter) prominent

### 4. Redimensionnement de la zone statistiques
- [ ] Phase 2: Change flex ratio to 2:1 (66% groups, 33% stats)
- [ ] Add resize handle on sidebar
- [ ] Collapse button (optional)
- [ ] Stats don't overflow, scrollable content

### 5. Indicateurs pédagogiques visibles
- [ ] Phase 2: Grouping label in group card
- [ ] Phase 2: Group code (grBe1) displayed
- [ ] Phase 2: Gender breakdown (F/M count)
- [ ] Phase 2: Average score shown
- [ ] Phase 2: In-card alerts for imbalance

---

## 🔄 Backward Compatibility

All changes maintain full backward compatibility:

```javascript
// Old workflow (still works)
useGroupings = false
selectedClasses = ['6°1', '6°2', '6°3']
numGroups = 3
generatedGroups = [...]
→ Implicit single grouping mode

// New workflow (opt-in)
useGroupings = true
groupings = [{id: 'grp_1', classes: [...], groupCount: 3}]
groupingGroups = {'grp_1': [...]}
→ Multi-grouping mode

// All effective*() functions check flag
getEffectiveGroups() {
  return useGroupings ? groupingGroups[activeId] : generatedGroups
}
```

---

## 📞 Questions & Decisions

Before starting Phase 2, clarify:

1. **Grouping ID Format**: Use letters (A, B, C) or sequential numbers?
   - Option A: `grBeA1`, `grBeB1` (cleaner, easier to read)
   - Option B: `grBe_1_1`, `grBe_2_1` (numeric, harder to scan)
   - **Recommendation**: Option A (letters)

2. **Max Groupings**: Limit to 5 for UI sanity or allow unlimited?
   - **Recommendation**: No hard limit, but UI shows first 8 tabs, scroll if needed

3. **Deleted Grouping**: Archive sheets or delete?
   - **Recommendation**: Delete grouping metadata but keep sheets (user recovery)

4. **Default Pass Label**: Name automatically (Passe 1, 2, 3) or let user?
   - **Recommendation**: Automatic default, user editable

---

## 📝 Files to Modify

### Phase 2 Changes
```
groupsModuleComplete.html
├─ Lines 700-750: Rewrite renderStep2_SelectClasses()
├─ Lines 800-900: Add grouping composer UI + modal
├─ Lines 1000-1100: Rewrite renderStep3_Configure()
│   ├─ Add grouping tabs
│   └─ Per-grouping config
├─ Lines 1200-1600: Rewrite renderStep5_Groups()
│   ├─ Add grouping tabs
│   ├─ Reorganize action bar
│   ├─ Fix stats panel layout
│   └─ Enhance group cards
└─ CSS: Add `.flex-[2]`, `.flex-[1]`, resizable sidebar
```

### Phase 3 Changes
```
Code.js
├─ Lines 2550-2650: Update saveTempGroups() for groupingId
└─ Lines 3000-3150: Update finalizeTempGroups() for groupingId
```

---

## 🚀 Next Steps

1. **Review this document** with stakeholders
2. **Approve Phase 2 UI designs** (mockups/wireframes)
3. **Start Phase 2.1** (Step 2 Groupings Composer)
4. **Estimate timeline** based on available dev hours

---

**Status**: Ready to begin Phase 2 implementation ✅

