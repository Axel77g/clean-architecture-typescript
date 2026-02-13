# Clean Architecture & DDD — Plateforme de Gestion de Concessionnaire Moto

> **Démonstration pratique** d'une architecture logicielle de niveau production, appliquant les principes de Clean Architecture, Domain-Driven Design, CQRS et Event Sourcing sur une application TypeScript full-stack.

![Node Version](https://img.shields.io/badge/node-%3E%3D20.0.0-brightgreen)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)

---

## 📋 Contexte & Objectif

### Problématique
Comment structurer une application complexe pour assurer **maintenabilité**, **évolutivité** et **testabilité** tout en préparant une éventuelle migration vers une architecture microservices ?

### Solution mise en œuvre
Ce projet démontre l'implémentation d'une plateforme de gestion de concessionnaire moto organisée autour de **3 sous-domaines métier indépendants**, chacun respectant strictement la Clean Architecture et prêt à être isolé en microservice.

**Contexte projet** : Développé dans le cadre d'un cursus 5ème année ESGI comme cas d'étude avancé en architecture logicielle.

---

## 💼 Compétences Démontrées

### Architecture & Patterns
- ✅ **Clean Architecture** — Séparation stricte Domaine / Application / Infrastructure
- ✅ **Domain-Driven Design (DDD)** — Modélisation riche du domaine avec Value Objects, Entities, Aggregates
- ✅ **CQRS & Event Sourcing** — Séparation lecture/écriture avec système d'événements et projections asynchrones
- ✅ **Microservices-ready** — Architecture en sous-domaines isolés et autonomes
- ✅ **Result Pattern** — Gestion fonctionnelle des erreurs avec typage strict TypeScript

### Stack Technique
- ✅ **TypeScript avancé** — Typage fort, génériques, Higher-Order Functions
- ✅ **Frameworks multi-couches** — API REST (Express), Web App (Next.js), CLI
- ✅ **Gestion de données** — MongoDB (sans ORM) + In-Memory implementations
- ✅ **CI/CD & Testing** — GitHub Actions, Jest (tests unitaires), ESLint
- ✅ **Validation & Schema** — Zod pour la validation stricte des inputs

### Bonnes Pratiques
- ✅ **Immutabilité** — Aucune mutation d'état, objets value-based
- ✅ **Testabilité** — Domaine totalement isolé et testable unitairement
- ✅ **API Documentation** — Génération automatique de collection Postman
- ✅ **Type Safety** — Zero `any`, interfaces contractuelles strictes

---

## 🏗️ Architecture

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE LAYER                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Express    │  │   Next.js    │  │   CLI Tool   │      │
│  │  (REST API)  │  │   (Web UI)   │  │  (Commands)  │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│         └──────────────────┴──────────────────┘              │
│                             │                                │
│                  ┌──────────▼──────────┐                     │
│                  │   Core / UseCase    │                     │
│                  │   Implementations   │                     │
│                  └──────────┬──────────┘                     │
└─────────────────────────────┼──────────────────────────────┘
                              │
┌─────────────────────────────▼──────────────────────────────┐
│                     APPLICATION LAYER                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Use Cases  │  │ Repositories│  │  Services   │        │
│  │  (Business  │  │ (Interfaces)│  │ (Interfaces)│        │
│  │   Logic)    │  └─────────────┘  └─────────────┘        │
│  └─────────────┘                                            │
│  ┌──────────────────────────────────────────────┐          │
│  │         Projections & Event Handlers         │          │
│  └──────────────────────────────────────────────┘          │
└─────────────────────────────┬──────────────────────────────┘
                              │
┌─────────────────────────────▼──────────────────────────────┐
│                       DOMAIN LAYER                          │
│  ┌──────────────────┐  ┌──────────────────┐               │
│  │  Entities        │  │  Value Objects   │               │
│  │  • Vehicle       │  │  • Address       │               │
│  │  • Maintenance   │  │  • SIRET         │               │
│  │  • Customer      │  │  • UploadedFile  │               │
│  └──────────────────┘  └──────────────────┘               │
│  ┌──────────────────────────────────────────────┐         │
│  │         Domain Events (Event Sourcing)       │         │
│  └──────────────────────────────────────────────┘         │
└────────────────────────────────────────────────────────────┘
```

### Sous-domaines Métier

| Sous-domaine | Responsabilité | Entités principales |
|--------------|----------------|---------------------|
| **InventoryManagement** | Gestion du stock de pièces détachées et des commandes | `Part`, `Order`, `Dealer`, `Stock` |
| **Maintenance** | Suivi des maintenances véhicules et notifications automatiques | `Maintenance`, `Vehicle`, `Failure`, `Notification` |
| **TestDrive** | Planification et suivi des essais clients | `TestDrive`, `Incident`, `Driver`, `Customer` |

Chaque sous-domaine :
- Possède ses propres **repositories** et **event stores**
- Est **complètement isolé** des autres domaines
- Peut être **déployé indépendamment** en microservice
- Communique via **événements** (pub/sub pattern)

### Pattern Event Sourcing + CQRS

#### Architecture de projection
```
Event Domain → Event Store (MongoDB)
                    ↓
            Projection Jobs Queue
                    ↓
         Projection Worker (async)
                    ↓
        Read Models (materialized views)
```

**Implémentation clés** :
- Séparation stricte **Write Model** (événements) et **Read Model** (projections)
- **Projection Worker** avec système de retry en cas de panne
- **Event Repository** dédié par sous-domaine (3 collections MongoDB)
- Événements métier générés directement par les entités domaine

**Point d'amélioration identifié** : Le worker utilise actuellement du polling ; une implémentation avec MongoDB Change Streams améliorerait les performances.

---

## ⚙️ Fonctionnalités Clés

### Métier
- 🚗 Gestion complète du parc véhicules et historique des pannes
- 🔧 Suivi des maintenances avec système de notifications automatiques (cron + CLI)
- 👥 Gestion clients et conducteurs (validations métier strictes)
- 🏍️ Planification d'essais et traçabilité des incidents
- 📦 Gestion de stock avec alertes de niveau bas
- 📁 Upload et gestion de fichiers (photos incidents, documents)

### Technique
- 🔄 **Event Sourcing** — Historique complet de tous les événements métier
- 📊 **Projections asynchrones** — Reconstruction des vues de lecture
- 🎯 **Use Cases fonctionnels** — Higher-Order Functions réutilisables
- 🛡️ **Validation d'entrée** — Schémas Zod partagés entre frameworks
- 📝 **Tests automatisés** — Couverture des domaines TestDrive et InventoryManagement
- 🤖 **CI/CD** — Build, lint, test automatiques sur GitHub Actions

---

## 🛠️ Choix Techniques & Patterns

### 1. Domaine Immutable & Type-Safe

**Choix** : Constructeurs privés, objets immutables, validation métier en amont.

```typescript
// Exemple : création d'entité validée
const result = Vehicle.create({
  vin: "1HGBH41JXMN109186",
  brand: "Honda",
  model: "CBR600RR"
});

if (!result.success) {
  // ApplicationException avec identifiant unique
  throw result.error;
}
```

**Bénéfices** :
- Garantie d'invariants métier à tout moment
- Simplification des tests (pas d'effets de bord)
- Réduction drastique des bugs d'état incohérent

### 2. Result Pattern avec TypeScript

**Choix** : Alternative fonctionnelle aux exceptions pour le flow applicatif.

```typescript
type Result<T> = SuccessResult<T> | FailureResult;
type ResultVoid = VoidResult | FailureResult;
type OptionalResult<T> = SuccessResult<T> | VoidResult | FailureResult;
```

**Bénéfices** :
- Typage exhaustif des cas d'erreur (TypeScript compiler vérifie tous les chemins)
- Composition fonctionnelle facile (`map`, `flatMap`, `fold`)
- Remplacement des `try/catch` par du code prévisible

### 3. Use Case Implementation Layer

**Choix** : Abstraction supplémentaire entre frameworks et application.

```typescript
// Partagé par Express & Next.js
export const createVehicleUseCase = (request: CreateVehicleRequest) => {
  // Parse + validate input (Zod schema)
  // Create domain objects
  // Execute use case
  // Return standardized result
};
```

**Bénéfices** :
- Zéro duplication de code entre frameworks
- Génération automatique de doc API (Postman collection)
- Remplacement facile d'un framework sans toucher la logique métier

### 4. Repository Pattern avec Implémentations Multiples

**Choix** : Abstractions applicatives + implémentations infrastructure.

**Disponible** :
- **MongoDB** (production) — Sans ORM (driver natif)
- **InMemory** (tests) — `InMemoryDataCollection` (tableau amélioré)

**Bénéfices** :
- Tests rapides sans base de données
- Switch facile entre implémentations (DI par configuration)
- Isolation complète de la persistence

---

## 🚀 Mise en Route Rapide

### Prérequis
- **Node.js** ≥ 20.0
- **Docker** (pour MongoDB)

### Installation

```bash
# Installer toutes les dépendances (root + frameworks)
npm run install:all
```

### Lancement des services

```bash
# 1. Démarrer MongoDB (Docker)
docker compose up -d

# 2a. Option API REST (port 3000)
npm run dev:express

# 2b. Option Web App (port 8080)
npm run dev:next
```

### Tests & CI

```bash
# Build TypeScript (hors Next.js)
npm run build

# Tests unitaires (Jest)
npm run test

# Linter (ESLint)
npm run lint
```

> ⚙️ **CI automatique** : Ces commandes s'exécutent automatiquement via GitHub Actions sur chaque PR et push sur `main`.

### Commandes CLI

```bash
# Interagir avec l'application via CLI
npm run command -- [nom-de-commande]
```

---

## 📝 Licence

Projet académique — ESGI 5ème année

---

<p align="center">
  <b>Développé par <a href="https://github.com/Axel77g">Axel77g</a> & <a href="https://github.com/InsaneBob">InsaneBob</a></b><br>
  Démonstration d'architecture logicielle avancée
</p>
