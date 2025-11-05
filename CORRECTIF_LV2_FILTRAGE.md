# Correctif : Filtrage LV2 pour Groupes de Langues

## 📋 Problème identifié

La fonction `loadStudentsFromClasses()` agrégeait **tous les élèves** des classes sélectionnées sans tenir compte du type de regroupement ni filtrer par langue LV2. Résultat : un regroupement LV2 mélangeait toutes les langues disponibles (ESP, ITA, ALL, etc.) au lieu de ne traiter qu'une seule langue à la fois.

La génération s'appuyait ensuite directement sur `state.students` sans appliquer de filtrage ou d'heuristique adaptée au type de groupe.

## ✅ Corrections apportées

### 1. **Filtrage par langue dans `loadStudentsFromClasses()`** (lignes 2511-2536)

Ajout d'un bloc de filtrage conditionnel après l'agrégation des élèves :

```javascript
// 🔴 FILTRAGE LV2 : Si on est en mode 'language', ne garder que les élèves de la langue sélectionnée
let filteredStudents = students;
if (state.groupType === 'language' && state.selectedLanguage) {
  const beforeFilter = students.length;
  filteredStudents = students.filter(s => {
    const studentLV2 = (s.lv2 || '').toString().trim().toUpperCase();
    const targetLang = state.selectedLanguage.toUpperCase();
    return studentLV2 === targetLang;
  });
  console.log(`[GroupsModule] 🔍 Filtrage LV2 "${state.selectedLanguage}": ${beforeFilter} élèves → ${filteredStudents.length} élèves`);
  
  if (filteredStudents.length === 0) {
    console.warn(`[GroupsModule] ⚠️ Aucun élève trouvé pour la langue "${state.selectedLanguage}"`);
    showToast(`⚠️ Aucun élève trouvé pour la langue ${state.selectedLanguage}`, 'warning');
  }
}

state.students = filteredStudents;
```

**Résultat** : Seuls les élèves de la langue sélectionnée (ex: ESP) sont chargés dans `state.students`.

---

### 2. **Tri adapté au type de groupe dans `generateGroupsLocally()`** (lignes 2588-2604)

Modification du tri pour prioriser la **PARTICIPATION** pour les groupes LV2 :

```javascript
// 1. Tri adapté au type de groupe
let sorted;
if (state.groupType === 'language') {
  // Pour les groupes LV2 : tri par PARTICIPATION (PART) décroissant
  sorted = [...state.students].sort((a, b) => {
    const partA = a.part ?? a.scores?.PART ?? 0;
    const partB = b.part ?? b.scores?.PART ?? 0;
    return partB - partA; // Décroissant : meilleurs participateurs en premier
  });
  console.log('[GroupsModule] 📊 Mode LV2 - Tri par PARTICIPATION');
  console.log('[GroupsModule] Top 3 élèves:', sorted.slice(0, 3).map(s => `${s.nom} (PART: ${s.part ?? s.scores?.PART ?? 0})`));
} else {
  // Pour les groupes de besoins : tri par niveau (note haute -> élève "fort")
  sorted = sortByNeedScoreDesc(state.students);
  console.log('[GroupsModule] 📊 Distribution par mode:', state.distributionType);
  console.log('[GroupsModule] Top 3 élèves:', sorted.slice(0, 3).map(s => `${s.nom} (${getNeedScoreForStudent(s).toFixed(1)})`));
}
```

**Résultat** : Les groupes LV2 sont maintenant générés en priorisant les élèves les plus participatifs.

---

### 3. **Détection automatique des langues disponibles** (lignes 2549-2576)

Nouvelle fonction `detectAvailableLanguages()` :

```javascript
function detectAvailableLanguages() {
  if (!state.classesData || Object.keys(state.classesData).length === 0) {
    state.availableLanguages = [];
    return;
  }

  const languageCounts = {};
  
  // Parcourir toutes les classes sélectionnées
  (state.selectedClasses || []).forEach(displayName => {
    const bucket = state.classesData[displayName];
    if (!bucket || !Array.isArray(bucket.eleves)) return;

    bucket.eleves.forEach(e => {
      if (!e) return;
      const lv2 = (e.lv2 || e.LV2 || '').toString().trim().toUpperCase();
      if (lv2) {
        languageCounts[lv2] = (languageCounts[lv2] || 0) + 1;
      }
    });
  });

  state.availableLanguages = Object.entries(languageCounts)
    .map(([lang, count]) => ({ lang, count }))
    .sort((a, b) => b.count - a.count); // Tri par nombre d'élèves décroissant

  console.log('[GroupsModule] 🗣️ Langues détectées:', state.availableLanguages);
}
```

Appelée automatiquement lors du passage à l'étape 3 en mode LV2 (ligne 2085-2087) :

```javascript
// Détecter les langues disponibles si on arrive à l'étape 3 en mode LV2
if (state.currentStep === 3 && state.groupType === 'language') {
  detectAvailableLanguages();
}
```

**Résultat** : L'interface affiche dynamiquement les langues présentes avec le nombre d'élèves.

---

### 4. **Interface améliorée pour l'étape 3** (lignes 1259-1300)

- Affichage du **nombre d'élèves par langue** dans des badges
- **Désactivation** des langues sans élèves
- Récapitulatif des langues détectées
- Message d'avertissement si aucune langue n'est détectée

```javascript
${['ESP', 'ITA', 'ALL', 'AUTRE'].map(lang => {
  const langInfo = state.availableLanguages.find(l => l.lang === lang);
  const count = langInfo ? langInfo.count : 0;
  const hasStudents = count > 0;
  return `
  <label class="radio-card ${state.selectedLanguage === lang ? 'selected' : ''} ${!hasStudents ? 'opacity-50' : ''}">
    <input
      type="radio"
      name="language"
      value="${lang}"
      ${state.selectedLanguage === lang ? 'checked' : ''}
      ${!hasStudents ? 'disabled' : ''}
    >
    <div class="flex items-center justify-between gap-2 w-full">
      <div class="flex items-center gap-2">
        <span class="font-medium">${lang === 'ESP' ? 'Espagnol' : lang === 'ITA' ? 'Italien' : lang === 'ALL' ? 'Allemand' : 'Autre'}</span>
        ${lang === 'ESP' ? '<span class="text-xl">🇪🇸</span>' : lang === 'ITA' ? '<span class="text-xl">🇮🇹</span>' : lang === 'ALL' ? '<span class="text-xl">🇩🇪</span>' : '<span class="text-xl">🌍</span>'}
      </div>
      ${hasStudents ? `<span class="text-xs font-semibold px-2 py-0.5 rounded-full bg-purple-100 text-purple-700">${count}</span>` : '<span class="text-xs text-slate-400">0</span>'}
    </div>
    <i class="fas fa-check-circle check-icon"></i>
  </label>
`;
}).join('')}
```

---

### 5. **Validation avant génération** (lignes 2073-2080)

Ajout d'une validation pour empêcher de continuer si la langue sélectionnée n'a pas d'élèves :

```javascript
} else if (state.currentStep === 3) {
  // Validation spécifique pour les groupes LV2
  if (state.groupType === 'language') {
    const selectedLangInfo = state.availableLanguages.find(l => l.lang === state.selectedLanguage);
    if (!selectedLangInfo || selectedLangInfo.count === 0) {
      showToast(`⚠️ Aucun élève trouvé pour la langue ${state.selectedLanguage}`, 'warning');
      return;
    }
  }
  nextStep();
```

---

## 🎯 Résultat final

### Avant la correction :
- ❌ Tous les élèves LV2 (ESP + ITA + ALL) mélangés dans les groupes
- ❌ Tri basé uniquement sur Math/Français
- ❌ Pas de visibilité sur les langues disponibles
- ❌ Génération non fonctionnelle pour LV2

### Après la correction :
- ✅ **Filtrage strict** : seuls les élèves de la langue sélectionnée (ex: ESP)
- ✅ **Tri adapté** : priorité à la PARTICIPATION pour les groupes LV2
- ✅ **Détection automatique** des langues avec comptage
- ✅ **Interface intelligente** : badges, désactivation, validation
- ✅ **Génération fonctionnelle** pour LV2/Options

---

## 📝 Fichier modifié

- **`groupsModuleComplete.html`** : 5 modifications majeures

---

## 🧪 Tests recommandés

1. **Test ESP uniquement** : Sélectionner des classes avec ESP → Vérifier que seuls les élèves ESP sont dans les groupes
2. **Test ITA uniquement** : Sélectionner des classes avec ITA → Vérifier que seuls les élèves ITA sont dans les groupes
3. **Test mixte** : Sélectionner des classes avec ESP + ITA → Choisir ESP → Vérifier filtrage correct
4. **Test tri PART** : Vérifier que les élèves avec PART élevé sont bien répartis en priorité
5. **Test validation** : Essayer de générer avec une langue sans élèves → Doit bloquer avec message d'erreur

---

## 📅 Date de correction

**29 octobre 2025**

---

## 👤 Auteur

Cascade AI - Correctif automatique suite à analyse du code
