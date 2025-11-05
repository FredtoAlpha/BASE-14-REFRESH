# Roadmap Production-Ready : groupsModuleComplete

**État actuel** : Prototype/démo fonctionnel
**Objectif** : Production-ready pour utilisation dans workflows réels
**Estimation totale** : 4-6 sprints (16-24 semaines avec équipe 1 dev)

---

## 📊 Priorisation des Chantiers

### 🔴 SPRINT 1-2 : BLOCKERS CRITIQUES (3-4 semaines)

#### Chantier #1 : Persistance Multi-Vagues
**Impact** : 🔴 **CRITIQUE** - Workflow multi-jours impossible sinon
**Effort** : 8-12 heures

**Problème** :
- État de continuation en mémoire JS seulement
- Rechargement page = perte complète
- Impossible pour workflow sur 2+ jours

**Solution** :
1. Backend (Code.js) - Nouvelles fonctions :
   ```javascript
   function saveContinuationMetadata(type, metadata) {
     const props = PropertiesService.getDocumentProperties();
     props.setProperty('CONTINUATION_' + type, JSON.stringify(metadata));
     console.log('✅ Continuation metadata saved:', metadata);
   }

   function loadContinuationMetadata(type) {
     const props = PropertiesService.getDocumentProperties();
     const json = props.getProperty('CONTINUATION_' + type);
     return json ? JSON.parse(json) : null;
   }

   function clearContinuationMetadata(type) {
     const props = PropertiesService.getDocumentProperties();
     props.deleteProperty('CONTINUATION_' + type);
   }
   ```

2. Frontend (groupsModuleComplete.html) - Modifications :
   - Au chargement du module (onOpen) : charger metadata
   - Avant fermeture : sauvegarder metadata
   - Au reset : proposer continuation avec metadata sauvegardée

3. Code location :
   - Backend : Ajouter après `finalizeTempGroups()` (après ligne 2832)
   - Frontend : Modifier `moduleReady()` (appelle au chargement)

**Tests** :
- [ ] Créer groupes 1-3, fermer navigateur
- [ ] Rouvrir → proposer continuation avec bons offsets
- [ ] Créer groupes 4-6, vérifier numérotation
- [ ] Finaliser groupes 1-6, vérifier tous présents

**Dépendances** : Aucune
**Blockers** : Aucun

---

#### Chantier #2 : Score Composite et Algorithme d'Optimisation
**Impact** : 🔴 **CRITIQUE** - Équilibrage pédagogique insuffisant sinon
**Effort** : 20-24 heures

**Problème** :
- Algorithme actuel = tri par note + snake draft
- Pas de pondération multi-critères (comportement, assiduité)
- Pas d'optimisation itérative
- Groupes peuvent être très déséquilibrés

**Solution** :
1. Backend validation (Code.js) - Helper pour calcul :
   ```javascript
   function getCompositeScore(student, weights) {
     // weights = {f: 0.3, m: 0.3, com: 0.2, part: 0.2, ...}
     const f = Number(student.scores?.F ?? 0);
     const m = Number(student.scores?.M ?? 0);
     const com = Number(student.com ?? 0);
     const part = Number(student.part ?? 0);

     return (f × weights.f) + (m × weights.m) +
            (com × weights.com) + (part × weights.part);
   }
   ```

2. Frontend (groupsModuleComplete.html) - Remplacer generateGroupsLocally() :
   ```javascript
   function generateGroupsLocally() {
     const num = state.numGroups || 3;

     // ÉTAPE 1 : Calculer scores composites
     const weights = {
       f: 0.3, m: 0.3, com: 0.2, part: 0.2
     };
     state.students.forEach(s => {
       s.compositeScore = getCompositeScore(s, weights);
     });

     // ÉTAPE 2 : Trier par score composite
     const sorted = state.students.sort((a,b) =>
       b.compositeScore - a.compositeScore
     );

     // ÉTAPE 3 : Équilibrer parité
     const girls = sorted.filter(s => s.sexe?.toUpperCase() === 'F');
     const boys = sorted.filter(s => s.sexe?.toUpperCase() === 'M');
     const balanced = interleave(girls, boys);

     // ÉTAPE 4 : Créer groupes via snake draft
     const groups = createGroupsSnakeDraft(balanced, num);

     // ÉTAPE 5 : Optimiser avec échanges itératifs
     optimizeGroupsWithSwaps(groups, weights);

     state.generatedGroups = groups;
   }

   function optimizeGroupsWithSwaps(groups, weights) {
     let improved = true;
     let iterations = 0;
     const maxIterations = 50;

     while (improved && iterations < maxIterations) {
       improved = false;
       iterations++;

       for (let i = 0; i < groups.length; i++) {
         for (let j = i + 1; j < groups.length; j++) {
           // Calculer écart-type actuel
           const currentStdDev = calculateGroupStdDev(groups, weights);

           // Essayer tous les échanges de paires
           for (let ki = 0; ki < groups[i].students.length; ki++) {
             for (let kj = 0; kj < groups[j].students.length; kj++) {
               // Swap temporaire
               [groups[i].students[ki], groups[j].students[kj]] =
               [groups[j].students[kj], groups[i].students[ki]];

               const newStdDev = calculateGroupStdDev(groups, weights);

               if (newStdDev < currentStdDev - 0.01) {
                 // Amélioration, garder le swap
                 improved = true;
                 currentStdDev = newStdDev;
                 break;
               } else {
                 // Remettre en place
                 [groups[i].students[ki], groups[j].students[kj]] =
                 [groups[j].students[kj], groups[i].students[ki]];
               }
             }
             if (improved) break;
           }
           if (improved) break;
         }
         if (improved) break;
       }
     }

     console.log(`[GroupsModule] Optimisation en ${iterations} itérations`);
   }

   function calculateGroupStdDev(groups, weights) {
     const scores = groups.map(g =>
       g.students.reduce((sum, s) => sum + s.compositeScore, 0) / g.students.length
     );
     const mean = scores.reduce((a,b) => a + b) / scores.length;
     const variance = scores.reduce((sum, s) => sum + Math.pow(s - mean, 2), 0) / scores.length;
     return Math.sqrt(variance);
   }
   ```

3. Code location :
   - Frontend : Remplacer ligne 2033-2119
   - Ajouter helper calculs dans section "HELPERS POUR RÉPARTITION"

**Tests** :
- [ ] Charger 120 élèves, générer 3 groupes
- [ ] Vérifier scores composites calculés
- [ ] Vérifier écart-type avant/après optimisation (doit diminuer)
- [ ] Vérifier parité F/M équilibrée
- [ ] Vérifier logs : "Optimisation en N itérations"

**Dépendances** : Aucune
**Blockers** : Aucun

---

### 🟠 SPRINT 2-3 : HAUTE PRIORITÉ (2-3 semaines)

#### Chantier #3 : Dashboard de Validation et Statistiques
**Impact** : 🟠 **HAUTE** - Utilisateur ne peut pas valider équilibrage sinon
**Effort** : 12-16 heures

**Problème** :
- Bouton "Statistiques" ne fait rien (stub)
- Pas de visibilité sur écarts-type, parité, anomalies
- Impossible de valider avant finalisation

**Solution** :
1. Frontend - Créer `renderStatisticsPanel()` :
   ```javascript
   function renderStatisticsPanel() {
     const html = `
       <div class="stats-container">
         <h3>📊 Statistiques des Groupes</h3>

         <table class="stats-table">
           <thead>
             <tr>
               <th>Groupe</th>
               <th>Effectif</th>
               <th>Avg Score</th>
               <th>Parité F/M</th>
               <th>Avg COM</th>
               <th>Avg PART</th>
             </tr>
           </thead>
           <tbody>
             ${state.generatedGroups.map((g, i) => {
               const avgScore = g.students.length > 0
                 ? (g.students.reduce((s,st) => s + st.compositeScore, 0) / g.students.length).toFixed(2)
                 : 0;
               const girls = g.students.filter(s => s.sexe === 'F').length;
               const boys = g.students.filter(s => s.sexe === 'M').length;
               const avgCom = g.students.length > 0
                 ? (g.students.reduce((s,st) => s + (st.com ?? 0), 0) / g.students.length).toFixed(2)
                 : 0;

               return `
                 <tr>
                   <td>Groupe ${state.tempOffsetStart + i}</td>
                   <td>${g.students.length}</td>
                   <td>${avgScore}</td>
                   <td>${girls}/${boys}</td>
                   <td>${avgCom}</td>
                   <td>N/A</td>
                 </tr>
               `;
             }).join('')}
           </tbody>
         </table>

         <h4>Métriques Globales</h4>
         <div class="metrics">
           <p>Écart-type (scores): <strong id="metric-stddev">0</strong></p>
           <p>Écart-type (effectifs): <strong id="metric-size-stddev">0</strong></p>
           <p>Parité globale F/M: <strong id="metric-parity">50/50</strong></p>
         </div>

         <h4>Alertes 🚨</h4>
         <div class="alerts" id="stats-alerts">
           <!-- Populated by JS -->
         </div>
       </div>
     `;

     return html;
   }

   function calculateStatistics() {
     const stats = {
       byGroup: [],
       global: {}
     };

     // Par groupe
     state.generatedGroups.forEach((g, i) => {
       stats.byGroup.push({
         idx: i,
         size: g.students.length,
         avgScore: g.students.reduce((s,st) => s + st.compositeScore, 0) / g.students.length,
         avgCom: g.students.reduce((s,st) => s + (st.com ?? 0), 0) / g.students.length,
         girls: g.students.filter(s => s.sexe === 'F').length,
         boys: g.students.filter(s => s.sexe === 'M').length
       });
     });

     // Global
     const scores = stats.byGroup.map(g => g.avgScore);
     const sizes = stats.byGroup.map(g => g.size);
     const meanScore = scores.reduce((a,b) => a+b) / scores.length;
     const meanSize = sizes.reduce((a,b) => a+b) / sizes.length;

     stats.global.scoreStdDev = Math.sqrt(
       scores.reduce((s,sc) => s + Math.pow(sc - meanScore, 2), 0) / scores.length
     );
     stats.global.sizeStdDev = Math.sqrt(
       sizes.reduce((s,sz) => s + Math.pow(sz - meanSize, 2), 0) / sizes.length
     );

     return stats;
   }

   function displayAlerts(stats) {
     const alerts = [];

     // Alerte parité
     const totalGirls = stats.byGroup.reduce((s,g) => s + g.girls, 0);
     const totalBoys = stats.byGroup.reduce((s,g) => s + g.boys, 0);
     if (Math.abs(totalGirls - totalBoys) > 5) {
       alerts.push('⚠️ Parité déséquilibrée (' + totalGirls + 'F / ' + totalBoys + 'M)');
     }

     // Alerte écart effectifs
     if (stats.global.sizeStdDev > 2) {
       alerts.push('⚠️ Groupes déséquilibrés en taille (écart-type: ' + stats.global.sizeStdDev.toFixed(2) + ')');
     }

     // Alerte écart scores
     if (stats.global.scoreStdDev > 1.5) {
       alerts.push('⚠️ Groupes déséquilibrés en niveau (écart-type: ' + stats.global.scoreStdDev.toFixed(2) + ')');
     }

     return alerts;
   }
   ```

2. UI - Bouton "Statistiques" :
   - Créer modal avec contenu du dashboard
   - Ajouter contrôle pour "Régénérer avec optimisation"
   - Afficher alertes en rouge/orange

3. Code location :
   - Frontend : Remplacer stub vers ligne 2100-2150

**Tests** :
- [ ] Afficher statistiques pour groupe quelconque
- [ ] Vérifier calculs écart-type corrects
- [ ] Tester alertes pour différents cas (parité, taille, score)
- [ ] Régénérer et vérifier amélioration des metrics

**Dépendances** : Chantier #2 (score composite)
**Blockers** : Aucun

---

#### Chantier #4 : Audit Logging Complet
**Impact** : 🟠 **HAUTE** - Conformité RGPD, traçabilité requise
**Effort** : 6-8 heures

**Problème** :
- Aucun journal des opérations sur groupes
- Impossible de savoir qui, quand, quoi
- Non-conformité RGPD

**Solution** :
1. Backend (Code.js) - Nouvelle feuille + fonction :
   ```javascript
   function ensureAuditLog() {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     let auditSheet = ss.getSheetByName('_AUDIT_LOG');
     if (!auditSheet) {
       auditSheet = ss.insertSheet('_AUDIT_LOG');
       auditSheet.appendRow([
         'Timestamp',
         'User',
         'Operation',
         'Type',
         'Prefix',
         'Range',
         'Count',
         'Mode',
         'Details'
       ]);
       auditSheet.hideSheet();
     }
     return auditSheet;
   }

   function logGroupOperation(operation, details) {
     try {
       const auditSheet = ensureAuditLog();
       auditSheet.appendRow([
         new Date().toISOString(),
         Session.getActiveUser().getEmail(),
         operation, // SAVE_TEMP | FINALIZE | DELETE
         details.type || 'unknown',
         details.prefix || '',
         details.range || '',
         details.count || 0,
         details.mode || '',
         JSON.stringify(details)
       ]);
       console.log('✅ Opération loggée:', operation);
     } catch (e) {
       console.error('Erreur log:', e);
       // Ne pas bloquer l'opération
     }
   }
   ```

2. Appeler `logGroupOperation()` dans :
   - `saveTempGroups()` après succès
   - `finalizeTempGroups()` après succès
   - `deleteGroup()` après suppression

3. Code location :
   - Ajouter après `finalizeTempGroups()` (vers ligne 2832)
   - Appels : ligne 2500 (saveTempGroups), ligne 2820 (finalizeTempGroups)

**Tests** :
- [ ] Créer TEMP groupes
- [ ] Vérifier entrée dans _AUDIT_LOG
- [ ] Finaliser groupes
- [ ] Vérifier entrée avec bons champs
- [ ] Supprimer groupe
- [ ] Vérifier entrée DELETE

**Dépendances** : Aucune
**Blockers** : Aucun

---

### 🟡 SPRINT 3-4 : MOYENNE PRIORITÉ (2-3 semaines)

#### Chantier #5 : UI Filtrage avec Confirmation
**Impact** : 🟡 **MOYENNE** - Utilisateur ne voit pas l'impact du filtre
**Effort** : 4-6 heures

**Problème** :
- Filtre LV2/Option appliqué silencieusement
- Pas de visibilité "avant: 120 → après: 45"
- Risque d'élèves "oubliés" accidentellement

**Solution** :
1. Frontend - Ajouter bandeau après loadStudentsFromClasses() :
   ```html
   <div class="filter-info-banner">
     <p>✅ Élèves chargés: <strong id="before-filter">120</strong></p>
     ${state.groupType === 'language' ? `
       <p>Filtré par langue: <strong>${state.selectedLanguage}</strong></p>
       <p>✅ Après filtre: <strong id="after-filter">45</strong> élèves</p>
     ` : ''}
     <label>
       <input type="checkbox" id="confirm-filter">
       J'ai vérifié le filtre et je souhaite continuer
     </label>
   </div>
   ```

2. Bloquer progression si filtre non confirmé

3. Code location :
   - Frontend : Ajouter après étape 3 (après sélection classes)

**Tests** :
- [ ] Sélectionner classe avec ESP et ITA
- [ ] Choisir type "language" + "ESP"
- [ ] Vérifier bandeau montre réduction (ex: 120 → 45)
- [ ] Bloquer progression sans confirmation

**Dépendances** : Aucune
**Blockers** : Aucun

---

#### Chantier #6 : Versioning et Snapshots
**Impact** : 🟡 **MOYENNE** - Impossible de revenir à version antérieure
**Effort** : 8-10 heures

**Problème** :
- Une fois finalisé, impossible d'annuler/revenir
- Pas de snapshots
- Une erreur = données perdues

**Solution** :
1. Backend - Créer snapshots :
   ```javascript
   function createGroupSnapshot(type, groupName) {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     const original = ss.getSheetByName(groupName);
     if (!original) return null;

     // Copier vers _archive_
     const timestamp = new Date().toISOString().slice(0,10);
     const snapshotName = groupName + '_snapshot_' + timestamp;
     const copy = original.copy();
     copy.setName(snapshotName);
     copy.hideSheet();

     // Garder seulement les 5 derniers snapshots
     const all = ss.getSheets();
     const snapshots = all.filter(sh => sh.getName().startsWith(groupName + '_snapshot'));
     if (snapshots.length > 5) {
       ss.deleteSheet(snapshots[0]);
     }

     return snapshotName;
   }

   function restoreFromSnapshot(snapshotName) {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     const snapshot = ss.getSheetByName(snapshotName);
     if (!snapshot) return { success: false, error: 'Snapshot not found' };

     const originalName = snapshotName.replace(/_snapshot.*/, '');
     const original = ss.getSheetByName(originalName);

     // Copier snapshot par-dessus original
     if (original) ss.deleteSheet(original);
     const restored = snapshot.copy();
     restored.setName(originalName);
     restored.showSheet();

     return { success: true, message: 'Restored from ' + snapshotName };
   }
   ```

2. Appeler `createGroupSnapshot()` dans `finalizeTempGroups()` avant changements

3. Code location :
   - Ajouter après ligne 2832
   - Appel : ligne 2760 (debut finalizeTempGroups)

**Tests** :
- [ ] Finaliser groups
- [ ] Vérifier snapshots créés avec date
- [ ] Supprimer groupe
- [ ] Restaurer depuis snapshot
- [ ] Vérifier >5 snapshots gardés seulement 5 derniers

**Dépendances** : Aucune
**Blockers** : Aucun

---

### 🟢 SPRINT 4+ : BASSE PRIORITÉ (1-2 semaines chacune)

#### Chantier #7 : Contraintes Structurelles (SOURCE, effectifs ±1)
**Impact** : 🟢 **BASSE** - Amélioration mais non-bloquant
**Effort** : 8-12 heures

- Ajouter paramètre pour max élèves par classe d'origine par groupe
- Ajouter contrainte ±1 effectif
- Intégrer dans algorithme d'optimisation

---

#### Chantier #8 : Prévisualisation PDF Avancée
**Impact** : 🟢 **BASSE** - Nice-to-have
**Effort** : 4-6 heures

- Ajouter modal de préview avant téléchargement
- Afficher pages, pouvoir scroller, zoom

---

#### Chantier #9 : Module Options Complet
**Impact** : 🟢 **BASSE** - Futur, non urgent
**Effort** : 12-16 heures

- UI complète pour sélection option
- Règles de filtrage spécifiques
- Dédié après groupes langue fonctionnels

---

## 📋 Tableau de Marche

| Sprint | Chantiers | Durée | Livrables |
|--------|-----------|-------|-----------|
| **#1** | #1 Persistance | 1-2 sem | Workflow multi-jours ✅ |
| **#2** | #2 Score + Algo | 2-3 sem | Équilibrage robuste ✅ |
| **#3** | #3 Dashboard, #4 Audit | 2-3 sem | Stats + traçabilité ✅ |
| **#4** | #5 Filtre UI, #6 Versioning | 2-3 sem | Sécurité + UX ✅ |
| **#5+** | #7-9 Enhancements | 1+ sem each | Production complet |

---

## ✅ Checklist Fin de Chaque Sprint

### Sprint #1
- [ ] Metadata sauvegardée en PropertiesService
- [ ] Rechargement page = récupération continuité
- [ ] Tests multi-jours passés
- [ ] Docs mises à jour

### Sprint #2
- [ ] Score composite implémenté
- [ ] Algo optimisation itérative
- [ ] Écart-type diminue après optimisation
- [ ] Tests équilibrage passés

### Sprint #3
- [ ] Dashboard stats fonctionnel
- [ ] Alertes affichées correctement
- [ ] _AUDIT_LOG rempli de logs
- [ ] Traçabilité complète

### Sprint #4
- [ ] Filtre UI affiche avant/après
- [ ] Snapshots créés et restaurables
- [ ] Tests sauvegarde/restauration passés

---

## 🎯 Conclusion

**Après 4 sprints** (16-20 semaines) :
- ✅ Module production-ready
- ✅ Workflow multi-jours robuste
- ✅ Équilibrage pédagogique solide
- ✅ Traçabilité complète
- ✅ Récupération sur erreur possible

**Estimer 2-4 heures/jour de dev (1 personne) pour respecter timeline**

---

**Document créé** : 2025-10-29
**Version** : 1.0
**Prochaine revue** : Après Sprint #1
