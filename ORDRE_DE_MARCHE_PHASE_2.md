# Ordre de Marche - Phase 2 : Corrections Ergonomiques & Workflow

**Date**: 29 octobre 2025
**Objectif**: Formaliser les 3 étapes de validation avant démarrage Phase 2
**Durée Phase 2**: 18,5 heures (2-3 jours intensifs)
**Livrables**: UI complètement restructurée (Steps 2, 3, 5 + action bar)

---

## 📋 ÉTAPE 1: RELIRE LA DOCUMENTATION

### Documents Critiques à Consulter

#### 🔴 **PRIORITÉ ABSOLUE: CORRECTIONS_OBLIGATOIRES.md**
Cette note synthétise le diagnostic utilisateur. **À lire d'abord par toute l'équipe.**

**Sections clés**:
- **1. Constat d'ergonomie et de workflow** (Les 4 problèmes UX)
  - Barre d'actions surchargée (8 boutons)
  - Espace statistiques envahissant (50%)
  - Étape 2 trop basique (pas de regroupements)
  - Absence de visualisation des passes

- **2. Corrections UI obligatoires** (Approche de solution)
  - Repenser Étape 2 autour des regroupements
  - Navigation claire entre passes (tabs/selector)
  - Hiérarchiser la barre d'actions
  - Redimensionner le panneau stats
  - Indicateurs pédagogiques visibles

- **3. Corrections moteur obligatoires** (Backend)
  - Persistance multi-passes réelle
  - Finalisation non destructive
  - Gestion structurée des regroupements
  - Filtres cohérents par type

- **4. Workflow cible recommandé**
  - L'utilisateur définit tous les regroupements à Step 2
  - Configure par regroupement à Step 3
  - Génère par regroupement à Step 4
  - Sauvegarde & finalise sans perte de données précédentes

- **5. Priorisation**
  1. Implémenter la structure de regroupements (sine qua non)
  2. Revoir la barre d'actions & mode continuation
  3. Réduire/ajuster le panneau stats
  4. Renforcer la persistance Apps Script

#### 📊 **Lecture Secondaire: groupsModuleComplete_status.md**
État actuel de l'implémentation (Sprints #1-5 + bugs critiques corrigés).

**Sections clés**:
- Sprint #1: Persistance multi-jours (état: 🚧 SEMI-FONCTIONNELLE)
- Sprint #2: Score composite & optimisation
- Sprint #3: Dashboard & validation
- Sprint #4: Versioning & snapshots
- Sprint #5: Audit logging & RGPD

#### 🏗️ **Lecture Tertiaire: ARCHITECTURE_GROUPINGS_REDESIGN.md**
Design complet du système de regroupements (Phase 2-3).

**Sections clés**:
- Concept: Groupings Model (voir Section 1)
- Data Structures (Section 2)
- UI/UX Redesign (Section 3) ← **MOCKUPS VISUELS À EXAMINER**
- Backend Changes (Section 4)

---

### Checklist de Lecture (Équipe Complète)

**À faire avant la validation formelle**:

- [ ] **Chef de projet**: Lire CORRECTIONS_OBLIGATOIRES.md (30 min)
- [ ] **Développeur frontend**: Lire CORRECTIONS_OBLIGATOIRES.md + ARCHITECTURE_GROUPINGS_REDESIGN.md Section 3 (1h30)
- [ ] **Responsable QA**: Lire CORRECTIONS_OBLIGATOIRES.md + groupsModuleComplete_status.md (1h)
- [ ] **Stakeholders/Métier**: Lire CORRECTIONS_OBLIGATOIRES.md Sections 4-5 (30 min)
- [ ] **Équipe complète**: Revoir SESSION_SUMMARY.md + GROUPINGS_REDESIGN_STATUS.md (20 min)

**Durée totale**: 3-4 heures pour l'équipe complète

---

## ✅ ÉTAPE 2: VALIDER L'APPROCHE PHASE 2 & LES MAQUETTES

### Validation Formelle: 5 Points de Contrôle

#### **Point 1: Diagnostic Agreed (Yes/No)**

**Question**: L'équipe est-elle d'accord que les 4 problèmes UX listés reflètent bien les limites du module actuel?

```
❌ Barre d'actions surchargée (8 boutons sans hiérarchie)
❌ Espace statistiques envahissant (50% de l'écran)
❌ Étape 2 trop basique (juste des checkboxes)
❌ Pas de visualisation des passes (comment savoir en quelle passe on est?)
```

**Validation**:
- [ ] Chef de projet: Oui / Non
- [ ] Développeur: Oui / Non
- [ ] Responsable QA: Oui / Non
- [ ] Stakeholder métier: Oui / Non

---

#### **Point 2: Approach Approved (Yes/No)**

**Question**: L'approche Groupings System (3 phases) est-elle validée?

**What**: Transformer le module d'une sélection simple (checkboxes) à une gestion de regroupements multiples:
- Passe 1: 6°1 + 6°2 + 6°3 → 3 groupes
- Passe 2: 6°4 + 6°5 → 4 groupes

**How**: 3 phases:
1. ✅ Phase 1 (DONE): Data model + 24 utility functions
2. 🚧 Phase 2 (NEXT): UI redesign (Steps 2, 3, 5) - 18.5h
3. 🚧 Phase 3 (AFTER): Backend integration - 9h

**Why**: Résout tous les 4 problèmes UX listés ci-dessus.

**Validation**:
- [ ] Chef de projet: Oui / Non
- [ ] Développeur: Oui / Non
- [ ] Responsable QA: Oui / Non
- [ ] Stakeholder métier: Oui / Non

---

#### **Point 3: Mockups Reviewed (Yes/No)**

**Question**: Les maquettes des 3 étapes ont-elles été examinées et approuvées?

**Mockups à examiner** (dans ARCHITECTURE_GROUPINGS_REDESIGN.md Section 3):

**Étape 2 - Groupings Composer**:
```
Current: Simple checkboxes
Target: Grouping manager avec modal d'édition
```

**Étape 3 - Per-Grouping Configuration**:
```
Current: Single global config
Target: Tabs montrant config par regroupement
```

**Étape 5 - Groups Display & Action Bar**:
```
Current: 8 boutons mélangés, stats 50%, pas de contexte
Target: Tabs + 4 groupes d'actions + stats 33% + contexte groupement
```

**Validation**:
- [ ] Développeur frontend: Oui / Non (Compris les maquettes?)
- [ ] Stakeholder UX: Oui / Non (L'UX répond aux besoins?)
- [ ] Chef de projet: Oui / Non (Faisable en 18.5h?)

---

#### **Point 4: UX Requirements Aligned (Yes/No)**

**Question**: Les maquettes répondent-elles à TOUTES les corrections obligatoires listées?

**Checklist de couverture**:

| Correction Obligatoire | Implémentée à | Couverte? |
|------------------------|---|----------|
| Barre actions hiérarchisée | Step 5 | [ ] |
| Panel stats réduit (max 1/3) | Step 5 | [ ] |
| Regroupements composables | Step 2 | [ ] |
| Navigation entre passes | Step 2 + Step 5 | [ ] |
| Mode continuation visible | Step 5 bandeau + tabs | [ ] |
| Indicateurs pédagogiques (code groupe) | Group cards | [ ] |
| Persistance multi-passes | Phase 3 | [ ] |
| Finalisation non-destructive | Phase 3 | [ ] |

**Validation**:
- [ ] Stakeholder métier: Oui / Non (Toutes les corrections couvertes?)
- [ ] Responsable QA: Oui / Non (Critères de réception clairs?)

---

#### **Point 5: Resource Allocation Confirmed (Yes/No)**

**Question**: L'équipe a-t-elle 18,5 heures de développement disponibles?

**Répartition du travail**:
- Step 2 Redesign: 5h
- Step 3 Redesign: 3h
- Step 5 Redesign: 7h
- Stats Panel: 2h
- Group Cards: 1.5h

**Validation**:
- [ ] Développeur frontend: Disponible? (18.5h consécutives?)
- [ ] Chef de projet: Slots calendrier confirmés?
- [ ] Responsable QA: Slots test confirmés?

---

### Formulaire de Validation Formelle

**À signer avant démarrage Phase 2**:

```
╔════════════════════════════════════════════════════════════════╗
║          VALIDATION FORMELLE - PHASE 2 START GATE              ║
║                       (à signer)                               ║
╠════════════════════════════════════════════════════════════════╣
║                                                                 ║
║ Point 1: Diagnostic agreed?      Oui [ ]  Non [ ]  N/A [ ]    ║
║ Point 2: Approach approved?      Oui [ ]  Non [ ]  N/A [ ]    ║
║ Point 3: Mockups reviewed?       Oui [ ]  Non [ ]  N/A [ ]    ║
║ Point 4: UX requirements met?    Oui [ ]  Non [ ]  N/A [ ]    ║
║ Point 5: Resources allocated?    Oui [ ]  Non [ ]  N/A [ ]    ║
║                                                                 ║
║ DECISION: [ ] GO  [ ] NO-GO  [ ] HOLD                          ║
║                                                                 ║
║ Signatures (5):                                                 ║
║ Chef de projet: ________________  Date: __________             ║
║ Dev frontend: ________________  Date: __________               ║
║ Responsable QA: ________________  Date: __________             ║
║ Stakeholder métier: ________________  Date: __________         ║
║ Lead technique: ________________  Date: __________             ║
║                                                                 ║
╚════════════════════════════════════════════════════════════════╝
```

**Rules**:
- **GO**: Tous les 5 points = Oui → Démarrage Phase 2
- **NO-GO**: ≥1 point = Non → STOP, revoir le point bloquant
- **HOLD**: ≥1 point = N/A → Clarifier avant démarrage

---

## 💰 ÉTAPE 3: ALLOUER 18,5 H DE DÉVELOPPEMENT

### Feuille de Route Phase 2 (Développeur Frontend)

#### **Day 1: Step 2 Redesign (5h)**

```
Matin (2h30):
- [ ] Lire CORRECTIONS_OBLIGATOIRES.md Section 2.1
- [ ] Examiner mockup Step 2 dans ARCHITECTURE_GROUPINGS_REDESIGN.md
- [ ] Identifier l'UI existante à remplacer (renderStep2_SelectClasses)

- [ ] Créer fonction renderStep2_Groupings() remplaçant renderStep2_SelectClasses()
- [ ] Implémenter affichage de state.groupings[]
- [ ] Implémenter bouton "+ Nouveau Regroupement"

Après-midi (2h30):
- [ ] Créer modal d'édition/création de regroupement
- [ ] Implémenter CRUD (Create, Edit, Delete, Activate/Deactivate)
- [ ] Tester: créer 2+ regroupements, modifier, supprimer
- [ ] Valider: state.groupings[] updated correctement
```

**Expected Output**: Step 2 utilisateur peut composer plusieurs regroupements

---

#### **Day 2-3: Steps 3 & 5 Redesign (10h)**

##### **Day 2 Matin (3h): Step 3 Per-Grouping Config**

```
- [ ] Lire CORRECTIONS_OBLIGATOIRES.md Section 2.2
- [ ] Examiner mockup Step 3

- [ ] Ajouter tabs de regroupements à Step 3
- [ ] Charger config per-grouping via getEffectiveConfig()
- [ ] Implémenter setEffectiveConfig() au changement d'input
- [ ] Ajouter navigation Prev/Next entre regroupements

Test:
- [ ] Créer 2 regroupements avec configs différentes
- [ ] Vérifier configs stockées indépendamment
- [ ] Vérifier navigation entre tabs fonctionne
```

**Expected Output**: Step 3 = configuration indépendante par regroupement

---

##### **Day 2 Après-midi + Day 3 Matin (7h): Step 5 Complete Redesign**

```
SOUS-TÂCHE 1: Grouping Tabs (1.5h)
- [ ] Ajouter tabs montrant tous les regroupements
- [ ] Implémenter activateGrouping() au clic tab
- [ ] Mettre à jour groups grid dynamiquement au changement tab

SOUS-TÂCHE 2: Reorganiser Action Bar (2h)
- [ ] Créer 4 sections d'actions:
     ├─ CALCUL: Régénérer, Statistiques
     ├─ SAUVEGARDE: Mode selector + Temp + Finalize
     ├─ PLUS: Dropdown exports
     └─ NAVIGATION: Prev/Next regroupement
- [ ] Ajouter séparations visuelles claires
- [ ] Wirer les handlers d'actions

SOUS-TÂCHE 3: Resize Stats Panel (2h)
- [ ] Passer layout de 50%:50% à 66%:33% (groups:stats)
- [ ] Ajouter resize handle sur sidebar
- [ ] Tester scrolling quand overflow

SOUS-TÂCHE 4: Enhance Group Cards (1.5h)
- [ ] Ajouter header avec contexte groupement
- [ ] Afficher code groupe (grBe1, pas juste "Groupe 1")
- [ ] Montrer offset range (e.g., "grBe1-3")
- [ ] Ajouter indicators (F/M count, avg score)
```

**Expected Output**: Step 5 = interface nettoyée, claire, contextuelle

---

#### **Day 4: Testing & Refinement (3.5h)**

```
Matin (2h):
- [ ] Créer 2 regroupements distincts
- [ ] Configurer chacun différemment
- [ ] Générer des groupes pour chaque
- [ ] Vérifier pas de contamination croisée

Après-midi (1.5h):
- [ ] Tester workflow ancien (single-pass) = toujours OK
- [ ] Tester UI responsiveness
- [ ] Tester tab switching smooth
- [ ] Vérifier resize panel fonctionne
- [ ] Vérifier group cards affichent correctly

Final (0h30):
- [ ] Relire code pour qualité
- [ ] Vérifier pas de console errors
- [ ] Documenter changements majeurs
```

**Expected Output**: Phase 2 = 100% fonctionnelle, testée, ready pour Phase 3

---

### Calendrier Proposé

```
Semaine du [DATE]:
┌────────────────────────────────────────────────────────────────┐
│ Lundi 8h-17h:      Step 2 Redesign (5h)                        │
│                    + Review docs & planning (1h)               │
├────────────────────────────────────────────────────────────────┤
│ Mardi 8h-17h:      Step 3 Config (3h) + Step 5 Part 1 (4h)     │
├────────────────────────────────────────────────────────────────┤
│ Mercredi 8h-17h:   Step 5 Part 2 (6h) + Testing (2h)           │
├────────────────────────────────────────────────────────────────┤
│ Jeudi matin 8h-12h: Final testing & refinement (3.5h)          │
│ Jeudi après-midi:  Code review, documentation                 │
├────────────────────────────────────────────────────────────────┤
│ TOTAL: 18.5h effectifs + 1h planning = 19.5h                   │
│ Durée calendaire: 3.5 jours intensifs (Lun 8h à Jeu 12h)      │
└────────────────────────────────────────────────────────────────┘
```

### Allocations Requises

| Rôle | Tâche | Durée | Semaine |
|------|-------|-------|---------|
| **Dev Frontend** | Implémenter Phase 2 | 18.5h | Lun-Jeu matin |
| **Responsable QA** | Review code, valider | 4h | Jeudi-Vendredi |
| **Chef de projet** | Coordination, blockers | 2h | Slots quotidiens |
| **Stakeholder** | Feedback sur UI | 1h | Jeudi matin |

**Remarque**: Pas d'interruptions externes pendant Lun-Jeu. Phone/Slack silencieux.

---

## 🎯 GO/NO-GO Decision Matrix

### Si tous les points de validation = YES

```
✅ DECISION: GO FOR PHASE 2

Démarrage: [DATE CONFIRMÉE]
Fin prévue: [DATE + 3.5 jours]
Livrables: Step 2-5 complètement restructurés
Validation finale: Code review + test end-to-end
Next phase: Phase 3 (Backend integration) dans 2 semaines
```

### Si ≥1 point = NO

```
❌ DECISION: NO-GO - HOLD UNTIL RESOLUTION

Point bloquant: [NUMÉRO & DESCRIPTION]
Root cause: [ANALYSE]
Action corrective: [À FAIRE]
Date de re-review: [+ 2-3 jours]
Escalade: [SI NÉCESSAIRE]
```

### Si ≥1 point = N/A (Non-Applicable)

```
⚠️ DECISION: CONDITIONAL GO

Points à clarifier:
- [ ] [POINT N/A 1]
- [ ] [POINT N/A 2]

Clarification deadline: [DATE]
Review après clarification: [DATE + 1j]
```

---

## 📋 Checklist Finale (À Cocher Avant Démarrage)

### Équipe Complète

- [ ] Tous les documents requis lus (CORRECTIONS_OBLIGATOIRES.md, ARCHITECTURE...)
- [ ] Les 5 points de validation formels signés
- [ ] Decision = GO (tous les points = Oui)
- [ ] Développeur frontend 100% disponible (18.5h)
- [ ] Responsable QA slots réservés
- [ ] Calendrier verrouillé (pas de réunions autres)
- [ ] Environnement dev/test prêt
- [ ] Accès aux codes de base (groupsModuleComplete.html, etc.)

### Développeur Frontend Spécifiquement

- [ ] Lire CORRECTIONS_OBLIGATOIRES.md (30 min)
- [ ] Lire ARCHITECTURE_GROUPINGS_REDESIGN.md Section 3 (1h)
- [ ] Lire SPRINT_6_GROUPINGS_KICKOFF.md Phase 2 (30 min)
- [ ] Examiner mockups & structures de données
- [ ] Lister les 15+ utility functions disponibles (Phase 1)
- [ ] Identifier code existant à remplacer/augmenter
- [ ] Tester environment ready (local dev server running)
- [ ] Slack/Discord setup for daily standup

### Responsable QA

- [ ] Test plan Draft pour Phase 2 (multi-regroupements scenario)
- [ ] Environnement test prêt
- [ ] Données de test préparées (Google Sheets template)
- [ ] Test cases documentés pour chaque correction obligatoire
- [ ] Sign-off criteria définis

---

## 🚀 Démarrage Phase 2

### Réunion de Lancement (30 min)

**Agenda**:
1. Confirmation des décisions de validation (5 min)
2. Review rapide des maquettes (5 min)
3. Répartition des tâches (5 min)
4. Points de coordination (Slack, daily standup, code review) (5 min)
5. Q&A (5 min)

**Participants**: Dev, QA, Chef de projet, Lead technique

**Output**: Kickoff notes + calendar blocks locked

---

### Daily Standup (Lun-Jeu)

**Format**: 15 min, 9h30 ou 16h

**Questions**:
1. Qu'ai-je livré hier?
2. Qu'est-ce que je livre aujourd'hui?
3. Quels sont mes blockers?

**Owner**: Chef de projet

---

## 📞 Escalade & Support

### Si blockers pendant Phase 2

**Level 1: Developer self-service** (première tentative)
- Consulter CORRECTIONS_OBLIGATOIRES.md
- Consulter ARCHITECTURE_GROUPINGS_REDESIGN.md
- Vérifier utility functions disponibles (Phase 1)

**Level 2: Lead technique** (30 min) - Si Level 1 insufficient
- Code review
- Architecture clarification
- Refactoring advice

**Level 3: Chef de projet** (decision escalation) - Si blockers majeurs
- Peut-on réduire scope?
- Peut-on demander délai?
- Faut-il ressource supplémentaire?

---

## ✨ Signoff & Go-Ahead

```
╔════════════════════════════════════════════════════════════════╗
║                    ORDRE DE MARCHE SIGNÉ                       ║
║                                                                 ║
║ Phase 2 Autorisée:        [ ] Oui   [ ] Non                    ║
║ Date démarrage: _________________                              ║
║ Date fin prévue: _________________                             ║
║                                                                 ║
║ Approuvé par:                                                   ║
║ • Chef de Projet: ________________  Date: __________            ║
║ • Lead Technique: ________________  Date: __________            ║
║ • Director/Sponsor: ________________  Date: __________          ║
║                                                                 ║
╚════════════════════════════════════════════════════════════════╝
```

---

**Ordre de Marche Établi**: 29 octobre 2025
**Prêt pour Phase 2**: Après validation des 3 étapes ci-dessus

