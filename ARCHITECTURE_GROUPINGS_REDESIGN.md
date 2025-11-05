# Architecture Redesign: Groupings System for Multi-Pass Workflows

**Status**: 🎯 Planning Phase
**Date**: 29 octobre 2025
**Objective**: Implement a coherent groupings model that enables true multi-pass group creation on the same student level

---

## 1. Core Concept: Groupings Model

### Current State (Broken)
```
Classes Selection (Simple checkboxes)
         ↓
Configuration (One per type)
         ↓
Generation (All classes at once)
         ↓
Step 5 (Display everything, no grouping context)
         ↓
Save (Everything or nothing, no separation)
```

**Problem**: No concept of "grouping" = no way to say "Pass A = 6°1+6°2 (3 groups)" and "Pass B = 6°3+6°5 (4 groups)" separately.

### Target State (Multi-Pass Aware)
```
Groupings Composition (Prepare multiple passes)
    ├─ Grouping A: 6°1 + 6°2 + 6°3 → 3 groups (Passe 1)
    ├─ Grouping B: 6°4 + 6°5 → 4 groups (Passe 2)
    └─ Grouping C: 6°1+6°2+6°3+6°4+6°5 → 2 large groups (Alt)
         ↓
Per-Grouping Configuration (Steps 3-4)
    ├─ Grouping A: LV2=English, Distribution=Gender
    ├─ Grouping B: LV2=Spanish, Distribution=Balanced
    └─ Grouping C: No LV2, Distribution=Score
         ↓
Per-Grouping Generation & Review (Step 5 with tabs/selector)
    ├─ Display [Grouping A tabs] with 3 groups
    ├─ Display [Grouping B tabs] with 4 groups
    └─ Each tab shows its own groups, statistics, action bar
         ↓
Per-Grouping Save & Finalization
    ├─ Save Grouping A: grBeA1TEMP, grBeA2TEMP, grBeA3TEMP
    ├─ Save Grouping B: grBeB1TEMP, grBeB2TEMP, grBeB3TEMP, grBeB4TEMP
    └─ Finalize: grBeA1, grBeA2, grBeA3, grBeB1, grBeB2, grBeB3, grBeB4
```

---

## 2. Data Structures

### State Object Extensions
```javascript
// Current state
state = {
  groupType: 'needs',
  selectedClasses: ['6°1', '6°2'],
  generatedGroups: [...],
  ...
}

// NEW: Groupings layer
state = {
  groupType: 'needs',

  // NEW: Groupings registry
  groupings: [
    {
      id: 'grp_a',
      label: 'Passe 1',
      classes: ['6°1', '6°2', '6°3'],
      groupCount: 3,
      active: true,
      description: 'Premier regroupement'
    },
    {
      id: 'grp_b',
      label: 'Passe 2',
      classes: ['6°4', '6°5'],
      groupCount: 4,
      active: false,
      description: 'Deuxième regroupement'
    }
  ],

  // Current grouping being edited/displayed
  activeGrouping: 'grp_a',

  // Per-grouping configuration
  groupingConfig: {
    'grp_a': {
      lv2Filter: 'English',
      distribution: 'gender',
      offsetStart: 1
    },
    'grp_b': {
      lv2Filter: 'Spanish',
      distribution: 'balanced',
      offsetStart: 4
    }
  },

  // Per-grouping generated groups
  groupingGroups: {
    'grp_a': [...3 groups...],
    'grp_b': [...4 groups...]
  },

  // Existing for single-grouping backward compatibility
  selectedClasses: ['6°1', '6°2', '6°3', '6°4', '6°5'],
  generatedGroups: [...all groups...]
}
```

### Backend: GroupingMetadata (PropertiesService)
```javascript
{
  "grouping_a": {
    "id": "grp_a",
    "label": "Passe 1",
    "created": "2025-10-29T09:00:00Z",
    "classes": ["6°1", "6°2", "6°3"],
    "groupCount": 3,
    "offsetStart": 1,
    "offsetEnd": 3,
    "sheetNames": ["grBeA1TEMP", "grBeA2TEMP", "grBeA3TEMP"],
    "status": "temp" // or "finalized"
  },
  "grouping_b": {
    ...
  }
}
```

---

## 3. UI/UX Redesign

### Step 2: Groupings Composer (REDESIGNED)
**Current**: Simple class checkboxes
**New**: Composition builder with multiple groupings

#### Layout
```
┌─────────────────────────────────────────────┐
│ Define Your Groupings                       │
├─────────────────────────────────────────────┤
│ [+ New Grouping] Button                     │
│                                              │
│ Grouping #1: "Passe 1"                      │
│ ├─ Classes: [6°1][6°2][6°3] [Edit]        │
│ ├─ Groups to create: 3                      │
│ ├─ Description: (optional text)             │
│ └─ [Delete] [Make inactive]                 │
│                                              │
│ Grouping #2: "Passe 2"                      │
│ ├─ Classes: [6°4][6°5] [Edit]              │
│ ├─ Groups to create: 4                      │
│ ├─ Description: (optional text)             │
│ └─ [Delete] [Make inactive]                 │
│                                              │
│ Grouping #3 (Inactive): "Alternative"       │
│ └─ [Reactivate] [Delete]                    │
│                                              │
│ Next: Configure per grouping →              │
└─────────────────────────────────────────────┘
```

#### Modal: Edit Grouping
```
┌─────────────────────────────────┐
│ Edit Grouping "Passe 1"         │
├─────────────────────────────────┤
│ Label: [Passe 1          ]      │
│ Description: [Premier...  ]     │
│                                  │
│ Select classes:                  │
│ [6°1]✓ [6°2]✓ [6°3]✓ [6°4]  │
│ [6°5] [6°6] [6°7]            │
│                                  │
│ Number of groups: [3         ]  │
│                                  │
│ [Cancel] [Save]                 │
└─────────────────────────────────┘
```

---

### Step 3: Per-Grouping Configuration (REDESIGNED)
**Current**: One configuration shared
**New**: Per-grouping config with clear indication

#### Layout
```
┌─────────────────────────────────────────┐
│ Configure Grouping "Passe 1"            │
│ (6°1, 6°2, 6°3 → 3 groups)             │
├─────────────────────────────────────────┤
│ Criteria Configuration                  │
│ ├─ Matière: [Besoins        ]          │
│ ├─ Distribution: [Gender    ]          │
│ ├─ LV2 Filter: [English only]          │
│ └─ Additional params...                │
│                                          │
│ [← Prev Grouping] [Next Grouping →]    │
│ [Back to Groupings] [Next to Review]   │
└─────────────────────────────────────────┘
```

---

### Step 5: Multi-Grouping Display (CRITICAL REDESIGN)
**Current**: All groups in one view, cluttered action bar
**New**: Tabs/Selector for active grouping, hierarchical actions

#### Layout
```
┌─────────────────────────────────────────────────────────┐
│ Generated Groups                                        │
│                                                          │
│ [Grouping A: Passe 1] [Grouping B: Passe 2]          │
│                                                          │
│ ┌─────────────────────────────────┐                    │
│ │ Grouping A (3 groups, 45 élèves)│                   │
│ │ Number range: grBe1-3            │                   │
│ │                                   │                   │
│ │ [Card 1: Group grBe1] [Card 2]...│                  │
│ │ [Card 3: Group grBe3]            │                  │
│ │                                   │                   │
│ │ ┌──STATS (1/3 width, right)────┐ │                  │
│ │ │ Group Statistics              │ │                  │
│ │ │ • Avg Score: 14.2            │ │                  │
│ │ │ • Std Dev: 0.8               │ │                  │
│ │ │ • Gender: 23F/22M            │ │                  │
│ │ └──────────────────────────────┘ │                  │
│ │                                   │                  │
│ │ ACTION GROUPS:                   │                  │
│ │ ┌─────────────────────────────┐  │                  │
│ │ │ Calcul                      │  │                  │
│ │ │ [Régénérer] [Statistiques]  │  │                  │
│ │ └─────────────────────────────┘  │                  │
│ │ ┌─────────────────────────────┐  │                  │
│ │ │ Sauvegarde (Choose mode)    │  │                  │
│ │ │ [← Remplacer] [Ajouter →]   │  │                  │
│ │ │ [💾 Temp] [✅ Finalize]      │  │                  │
│ │ └─────────────────────────────┘  │                  │
│ │ ┌─────────────────────────────┐  │                  │
│ │ │ Exports                     │  │                  │
│ │ │ [PDF] [CSV] [JSON]          │  │                  │
│ │ └─────────────────────────────┘  │                  │
│ └─────────────────────────────────┘ │                  │
│                                      │                  │
│ [← Prev Grouping] [Next Grouping →] │                  │
└─────────────────────────────────────────────────────────┘
```

#### Stats Panel (Resizable)
- Default: **1/3 width** (right sidebar, always visible)
- Expandable via drag handle
- Collapsible for full-width group manipulation
- Can be "sticky" to top when scrolling groups

#### Action Bar Restructuring
```
GROUP ACTIONS (Top row)
├─ Regenerate sub-group: [Régénérer]
└─ View stats: [Statistiques] (toggle sidebar)

SAVE/FINALIZE ACTIONS (Middle row)
├─ Save Mode selector: [← Remplacer | Ajouter →]
├─ Primary actions: [💾 Temp] [✅ Finalize]
└─ Notes: "Remplacer: effacera autres passes du même type"

NAVIGATION (Bottom row)
├─ Between groupings: [← Prev Grouping] [Next Grouping →]
├─ Or: Grouping selector: [Passe 1 ▼] → dropdown

EXPORT (Dropdown menu or separate section)
└─ [⋮ More] → [PDF] [CSV] [JSON]
```

#### Group Card Enhancements
```
┌─────────────────────────────┐
│ Grouping A / grBe1          │ ← Grouping label + number
├─────────────────────────────┤
│ Groupe 1 (Code: grBe1)      │ ← Title + code
│ Passe 1, Range grBe1-3      │ ← Grouping context
├─────────────────────────────┤
│ 15 élèves | 8F, 7M | ▲ 14.2 │ ← Stats: count, gender, score
├─────────────────────────────┤
│ [Drag to reorder students]  │
│ - Student 1 (F, 13.5)       │
│ - Student 2 (M, 14.8)       │
│ ...                          │
├─────────────────────────────┤
│ ⚠️ Alert: Unbalanced gender  │ ← In-card alerts (if any)
└─────────────────────────────┘
```

---

## 4. Backend Changes

### saveTempGroups() Refactor
**Current**:
- Saves all generated groups at once
- Uses single offset (startNum)
- No grouping awareness

**New**:
- Accepts groupingId, groupingLabel, offsetStart as parameters
- Creates TEMP sheets with grouping prefix: `grBeA1TEMP`, `grBeA2TEMP`... or `grBe1TEMP`, `grBe2TEMP`...
- Stores grouping metadata in PropertiesService under grouping key
- Supports multiple concurrent grouping saves (A, B, C can all be in TEMP)

```javascript
function saveTempGroups(payload) {
  // NEW: payload.groupingId, payload.groupingLabel, payload.offsetStart
  // Create TEMP sheets:
  // - If groupingId: grBe${groupingId}${number}TEMP
  // - Else: grBe${offsetStart + index}TEMP

  // Store metadata per grouping
  // Track which groupings are in TEMP state
}
```

### finalizeTempGroups() Refactor
**Current**:
- Finalizes ALL TEMP sheets
- Deletes old finalized sheets
- Replaces everything or appends everything

**New**:
- Accepts groupingId parameter
- Finalizes ONLY the TEMP sheets for that grouping
- Renaming: `grBeA1TEMP` → `grBeA1`, preserves other groupings untouched
- Merge vs Replace per-grouping (not global)

```javascript
function finalizeTempGroups(payload) {
  // NEW: payload.groupingId
  // Finalize only sheets matching this grouping:
  // - Find all grBe${groupingId}*TEMP
  // - Rename to grBe${groupingId}*
  // - Other groupings' sheets remain untouched

  // If no groupingId: finalize all (backward compat)
}
```

### New Helper Functions
```javascript
// Get all groupings for a type (needs, language, option)
function getGroupingsForType(groupType) {
  // Return from PropertiesService
}

// Get next available offset for a grouping
function getNextOffsetForGrouping(groupingId) {
  // Scan sheets grBe${groupingId}* and find max number
  // Return max + 1
}

// Get offset range for a grouping (min-max numbers used)
function getGroupingOffsetRange(groupingId) {
  // Return {min: 1, max: 3, count: 3}
}
```

---

## 5. Backward Compatibility

### Single-Grouping Mode (Default)
If user doesn't explicitly create multiple groupings:
- Treat entire selection as one implicit grouping (id='default')
- UI still shows tabs/selector, but only one tab
- Behavior identical to current

### Migration Path
- Old spreadsheets: Load with implicit grouping
- New spreadsheets: Encourage explicit grouping from start
- No data loss, full backward compat

---

## 6. Implementation Phases

### Phase 1: Data Model & Backend (Days 1-2)
- [x] Extend state with groupings array
- [x] Implement PropertiesService grouping metadata storage
- [x] Refactor saveTempGroups() for per-grouping save
- [x] Refactor finalizeTempGroups() for per-grouping finalization
- [ ] Create grouping helper functions (getNextOffset, getRange, etc.)

### Phase 2: Step 2 Redesign (Day 3)
- [ ] Build groupings composer UI (list + edit modal)
- [ ] Implement grouping CRUD (add, edit, delete, toggle active)
- [ ] Wire state management for groupings array
- [ ] Add validation (at least 1 active grouping required)

### Phase 3: Steps 3-4 Per-Grouping (Day 4)
- [ ] Redesign Step 3 to accept activeGrouping context
- [ ] Implement per-grouping config storage in state.groupingConfig
- [ ] Add navigation between groupings (Prev/Next)
- [ ] Wire configuration per grouping to Step 5 generation

### Phase 4: Step 5 Redesign (Days 5-6)
- [ ] Implement tab/selector UI for active grouping
- [ ] Refactor renderStep5_Groups() to display one grouping at a time
- [ ] Implement stats panel resizable sidebar (max 1/3 width)
- [ ] Reorganize action bar into hierarchical groups (Calcul, Sauvegarde, Exports)
- [ ] Update group cards with grouping label + offset context

### Phase 5: Testing & Polish (Day 7)
- [ ] End-to-end test: 2 groupings on same level
- [ ] Verify no cross-grouping contamination
- [ ] Test finalization per grouping
- [ ] Verify backward compatibility with single grouping
- [ ] Stress test with 3-4 groupings

---

## 7. Success Criteria

- [ ] User can define multiple groupings (passes) at Step 2
- [ ] Each grouping can have different class composition & group count
- [ ] Configuration is applied per-grouping at Step 3
- [ ] Step 5 displays one grouping at a time with clear tabs
- [ ] Action bar is hierarchical and uncluttered
- [ ] Stats panel doesn't consume >1/3 of width
- [ ] Saving grouping A doesn't affect grouping B's TEMP
- [ ] Finalizing grouping A doesn't delete grouping B's sheets
- [ ] Continuation mode shows correct offset per grouping
- [ ] All groups finalize with proper numbering (grBeA1, grBeA2, grBeB1, grBeB2...)

---

**Next**: Begin Phase 1 implementation of data model and backend

