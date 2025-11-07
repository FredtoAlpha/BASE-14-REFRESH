# 🔬 AUDIT TECHNIQUE - ALGORITHME DE GROUPES V4

**Date:** 2025-01-07
**Version:** 4.0
**Auteur:** Architecture Module Groupes
**Objet:** Analyse des z-scores, coefficients, parité et recommandations pédagogiques

---

## 📋 TABLE DES MATIÈRES

1. [Vue d'ensemble de l'algorithme V4](#1-vue-densemble-de-lalgorithme-v4)
2. [Analyse des z-scores et normalisation](#2-analyse-des-z-scores-et-normalisation)
3. [Coefficients de pondération](#3-coefficients-de-pondération)
4. [Équilibrage de la parité F/M](#4-équilibrage-de-la-parité-fm)
5. [Risques identifiés](#5-risques-identifiés)
6. [Recommandations pédagogiques](#6-recommandations-pédagogiques)
7. [Conclusion](#7-conclusion)

---

## 1. VUE D'ENSEMBLE DE L'ALGORITHME V4

### 1.1 Objectifs

L'algorithme V4 vise à créer des groupes **hétérogènes** ou **homogènes** en équilibrant :
- Les scores académiques (Français, Mathématiques)
- Les critères comportementaux (COM, TRA, PART, ABS)
- La parité garçons/filles (ratio F/M)
- Les contraintes optionnelles (LV2, options, dissociations)

### 1.2 Flux de l'algorithme

```
┌─────────────────────────────────────┐
│  1. Consolidation des données       │
│     - Extraction scores F, M        │
│     - Validation champs critiques   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  2. Normalisation (z-scores)        │
│     - Calcul moyenne et écart-type  │
│     - Transformation en z-scores    │
│     - Distribution centrée N(0,1)   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  3. Calcul indices pondérés         │
│     - Application coefficients V4   │
│     - Somme pondérée → score final  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  4. Distribution initiale           │
│     - Hétérogène : alternance H/B   │
│     - Homogène : regroupement       │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  5. Équilibrage parité F/M          │
│     - Objectif : 40-60% par groupe  │
│     - Max 10 itérations de swaps    │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│  6. Calcul statistiques finales     │
└─────────────────────────────────────┘
```

---

## 2. ANALYSE DES Z-SCORES ET NORMALISATION

### 2.1 Principe des z-scores

Un **z-score** transforme une valeur brute en nombre d'écarts-types par rapport à la moyenne :

```
z = (x - μ) / σ

Où:
  x = valeur brute
  μ = moyenne de la population
  σ = écart-type de la population
```

### 2.2 Exemple concret

**Classe de 30 élèves, scores Français :**

```
Scores bruts: [1.5, 2.0, 2.5, 3.0, 3.5, 4.0, ...]
Moyenne (μ): 2.8
Écart-type (σ): 0.9

Élève A: scoreF = 4.0
  → z_F = (4.0 - 2.8) / 0.9 = 1.33
  → 1.33 écarts-types AU-DESSUS de la moyenne

Élève B: scoreF = 1.5
  → z_F = (1.5 - 2.8) / 0.9 = -1.44
  → 1.44 écarts-types EN-DESSOUS de la moyenne
```

### 2.3 Avantages des z-scores

✅ **Comparabilité** : Permet de comparer des scores sur des échelles différentes
✅ **Détection des outliers** : z > 2 ou z < -2 = valeur extrême (2.5% de la population)
✅ **Distribution standard** : Moyenne = 0, Écart-type = 1

### 2.4 ⚠️ RISQUES DES Z-SCORES

#### Risque 1 : Écart-type nul ou très faible

**Scénario :**
Tous les élèves ont le même score (ex: scoreF = 3.0 pour tous)

**Conséquence :**
```
σ = 0
z = (x - μ) / 0 = Division par zéro → NaN ou Infinity
```

**Mitigation :**
```javascript
// Code V4 actuel
const std = Math.sqrt(variance) || 1;  // Fallback σ = 1 si 0
```

#### Risque 2 : Effectif faible

**Scénario :**
Classe de 5 élèves seulement

**Conséquence :**
- Écart-type peu représentatif
- Un seul outlier biaise massivement la distribution
- Z-scores exagérés

**Exemple :**
```
Classe A (30 élèves):  μ = 2.8, σ = 0.9
Classe B (5 élèves):   μ = 2.8, σ = 0.3  (scores plus homogènes)

Élève avec scoreF = 4.0:
  - Classe A: z = 1.33  (bon niveau)
  - Classe B: z = 4.00  (niveau exceptionnel)
```

**Recommandation :**
⚠️ Activer un **avertissement** si effectif < 10 élèves

#### Risque 3 : Scores à 0

**Scénario :**
Élève sans scoreF et scoreM (valeurs manquantes → 0 par défaut)

**Conséquence :**
```
scoreF = 0, scoreM = 0
→ z_F et z_M très négatifs
→ Élève considéré comme "très faible"
→ Biais dans la distribution hétérogène
```

**Mitigation actuelle :**
```javascript
// prepareStudentForAlgorithm
if (scoreF === 0 && scoreM === 0) {
  ErrorReporter.warn('Élève sans scores académiques → impact z-scores V4 nul');
}
```

**Recommandation :**
💡 **Option 1 :** Exclure les élèves sans scores de la normalisation
💡 **Option 2 :** Utiliser la médiane de la classe comme valeur de repli

---

## 3. COEFFICIENTS DE PONDÉRATION

### 3.1 Scénario "besoins" (défaut)

```javascript
const weights = {
  scoreF: 0.3,   // 30% Français
  scoreM: 0.3,   // 30% Mathématiques
  com:    0.15,  // 15% Comportement
  tra:    0.10,  // 10% Travail
  part:   0.10,  // 10% Participation
  abs:    0.05   //  5% Absences
};

Total: 1.0 (100%)
```

### 3.2 Impact des coefficients

**Calcul de l'indice pondéré :**
```
index = (z_F × 0.3) + (z_M × 0.3) + (z_COM × 0.15) +
        (z_TRA × 0.1) + (z_PART × 0.1) + (z_ABS × 0.05)
```

**Exemple :**
```
Élève A:
  z_F = 1.5, z_M = 1.2, z_COM = 0.8, z_TRA = 0.5, z_PART = 0.3, z_ABS = -0.2

  index_A = (1.5×0.3) + (1.2×0.3) + (0.8×0.15) + (0.5×0.1) + (0.3×0.1) + (-0.2×0.05)
          = 0.45 + 0.36 + 0.12 + 0.05 + 0.03 - 0.01
          = 1.00
```

### 3.3 ⚠️ RISQUES DES COEFFICIENTS

#### Risque 1 : Dominance des scores académiques

**Avec poids actuels :**
```
Académique (F+M) = 60%
Comportemental   = 40%
```

**Conséquence :**
Un élève avec d'excellents scores académiques mais comportement difficile sera considéré comme "bon élève" globalement.

**Recommandation :**
💡 Permettre de **personnaliser les coefficients** selon les objectifs pédagogiques :
- Classe difficile → augmenter poids comportement (ex: COM 25%, TRA 20%)
- Classe homogène → augmenter poids académique

#### Risque 2 : Sous-pondération des absences

**ABS = 5% seulement**

**Scénario :**
Élève avec 15 absences (z_ABS = 2.5) vs élève avec 0 absence (z_ABS = -0.5)

```
Impact sur index:
  Élève absentéiste: 2.5 × 0.05 = 0.125
  Élève assidu:     -0.5 × 0.05 = -0.025

Différence: 0.15 seulement sur un index total de ~1-2
```

**Recommandation :**
⚠️ Si les absences sont critiques, **augmenter le poids à 10-15%**

---

## 4. ÉQUILIBRAGE DE LA PARITÉ F/M

### 4.1 Objectif de parité

```javascript
const PARITY_TARGET_MIN = 40;  // 40% minimum
const PARITY_TARGET_MAX = 60;  // 60% maximum
```

**Zone acceptable :** 40-60% de filles (ou garçons)

### 4.2 Algorithme de swap

```
POUR chaque itération (max 10):
  POUR chaque groupe déséquilibré:
    1. Identifier les groupes excédentaires et déficitaires
    2. Trouver une paire F/M à échanger
    3. Calculer l'amélioration de parité globale
    4. Si amélioration > 0: effectuer le swap
    5. Si tous groupes équilibrés: STOP
```

### 4.3 ⚠️ RISQUES DE LA TOLÉRANCE FIXE (40-60%)

#### Risque 1 : Effectif faible

**Scénario :**
Groupe de 5 élèves

```
Configuration:
  - 3 filles, 2 garçons = 60% F → OK
  - 2 filles, 3 garçons = 40% F → OK

Mais si 4F-1G ou 1F-4G:
  - 4F-1G = 80% F → DÉSÉQUILIBRÉ
  - 1F-4G = 20% F → DÉSÉQUILIBRÉ
```

**Problème :**
Avec des petits groupes, il est **mathématiquement impossible** de respecter 40-60% tout en ayant des effectifs entiers.

**Recommandation :**
💡 **Adapter la tolérance selon l'effectif :**

```javascript
function calculateParityTolerance(groupSize) {
  if (groupSize <= 5) return [20, 80];   // ±30% de tolérance
  if (groupSize <= 10) return [30, 70];  // ±20% de tolérance
  return [40, 60];                       // ±10% de tolérance (défaut)
}
```

#### Risque 2 : Population déséquilibrée globalement

**Scénario :**
Classe de 30 élèves : 20 filles, 10 garçons (67% F globalement)

**Création de 3 groupes de 10 :**

```
Objectif 40-60% par groupe:
  - Groupe 1: 5F-5M (50% F) → OK
  - Groupe 2: 5F-5M (50% F) → OK
  - Groupe 3: 10F-0M (100% F) → IMPOSSIBLE À ÉQUILIBRER

Raison: Total garçons = 10, mais besoin de 15 (5 par groupe)
```

**Conséquence :**
L'algorithme va itérer 10 fois sans succès.

**Recommandation :**
⚠️ **Détection préalable :** Calculer si l'équilibrage est mathématiquement possible

```javascript
function isBalancingPossible(students, numGroups) {
  const totalGirls = students.filter(s => s.sexe === 'F').length;
  const totalBoys = students.filter(s => s.sexe === 'M').length;
  const groupSize = Math.ceil(students.length / numGroups);

  const minGirlsPerGroup = Math.floor(groupSize * 0.4);
  const minBoysPerGroup = Math.floor(groupSize * 0.4);

  const needGirls = minGirlsPerGroup * numGroups;
  const needBoys = minBoysPerGroup * numGroups;

  if (totalGirls < needGirls || totalBoys < needBoys) {
    return {
      possible: false,
      reason: `Pas assez de ${totalGirls < needGirls ? 'filles' : 'garçons'}`
    };
  }

  return { possible: true };
}
```

#### Risque 3 : Dégradation de l'hétérogénéité académique

**Problème principal :**
Les swaps F/M pour équilibrer la parité peuvent **détruire** l'équilibrage académique initial.

**Exemple concret :**

```
AVANT SWAP:
  Groupe A: [Fille(z=2.0), Fille(z=1.5), Garçon(z=-1.0), Garçon(z=-1.5)]
    → Parité: 50% F
    → Moy z: 0.25  (hétérogène)

  Groupe B: [Fille(z=-2.0), Garçon(z=0.5), Garçon(z=1.0), Garçon(z=1.5)]
    → Parité: 25% F (DÉSÉQUILIBRÉ)
    → Moy z: 0.25  (hétérogène)

SWAP: Fille(z=2.0) ↔ Garçon(z=1.5)

APRÈS SWAP:
  Groupe A: [Fille(z=1.5), Garçon(z=1.5), Garçon(z=-1.0), Garçon(z=-1.5)]
    → Parité: 25% F (TOUJOURS DÉSÉQUILIBRÉ)
    → Moy z: 0.125  (légèrement moins hétérogène)

  Groupe B: [Fille(z=2.0), Fille(z=-2.0), Garçon(z=0.5), Garçon(z=1.0)]
    → Parité: 50% F (AMÉLIORÉ)
    → Moy z: 0.375  (MOINS hétérogène, écart réduit)
```

**Conséquence :**
❌ Groupe B devient **plus homogène** académiquement (écart-type réduit)
❌ L'objectif pédagogique d'hétérogénéité est **compromis**

---

## 5. RISQUES IDENTIFIÉS

### 5.1 Risques CRITIQUES 🔴

| # | Risque | Impact | Probabilité | Mitigation |
|---|--------|--------|-------------|------------|
| 1 | **Écart-type nul** | Division par 0 → NaN | Faible | Fallback σ=1 ✅ |
| 2 | **Population F/M impossible à équilibrer** | 10 itérations inutiles | Moyenne | Détection préalable 🔧 |
| 3 | **Scores à 0 non distingués de scores manquants** | Biais distribution | Élevée | Warning actuel ⚠️, exclusion recommandée |

### 5.2 Risques MAJEURS 🟠

| # | Risque | Impact | Probabilité | Mitigation |
|---|--------|--------|-------------|------------|
| 4 | **Swaps dégradent hétérogénéité** | Objectif péda. compromis | Moyenne | Critère swap intelligent 🔧 |
| 5 | **Effectif faible** | Z-scores non fiables | Moyenne | Warning si N<10 🔧 |
| 6 | **Coefficients inadaptés** | Priorité incorrecte | Faible | Personnalisation 🔧 |

### 5.3 Risques MINEURS 🟡

| # | Risque | Impact | Probabilité | Mitigation |
|---|--------|--------|-------------|------------|
| 7 | **Tolérance parité fixe** | Petits groupes déséquilibrés | Faible | Tolérance adaptative 🔧 |
| 8 | **Absences sous-pondérées** | Élèves absentéistes mixés | Faible | Paramétrable 🔧 |

**Légende :**
✅ Implémenté | 🔧 Recommandé | ⚠️ Partiellement

---

## 6. RECOMMANDATIONS PÉDAGOGIQUES

### 6.1 AVANT la génération des groupes

#### ✅ Validation des données

```javascript
// 1. Vérifier les scores académiques
const studentsWithoutScores = students.filter(s => s.scoreF === 0 && s.scoreM === 0);
if (studentsWithoutScores.length > 0) {
  alert(`⚠️ ${studentsWithoutScores.length} élève(s) sans scores académiques.
         Ils seront considérés comme "faibles" dans l'algorithme.
         Voulez-vous les exclure ou leur attribuer la médiane ?`);
}

// 2. Vérifier l'équilibre F/M global
const parityCheck = isBalancingPossible(students, numGroups);
if (!parityCheck.possible) {
  alert(`⚠️ Équilibrage 40-60% impossible: ${parityCheck.reason}
         Acceptez-vous une tolérance élargie (ex: 30-70%) ?`);
}

// 3. Vérifier l'effectif
if (students.length < 10) {
  alert(`⚠️ Effectif faible (${students.length} élèves).
         Les z-scores peuvent être peu fiables.
         Préférez une répartition manuelle ou un algorithme simplifié.`);
}
```

### 6.2 PENDANT la génération

#### 💡 Améliorer l'algorithme de swap

**Problème actuel :**
Le swap est accepté dès qu'il améliore la parité, sans considérer l'impact académique.

**Solution recommandée :**
Calculer un **score composite** pour chaque swap :

```javascript
function evaluateSwap(student1, student2, group1, group2) {
  // Score actuel
  const currentParityScore = calculateParityScore(group1, group2);
  const currentHeterogeneityScore = calculateHeterogeneityScore(group1, group2);

  // Simulation du swap
  const [newGroup1, newGroup2] = simulateSwap(student1, student2, group1, group2);

  // Nouveau score
  const newParityScore = calculateParityScore(newGroup1, newGroup2);
  const newHeterogeneityScore = calculateHeterogeneityScore(newGroup1, newGroup2);

  // Amélioration pondérée
  const parityImprovement = newParityScore - currentParityScore;
  const heterogeneityDegradation = currentHeterogeneityScore - newHeterogeneityScore;

  // Accepter le swap SI:
  // 1. Amélioration parité > 0
  // 2. Dégradation hétérogénéité < seuil toléré (ex: 10%)
  return {
    accept: parityImprovement > 0 && heterogeneityDegradation < 0.1,
    parityImprovement,
    heterogeneityDegradation
  };
}
```

#### 💡 Limiter le nombre de swaps par groupe

**Problème :**
Un même groupe peut subir plusieurs swaps successifs → instabilité

**Solution :**
Marquer les groupes modifiés et limiter à **1 swap par groupe par itération**

```javascript
const swappedGroups = new Set();
if (swappedGroups.has(group1.id) || swappedGroups.has(group2.id)) {
  continue;  // Skip ce swap
}
// Effectuer le swap
swappedGroups.add(group1.id);
swappedGroups.add(group2.id);
```

### 6.3 APRÈS la génération

#### ✅ Rapport de qualité

Afficher un **rapport détaillé** pour chaque groupe :

```
===== RAPPORT DE QUALITÉ =====

Groupe A (16 élèves):
  ✅ Parité: 50% F (8F-8M)         → ÉQUILIBRÉ
  ✅ Hétérogénéité: σ_index = 1.2  → TRÈS HÉTÉROGÈNE
  ⚠️ LV2: 100% ESP                 → PEU DIVERSIFIÉ
  ✅ Score moyen: 2.8/4            → NIVEAU CORRECT

Groupe B (15 élèves):
  ⚠️ Parité: 67% F (10F-5M)        → DÉSÉQUILIBRÉ
  ✅ Hétérogénéité: σ_index = 1.1  → HÉTÉROGÈNE
  ✅ LV2: 60% ESP, 40% ALL         → DIVERSIFIÉ
  ⚠️ Score moyen: 2.1/4            → NIVEAU FAIBLE

Recommandations:
  - Groupe B: Envisager un swap manuel pour améliorer parité
  - Groupe B: Niveau faible global → prévoir soutien renforcé
```

#### 💡 Swaps manuels guidés

Permettre à l'utilisateur de **modifier manuellement** avec suggestions intelligentes :

```
Vous souhaitez améliorer la parité du Groupe B ?

Suggestions de swaps:
  1. Fille "DUPONT M." (z=1.5) ↔ Garçon "MARTIN P." (z=1.4)
     → Impact parité: +13%
     → Impact hétérogénéité: -2% (acceptable)
     [RECOMMANDÉ]

  2. Fille "BERNARD L." (z=-0.5) ↔ Garçon "DURAND J." (z=2.0)
     → Impact parité: +13%
     → Impact hétérogénéité: -18% (RISQUÉ)
     [NON RECOMMANDÉ]
```

---

## 7. CONCLUSION

### 7.1 Points forts de l'algorithme V4

✅ **Robustesse** : Gestion des cas limites (σ=0, données manquantes)
✅ **Flexibilité** : 2 modes (hétérogène/homogène), scénarios personnalisables
✅ **Traçabilité** : ErrorReporter centralise logs et warnings
✅ **Performance** : Normalisation en O(n), swaps limités à 10 itérations

### 7.2 Axes d'amélioration prioritaires

| Priorité | Amélioration | Effort | Impact |
|----------|-------------|--------|--------|
| 🔴 P1 | **Détection impossibilité équilibrage F/M** | Faible | Élevé |
| 🔴 P1 | **Swap intelligent (composite score)** | Moyen | Élevé |
| 🟠 P2 | Gestion scores manquants (médiane) | Faible | Moyen |
| 🟠 P2 | Tolérance parité adaptative | Faible | Moyen |
| 🟡 P3 | Personnalisation coefficients | Moyen | Faible |
| 🟡 P3 | Rapport qualité détaillé | Élevé | Moyen |

### 7.3 Prochaines étapes recommandées

1. **Court terme** (Sprint 1-2 semaines)
   - Implémenter détection impossibilité équilibrage
   - Améliorer algorithme de swap (score composite)
   - Ajouter warnings effectif faible

2. **Moyen terme** (Sprint 2-4 semaines)
   - Interface personnalisation coefficients
   - Tolérance parité adaptative
   - Rapport qualité détaillé

3. **Long terme** (3+ mois)
   - Tests A/B avec enseignants
   - Collecte feedback pédagogique
   - Affinement basé sur données réelles

---

## 📚 ANNEXES

### Annexe A : Formules mathématiques

**Z-score :**
```
z = (x - μ) / σ

Avec:
  μ = (1/n) × Σ(x_i)
  σ = √[(1/n) × Σ(x_i - μ)²]
```

**Indice pondéré :**
```
I = Σ(w_i × z_i)

Avec Σ(w_i) = 1
```

**Ratio de parité :**
```
R_F = (N_F / N_total) × 100%
```

### Annexe B : Glossaire

| Terme | Définition |
|-------|------------|
| **Z-score** | Nombre d'écarts-types par rapport à la moyenne |
| **Hétérogène** | Groupes avec diversité de niveaux (objectif: σ élevé) |
| **Homogène** | Groupes avec niveaux similaires (objectif: σ faible) |
| **Parité** | Équilibre garçons/filles dans un groupe |
| **Swap** | Échange de 2 élèves entre 2 groupes |
| **Outlier** | Valeur extrême (|z| > 2) |

---

**FIN DU DOCUMENT**

*Ce document doit être mis à jour à chaque évolution majeure de l'algorithme.*
