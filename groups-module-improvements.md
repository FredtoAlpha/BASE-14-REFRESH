# Propositions d'Amélioration pour le Module de Création des Groupes

## 1. Spécialisation par type de groupe
- Filtrer les élèves selon la langue vivante (LV2) lorsque le type de groupe est « langue » afin de garantir la cohérence pédagogique.
- Introduire un traitement dédié aux options en s'appuyant sur `student.opt` pour préparer un futur module spécifique.
- Affiner la dénomination des groupes en fonction du type choisi pour clarifier les exports et la lecture des onglets.

## 2. Pondération multi-critères et optimisation
- Construire un score composite configurable (notes, participation, assiduité) afin de mieux équilibrer les groupes.
- Mettre en place un algorithme glouton avec échanges successifs pour réduire l'écart-type des indicateurs clés.
- Exposer dans l'interface un tableau de bord des écarts (notes moyennes, parité, participation) pour valider l'équilibre obtenu.

## 3. Gestion des contraintes structurelles
- Contrôler la répartition des élèves par classe d'origine pour éviter les regroupements excessifs.
- Autoriser des effectifs cibles différents par groupe avec un delta maximal paramétrable (±1) pour couvrir les cas particuliers.
- Prévoir un mécanisme de verrouillage d'élèves (placements fixes) avant génération pour satisfaire des contraintes pédagogiques.

## 4. Fiabilisation de la sauvegarde et du suivi
- Remplacer la suppression basée sur la regex `^\w+\d+$` par un filtrage explicite sur le préfixe d'onglet pour éviter les résidus.
- Enregistrer la date de sauvegarde effective (propriété du document ou cellule masquée) et l'afficher dans le tableau de bord.
- Journaliser les versions de sauvegarde (identifiant, utilisateur, horodatage) pour tracer les modifications et faciliter le retour arrière.

## 5. Améliorations de l'expérience utilisateur
- Ajouter une prévisualisation PDF et proposer un export CSV enrichi (COM, TRA, PART, ABS) pour couvrir différents usages.
- Permettre la comparaison côte à côte entre la version courante et une sauvegarde antérieure avant finalisation.
- Intégrer des alertes guidées (par exemple, « parité déséquilibrée ») pour informer l'utilisateur des anomalies détectées.
