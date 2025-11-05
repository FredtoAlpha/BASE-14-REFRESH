# Module Groupes V4 - Documentation

## Vue d'ensemble

Le **Module Groupes V4** est une refonte complète du système de création de groupes, construit sur une architecture **triptyque** simple et modulaire. Il remplace les anciennes versions en offrant une interface claire, un algorithme robuste et une intégration transparente avec InterfaceV2.

## Architecture

Le module repose sur **3 fichiers principaux** :

### 1. `GroupsAlgorithmV4.js`
**Algorithme de répartition pur, sans UI**

- ✅ Mode **hétérogène** : mélange les niveaux (stratégie serpentin)
- ✅ Mode **homogène** : équilibre les niveaux (stratégie round-robin)
- ✅ Normalisation des scores entre 0 et 1
- ✅ Gestion de la parité F/M
- ✅ Statistiques détaillées par groupe

**API publique :**
```javascript
window.GroupsAlgorithmV4.distribute({
  students: [...],      // Tableau d'élèves
  mode: 'heterogeneous', // 'heterogeneous' | 'homogeneous'
  numGroups: 3,         // Nombre de groupes à créer
  scenario: 'besoins'   // 'besoins' | 'lv2' | 'options'
})
```

### 2. `GroupsInterfaceV4.js`
**Interface triptyque en vanilla JS (30/40/30)**

Structure en 3 volets :
- **Volet A (30%)** : Paramètres (scénario + mode de distribution)
- **Volet B (40%)** : Regroupements configurables (cartes avec actions)
- **Volet C (30%)** : Récapitulatif (statistiques + timeline + aperçu)

**Événements émis :**
- `groups:generate` : demande de génération avec config complète
- `groups:generated` : résultats disponibles (écoute)
- `groups:error` : erreur rencontrée (écoute)

**API publique :**
```javascript
window.GroupsInterfaceV4.renderInitialStructure(containerElement)
window.GroupsInterfaceV4.displayResults(results)
window.GroupsInterfaceV4.displayError(error)
```

### 3. `InterfaceV2_GroupsModuleV4_Script.html`
**Loader minimal qui connecte tout**

- ✅ Récupère les données depuis `window.STATE.students`
- ✅ Remplace `window.openGroupsInterface()` pour intercepter les clics sur le bouton header
- ✅ Crée le container plein écran avec bouton de fermeture
- ✅ Écoute les événements et fait le pont avec l'algorithme
- ✅ Affiche les résultats dans l'interface

## Intégration dans InterfaceV2

Le module est inclus via les balises `<?!= include(...) ?>` dans `InterfaceV2.html` :

```html
<!-- ========== MODULE GROUPES V4 - TRIPTYQUE (NOUVEAU) ========== -->
<?!= include('GroupsAlgorithmV4'); ?>
<?!= include('GroupsInterfaceV4'); ?>
<?!= include('InterfaceV2_GroupsModuleV4_Script'); ?>
```

**Ordre d'inclusion critique :** Algorithme → Interface → Loader

## Flux de données

```
1. USER clique sur "Groupes" dans le header
   ↓
2. openGroupsInterface('creator') intercepté par le loader
   ↓
3. Loader récupère les données depuis window.STATE.students
   ↓
4. Interface triptyque affichée en plein écran
   ↓
5. USER configure les paramètres (scénario, mode, regroupements)
   ↓
6. USER clique sur "Générer"
   ↓
7. Événement groups:generate émis avec la config
   ↓
8. Loader écoute l'événement et appelle l'algorithme
   ↓
9. Algorithme calcule les groupes et les statistiques
   ↓
10. Événement groups:generated émis avec les résultats
    ↓
11. Interface affiche les résultats dans le volet C
```

## Principes de conception

### ✅ SIMPLE
- Vanilla JS uniquement (pas de librairie externe)
- 3 fichiers clairement séparés
- API publique minimale et documentée

### ✅ MODULAIRE
- Chaque fichier a une responsabilité unique
- Couplage faible via événements
- Facile à tester et maintenir

### ✅ ROBUSTE
- Validation des données à l'entrée
- Refus systématique des données factices
- Messages d'erreur explicites dans la console

### ✅ INTÉGRÉ
- Connexion transparente au bouton header existant
- Utilise les données réelles de `window.STATE`
- Compatible avec l'architecture InterfaceV2

## Données attendues

Le module attend des données dans `window.STATE.students` au format :

```javascript
{
  1: {
    id: 1,
    firstName: "Alice",
    lastName: "Martin",
    class: "6°1",
    gender: "F",
    score: 3,        // Score 1-4 pour les besoins
    lv2: "Espagnol", // LV2 optionnelle
    options: []      // Options spécifiques
  },
  2: { ... },
  // etc.
}
```

Si aucune donnée n'est disponible, le module **bloque l'ouverture** et affiche un message explicite dans la console avec le diagnostic.

## Test en local

Un fichier de test autonome est fourni : `GroupsModuleV4_Test.html`

**Pour tester localement :**
1. Ouvrir `GroupsModuleV4_Test.html` dans un navigateur
2. Cliquer sur "Charger données de test" pour générer des élèves fictifs
3. Cliquer sur "Ouvrir le module" pour afficher l'interface triptyque
4. Tester les fonctionnalités (scénario, mode, regroupements, génération)
5. Observer les logs dans la console en bas de page

## État du projet

### ✅ Fonctionnalités complètes (Base 8)
- [x] Algorithme hétérogène et homogène
- [x] Interface triptyque 30/40/30
- [x] Gestion d'état centralisée
- [x] Événements groups:generate / groups:generated / groups:error
- [x] Connexion au bouton header
- [x] Validation des données
- [x] Timeline des actions
- [x] Diagnostics intégrés

### 📋 Fonctionnalités planifiées (non bloquantes)
- [ ] Swap manuel d'élèves entre groupes (drag & drop)
- [ ] Sauvegarde des regroupements en brouillon
- [ ] Export CSV/PDF des groupes générés
- [ ] Raccourcis clavier pour actions rapides
- [ ] Carrousel détaillé des groupes dans le volet C

## Migration depuis l'ancienne version

Les anciens modules `groupsModuleComplete` et `InterfaceV2_GroupsScript` ont été **désactivés** (commentés) dans InterfaceV2.html pour éviter les conflits.

Si besoin de revenir à l'ancienne version :
1. Décommenter les anciennes inclusions
2. Commenter les inclusions V4
3. Recharger la page

## Support et débogage

En cas de problème :

1. **Ouvrir la console navigateur** (F12)
2. Rechercher les messages préfixés :
   - `🔌` : Initialisation du loader
   - `✅` : Opération réussie
   - `❌` : Erreur détectée
   - `📊` : Événement reçu
   - `🚀` : Génération lancée

3. Vérifier les diagnostics automatiques :
   - Présence de `window.STATE`
   - Nombre d'élèves dans `STATE.students`
   - Chargement des modules (GroupsAlgorithmV4, GroupsInterfaceV4)

## Maintenance

Pour modifier le module :
- **Algorithme** : Éditer `GroupsAlgorithmV4.js`
- **Interface** : Éditer `GroupsInterfaceV4.js`
- **Connexion** : Éditer `InterfaceV2_GroupsModuleV4_Script.html`

**Règle d'or** : Ne jamais modifier les 3 fichiers en même temps. Tester après chaque modification.

## Licence

Ce module fait partie du projet BASE-14-REFRESH.

---

**Dernière mise à jour** : 2025-01-05
**Version** : 4.0
**Auteur** : Claude (Anthropic) via claude/groups-module-v4-triptyque
