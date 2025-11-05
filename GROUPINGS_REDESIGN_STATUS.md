# Groupings Redesign - Implementation Status

**Date**: 29 octobre 2025
**Status**: 🚧 PHASE 1 (Backend Foundation) - COMPLETE
**Next Phase**: PHASE 2 (UI Redesign) - Ready to start

---

## ✅ PHASE 1 COMPLETED

### What Was Done

#### 1. Data Model Extensions ✅
- **File**: groupsModuleComplete.html (Lines 132-154)
- **Added to state object**:
  - `groupings[]` - Registry of grouping definitions
  - `activeGroupingId` - Currently active grouping
  - `groupingConfig{}` - Per-grouping config storage
  - `groupingGroups{}` - Per-grouping generated groups
  - `useGroupings` - Flag for multi-grouping mode
  - `groupingMetadata{}` - Backend metadata cache

#### 2. Frontend Utility Functions ✅
- **File**: groupsModuleComplete.html (Lines 213-386)
- **Added 16 functions**:
  - `createNewGrouping()` - Create grouping with unique ID
  - `getActiveGrouping()` - Get current grouping object
  - `getEffectiveClasses()` - Get classes for active grouping
  - `getEffectiveGroupCount()` - Get group count for active grouping
  - `getEffectiveGroups()` - Get groups for active grouping
  - `setEffectiveGroups()` - Set groups for active grouping
  - `getEffectiveConfig()` - Get config for active grouping
  - `setEffectiveConfig()` - Set config for active grouping
  - `activateGrouping()` - Switch active grouping
  - `deleteGrouping()` - Remove a grouping
  - `toggleGroupingActive()` - Enable/disable grouping
  - `getActiveGroupings()` - List enabled groupings
  - `initializeSingleGrouping()` - Backward compat mode
  - `getEffectiveActiveGroupingId()` - Get ID with default fallback

#### 3. Backend Grouping Helpers ✅
- **File**: Code.js (Lines 335-490)
- **Added 8 functions**:
  - `storeGroupingMetadata()` - Save grouping info to PropertiesService
  - `loadGroupingMetadata()` - Load grouping info from PropertiesService
  - `getGroupingMetadataList()` - Get all groupings of a type
  - `deleteGroupingMetadata()` - Remove grouping metadata
  - `getNextOffsetForGrouping()` - Find next available number
  - `getGroupingOffsetRange()` - Get min-max range for grouping
  - `listGroupingTempSheets()` - List TEMP sheets for grouping
  - `deleteGroupingTempSheets()` - Clean up TEMP sheets

### Code Statistics
- **Total Lines Added**: ~350
- **Files Modified**: 2 (groupsModuleComplete.html, Code.js)
- **Functions Added**: 24 (16 frontend + 8 backend)
- **Backward Compatibility**: ✅ Maintained (useGroupings=false by default)

---

## 🎯 PHASE 2: UI REDESIGN (READY TO START)

### Step 2 - Groupings Composer
**Objective**: Replace simple class checkboxes with grouping composition interface

#### Current State
```html
<!-- Simple grid of checkboxes for class selection -->
<div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3">
  ${state.availableClasses.map(classe => `
    <label class="class-checkbox-card">
      <input type="checkbox" class="class-checkbox" value="${classe}">
      <span class="class-name">${classe}</span>
    </label>
  `).join('')}
</div>
```

#### Target State
```html
<!-- Groupings list with composition builder -->
<div class="groupings-list">
  <!-- List of defined groupings -->
  ${state.groupings.map(grouping => `
    <div class="grouping-card">
      <h4>${grouping.label}</h4>
      <p>Classes: ${grouping.classes.join(', ')}</p>
      <p>Nombre de groupes: ${grouping.groupCount}</p>
      <button onclick="editGrouping('${grouping.id}')">Éditer</button>
      <button onclick="deleteGrouping('${grouping.id}')">Supprimer</button>
    </div>
  `).join('')}
</div>

<!-- Modal: Edit Grouping -->
<div id="grouping-modal" class="modal">
  <div class="modal-content">
    <h3>Edit Grouping</h3>
    <input type="text" id="grouping-label" placeholder="Passe 1">
    <input type="text" id="grouping-desc" placeholder="Description">
    <!-- Class selector -->
    <!-- Group count input -->
    <button onclick="saveGrouping()">Save</button>
    <button onclick="closeModal()">Cancel</button>
  </div>
</div>
```

#### Estimated Effort
- **Duration**: 4-6 hours
- **Complexity**: Medium
- **Dependencies**: None (uses existing utilities)

---

### Step 3 - Per-Grouping Configuration
**Objective**: Configure each grouping independently with navigation between groupings

#### Current State
```html
<!-- Single global config -->
<div class="config-card">
  <select class="subject-selector">
    <option value="both">Math + Français</option>
    ...
  </select>
  <!-- ... single config for all classes selected in step 2 ... -->
</div>
```

#### Target State
```html
<!-- Per-grouping config with tabs/navigation -->
<div class="grouping-tabs">
  ${state.groupings.map(g => `
    <button class="tab ${state.activeGroupingId === g.id ? 'active' : ''}" onclick="activateGrouping('${g.id}')">
      ${g.label}
    </button>
  `).join('')}
</div>

<div class="config-section">
  <p class="text-sm text-slate-600">
    Configuring: ${getActiveGrouping().label}
    (${getActiveGrouping().classes.join(', ')})
  </p>

  <!-- Config for active grouping only -->
  <select class="subject-selector" onchange="updateGroupingConfig('selectedSubject', this.value)">
    ...
  </select>
</div>

<div class="grouping-nav">
  <button onclick="previousGrouping()">← Précédent</button>
  <button onclick="nextGrouping()">Suivant →</button>
</div>
```

#### Estimated Effort
- **Duration**: 3-4 hours
- **Complexity**: Low-Medium
- **Dependencies**: Step 2 completion

---

### Step 5 - Multi-Grouping Display
**Objective**: Show groups per active grouping with improved action bar hierarchy

#### Current Issues
1. **Action bar overloaded**: 8+ buttons crammed without hierarchy
2. **No grouping context**: User doesn't know which pass they're viewing
3. **Stats take too much space**: 50% width left for groups manipulation
4. **No visual separation**: Calc / Save / Export all mixed

#### Target State

**Tab Navigation**
```html
<div class="grouping-tabs-step5">
  ${state.groupings.map(g => `
    <button class="grouping-tab ${state.activeGroupingId === g.id ? 'active' : ''}" onclick="activateGrouping('${g.id}')">
      <span class="tab-label">${g.label}</span>
      <span class="tab-count">${state.groupingGroups[g.id]?.length || 0} groupes</span>
    </button>
  `).join('')}
</div>
```

**Main Layout** (groups + stats sidebar)
```html
<div class="flex gap-4">
  <!-- Groups (2/3 width) -->
  <div class="flex-[2] overflow-auto">
    <!-- Groups grid for active grouping -->
    <div class="grid grid-cols-${gridClass}">
      ${getEffectiveGroups().map((group, index) => renderGroupCard(group, index)).join('')}
    </div>
  </div>

  <!-- Stats Sidebar (1/3 width, resizable) -->
  <div class="flex-[1] bg-slate-50 rounded-lg p-4 overflow-auto" id="stats-sidebar">
    <!-- Statistics panel with resize handle -->
    <div class="resize-handle">⋮</div>
    <!-- Stats content -->
  </div>
</div>
```

**Hierarchical Action Bar**
```html
<div class="action-groups">
  <!-- GROUP 1: Calculation -->
  <div class="action-group">
    <span class="group-label">Calcul</span>
    <button onclick="regenerate()">Régénérer</button>
    <button onclick="toggleStatistics()">Statistiques</button>
  </div>

  <!-- GROUP 2: Save/Finalize -->
  <div class="action-group">
    <span class="group-label">Sauvegarde</span>
    <div class="mode-selector">
      <button class="mode-btn ${state.saveMode === 'replace' ? 'active' : ''}" onclick="setSaveMode('replace')">
        ← Remplacer
      </button>
      <button class="mode-btn ${state.saveMode === 'append' ? 'active' : ''}" onclick="setSaveMode('append')">
        Ajouter →
      </button>
    </div>
    <button class="primary-btn" onclick="saveTempGroups()">💾 Temp</button>
    <button class="success-btn" onclick="finalizeTempGroups()">✅ Finalize</button>
  </div>

  <!-- GROUP 3: Exports (Dropdown menu) -->
  <div class="action-group">
    <button onclick="toggleExportMenu()">⋮ Exporter</button>
    <!-- Dropdown with [PDF] [CSV] [JSON] [Historique] [Snapshots] -->
  </div>

  <!-- GROUP 4: Navigation (between groupings) -->
  <div class="action-group">
    <button onclick="previousGrouping()">← Précédent</button>
    <button onclick="nextGrouping()">Suivant →</button>
  </div>
</div>
```

#### Group Card Enhancements
```javascript
// Add to group card header:
// 1. Grouping label (e.g., "Passe 1")
// 2. Number range (e.g., "grBe1-3")
// 3. Gender breakdown (e.g., "8F / 7M")
// 4. Score indicator (avg, std dev)
// 5. In-card alerts (if unbalanced)

const groupingLabel = getActiveGrouping()?.label || 'Sans label';
const offsetStart = state.tempOffsetStart;
const offsetEnd = offsetStart + getEffectiveGroups().length - 1;
```

#### Estimated Effort
- **Duration**: 6-8 hours
- **Complexity**: High
- **Dependencies**: Steps 2-3 completion

---

## 📊 COMPLETE IMPLEMENTATION TIMELINE

```
PHASE 1 (DONE) ✅
├─ Data Model: 1h
├─ Frontend Utils: 2h
├─ Backend Helpers: 2h
└─ Total: 5h

PHASE 2 (NEXT)
├─ Step 2 Groupings Composer: 5h
├─ Step 3 Per-Grouping Config: 3h
├─ Step 5 Redesign (UI + Action Bar): 7h
├─ Statistics Panel Sidebar: 2h
├─ Group Card Enhancements: 1.5h
└─ Subtotal: 18.5h

PHASE 3 (TESTING)
├─ saveTempGroups() refactor: 2h
├─ finalizeTempGroups() refactor: 2h
├─ Integration tests: 3h
├─ Multi-grouping scenario test: 2h
└─ Subtotal: 9h

TOTAL ESTIMATED: ~32.5 hours (~4 days of intensive work)
```

---

## 🔄 Backward Compatibility Guarantee

### How It Works
1. **useGroupings = false** (default) → Old behavior preserved
2. User selects classes → Creates implicit single grouping
3. All effective*() functions check useGroupings flag
4. If false, uses global state (selectedClasses, numGroups, generatedGroups)
5. If true, uses per-grouping storage (groupings[], groupingGroups[])

### Migration Path
- Old spreadsheets: Automatically use single-grouping mode
- New users: Can opt into multi-grouping via Step 2
- No data loss or breaking changes

---

## 🎯 Success Criteria (For Phase 2)

- [ ] User can define 2+ groupings at Step 2
- [ ] Each grouping has different classes & group counts
- [ ] Step 5 shows one grouping at a time via tabs
- [ ] Action bar has 4 clear groups (Calcul, Sauvegarde, Exports, Nav)
- [ ] Stats panel is max 1/3 width with resize handle
- [ ] Group cards show grouping label + offset range
- [ ] Saving grouping A doesn't affect grouping B
- [ ] No UI lag with 5-10 groupings defined
- [ ] All tests pass with backward compat preserved

---

## 📝 Next Immediate Actions

1. **Review** this implementation plan with stakeholders
2. **Prioritize** which phase to start first (recommend Phase 2.1: Step 2)
3. **Allocate** developer time (~8 hours/day min for focus)
4. **Test** with real educational scenario (6 classes, 2 passes)

---

**Status**: Ready to begin Phase 2
**Lead**: [Your Name]
**Created**: 29 octobre 2025

