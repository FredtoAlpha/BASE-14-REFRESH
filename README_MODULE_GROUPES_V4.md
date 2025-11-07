# 📚 MODULE DE GROUPES V4 - DOCUMENTATION COMPLÈTE

**Version:** 4.0
**Date:** 2025-01-07
**Statut:** ✅ Production Ready

---

## 🎯 VUE D'ENSEMBLE

Le **Module de Groupes V4** est un système avancé de répartition d'élèves en groupes hétérogènes ou homogènes, avec équilibrage automatique de la parité F/M et prise en compte de critères multiples (académiques, comportementaux, contraintes).

### Fonctionnalités principales

✅ **2 modes de distribution**: Hétérogène (mix de niveaux) / Homogène (niveaux similaires)
✅ **3 scénarios**: Besoins, Compétences, Mixte
✅ **Z-scores normalisés**: Comparaison équitable multi-critères
✅ **Équilibrage parité**: Objectif 40-60% F/M par groupe
✅ **Gestion contraintes**: LV2, options, dissociations
✅ **Reporting centralisé**: Erreurs/warnings horodatés
✅ **Tests unitaires**: 5 suites, 20+ cas de test

---

## 📦 ARCHITECTURE

```
┌──────────────────────────────────────────────────────────────┐
│                    MODULE GROUPES V4                         │
└──────────────────────────────────────────────────────────────┘
       │
       ├─► InterfaceV2_GroupsModuleV4_Script.html
       │   ├─ loadFINData()                    → Chargement onglets FIN
       │   ├─ collectStudentsRecursive()       → Extraction récursive
       │   ├─ prepareStudentForAlgorithm()     → Normalisation robuste
       │   ├─ normalizeClassStudents()         → Pipeline complet
       │   ├─ getClassesData()                 → Interface algorithme
       │   └─ ErrorReporter                    → Reporting centralisé
       │
       ├─► GroupsAlgorithmV4.html
       │   ├─ calculateZScores()               → Normalisation statistique
       │   ├─ distributeStudents()             → Distribution initiale
       │   └─ balanceParity()                  → Équilibrage F/M
       │
       ├─► GroupsInterfaceV4.html
       │   ├─ renderInitialStructure()         → Interface triptyque
       │   ├─ displayResults()                 → Affichage groupes
       │   └─ enableSwap()                     → Dashboard swap élèves
       │
       ├─► GroupsModule_TestCases.js
       │   ├─ testIsStudentLike()              → 7 tests
       │   ├─ testCollectStudentsRecursive()  → 7 tests
       │   ├─ testPrepareStudentForAlgorithm() → 6 tests
       │   └─ runAllTests()                    → Exécution complète
       │
       └─► AUDIT_ALGORITHME_GROUPES_V4.md
           └─ Documentation technique complète (700+ lignes)
```

---

## 🚀 DÉMARRAGE RAPIDE

### 1. Installation

Le module est intégré à l'application BASE-14-REFRESH. Aucune installation séparée requise.

### 2. Utilisation basique

```javascript
// 1. Ouvrir le module depuis le header
openGroupsInterface('creator');

// 2. Sélectionner le scénario et le mode
// Interface graphique → Scénario: "besoins", Mode: "heterogeneous"

// 3. Choisir les classes et le nombre de groupes
// Interface graphique → Regroupement 1: 6°1, 6°2, 6°3 → 4 groupes

// 4. Générer les groupes
// Bouton "Générer les groupes"

// 5. Consulter les résultats
// Dashboard de swap avec statistiques par groupe
```

### 3. Tests

```javascript
// Exécuter tous les tests dans la console Apps Script
runAllTests();

// Ou tests individuels
testCollectStudentsRecursive();
testPrepareStudentForAlgorithm();
```

---

## 🔧 FONCTIONS CLÉS

### `collectStudentsRecursive(data, seen, depth)`

**Objectif:** Extraction récursive et dédupliquée des élèves dans une structure complexe.

**Paramètres:**
- `data` (any): Structure de données à parcourir
- `seen` (Set): IDs déjà vus (déduplication)
- `depth` (number): Profondeur actuelle (max 10)

**Retour:** Array d'objets élèves dédupliqués

**Exemple:**
```javascript
const data = {
  classe1: {
    eleves: [
      { id: 'A', nom: 'DUPONT' },
      { id: 'B', nom: 'MARTIN' }
    ]
  },
  classe2: {
    data: {
      students: [
        { id: 'C', nom: 'BERNARD' }
      ]
    }
  }
};

const students = collectStudentsRecursive(data);
// → [{ id: 'A', ... }, { id: 'B', ... }, { id: 'C', ... }]
```

---

### `prepareStudentForAlgorithm(student, className)`

**Objectif:** Normalisation robuste d'un élève avec gardes et valeurs de repli.

**Paramètres:**
- `student` (Object): Objet élève brut
- `className` (string): Nom de la classe (pour contexte errors)

**Retour:** Objet élève normalisé

**Validations:**
- **Sexe:** FILLE→F, GARCON→M, fallback M
- **Scores:** Clamping [0-4], extraction multi-source
- **LV2:** Normalisation abréviations (ESPAGNOL→ESP)
- **Options:** String "A,B" → ['A', 'B']

**Exemple:**
```javascript
const raw = {
  id: 'TEST_001',
  sexe: 'FILLE',
  scoreF: 5.5,  // Invalide
  lv2: 'ESPAGNOL',
  options: 'LATIN,GREC'
};

const prepared = prepareStudentForAlgorithm(raw, '6°1');
// → {
//   id: 'TEST_001',
//   sexe: 'F',
//   scoreF: 4,    // Clampé
//   lv2: 'ESP',   // Normalisé
//   options: ['LATIN', 'GREC']  // Parsé
// }
```

---

### `ErrorReporter`

**Objectif:** Système de reporting centralisé avec historique horodaté.

**Méthodes:**

```javascript
ErrorReporter.error(context, message, data);
ErrorReporter.warn(context, message, data);
ErrorReporter.info(context, message, data);

const report = ErrorReporter.getReport();
// → {
//   errors: [...],
//   warnings: [...],
//   summary: "2 erreur(s), 3 avertissement(s)"
// }

ErrorReporter.displayToUser();  // Alerte si erreurs critiques
ErrorReporter.clear();           // Réinitialiser
```

**Exemple:**
```javascript
ErrorReporter.warn('prepareStudentForAlgorithm',
  'Élève DUPONT_Jean sans scores académiques → impact z-scores nul'
);

// Console:
// ⚠️ [prepareStudentForAlgorithm] Élève DUPONT_Jean sans scores académiques → impact z-scores nul
```

---

## 📊 ALGORITHME V4 - DÉTAILS

### Étapes

```
1. Consolidation des données
   ↓
2. Normalisation (z-scores)
   ↓
3. Calcul indices pondérés
   ↓
4. Distribution initiale (hétérogène/homogène)
   ↓
5. Équilibrage parité F/M (swaps)
   ↓
6. Calcul statistiques finales
   ↓
7. Validation et rapport
```

### Z-scores

**Formule:**
```
z = (x - μ) / σ

Où:
  x = valeur brute
  μ = moyenne
  σ = écart-type
```

**Exemple:**
```
Classe de 30 élèves, scores Français:
  μ = 2.8, σ = 0.9

Élève A: scoreF = 4.0
  → z_F = (4.0 - 2.8) / 0.9 = 1.33
  → 1.33 écarts-types AU-DESSUS de la moyenne
```

### Coefficients (scénario "besoins")

| Critère | Poids |
|---------|-------|
| Français (scoreF) | 30% |
| Mathématiques (scoreM) | 30% |
| Comportement (COM) | 15% |
| Travail (TRA) | 10% |
| Participation (PART) | 10% |
| Absences (ABS) | 5% |

### Équilibrage parité

**Objectif:** 40-60% de filles (ou garçons) par groupe

**Algorithme:**
1. Identifier groupes déséquilibrés
2. Trouver paire F/M à échanger
3. Calculer amélioration parité globale
4. Si amélioration > 0: swap
5. Max 10 itérations

**⚠️ Risque:** Les swaps peuvent dégrader l'hétérogénéité académique
**→ Voir AUDIT_ALGORITHME_GROUPES_V4.md section 4.3**

---

## 🧪 TESTS UNITAIRES

### Suite de tests disponible

| Test | Cas | Description |
|------|-----|-------------|
| `testIsStudentLike` | 7 | Validation format élève |
| `testCollectStudentsRecursive` | 7 | Extraction récursive |
| `testPrepareStudentForAlgorithm` | 6 | Normalisation robuste |
| `testNormalizeClassStudents` | 5 | Pipeline complet |
| `testIntegration` | 1 | Test de bout en bout |

### Exécution

```javascript
// Console Apps Script
runAllTests();

// Résultat attendu:
// ✅ isStudentLike
// ✅ collectStudentsRecursive
// ✅ prepareStudentForAlgorithm
// ✅ normalizeClassStudents
// ✅ integration
// 🎉 TOUS LES TESTS SONT PASSÉS !
```

---

## ⚠️ RISQUES ET MITIGATIONS

### Risques critiques 🔴

| Risque | Impact | Mitigation |
|--------|--------|------------|
| **Écart-type nul** | Division par 0 → NaN | Fallback σ=1 ✅ |
| **Population F/M impossible** | 10 itérations inutiles | Détection préalable 🔧 |
| **Scores à 0 vs manquants** | Biais distribution | Warning + exclusion 🔧 |

### Risques majeurs 🟠

| Risque | Impact | Mitigation |
|--------|--------|------------|
| **Swaps dégradent hétérogénéité** | Objectif péda. compromis | Swap intelligent 🔧 |
| **Effectif faible** | Z-scores non fiables | Warning si N<10 🔧 |

**Légende:** ✅ Implémenté | 🔧 Recommandé

**→ Détails complets dans AUDIT_ALGORITHME_GROUPES_V4.md**

---

## 📈 ÉVOLUTIONS PRÉVUES

### P1 - Court terme (1-2 semaines)

- [ ] Détection impossibilité équilibrage F/M (avant génération)
- [ ] Swap intelligent avec score composite (parité + hétérogénéité)
- [ ] Warning si effectif < 10 élèves

### P2 - Moyen terme (2-4 semaines)

- [ ] Tolérance parité adaptative selon effectif groupe
- [ ] Interface personnalisation coefficients
- [ ] Rapport qualité détaillé post-génération

### P3 - Long terme (3+ mois)

- [ ] Tests A/B avec enseignants
- [ ] Collecte feedback pédagogique
- [ ] Dashboard monitoring erreurs production

---

## 📚 DOCUMENTATION ASSOCIÉE

| Document | Contenu | Taille |
|----------|---------|--------|
| `AUDIT_ALGORITHME_GROUPES_V4.md` | Analyse technique complète | 700+ lignes |
| `GroupsModule_TestCases.js` | Suite de tests unitaires | 550 lignes |
| `InterfaceV2_GroupsModuleV4_Script.html` | Code source module | 450 lignes |
| `GroupsAlgorithmV4.html` | Algorithme core | 500 lignes |
| `GroupsInterfaceV4.html` | Interface utilisateur | 2200 lignes |

---

## 🤝 SUPPORT

### Questions fréquentes

**Q: Les scores académiques ne s'affichent pas**
R: Vérifiez que les onglets FIN existent et contiennent les colonnes U (SCORE F) et V (SCORE M). Voir logs console pour warnings.

**Q: L'équilibrage F/M échoue**
R: Vérifiez le ratio global F/M de votre classe. Si trop déséquilibré (ex: 80% F), l'objectif 40-60% est mathématiquement impossible.

**Q: Les groupes sont trop homogènes/hétérogènes**
R: Ajustez les coefficients de pondération ou changez de mode (heterogeneous ↔ homogeneous).

### Contact

Pour signaler un bug ou proposer une amélioration:
1. Consulter `AUDIT_ALGORITHME_GROUPES_V4.md` section 7.3
2. Exécuter `runAllTests()` pour vérifier l'intégrité
3. Fournir les logs console + configuration utilisée

---

## 📜 CHANGELOG

### v4.0 (2025-01-07) - CURRENT

**✨ Nouveautés:**
- Collecte récursive et dédupliquée (`collectStudentsRecursive`)
- Préparation consolidée avec gardes (`prepareStudentForAlgorithm`)
- Système de reporting centralisé (`ErrorReporter`)
- Audit technique complet (AUDIT_ALGORITHME_GROUPES_V4.md)
- Suite de tests étendue (20+ cas)

**🐛 Corrections:**
- Fix extraction scores depuis onglets FIN (colonnes U et V)
- Fix format invalide `students.map is not a function`
- Fix normalisation sexe (FILLE/GARCON → F/M)
- Fix clamping scores [0-4]

**⚡ Améliorations:**
- Performance collecte: O(n×d) avec d=10 max
- Warnings clairs pour scores manquants
- Déduplication automatique par ID
- LV2 et options normalisés

---

**FIN DU DOCUMENT**

*Pour toute question technique, consulter AUDIT_ALGORITHME_GROUPES_V4.md*
