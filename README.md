### TL:DR

#### Contexte:
Ce projet a été conçu comme une démonstration de Clean Architecture et de Domain-Driven Design (DDD) appliqués à la gestion d’un concessionnaire moto (Projet de cours 5ème ESGI). Il illustre la structuration d’une application complexe en sous-domaines, l’usage de CQRS, Event Sourcing et la séparation claire entre domaine, application et infrastructure.

#### Skills:
- Architecture logicielle avancée (Clean Architecture, DDD, CQRS, Event Sourcing)
- TypeScript avancé et typage fort
- API REST avec Express et application web avec Next.js
- Gestion des bases de données multiples (MongoDB + in-memory)
- Mise en place de tests unitaires et CI via GitHub Actions
- Implémentation de patterns fonctionnels (Result Pattern) et pratiques


#### Architecture du projet :
- **InventoryManagement** : gestion du stock et des commandes
- **Maintenance** : suivi des maintenances et notifications
- **TestDrive** : planification et suivi des essais de véhicules

Chaque sous-domaine respecte la Clean Architecture et peut être isolé pour microservices.

-----
# Présentation

> Node Version : 20 minimum

## Fonctionnalités Importantes :

- Gestions des véhicules et des pannes
- Gestion des maintenances + Notifications maintenance (via cron et commandes CLI)
- Gestions des clients et des conducteurs
- Gestions des essais et des incidents lors des essais
- Gestions des stocks concessionnaires et commandes de pièces + Notifications stock bas
- Upload de fichiers
- Commandes CLI
- API REST express
- Application Next JS
- Tests du domaine (sous domaine testDrive et inventoryManagement pour exemple) 

## Installation

Installer les dépendances de chaque framework : 
```bash
npm run install:all
```

## Lancements

API Express : (port 3000)
```bash
npm run dev:express
```

Ou

Application Next JS : (port 8080)
```bash
npm run dev:next
```

Base de données Mongo
```bash
docker compose up
```

## Tests et build

Build typescript (sans next js):
```bash
npm run build
```

Tests unitaires :
```bash
npm run test
```

Tests linter avec eslint:
```bash
npm run lint
```

> Note : Ces trois tests sont lancés automatiquement lors d'un push sur chaque Pull Request et par commits sur la branche `main` via GitHub Actions.


## Séparation en sous-domaines

Nous avons séparé notre application en trois sous-domaines afin de pouvoir, par la suite, les séparer en microservices :
- **InventoryManagement** : Permet de gérer les pièces, les commandes et le stock des concessionnaires.
- **Maintenance** : Permet de gérer les maintenances et les suivis des véhicules.
- **TestDrive** : Permet de planifier et suivre les essais des véhicules.

Toute notre application respecte cette séparation dans chacune des couches de la Clean Architecture.

## Event Sourcing + CQRS

- **Event dans le domaine**
- **Projection dans l'application**
    - **Projection Worker & Projection Job** : Permet de gérer les projections en arrière-plan et de reprendre en cas de panne temporaire du worker (le read et le write sont séparés).
    - Un **Event** peut créer plusieurs **ProjectionJob**.
    - La configuration de la projection est facilitée par l’abstraction de la logique de projection.
- Un **repository Event** et un **repository ProjectionJob**.
- Une implémentation de **EventRepository** et **ProjectionJobRepository** par sous-domaine (trois implémentations et trois collections MongoDB en conséquence).
- Le **worker** pourrait être amélioré, car il effectue du polling pour détecter de nouveaux **ProjectionJob** à traiter.

## Domaine

- **Constructeur en privé** -> Permet de valider la logique métier en amont.
- **Aucune mutation des objets, tout est immutable** -> Recréation d’un objet à chaque modification.
- **Les events sont instanciés par l’entité elle-même** -> Accès direct aux propriétés privées / maintien plus simple (un seul endroit à modifier).
- **Tout renvoie des `ApplicationException` avec un identifiant unique**, ce qui permet de comparer les erreurs (cas des tests, par exemple).
- **Mise en place de tests Jest** pour tester unitairement le domaine.
- **Utilisation des DTO** (objets sérialisables en JSON) pour servir d’interface entre le domaine et une éventuelle base de données.
- **Shared** correspond à des classes/objets métier partagés entre plusieurs sous-domaines, comme les adresses ou les SIRET.

## Application

- **UseCase** -> Paradigme fonctionnel, fonctions d’ordre supérieur, permettant de limiter la base de code et d’améliorer sa lisibilité.
- **Repository** -> Abstraction de la logique de persistance, permettant de changer de base de données sans modifier le code métier.
- **Services (si applicable)** -> Abstraction des services interopérables utilisés par nos `UseCase` (ex: `NotificationService`, `FileUploadService`).
- **Projections** -> Logique de matérialisation des données, centralisant toutes les logiques post-event des sous-domaines.
- **Commands** -> Permet d’interagir avec l’application via CLI.

### Result Pattern

L’ensemble de l’application utilise le **Result Pattern** pour la majorité des échanges entre Application et Infrastructure.  
Avec **TypeScript**, nous avons profité d’un typage fort sur nos `UseCase` et `Repository` pour faciliter l’implémentation et la prédiction du comportement de l’application.

Il existe plusieurs types de résultats :
- `SuccessResult<T>` : Attend une valeur de type `T`, sans erreur.
- `FailureResult` : Contient une `ApplicationException` avec un message et un identifiant unique en cas d’erreur.
- `VoidResult` : Aucune valeur de retour, sans erreur.
- `PaginatedResult<T>` : Retourne une liste de valeurs de type `T`, avec les informations de pagination (`page`, `limit`, `total`).

Ensuite, il existe des types d’encapsulation de ces résultats :
- `Result<T> : SuccessResult<T> | FailureResult` -> Si `success` est vrai, on a un `SuccessResult<T>`, sinon un `FailureResult`.
- `ResultVoid : VoidResult | FailureResult` -> Si `success` est vrai, on a un `VoidResult`, sinon un `FailureResult`.
- `ResultPaginated<T> : PaginatedResult<T> | FailureResult` -> Si `success` est vrai, on a un `PaginatedResult<T>`, sinon un `FailureResult`.
- `OptionalResult<T> : SuccessResult<T> | VoidResult | FailureResult` -> Si `success` est vrai et `empty` est faux, on a un `SuccessResult<T>`, si `empty` est vrai, on a un `VoidResult`, sinon un `FailureResult`.

✅ **Fonctionne bien avec le paradigme fonctionnel**, permet d’enchaîner les résultats et de les manipuler facilement.  
✅ **Simplifie la gestion des erreurs** et permet de prédire le comportement de l’application.

## Infrastructure


- **common** : Implémentation des abstractions applicatives (`Repository`, `Services`)
    - **Repository** : Deux implémentations, une en mémoire et une en MongoDB.
    - **InMemory** : Objet `InMemoryDataCollection`, un tableau amélioré pour simplifier l’implémentation des repositories InMemory.
    - **MongoDB** : Utilisation de MongoDB sans Mongoose.

- **Core**
    - Instanciation de toutes les classes applicatives utilisées dans nos implémentations framework.
    - Sélection des implémentations de `common` et choix de celles qui seront utilisées.
    - Planification des projections et sélection des adaptateurs de `Repository`.
    - Spécification des **requests** sous forme de schémas **ZOD**, utilisés par **Next.js** et **Express**.
    - **UseCaseImplementation** : Passerelle entre notre application et notre infrastructure.
        - Prend en entrée des inputs parsés par nos `requests` (objets bruts).
        - Crée les objets métier nécessaires pour les `UseCase`.
        - Utilisé à la fois par **Next.js** et **Express**.

- **Express**
    - Les `controllers` sont directement nos `UseCaseImplementation`.
    - Enregistrement des routes sous la forme `METHOD(PATH, USECASE_IMPLEMENTATION, REQUEST)`.
    - Génération dynamique d’une **collection Postman** pour tester notre API en utilisant TypeScript pour générer la spécification des paramètres et payloads.

- **Next.js**
    - Les `UseCaseImplementation` prennent en entrée les payloads bruts définis dans `requests`.
    - Gestion des requêtes selon les règles de Next.js (**server actions**, **server components**).

- **cli** : Contient l'infrastructure pour le montage des commandes CLI ainsi que toutes les commandes executable via la commande npm run command -- [commande], permet d'intéragir via CLI avec l'application


Cette approche via **UseCaseImplementation** nous permet de séparer notre framework de notre application avec un cran supplémentaire.  
✅ **Évite la duplication de code**.  
✅ **Protège l’entrée avec des `requests` bien spécifiques**.  
✅ **Permet, dans le cas d’une API, de générer dynamiquement une documentation Postman** pour tester l’application.  
