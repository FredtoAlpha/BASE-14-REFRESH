# ANALYSE BUGS UX/ARCHITECTURE - Blocages Profonds

**Date** : 30 octobre 2025
**Statut** : 7 bugs critiques identifiés
**Priorité** : BLOQUER avant production

---

## 🔴 BUG #1 : LANGUE PAR DÉFAUT BLOQUANTE (CRITIQUE)

### Problème
```javascript
// state.js - ligne 105
selectedLanguage: 'ESP',  // ← HARDCODÉ !
```

- **Établissement avec UNIQUEMENT l'italien** :
  - `availableLanguages = [{ lang: 'ITA', count: 45 }]`
  - `selectedLanguage = 'ESP'` (défaut)
  - Filtrage : `students.filter(s => s.lv2 === 'ESP')` → **0 élève**
  - L'enseignant pense que le chargement a échoué !

- **Aucune logique d'auto-correction** :
  - `detectAvailableLanguages()` remplit `availableLanguages`
  - Mais ne repositionne jamais `selectedLanguage`
  - L'écran Phase 3 affiche "ESP" avec 0 élèves, puis crée des groupes **vides**

### Code Actuel (Défaillant)
```javascript
// renderStep3_Configure() - ligne 1295+
${isLanguage ? `
  <!-- Affiche les radio pour ESP, ITA, ALL, AUTRE -->
  ${['ESP', 'ITA', 'ALL', 'AUTRE'].map(lang => {
    const langInfo = state.availableLanguages.find(l => l.lang === lang);
    const count = langInfo ? langInfo.count : 0;
    return `
      <label class="radio-card ${state.selectedLanguage === lang ? 'selected' : ''} ${!hasStudents ? 'opacity-50' : ''}">
        <input type="radio" name="language" value="${lang}"
          ${state.selectedLanguage === lang ? 'checked' : ''}
          ${!hasStudents ? 'disabled' : ''}>  <!-- ← DÉSACTIVÉ si 0 élève -->
        ...
      </label>
    `;
  }).join('')}
` : ''}
```

**Le problème** :
1. Radio ESP est **sélectionnée** par défaut
2. Radio ESP est **désactivée** car 0 élève
3. L'utilisateur **NE PEUT PAS CLIQUER** sur ESP (grisé)
4. Mais c'est quand même la valeur active ! 😱
5. Génération des groupes utilise `state.selectedLanguage = 'ESP'` → 0 élève

### Solution

#### Option A : Auto-repositionner vers la première langue disponible
```javascript
function detectAvailableLanguages() {
  // ... code de détection ...
  state.availableLanguages = detected; // Tableau des langues trouvées

  // 🟢 AJOUTER : Si la langue sélectionnée n'est plus disponible, reposition
  const currentLangAvailable = state.availableLanguages.some(l => l.lang === state.selectedLanguage);
  if (!currentLangAvailable && state.availableLanguages.length > 0) {
    // Prendre la première langue disponible
    state.selectedLanguage = state.availableLanguages[0].lang;
  }
}
```

#### Option B : Pré-sélectionner la première langue au rendu
```javascript
const availableLangs = ['ESP', 'ITA', 'ALL', 'AUTRE'].filter(lang => {
  const langInfo = state.availableLanguages.find(l => l.lang === lang);
  return langInfo && langInfo.count > 0;
});

const defaultLang = availableLangs.length > 0
  ? availableLangs[0]
  : state.selectedLanguage;

// Utiliser `defaultLang` pour la sélection initiale du HTML
```

**Recommandation** : **Option A** + **Option B**
- Auto-correction en détection
- + HTML qui respecte la valeur correcte

---

## 🔴 BUG #2 : LANGUES NON RAFRAÎCHIES APRÈS CHANGEMENT

### Problème
```javascript
// renderStep3_Configure() - ligne 1189
function renderStep3_Configure() {
  const isLanguage = state.groupType === 'language';

  return `
    ${isLanguage ? `
      <!-- Affiche availableLanguages -->
      ${state.availableLanguages.map(l => ...)}
    ` : ''}
  `;
}
```

**Timeline problématique** :
```
Étape 2 : Utilisateur sélectionne "6°1" (45 élèves en ITA)
          → Aucun appel à detectAvailableLanguages()

Étape 3 : RenderStep3_Configure() s'affiche
          → APPEL à detectAvailableLanguages() (1ère fois)
          → state.availableLanguages = [{ lang: 'ITA', count: 45 }]

Utilisateur revient à Étape 2 (Previous)
          → Modifie sélection → "6°2" (30 élèves en ESP)

Retour à Étape 3 : RenderStep3_Configure()
          → state.availableLanguages ENCORE À [{ lang: 'ITA', count: 45 }]
          → Liste NON MISE À JOUR !
          → Affiche 45 élèves en ITA, mais il n'y en a que 0
```

### Code Actuel (Défaillant)
```javascript
// setupModalListeners() - ligne 2430+
function nextStep() {
  if (state.currentStep === 2) {
    // Pas d'appel à detectAvailableLanguages() ici !
    // L'appel n'arrive qu'à l'affichage de Phase 3
  }
}

// renderStep3_Configure() - ligne 1189
function renderStep3_Configure() {
  const isLanguage = state.groupType === 'language';

  // Appel IMPLICITE via renderModal() → updateUI() → renderStep3
  // Mais uniquement à l'affichage initial !
  // Pas de callback pour "quand les données changent"
}
```

**L'architecture** :
- `detectAvailableLanguages()` n'existe que dans **renderStep3_Configure()**
- Aucun événement ou hook pour recharger quand `state.selectedClasses` change
- Pas de invalidation du cache

### Solution

#### Créer une fonction dédiée + l'appeler aux bons moments
```javascript
function syncLanguageState() {
  if (state.groupType !== 'language') return;

  // Toujours régénérer la liste des langues dispo
  detectAvailableLanguages();

  // Vérifier que selectedLanguage est encore valide
  const langAvailable = state.availableLanguages.some(l => l.lang === state.selectedLanguage);
  if (!langAvailable && state.availableLanguages.length > 0) {
    state.selectedLanguage = state.availableLanguages[0].lang;
  }
}

// Appeler à CHAQUE fois que selectedClasses change
function activateRegroupement(regroupementId) {
  // ... code ...
  syncLanguageState();  // ← AJOUTER
  updateUI();
}

function nextStep() {
  // ...
  if (state.currentStep === 2 && state.groupType === 'language') {
    syncLanguageState();  // ← AJOUTER avant d'afficher phase 3
  }
}
```

---

## 🔴 BUG #3 : "AUTRE" NE CAPTURE PAS LES CODES RÉELS (CRITIQUE)

### Problème
```javascript
// generateGroupsLocally() - ligne 3100+
if (state.groupType === 'language' && state.selectedLanguage) {
  filteredStudents = state.students.filter(s => {
    if (state.selectedLanguage === 'AUTRE') {
      return !['ESP', 'ITA', 'ALL'].includes(s.lv2);  // ← COMPARAISON EXACTE
    }
    return s.lv2 === state.selectedLanguage;  // ← COMPARAISON EXACTE
  });
}
```

**Problème réel** :
- Données backend : `s.lv2 = 'POR'` (Portugais)
- Utilisateur choisit "AUTRE"
- Filtre cherche : `s.lv2 === 'AUTRE'`
- **Résultat** : 0 élève !

Le code suppose que les données contiennent littéralement `'AUTRE'`, ce qui n'arrive JAMAIS.

### Code Actuel (Défaillant)
```javascript
// renderStep3_Configure() - ligne 1307+
${['ESP', 'ITA', 'ALL', 'AUTRE'].map(lang => {
  // ...
  <span class="text-xs font-semibold px-2 py-0.5 rounded-full bg-purple-100 text-purple-700">
    ${count}  <!-- ← Affiche 0 pour AUTRE -->
  </span>
})
```

Et dans le comptage :
```javascript
function detectAvailableLanguages() {
  const langMap = {};
  state.students.forEach(student => {
    const lang = student.lv2 || 'AUTRE';  // ← Si pas de lv2, c'est AUTRE
    langMap[lang] = (langMap[lang] || 0) + 1;
  });

  // Retourne : { 'ESP': 15, 'ITA': 20, 'POR': 10 }
  // Jamais de clé 'AUTRE' dans les réelles données !
}
```

### Solution

#### Normaliser et capturer les codes réels
```javascript
function getNormalizedLanguage(langCode) {
  if (!langCode) return 'AUTRE';

  const normalized = (langCode || '').toUpperCase().trim();
  const knownLangs = {
    'ESP': 'ESP',
    'ESPAGNOL': 'ESP',
    'ITA': 'ITA',
    'ITALIEN': 'ITA',
    'ALL': 'ALL',
    'ALLEMAND': 'ALL'
  };

  return knownLangs[normalized] || 'AUTRE';
}

function detectAvailableLanguages() {
  const langMap = {};
  state.students.forEach(student => {
    const lang = getNormalizedLanguage(student.lv2);
    langMap[lang] = (langMap[lang] || 0) + 1;
  });

  // Exemple résultat : { 'ESP': 15, 'ITA': 20, 'AUTRE': 10 }
  // Maintenant on capture POR, ALL autres langues en AUTRE !

  state.availableLanguages = Object.entries(langMap)
    .map(([lang, count]) => ({ lang, count }));
}

function filterStudentsByLanguage(students, selectedLang) {
  return students.filter(s => {
    const studentLang = getNormalizedLanguage(s.lv2);
    return studentLang === selectedLang;
  });
}
```

#### Mise à jour du rendu
```javascript
// renderStep3_Configure()
${['ESP', 'ITA', 'ALL', 'AUTRE'].map(lang => {
  const langInfo = state.availableLanguages.find(l => l.lang === lang);
  const count = langInfo ? langInfo.count : 0;

  // Afficher le label lisible + code
  const langLabel = {
    'ESP': 'Espagnol',
    'ITA': 'Italien',
    'ALL': 'Allemand',
    'AUTRE': 'Autre'
  }[lang];

  return `
    <label class="radio-card ${count === 0 ? 'opacity-50 cursor-not-allowed' : ''}">
      <input type="radio" ... ${count === 0 ? 'disabled' : ''}>
      <div>${langLabel}</div>
      <span class="text-xs font-semibold px-2 py-0.5 rounded-full
        ${count === 0 ? 'bg-slate-100 text-slate-500' : 'bg-purple-100 text-purple-700'}">
        ${count} élève${count !== 1 ? 's' : ''}
      </span>
    </label>
  `;
})
```

---

## 🟡 BUG #4 : TOASTS NON MAÎTRISÉS (URGENT UX)

### Problème
```javascript
function showToast(message, type = 'info', duration = 3000) {
  const toast = document.createElement('div');
  toast.className = `toast toast-${type} fixed top-4 right-4 ...`;
  toast.textContent = message;
  document.body.appendChild(toast);

  // ← PAS DE FILE D'ATTENTE
  // ← PLUSIEURS TOASTS PEUVENT SE CHEVAUCHER

  setTimeout(() => toast.remove(), duration);
}
```

**Scénario problématique** :
```
T0:     Changement de regroupement
        → Toast: "Chargement des élèves..." (4s)

T0+100: Changement de classe
        → Toast: "Mise à jour des langues..." (4s)
        → Première toast TOUJOURS AFFICHÉE

T0+200: Détection langue
        → Toast: "5 élèves en ITA" (4s)
        → TROIS TOASTS SE CHEVAUCHENT ! 😱
```

**Résultat** : L'écran se remplit de messages, certains se superposent, l'utilisateur ne peut pas lire.

### Code Actuel (Défaillant)
```javascript
// Utilisé dans ~30 endroits
showToast('Changement regroupement');
showToast('Langues détectées : ESP, ITA');
showToast('Groupes générés');
showToast('Aucun élève dispo');
// Etc...

// Aucun système d'empilage
// Aucun bouton fermer
```

### Solution

#### Créer un système de queue de toasts
```javascript
const toastQueue = {
  items: [],
  maxVisible: 3,

  add(message, type = 'info', duration = 3000) {
    const id = Math.random().toString(36).substr(2, 9);
    const toast = {
      id,
      message,
      type,
      duration,
      createdAt: Date.now()
    };

    this.items.push(toast);
    this.render();

    // Auto-remove après duration
    setTimeout(() => this.remove(id), duration);
  },

  remove(id) {
    this.items = this.items.filter(t => t.id !== id);
    this.render();
  },

  render() {
    const container = document.getElementById('toast-container')
      || this.createContainer();

    const visibleToasts = this.items.slice(-this.maxVisible);

    container.innerHTML = visibleToasts
      .map((toast, idx) => `
        <div class="toast toast-${toast.type} animate-slide-in"
          style="bottom: ${idx * 80 + 20}px;">
          <span>${toast.message}</span>
          <button class="toast-close" data-toast-id="${toast.id}">
            <i class="fas fa-times"></i>
          </button>
        </div>
      `)
      .join('');

    // Event listeners pour fermer
    container.querySelectorAll('.toast-close').forEach(btn => {
      btn.onclick = () => this.remove(btn.dataset.toastId);
    });
  },

  createContainer() {
    const div = document.createElement('div');
    div.id = 'toast-container';
    div.className = 'fixed bottom-4 right-4 flex flex-col gap-2 z-50 pointer-events-none';
    document.body.appendChild(div);
    return div;
  }
};

// Remplacer showToast() partout
function showToast(message, type = 'info', duration = 3000) {
  toastQueue.add(message, type, duration);
}
```

#### CSS pour l'animation
```css
.toast-container .toast {
  pointer-events: auto;
  animation: slide-in-right 0.3s ease-out;
  transition: all 0.3s ease;
}

.toast-container .toast:hover {
  transform: translateX(-5px);
}

.toast-close {
  margin-left: auto;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.875rem;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.toast-close:hover {
  opacity: 1;
}

@keyframes slide-in-right {
  from {
    opacity: 0;
    transform: translateX(400px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

---

## 🟡 BUG #5 : AUTO-TRANSITION PHASE 1→2 DÉROUTANTE

### Problème
```javascript
// setupModalListeners() - ligne 2500+
// Après sélection du type
qs('[data-action="select-group-type"]', state.modal).onclick = function() {
  state.groupType = this.dataset.type;  // Ex: 'needs'
  showToast(`Type sélectionné: ${state.groupType}`);

  // 🔴 AUTO-PASSAGE À ÉTAPE 2 APRÈS 300ms
  setTimeout(() => {
    state.currentStep = 2;
    updateUI();
  }, 300);
};
```

**Problème UX** :
1. L'utilisateur **clique** sur "Groupes de Besoins"
2. La description s'affiche (300ms)
3. **BOOM** : Écran change automatiquement
4. L'utilisateur n'a pas eu le temps de **lire** ou **revenir en arrière**
5. Sensation de "magie noire" : où suis-je ? Que s'est-il passé ?

### Solution

#### Retirer le setTimeout, laisser l'utilisateur valider
```javascript
// OPTION A : Bouton "Continuer" explicite
qs('[data-action="select-group-type"]', state.modal).onclick = function() {
  state.groupType = this.dataset.type;
  // ← PAS DE nextStep() automatique !
  updateUI();  // Juste afficher la description
};

// L'utilisateur clique sur "Continuer" en bas
qs('[data-action="next-step"]', state.modal).onclick = function() {
  if (state.currentStep === 1 && !state.groupType) {
    showToast('Veuillez d\'abord sélectionner un type de groupe', 'warning');
    return;
  }
  nextStep();
};
```

#### Ou améliorer le feedback visuel
```javascript
// OPTION B : Garder le setTimeout mais afficher plus clairement
qs('[data-action="select-group-type"]', state.modal).onclick = function() {
  state.groupType = this.dataset.type;
  updateUI();

  // Afficher message temporaire (2s) au lieu de (0.3s)
  showToast(`Type sélectionné: ${state.groupType}. Redirection en cours...`, 'info', 2000);

  // Laisser plus de temps (2s au lieu de 300ms)
  setTimeout(() => {
    state.currentStep = 2;
    updateUI();
  }, 2000);
};
```

**Recommandation** : **Option A**
- Donne le contrôle à l'utilisateur
- Permet de relire avant de continuer
- Cohérent avec le UX web standard

---

## 🟡 BUG #6 : INCOHÉRENCE ÉTAPE 4 vs 5 (ARCHITECTURE)

### Problème
```javascript
// state.js - ligne 83-84
currentStep: 1,
totalSteps: 5,  // ← DÉCLARE 5 étapes

// main switch - ligne 587-593
switch(state.currentStep) {
  case 1: return renderStep1_SelectType();
  case 2: return renderStep2_SelectClasses();
  case 3: return renderStep3_Configure();
  case 4: return renderStep4_Preview();
  case 5: return renderStep5_Groups();  // ← EXISTE !
  default: return '<!-- Étape inconnue -->';
}
```

**Mais l'UI affiche 4 étapes** :
```javascript
// renderStepper() - ligne 430+
const steps = [
  { num: 1, label: 'Type' },
  { num: 2, label: 'Classes' },
  { num: 3, label: 'Config' },
  { num: 4, label: 'Aperçu' }
];
// ← SEULEMENT 4 ! Où est l'étape 5 ?
```

**Conséquence** :
- Navigation : `nextStep()` et `previousStep()` bordées par `totalSteps = 5`
- Stepper : Affiche que 4 jalons
- Certains appels poussent `currentStep = 5`
- L'étape 5 s'affiche (bon contenu) mais **sans libellé dans le stepper** !
- Retour arrière depuis étape 5 : décrémente à 4 mais aucune indication de "c'est la fin"

### Code Actuel (Défaillant)

```javascript
// nextStep()
function nextStep() {
  if (state.currentStep < state.totalSteps) {
    state.currentStep++;  // ← Peut aller jusqu'à 5
    // ...
  }
}

// previousStep()
function previousStep() {
  if (state.currentStep > 1) {
    state.currentStep--;  // ← Peut revenir de 5 à 4
  }
}

// generateGroups() - lance régen temp ou finale
function generateGroups() {
  // ...
  state.currentStep = 5;  // ← FORCE L'ÉTAPE 5 (même si totalSteps = 5)
  updateUI();
}
```

### Solution

#### Option A : Réintroduire une vraie étape 5
```javascript
// state.js
currentStep: 1,
totalSteps: 5,  // ← Cohérent avec le code

// renderStepper()
const steps = [
  { num: 1, label: 'Type' },
  { num: 2, label: 'Classes' },
  { num: 3, label: 'Config' },
  { num: 4, label: 'Aperçu' },
  { num: 5, label: 'Résultat' }  // ← AJOUTER
];

// Afficher un petit jeton "5" dans le stepper qui indique "Résultat"
```

#### Option B : Fixer à 4 étapes, éliminer toute référence à l'étape 5
```javascript
// state.js
currentStep: 1,
totalSteps: 4,  // ← Réduit

// Renommer :
// - renderStep4_Preview() → renderStep4_Groups()
// - Fusionner Step 4 Preview + Step 5 Groups en une seule vue

// main switch
switch(state.currentStep) {
  case 1: return renderStep1_SelectType();
  case 2: return renderStep2_SelectClasses();
  case 3: return renderStep3_Configure();
  case 4: return renderStep4_Groups();  // Combiné Preview + Display
}

// Stepper reste à 4 jalons, cohérent
```

**Recommandation** : **Option A** (plus clair)
- Permet de montrer un "bilan" distinct d'un "aperçu"
- Stepper affiche explicitement les 5 étapes
- Architecture cohérente

---

## 🟡 BUG #7 : BARRE ACTIONS SURPEUPLÉE ET CONFUSE

### Problème

**Barre d'actions (étape 5)** affiche ~9 boutons :
```html
<!-- renderActionToolbar() - ligne 1700+-->
<div class="action-toolbar">
  <!-- Stats -->
  <button data-action="stats">📊 Stats</button>

  <!-- Drag Help -->
  <button data-action="drag-help">👆 Aide</button>

  <!-- Export -->
  <button data-action="export-pdf">📄 PDF</button>
  <button data-action="export-groups">📋 Groupes</button>
  <button data-action="export-csv">📊 CSV</button>

  <!-- Reset -->
  <button data-action="reset-module">🔄 Réinitialiser</button>

  <!-- Navigation -->
  <button data-action="goto-step-2">← Paramètres</button>
  <button data-action="go-home">🏠 Accueil</button>

  <!-- Focus Mode -->
  <button data-action="toggle-focus-mode">🎯 Focus</button>
</div>
```

**Problèmes d'ergonomie** :
1. **Profusion d'icônes** → Distraction de l'objectif principal (vérifier/ajuster les groupes)
2. **Actions redondantes** :
   - "Réinitialiser" + "Paramètres" → Deux façons différentes de revenir
   - "Réinitialiser" = DANGER (perte de tout le travail)
   - "Paramètres" = SAFE (retour ajustable)
3. **Export non implémentés** → Les boutons ne font rien
4. **Aide drag** affichée à chaque fois → Bruit visuel pour les utilisateurs expérimentés

### Solution

#### Hiérarchiser les actions
```html
<!-- PRIMARY ACTIONS (toujours visibles) -->
<button data-action="stats" class="btn-primary">
  <i class="fas fa-chart-bar"></i> Stats
</button>
<button data-action="goto-step-2" class="btn-primary">
  <i class="fas fa-arrow-left"></i> Ajuster
</button>

<!-- SECONDARY ACTIONS (dropdown menu ou modal) -->
<div class="action-menu">
  <button class="btn-menu-toggle">
    <i class="fas fa-ellipsis-v"></i> Outils
  </button>
  <div class="menu-dropdown">
    <a href="#" data-action="export-groups">Exporter Groupes</a>
    <a href="#" data-action="export-pdf">Exporter PDF</a>
    <a href="#" data-action="export-csv">Exporter CSV</a>
    <hr/>
    <a href="#" data-action="reset-module" class="danger">Réinitialiser Module</a>
    <a href="#" data-action="go-home">Retour Accueil</a>
  </div>
</div>

<!-- FOCUS MODE TOGGLE (contrôle rapide) -->
<button data-action="toggle-focus-mode" class="btn-focus" title="Mode Focus (Raccourci: F)">
  <i class="fas fa-expand"></i>
</button>
```

#### Masquer aide drag après première visite
```javascript
function renderDragHelper() {
  // Afficher SEULEMENT si c'est la première génération de ce regroupement
  const regroupement = getActiveRegroupement();
  if (!regroupement || regroupement.dragHelpShown) return '';

  return `
    <div class="alert alert-info flex items-start gap-3">
      <i class="fas fa-hand-pointer text-lg"></i>
      <div class="flex-1">
        <p class="font-semibold">💡 Astuce : Drag & Drop</p>
        <p class="text-sm">Vous pouvez glisser-déposer les élèves entre groupes</p>
      </div>
      <button class="btn-sm" data-action="hide-drag-help">OK</button>
    </div>
  `;
}

// Marquer comme "vu"
function hideDragHelper() {
  const regroupement = getActiveRegroupement();
  if (regroupement) {
    regroupement.dragHelpShown = true;
  }
  updateUI();
}
```

#### Ne montrer les exports que s'ils sont implémentés
```javascript
// renderActionToolbar()
${state.exportFeaturesAvailable ? `
  <div class="menu-dropdown">
    <a href="#" data-action="export-groups">Exporter Groupes</a>
  </div>
` : ''}
```

---

## 📊 RÉSUMÉ DES 7 BUGS

| Bug | Catégorie | Sévérité | Ligne | Correction |
|-----|-----------|----------|-------|-----------|
| #1 | Langue défaut hardcodée | 🔴 BLOQUER | 105 | Auto-correction + détection |
| #2 | Langues non rafraîchies | 🔴 BLOQUER | 1189 | `syncLanguageState()` |
| #3 | "AUTRE" ne capture pas codes réels | 🔴 BLOQUER | 3100 | Normalisation + mapping |
| #4 | Toasts se chevauchent | 🟡 UX | ~30 | Queue + max-visible |
| #5 | Auto-transition Phase 1→2 | 🟡 UX | 2500 | Retirer setTimeout |
| #6 | Incohérence étape 4 vs 5 | 🟡 ARCH | 83-593 | Réintroduire étape 5 |
| #7 | Barre actions surpeuplée | 🟡 UX | 1700 | Menu déroulant + hiérarchie |

---

## 🎯 PRIORITÉ DE CORRECTION

### **Immédiat (BLOQUER production)** 🔴
1. Bug #1 : Langue par défaut (5 min)
2. Bug #2 : Rafraîchissement langues (10 min)
3. Bug #3 : Normalisation "AUTRE" (15 min)

### **Urgent UX** 🟡
4. Bug #4 : Queue toasts (30 min)
5. Bug #5 : Auto-transition (5 min)

### **Architecture (refactorisation)** 🟡
6. Bug #6 : Étape 4 vs 5 (45 min - optionnel mais recommandé)
7. Bug #7 : Barre actions (30 min)

---

## ✨ GAINS ATTENDUS

- ✅ LV2 fonctionne même avec une seule langue
- ✅ Pas de toasts superposés et confus
- ✅ UX plus claire et moins déroutante
- ✅ Architecture cohérente 4 ou 5 étapes (pas mélangé)
- ✅ Actions hiérarchisées, moins de bruit visuel
