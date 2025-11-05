# Groupings Implementation Plan - Detailed Code Changes

**Objective**: Add groupings data model and core backend support
**Phase**: 1 (Data Model + Backend Foundation)
**Files Modified**: groupsModuleComplete.html (state), Code.js (helpers)

---

## Part A: Frontend - State Object Extensions

### Location: groupsModuleComplete.html, Lines 81-131 (State initialization)

#### Add to state object (after line 130):
```javascript
// ═══════════════════════════════════════════════════════════════
// 🆕 GROUPINGS MODEL (Multi-pass support)
// ═══════════════════════════════════════════════════════════════

// Groupings registry: Array of {id, label, classes, groupCount, active, description}
groupings: [],

// Currently active grouping being edited/displayed
activeGroupingId: null,

// Per-grouping configuration (overrides global config)
// Format: { groupingId: { lv2Filter, distribution, offsetStart, ... } }
groupingConfig: {},

// Per-grouping generated groups (stores results per grouping)
// Format: { groupingId: [...groups] }
groupingGroups: {},

// Track if grouping mode is active
useGroupings: false,  // true if user has created multiple groupings, false for backward compat

// Grouping metadata from backend (for continuation mode)
groupingMetadata: {},  // { groupingId: { label, classes, groupCount, offsetStart, offsetEnd, status } }
```

---

## Part B: Frontend - Helper Functions

### Add these functions to groupsModuleComplete.html (around line 200-300, with other utilities):

```javascript
// ═══════════════════════════════════════════════════════════════
// 🆕 GROUPINGS UTILITIES
// ═══════════════════════════════════════════════════════════════

/**
 * Create a new grouping with unique ID
 */
function createNewGrouping(label, classes, groupCount) {
  const id = 'grp_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  return {
    id,
    label: label || 'Regroupement ' + (state.groupings.length + 1),
    classes: classes || [],
    groupCount: groupCount || 3,
    active: true,
    description: ''
  };
}

/**
 * Get active grouping object
 */
function getActiveGrouping() {
  if (!state.activeGroupingId) return null;
  return state.groupings.find(g => g.id === state.activeGroupingId);
}

/**
 * Get active grouping ID (or implicit default if none)
 */
function getEffectiveActiveGroupingId() {
  if (state.useGroupings && state.activeGroupingId) {
    return state.activeGroupingId;
  }
  return 'default';
}

/**
 * Get classes for active grouping (or selected classes if no groupings)
 */
function getEffectiveClasses() {
  if (!state.useGroupings || !state.activeGroupingId) {
    return state.selectedClasses;
  }
  const grouping = getActiveGrouping();
  return grouping ? grouping.classes : state.selectedClasses;
}

/**
 * Get group count for active grouping
 */
function getEffectiveGroupCount() {
  if (!state.useGroupings || !state.activeGroupingId) {
    return state.numGroups;
  }
  const grouping = getActiveGrouping();
  return grouping ? grouping.groupCount : state.numGroups;
}

/**
 * Get effective groups array (per-grouping or global)
 */
function getEffectiveGroups() {
  if (!state.useGroupings || !state.activeGroupingId) {
    return state.generatedGroups;
  }
  return state.groupingGroups[state.activeGroupingId] || [];
}

/**
 * Set effective groups (per-grouping or global)
 */
function setEffectiveGroups(groups) {
  if (!state.useGroupings || !state.activeGroupingId) {
    state.generatedGroups = groups;
  } else {
    state.groupingGroups[state.activeGroupingId] = groups;
  }
}

/**
 * Get config for active grouping (or global config if no groupings)
 */
function getEffectiveConfig() {
  if (!state.useGroupings || !state.activeGroupingId) {
    return {
      selectedSubject: state.selectedSubject,
      distributionType: state.distributionType,
      selectedLanguage: state.selectedLanguage
    };
  }
  const groupingId = state.activeGroupingId;
  return state.groupingConfig[groupingId] || {
    selectedSubject: state.selectedSubject,
    distributionType: state.distributionType,
    selectedLanguage: state.selectedLanguage
  };
}

/**
 * Set config for active grouping
 */
function setEffectiveConfig(config) {
  if (!state.useGroupings || !state.activeGroupingId) {
    state.selectedSubject = config.selectedSubject || state.selectedSubject;
    state.distributionType = config.distributionType || state.distributionType;
    state.selectedLanguage = config.selectedLanguage || state.selectedLanguage;
  } else {
    state.groupingConfig[state.activeGroupingId] = config;
  }
}

/**
 * Activate a grouping (switch view to that grouping)
 */
function activateGrouping(groupingId) {
  state.activeGroupingId = groupingId;
  render(); // Re-render to show that grouping
}

/**
 * Delete a grouping
 */
function deleteGrouping(groupingId) {
  state.groupings = state.groupings.filter(g => g.id !== groupingId);
  delete state.groupingConfig[groupingId];
  delete state.groupingGroups[groupingId];

  // If we deleted the active one, switch to another
  if (state.activeGroupingId === groupingId && state.groupings.length > 0) {
    state.activeGroupingId = state.groupings[0].id;
  } else if (state.groupings.length === 0) {
    state.useGroupings = false;
    state.activeGroupingId = null;
  }
}

/**
 * Toggle grouping active/inactive status
 */
function toggleGroupingActive(groupingId) {
  const grouping = state.groupings.find(g => g.id === groupingId);
  if (grouping) {
    grouping.active = !grouping.active;
  }
}

/**
 * Get list of active (enabled) groupings only
 */
function getActiveGroupings() {
  return state.groupings.filter(g => g.active);
}

/**
 * Convert current state into a single implicit grouping
 * (for backward compat when user doesn't use groupings)
 */
function initializeSingleGrouping() {
  if (state.groupings.length === 0 && state.useGroupings === false) {
    // Create implicit single grouping from current selection
    const grouping = {
      id: 'default',
      label: 'Défaut',
      classes: state.selectedClasses,
      groupCount: state.numGroups,
      active: true,
      description: 'Regroupement unique'
    };
    state.groupings = [grouping];
    state.activeGroupingId = 'default';
    state.useGroupings = false; // Still backward compat mode
  }
}
```

---

## Part C: Backend - PropertiesService Helpers

### Location: Code.js, add new functions around line 350-450

```javascript
/**
 * 🆕 GROUPINGS METADATA - PropertiesService storage/retrieval
 */

function storeGroupingMetadata(groupType, groupingId, metadata) {
  const ps = PropertiesService.getUserProperties();
  const key = `GROUPING_${groupType}_${groupingId}`;
  ps.setProperty(key, JSON.stringify(metadata));
  console.log(`✅ Stored grouping metadata: ${key}`);
}

function loadGroupingMetadata(groupType, groupingId) {
  const ps = PropertiesService.getUserProperties();
  const key = `GROUPING_${groupType}_${groupingId}`;
  const data = ps.getProperty(key);
  return data ? JSON.parse(data) : null;
}

function getGroupingMetadataList(groupType) {
  const ps = PropertiesService.getUserProperties();
  const allProps = ps.getProperties();
  const prefix = `GROUPING_${groupType}_`;
  const result = {};

  for (const key in allProps) {
    if (key.startsWith(prefix)) {
      const groupingId = key.substring(prefix.length);
      result[groupingId] = JSON.parse(allProps[key]);
    }
  }
  return result;
}

function deleteGroupingMetadata(groupType, groupingId) {
  const ps = PropertiesService.getUserProperties();
  const key = `GROUPING_${groupType}_${groupingId}`;
  ps.deleteProperty(key);
  console.log(`🗑️  Deleted grouping metadata: ${key}`);
}

/**
 * Get next available offset for a specific grouping
 * Scans sheets for pattern: typePrefix + groupingId + number + optional TEMP
 * Example: if using "A" as groupingId: grBeA1, grBeA2, grBeA3TEMP → returns 4
 * Or if no groupingId: grBe1, grBe2, grBe3 → returns 4
 */
function getNextOffsetForGrouping(typePrefix, groupingId) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheets = ss.getSheets();
  let maxNum = 0;

  for (const sheet of sheets) {
    const name = sheet.getName();

    // Pattern matching depends on whether groupingId is used
    let pattern;
    if (groupingId && groupingId !== 'default') {
      // Pattern: grBe${groupingId}${number}(TEMP)?
      // Example: grBeA1, grBeA2, grBeA3TEMP
      pattern = new RegExp(`^${typePrefix}${groupingId}(\\d+)(TEMP)?$`);
    } else {
      // Pattern: grBe${number}(TEMP)?
      // Example: grBe1, grBe2, grBe3TEMP
      pattern = new RegExp(`^${typePrefix}(\\d+)(TEMP)?$`);
    }

    const match = name.match(pattern);
    if (match) {
      const num = parseInt(match[1], 10);
      if (num > maxNum) maxNum = num;
    }
  }

  return maxNum + 1;
}

/**
 * Get offset range (min-max) for a grouping
 * Returns {min: X, max: Y, count: Z}
 */
function getGroupingOffsetRange(typePrefix, groupingId) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheets = ss.getSheets();
  let minNum = null;
  let maxNum = null;

  for (const sheet of sheets) {
    const name = sheet.getName();

    let pattern;
    if (groupingId && groupingId !== 'default') {
      pattern = new RegExp(`^${typePrefix}${groupingId}(\\d+)(TEMP)?$`);
    } else {
      pattern = new RegExp(`^${typePrefix}(\\d+)(TEMP)?$`);
    }

    const match = name.match(pattern);
    if (match) {
      const num = parseInt(match[1], 10);
      if (minNum === null || num < minNum) minNum = num;
      if (maxNum === null || num > maxNum) maxNum = num;
    }
  }

  if (minNum === null || maxNum === null) {
    return { min: null, max: null, count: 0 };
  }

  return {
    min: minNum,
    max: maxNum,
    count: maxNum - minNum + 1
  };
}

/**
 * List all TEMP sheets for a grouping
 */
function listGroupingTempSheets(typePrefix, groupingId) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheets = ss.getSheets();
  const result = [];

  for (const sheet of sheets) {
    const name = sheet.getName();

    let pattern;
    if (groupingId && groupingId !== 'default') {
      pattern = new RegExp(`^${typePrefix}${groupingId}(\\d+)TEMP$`);
    } else {
      pattern = new RegExp(`^${typePrefix}(\\d+)TEMP$`);
    }

    if (name.match(pattern)) {
      result.push(name);
    }
  }

  return result.sort();
}

/**
 * Delete all TEMP sheets for a grouping (used for cleanup/reset)
 */
function deleteGroupingTempSheets(typePrefix, groupingId) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const tempSheets = listGroupingTempSheets(typePrefix, groupingId);

  for (const sheetName of tempSheets) {
    const sheet = ss.getSheetByName(sheetName);
    if (sheet) {
      ss.deleteSheet(sheet);
      console.log(`🗑️  Deleted TEMP sheet for grouping: ${sheetName}`);
    }
  }
}
```

---

## Part D: Backend - Updated saveTempGroups() Signature

### Current Function Signature (Line ~2535):
```javascript
function saveTempGroups(payload) {
```

### NEW Support for Grouping Parameter:
```javascript
/**
 * Save groups to TEMP sheets
 * @param payload {
 *   groups: [...],
 *   groupType: 'needs' | 'language' | 'option',
 *   saveMode: 'replace' | 'append',
 *   groupingId?: 'grp_123' (optional, for multi-grouping mode),
 *   groupingLabel?: 'Passe 1' (optional, for metadata),
 *   offsetStart?: number (optional, force specific offset)
 * }
 */
function saveTempGroups(payload) {
  // At start, determine if this is a grouping-aware call
  const groupingId = payload.groupingId || null;
  const groupingLabel = payload.groupingLabel || 'Sans label';

  // ... rest of function ...

  // When creating sheet names:
  if (groupingId && groupingId !== 'default') {
    // Pattern: grBe${groupingId}${number}TEMP
    const finalName = typePrefix + groupingId + (startNum + i) + 'TEMP';
  } else {
    // Pattern: grBe${number}TEMP (backward compat)
    const finalName = typePrefix + (startNum + i) + 'TEMP';
  }

  // ... rest ...

  // Store grouping metadata if groupingId provided
  if (groupingId && groupingId !== 'default') {
    const metadata = {
      id: groupingId,
      label: groupingLabel,
      created: new Date().toISOString(),
      status: 'temp',
      offsetStart: startNum,
      offsetEnd: startNum + payload.groups.length - 1,
      groupCount: payload.groups.length,
      sheetNames: savedSheetNames // array of created sheets
    };
    storeGroupingMetadata(payload.groupType, groupingId, metadata);
  }
}
```

---

## Part E: Backend - Updated finalizeTempGroups() Signature

### NEW Support for Grouping Parameter:
```javascript
/**
 * Finalize TEMP groups to permanent sheets
 * @param payload {
 *   groupType: 'needs' | 'language' | 'option',
 *   finalizeMode: 'merge' | 'replace',
 *   groupingId?: 'grp_123' (optional, finalize only this grouping)
 * }
 */
function finalizeTempGroups(payload) {
  const groupingId = payload.groupingId || null;

  // When finding TEMP sheets:
  if (groupingId && groupingId !== 'default') {
    // Find only sheets matching: grBe${groupingId}*TEMP
    tempSheets = listGroupingTempSheets(typePrefix, groupingId);
  } else {
    // Backward compat: find all TEMP sheets with prefix
    tempSheets = ... existing logic ...
  }

  // ... rest of function ...

  // When renaming TEMP sheets:
  if (groupingId && groupingId !== 'default') {
    // Rename: grBeA1TEMP → grBeA1
    const finalName = sheetName.replace('TEMP', '');
  } else {
    // Backward compat: grBe1TEMP → grBe1
    const finalName = sheetName.replace('TEMP', '');
  }

  // Update grouping metadata status if applicable
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

## Part F: Backward Compatibility Notes

1. **Implicit Single Grouping**: If `groupingId` not provided, functions behave as before
2. **Sheet Naming**:
   - Old: `grBe1TEMP`, `grBe1` (no grouping marker)
   - New with grouping: `grBeA1TEMP`, `grBeA1` (includes grouping ID)
3. **PropertiesService**: New grouping metadata stored separately from continuation metadata
4. **UI Migration**: Frontend can progressively enable groupings without breaking old workflows

---

## Implementation Checklist

- [ ] Add state extensions to groupsModuleComplete.html (groupings, activeGroupingId, etc.)
- [ ] Add grouping utility functions to groupsModuleComplete.html
- [ ] Add PropertiesService helper functions to Code.js
- [ ] Update saveTempGroups() to accept and handle groupingId
- [ ] Update finalizeTempGroups() to accept and handle groupingId
- [ ] Update getNextOffsetForGrouping() helper in Code.js
- [ ] Test backward compatibility (non-grouping calls still work)
- [ ] Test grouping-aware calls (groupingId parameter respected)
- [ ] Create test scenario: 2 groupings on same level

---

**Next Step**: Begin implementation of Part A (state extensions) and Part B (frontend utilities)

