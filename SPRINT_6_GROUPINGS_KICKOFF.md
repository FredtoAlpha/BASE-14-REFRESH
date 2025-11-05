# 🚀 SPRINT #6: Groupings System - Implementation Kickoff

**Status**: Phase 1 Complete ✅ / Phase 2-3 Ready to Start 🎯
**Date**: 29 octobre 2025
**Objective**: Transform the module from single-pass to true multi-pass architecture

---

## 📋 What Is This Sprint?

This sprint addresses the critical UX/workflow issues identified in the requirements:

### Problems Being Solved

| Problem | Impact | Solution |
|---------|--------|----------|
| **Surcharged action bar** | User confusion, accidental clicks | Hierarchical grouping (Calcul / Sauvegarde / Exports) |
| **Stats consuming 50% space** | Groups hard to manipulate | Resizable sidebar (max 1/3 width) |
| **No grouping concept** | Can't prepare 2 passes simultaneously | Explicit groupings registry with composer |
| **All-or-nothing save** | Can't finalize pass 1 while working on pass 2 | Per-grouping save/finalize |
| **No continuation feedback** | User doesn't see which pass they're on | Tabs/selector showing active grouping |
| **Flat group numbering** | Groups restart at 1 every time | Grouping-aware numbering (grBeA1, grBeB1) |

### Deliverables

```
PHASE 1: Backend Foundation (DONE) ✅
├─ Data model with groupings array
├─ Frontend utilities (getEffectiveGroups, etc.)
├─ Backend helpers (getNextOffsetForGrouping, etc.)
├─ PropertiesService grouping persistence
└─ Backward compatibility layer

PHASE 2: UI Redesign (NEXT - ~18.5 hours)
├─ Step 2: Groupings Composer
├─ Step 3: Per-Grouping Configuration
├─ Step 5: Tabs + Hierarchical Action Bar
├─ Statistics Panel: Resizable Sidebar
└─ Group Cards: Enhanced with Context

PHASE 3: Integration & Testing (After Phase 2 - ~9 hours)
├─ saveTempGroups() refactor for groupingId
├─ finalizeTempGroups() refactor for per-grouping
├─ Full integration test (2-3 groupings on same level)
└─ Backward compatibility verification
```

---

## 🏗️ Architecture

### Data Model

**Frontend State**
```javascript
state = {
  // Existing (backward compat)
  groupType: 'needs',
  selectedClasses: [...],
  generatedGroups: [...],

  // NEW: Groupings system
  groupings: [
    { id: 'grp_1', label: 'Passe 1', classes: ['6°1', '6°2'], groupCount: 3, active: true },
    { id: 'grp_2', label: 'Passe 2', classes: ['6°3', '6°4'], groupCount: 4, active: true }
  ],
  activeGroupingId: 'grp_1',
  useGroupings: true,  // Flag: multi-grouping mode active

  // NEW: Per-grouping storage
  groupingConfig: {
    'grp_1': { selectedSubject: 'both', distributionType: 'heterogeneous' },
    'grp_2': { selectedSubject: 'both', distributionType: 'heterogeneous' }
  },
  groupingGroups: {
    'grp_1': [...3 generated groups...],
    'grp_2': [...4 generated groups...]
  }
}
```

**Sheet Naming**
```
Single-Pass (backward compat): grBe1TEMP, grBe2TEMP, grBe1, grBe2
Multi-Pass (with groupings):   grBeA1TEMP, grBeA2TEMP, grBeA1, grBeA2 (Passe A)
                               grBeB1TEMP, grBeB2TEMP, grBeB1, grBeB2 (Passe B)
```

**PropertiesService**
```javascript
{
  "GROUPING_needs_grp_1": {
    "id": "grp_1",
    "label": "Passe 1",
    "classes": ["6°1", "6°2"],
    "groupCount": 3,
    "offsetStart": 1,
    "offsetEnd": 3,
    "status": "temp" | "finalized"
  },
  "GROUPING_needs_grp_2": { ... }
}
```

---

## 🛠️ Phase 1 - What Was Implemented

### 1. State Extensions (groupsModuleComplete.html:132-154)
- 6 new state properties for groupings management
- Tracks active grouping and per-grouping configuration
- Enables single-pass backward compatibility mode

### 2. Frontend Utilities (groupsModuleComplete.html:213-386)
**16 Functions Added**:
- **Grouping CRUD**: `createNewGrouping()`, `deleteGrouping()`, `toggleGroupingActive()`
- **Navigation**: `activateGrouping()`, `getActiveGrouping()`, `getActiveGroupings()`
- **Data Access**: `getEffectiveClasses()`, `getEffectiveGroupCount()`, `getEffectiveGroups()`, `getEffectiveConfig()`
- **Config Management**: `setEffectiveGroups()`, `setEffectiveConfig()`
- **Compat**: `initializeSingleGrouping()`, `getEffectiveActiveGroupingId()`

All functions have dual-mode logic:
```javascript
function getEffectiveGroups() {
  if (!state.useGroupings || !state.activeGroupingId) {
    return state.generatedGroups;  // Single-pass mode
  }
  return state.groupingGroups[state.activeGroupingId] || [];  // Multi-pass mode
}
```

### 3. Backend Helpers (Code.js:335-490)
**8 Functions Added**:
- **Metadata Storage**: `storeGroupingMetadata()`, `loadGroupingMetadata()`, `getGroupingMetadataList()`
- **Offset Detection**: `getNextOffsetForGrouping()`, `getGroupingOffsetRange()`
- **Sheet Management**: `listGroupingTempSheets()`, `deleteGroupingTempSheets()`
- **Cleanup**: `deleteGroupingMetadata()`

All functions support both single-pass (groupingId=null) and multi-pass (groupingId='grp_1') modes.

---

## 🎯 Phase 2 Breakdown: UI Redesign

### 2.1 Step 2: Groupings Composer (5 hours)

**Current UI**:
```
Simple grid of checkboxes
Selected: 3 classes
```

**Target UI**:
```
┌─────────────────────────────────────────┐
│ Define Your Groupings (Passes)          │
├─────────────────────────────────────────┤
│ [+ Add New Grouping]                    │
│                                          │
│ ┌─ Grouping 1: "Passe 1" ────────────┐ │
│ │ Classes: 6°1 + 6°2 + 6°3          │ │
│ │ Groups to create: 3                │ │
│ │ [Edit] [Deactivate] [Delete]       │ │
│ └──────────────────────────────────────┘ │
│                                          │
│ ┌─ Grouping 2: "Passe 2" ────────────┐ │
│ │ Classes: 6°4 + 6°5                │ │
│ │ Groups to create: 4                │ │
│ │ [Edit] [Deactivate] [Delete]       │ │
│ └──────────────────────────────────────┘ │
│                                          │
│ [Back] [Configure Each Pass →]          │
└─────────────────────────────────────────┘
```

**Modal: Edit Grouping**:
```
Label: [Passe 1        ]
Description: [Optional      ]
Classes: [6°1] [6°2] [6°3] [6°4] [6°5]
Groups: [3         ]
[Cancel] [Save]
```

**Implementation Notes**:
- Uses `createNewGrouping()` to add
- Modal state in `state.modal`
- Triggers `state.useGroupings = true` when 2+ groupings active
- Reuses validation logic from Step 3

---

### 2.2 Step 3: Per-Grouping Configuration (3 hours)

**Current UI**:
```
Single configuration form
for all selected classes
```

**Target UI**:
```
┌─ Grouping Tabs ─────────────────────┐
│ [Passe 1 (Active)] [Passe 2]        │
├─────────────────────────────────────┤
│ Configuring: Passe 1                │
│ Classes: 6°1, 6°2, 6°3              │
│                                      │
│ • Matière: [Besoins ▼]             │
│ • Distribution: [Hétérogène ▼]     │
│ • LV2 Filter: [Tous ▼]             │
│                                      │
│ [← Précédent] [Suivant →]          │
│ [Retour Step 2] [Générer →]        │
└─────────────────────────────────────┘
```

**Implementation Notes**:
- Tabs render from `state.groupings`
- Config stored per grouping in `state.groupingConfig[id]`
- `setEffectiveConfig()` called on input change
- Prev/Next navigate groupings array

---

### 2.3 Step 5: Groups Display + Action Bar (7 hours)

This is the most complex UI change.

**Layout (Current vs Target)**:
```
CURRENT:
┌─────────────────────────────────────┐
│ Bandeau CONTINUATION (if multi-day) │
├─────────────────────────────────────┤
│ [8 buttons in one row - CLUTTERED] │
├─────────────────────────────────────┤
│   Groups Grid          │ Stats (50%) │
│                        │             │
│  [G1] [G2] [G3]      │ [Stats]    │
│  [G4] [G5] [G6]      │ [Details]  │
│                        │             │
└─────────────────────────────────────┘

TARGET:
┌─────────────────────────────────────┐
│ [Passe 1] [Passe 2] [Passe 3]       │ ← Tabs
├─────────────────────────────────────┤
│   Groups Grid (2/3)        │ Stats (1/3)
│                             │ [resize]
│ [G1] [G2] [G3]            │
│ [G4] [G5] [G6]            │ Avg: 14.2
│                             │ Std: 0.8
├─────────────────────────────────────┤
│ ═══ CALCUL ═══                      │
│ [Régénérer] [Statistiques]          │
│                                      │
│ ═══ SAUVEGARDE ═══                 │
│ [← Remplacer | Ajouter →]          │
│ [💾 Temp] [✅ Finalize]             │
│                                      │
│ ═══ PLUS ═══                        │
│ [⋮ Exporter] [📸 Snapshots] [📋 Log] │
│                                      │
│ ═══ NAVIGATION ═══                 │
│ [← Précédent] [Suivant →]          │
└─────────────────────────────────────┘
```

**Implementation Details**:

1. **Tabs Section** (top)
   ```html
   <div class="grouping-tabs">
     ${state.groupings.map(g => `
       <button class="${state.activeGroupingId === g.id ? 'active' : ''}"
               onclick="activateGrouping('${g.id}')">
         ${g.label} (${state.groupingGroups[g.id]?.length || 0} groupes)
       </button>
     `).join('')}
   </div>
   ```

2. **Groups + Stats Layout** (main)
   ```html
   <div class="flex gap-4">
     <!-- Groups (66% width) -->
     <div class="flex-[2] min-h-[600px] overflow-auto">
       <div class="grid grid-cols-${gridClass}">
         ${getEffectiveGroups().map(renderGroupCard).join('')}
       </div>
     </div>

     <!-- Stats Sidebar (33% width, resizable) -->
     <div class="flex-[1] bg-slate-50 rounded-lg p-4 overflow-auto"
          id="stats-sidebar"
          style="resize: both; overflow: hidden;">
       <div class="resize-handle">⋮</div>
       <!-- Stats content -->
     </div>
   </div>
   ```

3. **Hierarchical Action Bar** (bottom)
   ```html
   <div class="action-bar space-y-3">
     <!-- Calcul Group -->
     <div class="action-group">
       <span class="group-title">CALCUL</span>
       <button onclick="regenerate()">Régénérer</button>
       <button onclick="toggleStatistics()">Statistiques</button>
     </div>

     <!-- Sauvegarde Group -->
     <div class="action-group">
       <span class="group-title">SAUVEGARDE</span>
       <div class="mode-buttons">
         <button class="${state.saveMode === 'replace' ? 'active' : ''}"
                 onclick="setSaveMode('replace')">← Remplacer</button>
         <button class="${state.saveMode === 'append' ? 'active' : ''}"
                 onclick="setSaveMode('append')">Ajouter →</button>
       </div>
       <button class="primary" onclick="saveTempForGrouping()">💾 Temp</button>
       <button class="success" onclick="finalizeTempForGrouping()">✅ Finalize</button>
     </div>

     <!-- More (Dropdown) -->
     <div class="action-group">
       <button onclick="toggleExportMenu()">⋮ Plus</button>
       <!-- Dropdown: [PDF] [CSV] [Snapshots] [Historique] -->
     </div>

     <!-- Navigation -->
     <div class="action-group">
       <button onclick="previousGrouping()">← Grouping précédent</button>
       <button onclick="nextGrouping()">Grouping suivant →</button>
     </div>
   </div>
   ```

---

### 2.4 Statistics Panel: Resizable Sidebar (2 hours)
- Change from `position: absolute; width: 50%` to `flex: 1; max-width: 33.333%`
- Add CSS resize handles
- Implement JS resize listener (optional: snap to 1/3, 1/2, 2/3)
- Keep stats content unchanged, just reposition

---

### 2.5 Group Card Enhancements (1.5 hours)

**Current Card**:
```
┌─────────────────────┐
│ Groupe 1            │
│ 15 élèves           │
├─────────────────────┤
│ [Drag to reorder]   │
│ - Student 1         │
│ - Student 2         │
│ ...                 │
└─────────────────────┘
```

**Target Card**:
```
┌─────────────────────────────────────┐
│ Passe 1 / grBe1 (Range 1-3)        │ ← Grouping context
│                                      │
│ Groupe 1 (Code: grBe1)             │ ← Full code
├─────────────────────────────────────┤
│ 15 élèves | 8F, 7M | Avg: 14.2     │ ← Enhanced stats
│                                      │
│ ⚠️ Alert (if applicable)            │ ← Inline alerts
│                                      │
│ [Drag to reorder]                   │
│ - Student 1 (F, 13.5)               │ ← Score visible
│ - Student 2 (M, 14.8)               │
│ ...                                  │
└─────────────────────────────────────┘
```

**Code Changes**:
```javascript
function renderGroupCard(group, index) {
  const grouping = getActiveGrouping();
  const groupingLabel = grouping?.label || 'Default';
  const groupNumber = state.tempOffsetStart + index;
  const groupPrefix = getGroupPrefix();
  const groupCode = `${groupPrefix}${groupNumber}`;

  const offsetStart = state.tempOffsetStart;
  const offsetEnd = offsetStart + getEffectiveGroups().length - 1;
  const rangeLabel = `${offsetStart}-${offsetEnd}`;

  // Calculate gender breakdown
  const females = group.students.filter(s => s.gender === 'F').length;
  const males = group.students.length - females;

  // Calculate average score
  const avgScore = (group.students.reduce((sum, s) => {
    const score = s.scores?.F && s.scores?.M ? (s.scores.F + s.scores.M) / 2 : 0;
    return sum + score;
  }, 0) / group.students.length).toFixed(1);

  return `
    <div class="group-card" data-group-index="${index}">
      <div class="group-header">
        <p class="text-xs text-slate-600">${groupingLabel} / ${groupCode} (Range ${rangeLabel})</p>
        <h4 class="group-title">${group.name || `Groupe ${groupNumber}`}</h4>
        <p class="text-xs text-slate-500">Code: <code>${groupCode}</code></p>
      </div>
      <div class="group-stats">
        ${group.students.length} élèves | ${females}F, ${males}M | Avg: ${avgScore}
      </div>
      <!-- ... rest of card ... -->
    </div>
  `;
}
```

---

## 📊 Phase 3: Integration & Testing

### 3.1 saveTempGroups() Refactor (2 hours)

**Current Signature**:
```javascript
function saveTempGroups(payload) {
  // payload = { groups, groupType, saveMode, offsetStart, ... }
}
```

**New Signature**:
```javascript
function saveTempGroups(payload) {
  // payload = { groups, groupType, saveMode, offsetStart,
  //             groupingId, groupingLabel, ... }

  const groupingId = payload.groupingId || 'default';
  const typePrefix = getGroupPrefix(payload.groupType);

  // NEW: When creating TEMP sheet names
  if (groupingId !== 'default') {
    // Pattern: grBeA1TEMP, grBeA2TEMP
    const sheetName = `${typePrefix}${groupingId}${startNum + i}TEMP`;
  } else {
    // Backward compat: grBe1TEMP, grBe2TEMP
    const sheetName = `${typePrefix}${startNum + i}TEMP`;
  }

  // NEW: Store grouping metadata
  if (groupingId !== 'default') {
    const metadata = {
      id: groupingId,
      label: payload.groupingLabel,
      offsetStart: startNum,
      offsetEnd: startNum + payload.groups.length - 1,
      status: 'temp'
    };
    storeGroupingMetadata(payload.groupType, groupingId, metadata);
  }
}
```

**Key Changes**:
- Accept `groupingId` parameter
- Use groupingId in sheet naming
- Store grouping metadata to PropertiesService
- Preserve backward compatibility (groupingId='default' → old behavior)

---

### 3.2 finalizeTempGroups() Refactor (2 hours)

**Current Logic**: Finalize ALL TEMP sheets

**New Logic**: Finalize ONLY sheets for a grouping

```javascript
function finalizeTempGroups(payload) {
  // payload = { groupType, finalizeMode, groupingId }

  const groupingId = payload.groupingId || null;
  const typePrefix = getGroupPrefix(payload.groupType);

  // NEW: Only find TEMP sheets for this grouping
  if (groupingId && groupingId !== 'default') {
    tempSheets = listGroupingTempSheets(typePrefix, groupingId);
  } else {
    // Backward compat: find all TEMP sheets
    tempSheets = ... existing logic ...
  }

  // NEW: When renaming
  if (groupingId && groupingId !== 'default') {
    // Pattern: grBeA1TEMP → grBeA1
    const finalName = sheetName.replace('TEMP', '');
  } else {
    // Backward compat: grBe1TEMP → grBe1
    const finalName = sheetName.replace('TEMP', '');
  }

  // NEW: Update metadata status
  if (groupingId && groupingId !== 'default') {
    const metadata = loadGroupingMetadata(payload.groupType, groupingId);
    if (metadata) {
      metadata.status = 'finalized';
      storeGroupingMetadata(payload.groupType, groupingId, metadata);
    }
  }
}
```

---

### 3.3 Integration Tests (3 hours)

**Test Scenario: Two Passes on Same Level**

```javascript
// Day 1 - Pass 1
1. Step 2: Create "Passe 1" (6°1 + 6°2 + 6°3, 3 groups)
2. Step 3: Configure (Besoins, Hétérogène)
3. Step 5: Generate, see grBe1, grBe2, grBe3
4. Save TEMP: Sheets grBe1TEMP, grBe2TEMP, grBe3TEMP created ✓
5. Metadata stored: GROUPING_needs_grp_1 ✓

// Day 1 - Pass 2
6. Step 2: Add "Passe 2" (6°4 + 6°5, 4 groups)
7. Step 3: Configure (Besoins, Hétérogène)
8. Step 5: Switch to Passe 2 tab, generate
9. See grBe4, grBe5, grBe6, grBe7 (offset continues!) ✓
10. Save TEMP for Passe 2: Sheets grBeB1TEMP, grBeB2TEMP, grBeB3TEMP, grBeB4TEMP ✓
11. Sheets from Passe 1 (grBe1TEMP-3) still exist ✓

// Day 1 - Finalization
12. Switch to Passe 1 tab, click "Finalize"
    → Only grBe1TEMP-3 renamed to grBe1-3 ✓
    → grBeB1TEMP-4 remain untouched ✓
13. Switch to Passe 2 tab, click "Finalize"
    → Only grBeB1TEMP-4 renamed to grBeB1-4 ✓
    → grBe1-3 from Passe 1 remain ✓

// Result
Final sheets: grBe1, grBe2, grBe3, grBeB1, grBeB2, grBeB3, grBeB4 ✓
```

---

## 📋 Implementation Checklist

### Phase 2 - UI Redesign
- [ ] **Step 2 Composer**
  - [ ] Render groupings list
  - [ ] Add grouping modal
  - [ ] Save new grouping (calls createNewGrouping)
  - [ ] Delete grouping (calls deleteGrouping)
  - [ ] Edit existing grouping
  - [ ] Test: Create 2+ groupings, all stored in state

- [ ] **Step 3 Config**
  - [ ] Render grouping tabs
  - [ ] Activate grouping on tab click
  - [ ] Load per-grouping config
  - [ ] Save config changes with setEffectiveConfig
  - [ ] Prev/Next button navigation

- [ ] **Step 5 Display**
  - [ ] Render grouping tabs
  - [ ] Activate grouping on tab click
  - [ ] Dynamically update groups grid on tab change
  - [ ] Update group card rendering (add context)
  - [ ] Hierarchical action bar (4 groups)
  - [ ] Button handlers for each action
  - [ ] Test with 3 groupings: tabs work, groups switch correctly

- [ ] **Stats Sidebar**
  - [ ] Move stats to right sidebar
  - [ ] Set flex layout (1/3 width)
  - [ ] Add resize handle
  - [ ] Test resize functionality

### Phase 3 - Integration
- [ ] **saveTempGroups() Update**
  - [ ] Accept groupingId parameter
  - [ ] Generate correct sheet names per grouping
  - [ ] Store grouping metadata
  - [ ] Test: Save Pass 1 & Pass 2 separately

- [ ] **finalizeTempGroups() Update**
  - [ ] Accept groupingId parameter
  - [ ] Finalize only target grouping's TEMP
  - [ ] Update metadata status
  - [ ] Test: Finalize Pass 1 without affecting Pass 2

- [ ] **Full Integration Test**
  - [ ] 2 passes on same level scenario
  - [ ] Verify sheet naming (grBe1-3, grBeB1-4)
  - [ ] Verify no cross-contamination
  - [ ] Verify continuation mode works per grouping

---

## 🎯 Definition of Done

For each phase to be considered complete:

✅ **Code Complete**: All functions implemented, no TODO comments
✅ **Tests Pass**: Manual end-to-end scenario works
✅ **Backward Compat**: Single-pass workflow still works (useGroupings=false)
✅ **Documented**: Code comments updated, no confusion points
✅ **Performance**: No UI lag with 5+ groupings
✅ **Review Ready**: Code clean, styled consistently

---

## 📞 Questions Before Starting?

1. Should grouping IDs use letters (grBeA, grBeB) or numbers (grBe_1, grBe_2)?
2. Should we support >10 groupings or cap at 5 for UI sanity?
3. Should deleted grouping's sheets be archived or destroyed?
4. Should stats panel be sticky/floating or fixed sideb bar?

---

**Ready to begin Phase 2?** ✅

Implementation estimated at 32.5 hours total (~4 days intensive)

