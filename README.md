# TransportOptim

Application d'optimisation du transport personnel pour le site Tosyali Algérie.

## Fonctionnalités

- **Collecte GPS** : Page publique pour enregistrer sa position de ramassage
- **Gestion des employés** : CRUD, import CSV, filtres, recherche
- **Points de rassemblement** : Gestion + clustering spatial automatique
- **Véhicules** : Gestion de la flotte (bus, minibus, vans)
- **Optimisation des lignes** : Algorithme sweep + 2-opt avec respect des capacités et durées
- **Planning** : Cycles de travail 6×2 (Production) et 5×2 (Jour)
- **Statistiques** : Graphiques interactifs avec filtres par période (jour, semaine, mois)
- **Exports** : CSV, KML, Google Maps, feuilles de route PDF

## Stack

- React 18 + TypeScript + Vite 7
- Tailwind CSS 3
- Supabase (PostgreSQL + RLS)
- Lucide React (icônes)

## Installation

```bash
npm install
npm run dev
```

## Code d'accès par défaut

`admin` (modifiable dans Paramètres)

## Documentation

Voir [CAHIER_DES_CHARGES.md](CAHIER_DES_CHARGES.md) pour le cahier des charges complet.
