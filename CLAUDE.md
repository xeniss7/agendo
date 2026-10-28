# Agendo
 
SaaS multi-tenant de prise de rendez-vous pour professionnels (coachs, cabinets, salons, consultants).
Projet portfolio : le niveau visé est celui d'un produit en production, pas d'une démo.
 
## Structure du monorepo
 
- `backend/` : API Kotlin + Spring Boot (Gradle Kotlin DSL)
- `web/` : Next.js (TypeScript) : back-office pro + pages de réservation publiques
- `mobile/` : Kotlin Multiplatform + Compose Multiplatform (Android d'abord, iOS compilé en CI)
- `shared/` : module Kotlin partagé (modèles, validation, client API) entre backend et mobile
- `docs/specs/` : spécifications fonctionnelles (`NNNN-sujet.md`)
- `docs/design/` : documents de conception (`NNNN-sujet.md`)
- `docs/adr/` : Architecture Decision Records (`NNNN-titre.md`)
- `docs/reviews/` : rapports de tests et de review (`NNNN-sujet.qa.md`, `NNNN-sujet.review.md`)
- `docs/mentor/` : fiches d'explication et quiz
- `LEARNING.md` : journal d'apprentissage
Chaque sous-projet a son propre `CLAUDE.md` avec ses commandes et conventions.
 
## Stack
 
- Backend : Kotlin, Spring Boot, PostgreSQL, Flyway, JUnit 5, MockK, Testcontainers
- Contrat d'API : OpenAPI généré par le backend (springdoc) = source de vérité ; clients générés pour web et mobile
- Web : Next.js, TypeScript strict, Playwright pour l'E2E
- Mobile : KMP + Compose Multiplatform
- IA produit : API Claude (tool use) côté backend, serveur MCP exposant l'API Agendo
- Paiement : Stripe (mode test uniquement)
- Local : Docker Compose ; CI : GitHub Actions
- Les scripts doivent fonctionner sous Windows (Git Bash) et en CI Linux.

## Processus de développement
 
Le projet utilise le plugin `devflow`. Toute fonctionnalité suit ce cycle, chaque étape produisant un livrable écrit :
 
1. Spec (`devflow:analyst`), validée par le développeur
2. Conception + ADR si une décision d'architecture est prise (`devflow:architect`), validée par le développeur
3. Tests écrits avant le code, en échec (`devflow:qa`) : unitaires, intégration et E2E, validés par le développeur
4. Implémentation jusqu'au vert, puis refactoring (`devflow:developer`)
5. Review (`devflow:reviewer`)
6. Fiche Mentor ; quiz obligatoire pour les fonctionnalités critiques (`devflow:mentor`)
7. Validation du développeur, puis merge

Le projet suit le TDD : les tests validés à l'étape 3 sont la spécification exécutable. Le développeur ne les modifie pas ; il peut seulement en ajouter.
 
Niveaux de test :
- Unitaires : JUnit 5 + MockK, sans Spring ni base de données
- Intégration : Spring Boot Test + Testcontainers (PostgreSQL réel)
- E2E : backend complet appelé en HTTP en boîte noire ; Playwright pour les parcours web
Fonctionnalités critiques sur ce projet : réservation et disponibilités, multi-tenancy et droits, gestion du temps, paiements, actions de l'assistant IA.
 
Ne jamais commencer à coder sans tests validés en échec. Ne jamais merger sans tests verts.
 
## Règles métier non négociables
 
- Multi-tenancy : toute donnée métier appartient à un tenant ; toute requête est filtrée par tenant. Aucune fuite inter-tenant n'est acceptable.
- Temps : stockage en UTC (`Instant`), chaque tenant a un fuseau IANA ; conversion uniquement à l'affichage.
- Réservations : la double réservation d'un même créneau doit être impossible, y compris en accès concurrent (garantie en base, pas seulement dans le code).
- Argent : montants en centimes (entiers) + devise ; jamais de flottants.
- Paiements : webhooks Stripe idempotents.
- IA : l'assistant ne réalise jamais d'action (réserver, annuler, envoyer) sans confirmation explicite de l'utilisateur.

## Conventions
 
- Code, identifiants, commits : en anglais. Documentation, specs, ADR, fiches Mentor : en français.
- Commits : Conventional Commits (`feat:`, `fix:`, `chore:`, `test:`, `docs:`, `refactor:`).
- Branches : `feat/<sujet>`, `fix/<sujet>`, `chore/<sujet>`.
- Code idiomatique et lisible plutôt qu'astucieux ; chaque choix doit pouvoir être justifié.
- Petites modifications ciblées ; pas de refactoring non demandé.
- Demander avant d'ajouter une dépendance, en justifiant le choix.
- Ne jamais modifier une migration Flyway déjà commitée : en créer une nouvelle.
- Aucun secret dans le repo : variables d'environnement + fichiers `.env.example`.

## Avant de déclarer une tâche terminée
 
- Le code compile, les tests passent, le lint est propre.
- La documentation concernée (spec, ADR, README) est à jour.
- Résumer ce qui a été fait et ce qui reste éventuellement à faire.