# Cahier des charges — TransportOptim

## Application d'optimisation du transport personnel pour le site Tosyali Algérie

Dernière mise à jour : 3 novembre 2026

---

## 1. Contexte et objectifs

### 1.1 Contexte

L'usine Tosyali (complexes sidérurgiques) située près d'Oran emploie plusieurs centaines de travailleurs répartis sur les wilayas d'Oran, Mostaganem et Mascara. Le transport du personnel vers le site industriel représente un défi logistique majeur :

- Multiplicité des points de ramassage (domiciles, arrêts de bus, points de rassemblement)
- Flotte hétérogène de véhicules (bus 29 places, minibus, vans)
- Cycles de travail complexes (Production 6×2, Jour 5×2) avec tranches horaires (nuit/jour/soir)
- Nécessité de minimiser le nombre de bus, la distance parcourue et le temps de trajet

### 1.2 Objectifs

L'application **TransportOptim** vise à :

1. **Collecter** les positions GPS des employés via une page publique accessible depuis un smartphone
2. **Gérer** les données employés, points de rassemblement, véhicules et groupes de travail, réservées aux administrateurs
3. **Optimiser** les lignes de bus en respectant les capacités, durées maximales et horaires d'arrivée
4. **Planifier** les équipes selon les cycles de travail (6×2 et 5×2)
5. **Projeter** les lignes optimisées sur un mois pour chaque tranche (jour, soir, nuit)
6. **Exporter** les résultats (CSV, KML, Google Maps, feuilles de route PDF)
7. **Visualiser** les statistiques avec des graphiques interactifs et filtres temporels

---

## 2. Périmètre fonctionnel

### 2.1 Page publique de collecte (accessible sans authentification)

Seule la collecte GPS est publique. Toutes les autres pages exigent une connexion.

| Fonctionnalité | Description |
|---|---|
| Saisie du badge | L'employé entre son numéro de badge (ex: EMP001) |
| Capture GPS | Géolocalisation automatique via l'API du navigateur, ou saisie manuelle des coordonnées |
| Numéro de bus | Champ facultatif pour indiquer le bus habituel |
| Validation de zone | Vérification que la position est dans la zone couverte (Oran, Mostaganem, Mascara) |
| Lien Google Maps | Affichage d'un lien vers Google Maps pour vérifier sa position |
| Distance au site | Calcul de la distance à vol d'oiseau vers le site Tosyali |
| Détection de doublon | Alerte si le badge a déjà enregistré un point |

### 2.2 Espace de gestion (connexion obligatoire)

#### 2.2.0 Accès et comptes

| Fonctionnalité | Détail |
|---|---|
| Premier administrateur | Créé à l'installation si aucun compte administrateur n'existe |
| Connexion | Adresse e-mail et mot de passe (Supabase Auth) |
| Rôles | `admin` (accès complet) et `employe` |
| Gestion des comptes | Création et suppression de comptes réservées aux administrateurs, via une fonction serveur sécurisée |
| Garde-fous | Un administrateur ne peut pas supprimer son propre compte |

#### 2.2.1 Tableau de bord

- KPIs : employés actifs, bus disponibles, repères actifs, taux de collecte
- Indicateurs de qualité des données (sans GPS, doublons, collectes enregistrées)
- Accès rapide aux principales sections

#### 2.2.2 Gestion des employés

| Fonctionnalité | Détail |
|---|---|
| CRUD complet | Ajouter, modifier, supprimer des employés |
| Import en masse | Fichier CSV ou collage de texte, avec détection automatique du séparateur (; ou ,), modèle téléchargeable et format affiché. Colonnes : matricule, nom, adresse, gps, arret_min, groupe, wilaya, daira, commune. Les groupes absents sont créés automatiquement et chaque employé est rattaché à son groupe. Les lignes en erreur sont listées sans bloquer les autres |
| Champs gérés | Badge, nom, adresse, wilaya, daïra, commune, GPS, point de rassemblement, bus habituel, durée d'arrêt, groupe, téléphone, statut actif |
| Filtres | Par groupe, recherche textuelle, localisation (wilaya, daïra, commune) |
| Filtres de localisation | Listes dépendantes : la daïra se limite à la wilaya choisie, la commune à la daïra |
| Sélection | Sélection individuelle ou de tous les employés affichés |
| Actions groupées | Affectation de la sélection à un groupe (ou « Sans groupe »), suppression de la sélection avec confirmation. La suppression est envoyée par lots pour supporter de grandes sélections |
| Export | CSV de la liste filtrée, PDF de la liste filtrée |

#### 2.2.3 Gestion des repères (points de rassemblement)

| Fonctionnalité | Détail |
|---|---|
| CRUD complet | Ajouter, modifier, supprimer des repères |
| Champs | Nom, adresse, wilaya, daïra, commune, GPS, rayon de marche, région, type (arrêt, parking, etc.) |
| Import en masse | CSV avec modèle téléchargeable. Colonnes : nom, adresse, gps, rayon_m, region, wilaya, daira, commune |
| Sélection | Sélection individuelle ou de tous les repères affichés |
| Actions groupées | Suppression de la sélection avec confirmation |
| Filtre | Par région, recherche textuelle |
| Regroupement automatique | Algorithme de clustering par grille spatiale + sélection gloutonne du centroïde le plus proche |
| Statistiques | Effectif par repère, distance moyenne et maximale de marche |

#### 2.2.4 Gestion des véhicules

| Fonctionnalité | Détail |
|---|---|
| CRUD | Ajouter, modifier, supprimer des véhicules |
| Champs | Nom, capacité, nombre disponible, couleur (pour distinguer les lignes), numéro de bus |
| Types | Bus 29 places, minibus, van |

#### 2.2.5 Lignes & Optimisation

| Fonctionnalité | Détail |
|---|---|
| Algorithme | Sweep (tri par angle) + plus proche voisin + 2-opt |
| Paramètres | Heure d'arrivée au site, capacité bus, durée max de trajet, vitesse moyenne, coefficient de détour, stratégie (minimiser bus ou remplissage uniforme) |
| Tranches | Nuit, Jour, Soir — avec heures d'arrivée distinctes |
| Contraintes | Capacité du véhicule, durée maximale de trajet, regroupement par point de rassemblement, nombre de véhicules disponibles par type |
| Visualisation | Liste des lignes avec arrêts ordonnés, horaires calculés, distance, durée, remplissage |
| Non-attribués | Liste des employés non affectés avec raison (capacité insuffisante, trajet trop long) |
| Exports | CSV, KML (Google Earth), liens Google Maps, feuilles de route PDF (impression) |

#### 2.2.6 Projection mensuelle

| Fonctionnalité | Détail |
|---|---|
| Sélection | Choix d'un mois (mois courant par défaut) |
| Calcul | Pour chaque jour du mois et chaque tranche (jour, soir, nuit), optimisation des lignes avec les employés actifs dont le groupe travaille sur cette tranche |
| Synthèse par tranche | Effectif, nombre de lignes, passagers, kilomètres, durée, non-attribués |
| Détail jour par jour | Tableau des indicateurs par date et par tranche |
| Exports | CSV de la synthèse par tranche et jour, CSV du détail des lignes projetées |
| Calcul | Effectué à la demande, sans enregistrement en base |

#### 2.2.7 Planning des équipes

| Fonctionnalité | Détail |
|---|---|
| Cycle 6×2 Production | 8 jours, 4 groupes en rotation (Repos → Nuit → Jour → Soir) |
| Cycle 5×2 Jour | 7 jours, travail Dimanche→Jeudi, repos Vendredi→Samedi |
| Calendrier | Navigation par semaine, codes couleurs par poste (Repos/Nuit/Jour/Soir) |
| Groupes | Création, modification du nom et des jours de repos (date de référence), décalage de jours, couleur, suppression |
| Filtres de localisation | Affichage limité aux groupes ayant des employés dans la wilaya, la daïra ou la commune choisie, avec le nombre d'employés concernés. Un tableau sous le calendrier liste ces employés avec leur groupe, leur poste du jour et leur localisation |
| Calcul du poste | Compte les jours calendaires à partir de la date de référence, sans décalage lié au fuseau horaire |
| Heures d'arrivée | Par tranche : Nuit 23:30, Jour 07:45, Soir 15:30 |

#### 2.2.8 Statistiques

| Fonctionnalité | Détail |
|---|---|
| Sélecteur de période | Jour, 7 jours, 30 jours, Tout |
| Cartes de synthèse | Effectif total, avec GPS, bus disponibles, collectes sur la période |
| Graphique de tendance | Évolution des collectes jour par jour (courbe + barres) |
| Graphique en barres | Répartition des employés par wilaya |
| Graphique en anneau | Couverture GPS (avec / sans GPS) |
| Graphique en barres horizontales | Top 10 points de rassemblement |
| Graphique de remplissage | Répartition par ligne (passagers vs capacité) |
| Barres de progression | Avancement de la collecte GPS et terrain |
| Alertes | Employés sans GPS, collectes manquantes, période sans collecte |

#### 2.2.9 Paramètres

| Paramètre | Détail |
|---|---|
| Site Tosyali | Latitude, longitude, nom |
| Capacités | Capacité bus par défaut |
| Durée max trajet | En minutes |
| Vitesse moyenne | En km/h |
| Coefficient de détour | Ratio de distance réelle vs à vol d'oiseau |
| Stratégie d'optimisation | Minimiser le nombre de bus ou remplissage uniforme |
| Heures d'arrivée | Par tranche (nuit, jour, soir) |
| Zone géographique | Limites nord/sud/est/ouest pour validation GPS |
| Comptes | Gestion des comptes utilisateurs (administrateurs uniquement) |

---

## 3. Architecture technique

### 3.1 Stack

| Composant | Technologie |
|---|---|
| Frontend | React + TypeScript + Vite |
| Styles | Tailwind CSS |
| Icônes | Lucide React |
| Backend | Supabase (PostgreSQL + Auth + API REST + fonctions serveur) |
| Base de données | PostgreSQL (Supabase) avec RLS (Row Level Security) |
| Fonctions serveur | Edge functions Supabase (gestion des comptes, premier administrateur) |
| Persistance locale | localStorage (repli si Supabase indisponible) |

### 3.2 Structure du projet

```
src/
├── App.tsx                  # Routing, auth, layout
├── types.ts                 # Types TypeScript (Employee, Repere, Ligne, etc.)
├── components/
│   ├── GpsMapPicker.tsx     # Sélection GPS sur carte
│   └── LocationFilterBar.tsx # Filtres wilaya / daïra / commune
├── lib/
│   ├── auth.ts              # Connexion, comptes (via fonction serveur)
│   ├── geo.ts               # Haversine, parseCoord (DMS, Plus Codes, Maps links), validation de zone
│   ├── storage.ts           # Couche d'accès données (Supabase + localStorage)
│   ├── location.ts          # Filtres de localisation et options dépendantes
│   ├── optimization.ts      # Sweep + nearest neighbor + 2-opt
│   ├── projection.ts        # Projection mensuelle par tranche
│   ├── clustering.ts        # Regroupement spatial par grille
│   ├── planning.ts          # Cycles 6×2 et 5×2
│   ├── import.ts            # Import CSV (détection séparateur)
│   ├── export.ts            # CSV, KML, Google Maps, PDF (feuilles de route)
│   └── cleaning.ts          # Nettoyage et déduplication des données
├── pages/
│   ├── CollectePage.tsx     # Page publique de collecte GPS
│   ├── DashboardPage.tsx    # Tableau de bord
│   ├── EmployeesPage.tsx    # Gestion des employés
│   ├── ReperesPage.tsx      # Gestion des repères
│   ├── VehiculesPage.tsx    # Gestion des véhicules
│   ├── LignesPage.tsx       # Optimisation des lignes
│   ├── ProjectionPage.tsx   # Projection mensuelle
│   ├── PlanningPage.tsx     # Planning des équipes
│   ├── StatistiquesPage.tsx # Statistiques avec graphiques
│   └── ParametresPage.tsx   # Paramètres et comptes
supabase/
├── functions/
│   ├── create-first-admin/  # Création du premier administrateur
│   └── manage-users/        # Création et suppression de comptes (administrateurs)
└── migrations/              # Schéma, RLS et politiques d'accès
```

### 3.3 Base de données

| Table | Description |
|---|---|
| `employees` | Employés (badge, nom, GPS, wilaya, daïra, commune, groupe, etc.) |
| `collectes` | Collectes terrain (badge, GPS, horodatage, bus) |
| `reperes` | Points de rassemblement (nom, GPS, rayon, région, type) |
| `vehicules` | Véhicules (nom, capacité, nombre disponible, couleur) |
| `groupes` | Groupes de travail (cycle, décalage, date de référence, couleur) |
| `profiles` | Profils des comptes (nom, e-mail, rôle `admin` ou `employe`, statut) |
| `lignes` | Lignes calculées (véhicule, horaires, tranche) |
| `parametres` | Paramètres (clé-valeur) |
| `wilayas_ref` | Référentiel géographique wilayas/dairas/communes |
| `arrets_ligne` | Arrêts des lignes |

Toutes les tables ont RLS activé. Les politiques sont détaillées dans la section 5.

### 3.4 Algorithmes

#### Optimisation des lignes (sweep + 2-opt)

1. **Regroupement** : Les employés sont regroupés par point de rassemblement
2. **Tri par angle** : Les arrêts sont triés par angle autour du site (algorithme sweep)
3. **Assignation gloutonne** : Chaque véhicule est rempli jusqu'à sa capacité ou la durée maximale ; le nombre de véhicules disponibles par type est suivi explicitement. Un arrêt dépassant la capacité d'un véhicule est signalé comme non attribué
4. **2-opt** : Amélioration locale de l'ordre des arrêts en inversant des segments pour réduire la distance
5. **Calcul des horaires** : Départ calculé en remontant depuis l'heure d'arrivée au site

#### Projection mensuelle

1. Pour chaque jour du mois, le poste de chaque groupe (jour, soir, nuit ou repos) est déterminé par le cycle et la date de référence
2. Pour chaque tranche, seuls les employés actifs, géolocalisés et affectés à un groupe en poste sont retenus
3. L'optimisation des lignes est exécutée pour chaque couple (jour, tranche) et les indicateurs sont agrégés

#### Clustering spatial

1. **Grille** : Division de l'espace en cellules de taille `marcheMaxM / 111000` degrés
2. **Sélection gloutonne** : Tri des cellules par densité, sélection du centroïde le plus proche
3. **Attribution** : Points dans un rayon `marcheMaxM` (700m par défaut) attribués au repère
4. **Points restants** : Rattachés au repère le plus proche

---

## 4. Données de test

La page de collecte peut s'appuyer sur un jeu de 15 employés fictifs positionnés autour d'Oran, utilisable pour simuler des collectes et alimenter les statistiques. Ces données ne doivent pas être utilisées pour les besoins réels.

---

## 5. Sécurité

- **RLS activé** sur toutes les tables Supabase
- **Collecte GPS publique** : seules les tables nécessaires à la collecte (`collectes`, lecture des `reperes`) sont accessibles sans connexion
- **Données de gestion réservées aux administrateurs** : employés, repères, véhicules, groupes, lignes, paramètres et référentiel géographique ne sont lisibles et modifiables que par un administrateur connecté
- **Fonctions d'administration protégées** : la gestion des comptes vérifie l'identité et le rôle de l'appelant côté serveur avant toute action ; la clé d'administration n'est jamais exposée au navigateur
- **Pas de compte par défaut** : aucun mot de passe administrateur n'est écrit dans le code ; le premier compte est créé lors de l'installation
- **Validation de zone GPS** pour les collectes terrain

---

## 6. Compatibilité et responsive

- Application responsive (mobile, tablette, desktop)
- Optimisée pour l'accès mobile (page de collecte utilisée sur smartphone)
- Breakpoints Tailwind : sm (640px), md (768px), lg (1024px)
- Navigation par menu latéral sur desktop, menu coulissant sur mobile

---

## 7. Évolutions futures possibles

- Région renseignée directement sur les employés, pour un filtrage par région
- Enregistrement des projections mensuelles pour comparaison entre versions
- Authentification des employés par badge QR code
- Notifications push pour rappels de collecte
- Intégration temps réel avec GPS des bus
- Optimisation multi-objectifs (minimiser CO₂, équité de temps de trajet)
- Mode hors-ligne complet avec synchronisation différée
- Tableau de bord temps réel des bus en circulation
