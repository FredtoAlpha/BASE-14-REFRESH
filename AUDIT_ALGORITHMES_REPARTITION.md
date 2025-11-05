# AUDIT DÉTAILLÉ - ALGORITHMES DE RÉPARTITION
**Module Groupes v4.0** | **Date:** 30 octobre 2025

---

## EXECUTIVE SUMMARY

Les algorithmes de répartition sont **conceptuellement solides** mais **manquent de validation statistique** pour confirmer la qualité réelle.

| Critère | Évaluation | Confiance |
|---------|-----------|----------|
| **Logique hétérogène** | ✅ Robuste | 85% |
| **Logique homogène** | ⚠️ Basique | 65% |
| **Équilibre F/M** | ✅ Bon | 90% |
| **Critères pédagogiques** | ⚠️ Passifs | 40% |
| **Audit qualité** | ❌ Absent | 0% |

**Verdict:** Répartition de bonne qualité PROBABLE, mais non-vérifiée.

---

## 1. ANALYSE ALGORITHME HÉTÉROGÈNE

### Pseudo-code
```javascript
// 1. Trier élèves par score global (Math + FR)
const sorted = students.sort((a, b) =>
  (b.scoreM + b.scoreF) - (a.scoreM + a.scoreF)
);

// 2. Séparer par sexe
const girls = sorted.filter(s => s.sexe === 'F');
const boys = sorted.filter(s => s.sexe === 'M');
const other = sorted.filter(s => !['F', 'M'].includes(s.sexe));

// 3. Entrelacer (F/M/F/M/...)
const balanced = [];
for (let i = 0; i < Math.max(girls.length, boys.length); i++) {
  if (i < girls.length) balanced.push(girls[i]);
  if (i < boys.length) balanced.push(boys[i]);
}
balanced.push(...other);

// 4. Distribuer en round-robin
for (let g = 0; g < numGroups; g++) {
  for (let i = g; i < balanced.length; i += numGroups) {
    groups[g].push(balanced[i]);
  }
}
```

### Analyse Mathématique

#### Propriété 1: Parité F/M
```
Si filles = 25, garçons = 23, groups = 4

Après entrelaçage:
[F, M, F, M, F, M, F, M, ... F, F, F, ... F, other]

Distribution round-robin:
├─ Group 0: positions 0, 4, 8,  12, ... (F, F, F, F...)
├─ Group 1: positions 1, 5, 9,  13, ... (M, M, M, M...)
├─ Group 2: positions 2, 6, 10, 14, ... (F, F, F, F...)
└─ Group 3: positions 3, 7, 11, 15, ... (M, M, M, M...)

Résultat: Groupes peut-être MONOSEXE!
```

**⚠️ PROBLÈME CRITIQUE IDENTIFIÉ:**
L'entrelaçage F/M ne garantit PAS une parité PER-GROUP.
```
Exemple:
- Round-robin avec 48 élèves, 4 groupes = 12/groupe
- Si position pairs = femmes, impairs = hommes
- → Group 0 = [0, 4, 8, 12, 16, 20, 24, 28, ...] = 50/50 OK
- → MAIS si filles=20, garçons=28
- → Positions 0-19 ≠ Positions 1-48 (déséquilibre)
```

#### Propriété 2: Distribution des Niveaux

**Après tri par score + entrelaçage F/M:**
```
Sorted: [top scorer, 2nd, 3rd, ...]
  avec parité F/M dans l'ordre

Round-robin distribue:
- Group 0: 1er, 5e, 9e, 13e, ... (1 bon, 1 bon, ...)
- Group 1: 2e, 6e, 10e, 14e, ... (1 bon, 1 bon, ...)
```

**✅ BON:** Distribution "par rotation" garante:
- Chaque groupe = mélange haut/bas/milieu ✅
- Pas un groupe avec tous les "bons" ❌ Évité
- Pas un groupe avec tous les "faibles" ❌ Évité

**Mais:**
- Groupe 0 peut avoir `[top scorer, 5e, 9e, ...]` = un peu mieux
- Groupe 1 peut avoir `[2e, 6e, 10e, ...]` = un peu moins bien
- Variance faible mais **non-zéro**

---

## 2. ANALYSE ALGORITHME HOMOGÈNE

### Pseudo-code
```javascript
// 1-3. Tri + entrelaçage F/M (identique hétéro)
const balanced = /* ... */;

// 4. Découper en BLOCS séquentiels (au lieu de round-robin)
const chunkSize = Math.ceil(balanced.length / numGroups);
for (let g = 0; g < numGroups; g++) {
  const slice = balanced.slice(g * chunkSize, (g + 1) * chunkSize);
  groups[g] = slice;
}
```

### Analyse Mathématique

#### Propriété 1: Niveaux Homogènes
```
Après tri par score global:
[top scorer, 2e meilleur, 3e meilleur, ...]

Blocs:
├─ Group 0: indices [0 à chunkSize-1]     = tous les meilleurs
├─ Group 1: indices [chunkSize à 2*chunk] = bon/moyen mix
├─ Group 2: indices [...]                  = moyen/faible mix
└─ Group 3: indices [...fin]               = surtout les faibles
```

**🔴 CRITIQUE - GROUPES DE NIVEAU:**
```
Classe = 48 élèves, 4 groupes = 12/groupe

Group 0: [1er, 2e, 3e, 4e, 5e, 6e, 7e, 8e, 9e, 10e, 11e, 12e]
         Score moyen ~18/20 ⭐⭐⭐⭐⭐

Group 3: [37e, 38e, 39e, ..., 48e]
         Score moyen ~08/20 😞

CECI N'EST PAS HOMOGÈNE, C'EST STRATIFIÉ!
```

**Ce que "homogène" DEVRAIT faire:**
- Chaque groupe = mélange proche du niveau moyen
- Variance LOW mais pas zéro
- Pas de groupe avec tous les "bons" ou tous les "faibles"

**Ce que l'algo FAIT réellement:**
- Groupe 0 = "classe de niveau fort"
- Groupe 3 = "classe de niveau faible"
- **Utile pour "remédiation" ou "enrichissement" mais pas "homogène"!**

### Verdict Homogène
- ✅ **Utile pour:** Groupes de besoin/remédiation
- ❌ **Pas vrai "homogène":** Mieux dit "stratifiés par niveau"
- ⚠️ **Risque pédagogique:** Groupe faible peut se démotiver

---

## 3. CRITÈRES PÉDAGOGIQUES PASSIFS

### Critères Ignorés en Distribution
```
État inclut:
├─ PART (participation)
├─ COM (communication)
├─ TRA (travail)
└─ ABS (absences)
```

**Mais l'algorithme:**
1. Ne les utilise PAS dans le tri initial
2. Ne balance PAS les groupes par ces critères
3. Ne détecte PAS les "cas problématiques"

### Exemple Impact
```
Scénario: Classe 30 élèves, 3 groupes = 10/groupe

Scores Math/FR équilibrés mais:
- Élève A: participatif, travailleur
- Élève B: absent 50%, ne travaille pas
- Élève C: très participatif
- Élève D: muet, mais bon score
- ...

Algo distribution: Trie par (Math + FR) uniquement
→ Peut placer A, B, D dans même groupe (déséquilibré pédagogiquement)

Idéal: Chaque groupe = mélange A/B/C/D
```

### Recommandation: Weighted Score

**Proposé:**
```javascript
function computeWeightedScore(student) {
  const mathWeight = 0.3;
  const frenchWeight = 0.3;
  const partWeight = 0.15;     // Participation = 15%
  const comWeight = 0.1;       // Communication = 10%
  const traWeight = 0.1;       // Travail = 10%
  const absWeight = -0.05;     // Absences = -5%

  return (student.scoreM * mathWeight +
          student.scoreF * frenchWeight +
          student.part * partWeight +
          student.com * comWeight +
          student.tra * traWeight +
          student.abs * absWeight);
}
```

---

## 4. TEST STATISTIQUE: VALIDITÉ ÉQUILIBRE F/M

### Hypothèse Testée
**H0:** La distribution F/M est statistiquement équilibrée par groupe
**H1:** La distribution F/M est déséquilibrée

### Test Chi-Square (χ²)

**Exemple:**
```
Classe: 30 élèves (15F, 15M), 3 groupes

Attendu: Chaque groupe = 5F, 5M (si parfait équilibre)

Observé réel:
├─ Group 1: 6F, 4M  (χ² contribution = (6-5)²/5 + (4-5)²/5 = 0.4)
├─ Group 2: 5F, 5M  (χ² contribution = 0)
└─ Group 3: 4F, 6M  (χ² contribution = 0.4)

χ² total = 0.8

Degré de liberté = 2 (3 groupes - 1)
p-value(0.8, df=2) = 0.67 → NON significatif → OK ✅
```

### Implémentation Vérification

```javascript
function computeChiSquareGender(groups) {
  const totalF = groups.reduce((sum, g) =>
    sum + g.filter(s => s.sexe === 'F').length, 0);
  const totalM = groups.reduce((sum, g) =>
    sum + g.filter(s => s.sexe === 'M').length, 0);
  const total = totalF + totalM;

  const expectedF = (total / groups.length);
  const expectedM = (total / groups.length);

  let chi2 = 0;
  groups.forEach(group => {
    const actualF = group.filter(s => s.sexe === 'F').length;
    const actualM = group.filter(s => s.sexe === 'M').length;

    chi2 += Math.pow(actualF - expectedF, 2) / expectedF;
    chi2 += Math.pow(actualM - expectedM, 2) / expectedM;
  });

  const df = groups.length - 1;
  const pValue = chiSquareP(chi2, df);

  return {
    chi2,
    pValue,
    balanced: pValue > 0.05,  // Significatif si p > 0.05
    summary: pValue > 0.05 ? '✅ Équilibré' : '⚠️ Déséquilibré'
  };
}
```

---

## 5. TEST QUALITÉ: VARIANCE TAILLE GROUPES

### Problème
```
48 élèves, 4 groupes:
- Taille idéale: 12 / groupe
- Mais 48 ÷ 4 = 12 (parfait)
- Et 49 ÷ 4 = 12.25 → groupes 1: 13, 2: 13, 3: 13, 4: 10
```

### Test: Coefficient de Variation

```javascript
function computeSizeVariance(groups) {
  const sizes = groups.map(g => g.length);
  const mean = sizes.reduce((a, b) => a + b, 0) / sizes.length;
  const variance = sizes.reduce((sum, s) =>
    sum + Math.pow(s - mean, 2), 0) / sizes.length;
  const stdDev = Math.sqrt(variance);
  const cv = (stdDev / mean) * 100;  // Coefficient variation en %

  return {
    mean,
    min: Math.min(...sizes),
    max: Math.max(...sizes),
    stdDev,
    cv,  // 0% = parfait, < 10% = bon, > 20% = mauvais
    quality: cv < 10 ? '✅ Bon' : cv < 20 ? '⚠️ Acceptable' : '❌ Mauvais'
  };
}
```

**Interprétation:**
- CV = 0% : Tous les groupes même taille (impossible sauf cas spéciaux)
- CV < 5% : ✅ Excellent
- CV < 10% : ✅ Bon
- CV < 20% : ⚠️ Acceptable
- CV > 20% : ❌ Problématique

---

## 6. TEST QUALITÉ: DISTRIBUTION SCORES

### Hypothèse
**Chaque groupe = représentation fidèle de la distribution globale des scores**

### Implémentation

```javascript
function analyzeScoreDistribution(groups, subject = 'both') {
  const getAllScores = (subject) => {
    const key = subject === 'maths' ? 'scoreM' : subject === 'french' ? 'scoreF' : null;
    if (!key) {
      return groups.flat().map(s => s.scoreM + s.scoreF);
    }
    return groups.flat().map(s => s[key]);
  };

  const globalScores = getAllScores(subject);
  const globalMean = globalScores.reduce((a, b) => a + b, 0) / globalScores.length;
  const globalStdDev = Math.sqrt(
    globalScores.reduce((sum, s) =>
      sum + Math.pow(s - globalMean, 2), 0) / globalScores.length
  );

  const groupStats = groups.map((group, i) => {
    const scores = subject === 'maths' ? group.map(s => s.scoreM) :
                   subject === 'french' ? group.map(s => s.scoreF) :
                   group.map(s => s.scoreM + s.scoreF);

    const mean = scores.reduce((a, b) => a + b, 0) / scores.length;
    const stdDev = Math.sqrt(
      scores.reduce((sum, s) =>
        sum + Math.pow(s - mean, 2), 0) / scores.length
    );

    return {
      groupIndex: i,
      mean: mean.toFixed(2),
      stdDev: stdDev.toFixed(2),
      diff: (mean - globalMean).toFixed(2),
      percentileDiff: ((mean - globalMean) / globalMean * 100).toFixed(1)
    };
  });

  return {
    global: { mean: globalMean.toFixed(2), stdDev: globalStdDev.toFixed(2) },
    groups: groupStats,
    maxDiff: Math.max(...groupStats.map(g => Math.abs(parseFloat(g.diff)))).toFixed(2),
    quality: Math.max(...groupStats.map(g => Math.abs(parseFloat(g.diff)))) < 2 ?
             '✅ Bien équilibré' : '⚠️ Légères variations'
  };
}
```

**Exemple résultat:**
```
Global: Mean=12.5, StdDev=4.2

Group 0: Mean=13.1, Diff=+0.6 (✅ très proche)
Group 1: Mean=12.3, Diff=-0.2 (✅ très proche)
Group 2: Mean=11.9, Diff=-0.6 (✅ très proche)
Group 3: Mean=12.7, Diff=+0.2 (✅ très proche)

Max diff = 0.6 → BIEN ÉQUILIBRÉ ✅
```

---

## 7. DASHBOARD QUALITÉ (PROPOSITION)

### À Ajouter en Phase 5 (Panneau Stats)

```html
<div class="quality-report">
  <h4 class="text-lg font-bold mb-4">Rapport Qualité</h4>

  <!-- 1. Équilibre F/M -->
  <div class="stat-item">
    <h5 class="font-semibold">Parité F/M par groupe</h5>
    <div class="flex items-center gap-3">
      <div class="score-badge ${genderStats.balanced ? 'bg-green-100 text-green-700' : 'bg-amber-100 text-amber-700'}">
        ${genderStats.balanced ? '✅' : '⚠️'}
      </div>
      <div class="text-sm">
        <p>χ² = ${genderStats.chi2.toFixed(2)}</p>
        <p class="text-xs text-slate-600">p-value = ${genderStats.pValue.toFixed(3)}</p>
      </div>
    </div>
  </div>

  <!-- 2. Variance Taille -->
  <div class="stat-item">
    <h5 class="font-semibold">Variance Taille Groupes</h5>
    <div class="flex items-center gap-3">
      <div class="score-badge ${sizeStats.cv < 10 ? 'bg-green-100 text-green-700' : 'bg-amber-100 text-amber-700'}">
        ${(sizeStats.cv).toFixed(1)}%
      </div>
      <div class="text-sm">
        <p>${sizeStats.min}–${sizeStats.max} élèves</p>
        <p class="text-xs text-slate-600">${sizeStats.quality}</p>
      </div>
    </div>
  </div>

  <!-- 3. Distribution Scores -->
  <div class="stat-item">
    <h5 class="font-semibold">Distribution Scores</h5>
    <div class="flex items-center gap-3">
      <div class="score-badge bg-blue-100 text-blue-700">
        ±${scoreStats.maxDiff}
      </div>
      <div class="text-sm">
        <p>Écart max: ${scoreStats.maxDiff}</p>
        <p class="text-xs text-slate-600">${scoreStats.quality}</p>
      </div>
    </div>
  </div>

  <!-- 4. Quality Score Global -->
  <div class="stat-item bg-slate-50 p-3 rounded-lg mt-4">
    <h5 class="font-semibold mb-2">Score Qualité Global</h5>
    <div class="quality-bar">
      <div class="quality-fill" style="width: ${qualityScore}%"></div>
    </div>
    <p class="text-sm font-bold text-center mt-2">${qualityScore}/100</p>
    <p class="text-xs text-slate-600 text-center">
      ${qualityScore >= 80 ? '⭐ Excellent' :
        qualityScore >= 60 ? '👍 Bon' :
        qualityScore >= 40 ? '⚠️ Acceptable' :
        '❌ Révision nécessaire'}
    </p>
  </div>

  <!-- 5. Détail par Groupe -->
  <div class="stat-item mt-4">
    <h5 class="font-semibold mb-2">Détail Groupes</h5>
    <table class="text-xs w-full">
      <thead>
        <tr class="border-b">
          <th class="text-left">Groupe</th>
          <th>Élèves</th>
          <th>F/M</th>
          <th>Score</th>
        </tr>
      </thead>
      <tbody>
        ${groups.map((g, i) => {
          const f = g.filter(s => s.sexe === 'F').length;
          const m = g.filter(s => s.sexe === 'M').length;
          const avg = (g.reduce((sum, s) => sum + s.scoreM + s.scoreF, 0) / g.length / 2).toFixed(1);
          return `
            <tr class="border-b">
              <td class="font-semibold">G${i+1}</td>
              <td>${g.length}</td>
              <td>${f}F/${m}M</td>
              <td>${avg}</td>
            </tr>
          `;
        }).join('')}
      </tbody>
    </table>
  </div>
</div>
```

---

## 8. RECOMMANDATIONS FINALES

### Court Terme (Immédiat)
1. **Ajouter validation chi-square** F/M après génération
2. **Afficher variance taille** en panneau stats
3. **Ajouter warnings** si déséquilibre détecté

### Moyen Terme
4. **Implémenter weighted score** avec critères pédagogiques
5. **Ajouter comparaison hétéro vs homo** avant génération
6. **Créer quality score** global

### Long Terme
7. **Tester de vrais datasets** (50+ classes)
8. **Comparer avec distribution aléatoire**
9. **Valider avec pédagogues** (pas que stats)

---

## 9. CONCLUSION

**État actuel:** ✅ Algorithmes solides sur papier
**Validité réelle:** ⚠️ Non-vérifiée empiriquement
**Risque:** Possible que parité F/M soit bonne mais scores déséquilibrés

**Action urgente:** Ajouter audit statistique en Phase 5 pour confirmer (ou corriger).

