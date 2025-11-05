# 🧪 GUIDE DE TEST - 3 QUICK WINS
**Version:** 4.1 | **Date:** 30 octobre 2025

---

## 🎯 Objectif Test

Valider les 3 quick wins implémentés :
1. Masquage header amélioré
2. Mode focus groupes
3. Validation Phase 4

---

## 📋 PROCÉDURE TEST - MASQUAGE HEADER

### Test 1: Bouton Masquer Visible
**Étape:** Aller en Phase 5 (générer des groupes d'abord)
**Action:** Regarder le header, chercher bouton "Masquer"
**Attendre:** Bouton doit être visible top-right du header, avec icône chevrons up
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 2: Clic Masquer - Animation
**Étape:** Phase 5 avec header visible
**Action:** Clic bouton "Masquer"
**Attendre:**
- Header disparaît fluidement (animation 300ms)
- Bouton flottant apparaît top-right (fixed position)
- Bouton flottant doit dire "En-tête" avec chevrons down
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 3: Bouton Flottant - Réapparition
**Étape:** Header masqué
**Action:** Clic bouton flottant bleu
**Attendre:**
- Header réapparaît fluidement
- Bouton flottant disparaît
- Bouton "Masquer" visible à nouveau
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 4: Raccourci H - Masquer
**Étape:** Phase 5, header visible
**Action:** Appuyer touche H
**Attendre:**
- Header masque (animation)
- Bouton flottant apparaît
- Toast: "En-tête masqué (Raccourci: H)"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 5: Raccourci H - Afficher
**Étape:** Phase 5, header masqué
**Action:** Appuyer touche H
**Attendre:**
- Header réapparaît (animation)
- Bouton flottant disparaît
- Toast: "En-tête affiché (Raccourci: H)"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 6: Persistance État
**Étape:** Phase 5, header masqué
**Action:** Clic "Recommencer" → Phase 1
**Attendre:** Recommencer création groupes
**Action:** Aller à Phase 5 à nouveau
**Attendre:** Header doit être MASQUÉ (même que avant)
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 7: Focus not on Input
**Étape:** Phase 5, aucun input focus
**Action:** Appuyer H
**Attendre:** Header toggle
**Puis:** Clic dans input (label "Numéro de départ" par exemple)
**Action:** Appuyer H
**Attendre:** RIEN ne se passe (ignoré si input focus)
**Résultat:** ✅ PASS ou ❌ FAIL

---

## 🚀 PROCÉDURE TEST - MODE FOCUS

### Test 8: Raccourci F - Activer Focus
**Étape:** Phase 5 avec groupes générés, mode normal
**Action:** Appuyer touche F
**Attendre:**
- Header disparaît
- Stepper disparaît
- Persistance controls disparaît
- Barre actions normale disparaît
- Toolbar apparaît EN BAS (fixed), très compacte
- Grille s'élargit (plus de colonnes visibles)
- Toast: "Mode focus activé (F)"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 9: Grille Colonnes - Focus
**Étape:** Mode focus actif
**Action:** Compter les colonnes du groupe
**Attendre:**
- Résolution 1920px: 4-5 colonnes au lieu de 2-3
- Résolution 1366px: 3-4 colonnes
- Résolution 1024px: 2-3 colonnes
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 10: Toolbar Compact - Boutons Micro
**Étape:** Mode focus actif
**Action:** Regarder toolbar en bas
**Attendre:**
- Boutons petit (36x36px)
- Icônes uniquement, pas de texte
- Titles au hover (Stats, Regen, Load, Save, etc)
- Séparateurs visuels entre groupes
- Bouton "Expand" ou icon à droite pour quitter focus
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 11: Drag & Drop en Focus
**Étape:** Mode focus actif
**Action:** Drag un élève d'un groupe à l'autre
**Attendre:**
- Drag fonctionne normalement
- Drop fonctionne
- Grille se réorganise
- Stats se mettent à jour (si visible)
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 12: Stats Toggle en Focus
**Étape:** Mode focus actif
**Action:** Appuyer S (ou clic bouton stats toolbar)
**Attendre:**
- Toast: "Stats affichées (S)"
- Panneau stats apparaît à CÔTÉ de la grille
- Grille rétrécit légèrement
- Appuyer S à nouveau → Stats disparaissent
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 13: Raccourci F - Désactiver Focus
**Étape:** Mode focus actif
**Action:** Appuyer touche F
**Attendre:**
- Header réapparaît
- Stepper réapparaît
- Toolbar normal réapparaît
- Grille rétrécit (moins de colonnes)
- Toast: "Mode normal activé (F)"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 14: Bouton Toggle Focus
**Étape:** Phase 5, mode normal
**Action:** Chercher bouton "Focus" ou icon expand
**Attendre:** Bouton visible dans toolbar
**Action:** Clic bouton
**Attendre:** Focus mode s'active (même que F)
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 15: Focus Mode Multi-Viewport
**Étape:** Tester sur 3 résolutions
**Action:** 1920x1080 → F → 4-5 colonnes
**Action:** 1366x768 → F → 3-4 colonnes
**Action:** 1024x600 → F → 2-3 colonnes, pas scroll horizontal
**Résultat:** ✅ PASS ou ❌ FAIL

---

## 📊 PROCÉDURE TEST - VALIDATION PHASE 4

### Test 16: Stats Affichées Phase 4
**Étape:** Passer Phases 1-3, arriver Phase 4
**Action:** Observer le contenu
**Attendre:**
- 4 cartes stats: "Total élèves", "Total groupes", "Taille/groupe", "Temps estimé"
- Chaque carte affiche nombre + unité
- Chiffres cohérents avec sélection
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 17: Pas de Warnings - OK Badge
**Étape:** Phase 4, cas normal (15-30 élèves, 3 groupes)
**Action:** Observer le contenu
**Attendre:**
- Badge vert: "Configuration validée ✓"
- Pas de section "Vérifications" (warnings)
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 18: Warning - Classe Petite
**Étape:** Phase 2: Sélectionner 1 classe, puis Phase 4
**Action:** Observer le contenu
**Attendre:**
- Section "Vérifications" affichée (fond amber)
- Warning: "⚠️ Classe très petite (< 10 élèves)..."
- User peut quand même continuer
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 19: Warning - Groupes Petits
**Étape:** Phase 2-3: Sélectionner 1 classe (20 élèves), 8 groupes, Phase 4
**Action:** Observer le contenu
**Attendre:**
- Warning: "⚠️ Groupes très petits (< 5 élèves)..."
- Suggestion: "Considérez réduire le nombre de groupes"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 20: Warning - Groupes Grands
**Étape:** Phase 2-3: Sélectionner 5 classes (300 élèves), 3 groupes, Phase 4
**Action:** Observer le contenu
**Attendre:**
- Warning: "⚠️ Groupes très grands (> 25 élèves)..."
- Suggestion: "Considérez augmenter le nombre de groupes"
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 21: Info - Effectifs Inégaux
**Étape:** Phase 4, si 10 élèves / 3 groupes (inégal)
**Action:** Observer le contenu
**Attendre:**
- Info: "ℹ️ Effectifs inégaux: 1 groupe(s) aura/auront 1 élève de plus."
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 22: Recommandation Algo - Petit Groupe
**Étape:** Phase 2-3: 12 élèves, 6 groupes (2 élèves/groupe), Phase 4
**Action:** Observer section "Recommandation"
**Attendre:**
- Texte: "💡 Avec des groupes si petits, la répartition hétérogène garantit..."
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 23: Recommandation Algo - Grand Groupe
**Étape:** Phase 2-3: 100 élèves, 3 groupes (~33/groupe), Phase 4
**Action:** Observer section "Recommandation"
**Attendre:**
- Texte: "💡 Avec de grands groupes, une répartition homogène (par niveau)..."
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 24: Recommandation Algo - Défaut
**Étape:** Phase 2-3: 40 élèves, 3 groupes (~13/groupe), Phase 4
**Action:** Observer section "Recommandation"
**Attendre:**
- Texte: "💡 La distribution hétérogène est recommandée..."
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 25: Continuer avec Warnings
**Étape:** Phase 4 avec warnings
**Action:** Clic "Générer les groupes" (bouton primary)
**Attendre:**
- Phase 5 se lance
- Groupes générés quand même
- User peut utiliser groupes normalement
**Résultat:** ✅ PASS ou ❌ FAIL

---

## 📱 TESTS ADDITIONNELS

### Test 26: Responsive Design
**Étape:** Tester sur 3 résolutions
**Action:** 1920x1080 → Normal, Focus → Ok
**Action:** 1366x768 → Normal, Focus → Ok
**Action:** 1024x600 → Normal, Focus → Ok
**Attendre:** Tous les tests précédents OK sur chaque résolution
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 27: Accessibility - Focus Clavier
**Étape:** Phase 5
**Action:** Tab plusieurs fois → parcourir buttons, inputs
**Attendre:**
- Focus visuel clair sur chaque élément
- Raccourcis clavier (H, F, S) restent actifs
- Focus ring visible (pas caché)
**Résultat:** ✅ PASS ou ❌ FAIL

### Test 28: Browser Compatibility
**Étape:** Tester sur Chrome, Firefox, Safari (si possible)
**Action:** Tous les tests précédents sur chaque navigateur
**Attendre:** Pas de différences critiques
**Résultat:** ✅ PASS ou ❌ FAIL

---

## 🏁 RÉSUMÉ TEST

### Checklist Finale

**Masquage Header (Tests 1-7):**
- [ ] Test 1: Bouton masquer visible
- [ ] Test 2: Clic masquer + animation
- [ ] Test 3: Bouton flottant + réapparition
- [ ] Test 4: Raccourci H masquer
- [ ] Test 5: Raccourci H afficher
- [ ] Test 6: Persistance état
- [ ] Test 7: Focus sur input ignore

**Mode Focus (Tests 8-15):**
- [ ] Test 8: Raccourci F activer
- [ ] Test 9: Grille colonnes expandées
- [ ] Test 10: Toolbar micro buttons
- [ ] Test 11: Drag & drop focus
- [ ] Test 12: Stats toggle focus
- [ ] Test 13: Raccourci F désactiver
- [ ] Test 14: Bouton toggle focus
- [ ] Test 15: Multi-viewport

**Validation Phase 4 (Tests 16-25):**
- [ ] Test 16: Stats affichées
- [ ] Test 17: OK badge normal
- [ ] Test 18: Warning petite classe
- [ ] Test 19: Warning petits groupes
- [ ] Test 20: Warning grands groupes
- [ ] Test 21: Info effectifs inégaux
- [ ] Test 22: Recommandation petit
- [ ] Test 23: Recommandation grand
- [ ] Test 24: Recommandation défaut
- [ ] Test 25: Continuer avec warnings

**Additionnels (Tests 26-28):**
- [ ] Test 26: Responsive 3 résolutions
- [ ] Test 27: Accessibility clavier
- [ ] Test 28: Browser compatibility

### Score Final
- **Total Tests:** 28
- **Réussi:** ___ / 28
- **Score:** ___%

**Verdict:**
- ✅ **OK** si 28/28 PASS
- 🟡 **Minor Issues** si 26-27/28 PASS
- 🔴 **Major Issues** si < 25/28 PASS

---

## 📝 NOTES TEST

**Anomalies Trouvées:**
(Lister ici toute issue trouvée)

**Configurations Testées:**
- Browser: ___________
- Résolution: ___________
- OS: ___________

**Testeur:** ___________
**Date:** ___________
**Statut:** ☐ Validé | ☐ À corriger

