# MTG Vault — Project Plan

## 1. Project Overview

**MTG Vault** is a full-stack Magic: The Gathering application focused on:

- Card discovery and local card data caching
- Personal collection management
- ManaBox CSV import
- Card scanning
- Deck building
- Multiple Magic formats
- Deck legality and deck-building warnings
- Mana curve and deck statistics
- Prerelease / Sealed pool building
- Card pricing and collection valuation
- User authentication
- Demo-user support

The project is also intended as a practical learning project for modern frontend, backend, database, DevOps, and software-design concepts.

---

# 2. Main Goals

The application should allow a user to:

1. Register and log in.
2. Use a demo account without registering.
3. Import an existing MTG collection from ManaBox CSV.
4. Add cards manually.
5. Search cards.
6. Scan physical cards using a phone camera.
7. Store the user's collection.
8. Build decks using cards from the global card database.
9. Optionally identify which cards in a deck are already owned.
10. Support multiple formats such as:
   - Commander
   - Standard
   - Modern
   - Pioneer
   - Legacy
   - Vintage
   - Pauper
   - Sealed / Prerelease
   - Freeform
11. Validate format legality.
12. Show warnings without preventing the user from continuing.
13. Show deck statistics such as:
   - Mana curve
   - Card type distribution
   - Color distribution
   - Mana sources
   - Average mana value
14. Show recommended deck-building guidelines.
15. Track card prices.
16. Estimate:
   - Collection value
   - Deck value
   - Cost of missing deck cards

---

# 3. Proposed Technology Stack

## Frontend

- Angular 22
- TypeScript
- Angular Signals
- Signal-based APIs where appropriate
- RxJS where event streams / cancellation / async composition are appropriate
- Angular Router
- Angular Forms / Signal Forms where appropriate
- Vitest
- Responsive / mobile-first UI

## Backend

- Java 17
- Spring Boot 4.x
- Spring Web
- Spring Data JPA
- Hibernate
- Bean Validation
- Spring Security
- Flyway
- OpenAPI / Swagger
- Maven

## Database

- PostgreSQL

## Infrastructure

- Docker
- Docker Compose
- Git
- GitHub
- CI/CD later in the project

## External Card Data

Primary external card provider:

- Scryfall API

Scryfall will be used as an external source of truth / enrichment source.

The application should prefer local data whenever possible.

---

# 4. High-Level Architecture

```text
Angular Frontend
       |
       | HTTPS / REST
       v
Spring Boot Backend
       |
       +--------------------+
       |                    |
       v                    v
PostgreSQL              Scryfall API
Local Data              External Card Data
```

Important rule:

> The frontend should not use Scryfall directly for core application behavior.

External card data should flow through the backend so that the backend can:

- Cache cards locally
- Normalize data
- Control rate limits
- Handle failures
- Enrich local data
- Keep external API logic isolated from the frontend

---

# 5. Core Domain Concepts

## 5.1 Card

Represents the conceptual Magic card.

Example:

```text
Sol Ring
```

This should store data that is shared between printings.

Possible fields:

```text
id
oracleId
name
manaCost
manaValue
typeLine
oracleText
colors
colorIdentity
keywords
layout
```

---

## 5.2 Card Printing

Represents one specific printing of a card.

Example:

```text
Sol Ring
Commander Masters
Collector Number 396
```

Possible fields:

```text
id
scryfallId
cardId
setCode
setName
collectorNumber
rarity
language
imageUrl
releasedAt
```

A single Card can have multiple CardPrintings.

```text
Card
  |
  +-- CardPrinting
  +-- CardPrinting
  +-- CardPrinting
```

---

## 5.3 User

Represents an application user.

Possible fields:

```text
id
username
email
passwordHash
role
createdAt
updatedAt
```

Roles may initially be:

```text
USER
DEMO
ADMIN
```

---

## 5.4 Collection

A user's MTG collection.

A user should not own a `Card` directly.

They own a specific `CardPrinting`.

Example:

```text
User owns:

Sol Ring
Fallout printing
2 copies
Non-foil
English
```

---

## 5.5 Collection Item

Possible fields:

```text
id
userId
cardPrintingId
quantity
finish
condition
language
acquiredPrice
createdAt
updatedAt
```

Possible finish values:

```text
NONFOIL
FOIL
ETCHED
```

Condition may be added later.

---

## 5.6 Deck

Possible fields:

```text
id
userId
name
format
commanderId
description
createdAt
updatedAt
```

---

## 5.7 Deck Card

Represents a card contained in a deck.

Possible fields:

```text
id
deckId
cardId
quantity
section
```

Possible sections:

```text
MAINBOARD
SIDEBOARD
COMMANDER
MAYBEBOARD
```

---

## 5.8 Format

Initial formats:

```text
COMMANDER
STANDARD
MODERN
PIONEER
LEGACY
VINTAGE
PAUPER
SEALED
FREEFORM
```

Format behavior must not be hard-coded directly inside controllers.

Rules should be handled through a dedicated rules engine.

---

# 6. Card Search Strategy

Card searches should use a **local-first strategy**.

Example:

```text
User searches:
Ancient Greenwarden

        |
        v

Search PostgreSQL

        |
   +----+----+
   |         |
 FOUND     MISSING
   |         |
   |         v
   |      Scryfall
   |         |
   |      Found
   |         |
   |         v
   |     Save locally
   |         |
   +----+----+
        |
        v

Return Card
```

This allows the local card database to grow naturally as the application is used.

---

# 7. Card Data Synchronization

The project may eventually use two methods.

## On-demand enrichment

When the user searches for a missing card:

```text
Local DB
   |
 missing
   |
Scryfall
   |
 save locally
```

## Bulk synchronization

Later, the project may use Scryfall bulk data to populate or refresh card information.

Potential future job:

```text
ScheduledCardSyncJob
```

Responsibilities:

- Download updated bulk data
- Update cards
- Update printings
- Update legality information
- Avoid duplicate cards
- Record sync state

---

# 8. Collection Features

The Collection module should support:

## Manual add

The user searches for a card and adds it to the collection.

Example:

```text
Ancient Greenwarden

Printing:
Zendikar Rising

Quantity:
2

Finish:
Non-foil

[ Add to Collection ]
```

---

## ManaBox CSV Import

The application should accept ManaBox exports.

Suggested flow:

```text
ManaBox
   |
CSV export
   |
Angular upload
   |
Spring Boot
   |
Parse rows
   |
Match card printing
   |
Missing locally?
   |
Scryfall lookup
   |
Save card
   |
Create CollectionItem
```

The import should not fail completely because of a few bad rows.

Use an import job.

Possible states:

```text
PENDING
PROCESSING
SUCCESS
PARTIAL_SUCCESS
FAILED
```

Possible result:

```text
Import complete

487 cards imported
12 existing entries updated
3 rows could not be identified
```

Failures should be inspectable.

---

# 9. Card Scanner

Card scanning is an advanced feature and should not be part of the first MVP.

Possible initial implementation:

```text
Camera
  |
Image Capture
  |
OCR
  |
Card Name Detection
  |
Local Search
  |
Scryfall Fallback
  |
Candidate Match
  |
User Confirmation
  |
Add To Collection
```

Example UI:

```text
We think this is:

Ancient Greenwarden
Zendikar Rising #178

[ Confirm ]
[ Choose another printing ]
[ Cancel ]
```

Future scanner improvements:

- Better OCR
- Set symbol detection
- Collector-number recognition
- Image matching
- Confidence scores
- Batch scanning

---

# 10. Deck Builder

The deck builder should allow creation of decks based on a selected format.

Example:

```text
Create Deck

Name:
Hearthhull Lands

Format:
Commander
```

Deck editor should provide:

- Card search
- Add / remove cards
- Quantity changes
- Sections
- Search own collection
- Search full card database
- Missing-card indicator
- Card preview
- Deck legality state
- Deck-building warnings

---

# 11. Deck Rules Engine

Deck rules should be implemented independently from UI and controllers.

Suggested design:

```java
public interface DeckRule {

    RuleResult validate(Deck deck);

}
```

Possible result:

```text
PASS
INFO
WARNING
ERROR
```

A RuleResult could contain:

```text
code
severity
title
message
```

Example:

```text
severity: ERROR
title: Commander deck size
message: Commander decks must contain exactly 100 cards.
```

Important:

> Rules should warn the user but should not prevent deck editing.

Users should be able to intentionally create illegal or unfinished decks.

---

# 12. Format Legality Rules

Examples for Commander:

```text
Deck size
Singleton restrictions
Color identity
Commander validity
Banned cards
```

Possible classes:

```text
CommanderDeckSizeRule
CommanderSingletonRule
CommanderColorIdentityRule
CommanderBannedCardRule
CommanderValidityRule
```

Examples for Sealed:

```text
Minimum deck size
Pool ownership
Basic land exception
```

---

# 13. Deck-Building Recommendations

Recommendations must be kept separate from actual format legality.

Example:

```text
LEGALITY

✓ 100 cards
✓ Commander valid
✓ Color identity valid
```

Then separately:

```text
GUIDANCE

⚠ 31 lands

Many Commander decks run more lands.
This is not a legality error.
```

Possible recommendation rules:

```text
RecommendedLandCountRule
RecommendedCreatureCountRule
RecommendedRemovalRule
RecommendedCardDrawRule
HighManaCurveRule
LowManaSourceRule
```

Recommendations may eventually become configurable.

---

# 14. Deck Statistics

Each deck should calculate useful statistics.

## Card types

Example:

```text
Creatures        27
Instants          8
Sorceries        10
Enchantments      7
Artifacts         9
Planeswalkers     1
Lands             38
```

## Mana curve

Example:

```text
1    ██
2    █████
3    █████████
4    ███████
5    ████
6+   ██
```

## Additional statistics

- Average mana value
- Color distribution
- Color identity
- Mana sources
- Land count
- Creature count
- Spell count
- Removal count later
- Card draw count later
- Ramp count later

Some advanced categories may require card-tagging logic.

---

# 15. Angular Signals Usage

Angular Signals should be used where they provide real value.

Example deck state:

```ts
deckCards = signal<DeckCard[]>([]);

totalCards = computed(() => ...);

creatures = computed(() => ...);

lands = computed(() => ...);

averageManaValue = computed(() => ...);

manaCurve = computed(() => ...);
```

Important learning goal:

Understand when to use:

```text
Signal
Observable
Promise
```

Signals should not replace RxJS blindly.

RxJS remains useful for:

- HTTP search pipelines
- Debouncing
- Cancellation
- Event streams
- Complex async composition

Example:

```text
Search input
   |
debounce
   |
switchMap
   |
HTTP request
```

---

# 16. Prerelease / Sealed Mode

Prerelease should be treated as its own workflow.

Example:

```text
Create Prerelease Pool

Set:
<set>

Input:
[ Scan Cards ]
[ Import List ]
[ Add Manually ]
```

A prerelease session should contain a Sealed Pool.

Possible domain:

```text
SealedPool
  |
  +-- PoolCard
  +-- PoolCard
  +-- PoolCard
```

Deck building can then use only cards available in the pool, except where rules allow otherwise.

Example UI:

```text
SEALED POOL

White         14
Blue          17
Black         16
Red           19
Green         15
Multicolor     6
```

Deck analysis:

```text
Deck 40 / 40

Creatures     16
Instants       4
Sorceries      3
Artifacts      1
Lands         16
```

Possible guidance:

```text
⚠ Low creature count
⚠ High average mana value
⚠ Few blue mana sources
```

---

# 17. Prices

Prices should belong to card printings.

Suggested model:

```text
CardPrice

id
cardPrintingId
currency
finish
price
source
recordedAt
```

Initially the application may keep only current prices.

Later:

```text
Price History
```

Possible features:

```text
Collection Value
Deck Value
Missing Cards Cost
Price Movement
Historical Graph
```

Example:

```text
Collection

Cards:
1,438

Estimated Value:
€2,417.38
```

---

# 18. Collection + Deck Integration

One important feature:

```text
Deck:
Hearthhull Commander

Cards owned:
87 / 100

Missing:
13

Estimated missing-card cost:
€34.82
```

Users should be able to filter:

```text
Show owned cards
Show missing cards
Show cards available in other decks
```

Advanced ownership logic may later account for physical cards already allocated to multiple decks.

---

# 19. Authentication

Authentication should be added once the core card functionality works.

Features:

```text
Register
Login
Logout
Refresh token
Protected endpoints
Password hashing
Roles
```

Potential approach:

```text
JWT access token
Refresh token
```

Exact authentication design should be decided when this phase begins.

---

# 20. Demo User

A demo account should allow visitors to try the application immediately.

Possible demo content:

```text
Demo collection
Demo Commander deck
Demo Prerelease pool
Demo price data
```

Demo modifications should eventually reset automatically.

Possible strategy:

```text
Scheduled demo reset
```

---

# 21. Database Relationships

Initial conceptual model:

```text
User
 |
 +------ CollectionItem
 |              |
 |              v
 |        CardPrinting
 |              |
 |              v
 |             Card
 |
 +------ Deck
 |         |
 |         +------ DeckCard ------> Card
 |
 +------ SealedPool
 |              |
 |              +------ PoolCard
 |
 +------ ImportJob


Card
 |
 +------ CardPrinting
            |
            +------ CardPrice
```

---

# 22. Docker Strategy

Docker should be introduced gradually.

## Development Stage 1

Run locally:

```text
Angular
Spring Boot
```

Run in Docker:

```text
PostgreSQL
```

Example:

```text
docker compose up -d
```

---

## Development Stage 2

Containerize:

```text
PostgreSQL
Spring Boot
```

---

## Development Stage 3

Containerize the whole stack:

```text
Angular
Spring Boot
PostgreSQL
```

Desired final development command:

```bash
docker compose up
```

---

# 23. Suggested Repository Structure

```text
mtg-vault/
|
+-- frontend/
|   +-- src/
|   +-- package.json
|   +-- Dockerfile
|
+-- backend/
|   +-- src/
|   +-- pom.xml
|   +-- Dockerfile
|
+-- docker-compose.yml
+-- .env.example
+-- README.md
+-- PROJECT_PLAN.md
```

The project should initially remain a modular monolith.

Do not introduce microservices unless there is a genuine technical reason.

---

# 24. Backend Package Direction

Possible Spring package structure:

```text
com.mtgvault

auth/
card/
collection/
deck/
format/
importer/
pricing/
sealed/
user/
shared/
```

Each domain may later contain:

```text
controller
service
repository
domain
dto
mapper
```

However, package structure should evolve naturally rather than creating empty architecture upfront.

---

# 25. API Direction

Possible endpoints.

## Cards

```http
GET /api/cards/search?q=sol+ring
GET /api/cards/{id}
GET /api/cards/{id}/printings
```

## Collection

```http
GET    /api/collection
POST   /api/collection/items
PATCH  /api/collection/items/{id}
DELETE /api/collection/items/{id}

POST   /api/collection/import
GET    /api/collection/imports/{id}
```

## Decks

```http
GET    /api/decks
POST   /api/decks
GET    /api/decks/{id}
PATCH  /api/decks/{id}
DELETE /api/decks/{id}

POST   /api/decks/{id}/cards
PATCH  /api/decks/{id}/cards/{deckCardId}
DELETE /api/decks/{id}/cards/{deckCardId}

GET    /api/decks/{id}/analysis
GET    /api/decks/{id}/validation
```

## Authentication

```http
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
```

Exact API contracts will evolve during implementation.

---

# 26. Testing Strategy

Testing should be introduced from the beginning, but incrementally.

## Backend

Priorities:

```text
Rules Engine
Services
Repository integration tests
External API adapter
CSV parser
```

Possible tools:

```text
JUnit
Mockito
Spring Boot Test
Testcontainers
```

## Frontend

Priorities:

```text
Computed deck statistics
State behavior
Important components
Forms
Services
```

---

# 27. Development Roadmap

---

## Phase 0 — Project Setup

Goal:

Create the project skeleton.

Tasks:

- [ ] Create GitHub repository
- [ ] Create Angular application
- [ ] Create Spring Boot application
- [ ] Add PostgreSQL Docker container
- [ ] Configure backend database connection
- [ ] Add Flyway
- [ ] Add `.env.example`
- [ ] Create initial README
- [ ] Verify frontend -> backend communication

Definition of Done:

```text
Angular starts
Spring Boot starts
PostgreSQL starts
Frontend can call backend health endpoint
```

---

## Phase 1 — Card Engine

Goal:

Build the core card search system.

Tasks:

- [ ] Create Card entity
- [ ] Create CardPrinting entity
- [ ] Create database migrations
- [ ] Build card repository
- [ ] Build card service
- [ ] Integrate Scryfall
- [ ] Implement local-first search
- [ ] Persist externally discovered cards
- [ ] Build search endpoint
- [ ] Build Angular card search page
- [ ] Build card details view
- [ ] Add basic error handling
- [ ] Add backend tests

Definition of Done:

```text
Search card
   ->
find locally OR fetch externally
   ->
save missing data
   ->
display in Angular
```

---

## Phase 2 — Collection

Goal:

Create personal card collections.

Tasks:

- [ ] Add User model
- [ ] Add authentication
- [ ] Add CollectionItem
- [ ] Manual card add
- [ ] Remove card
- [ ] Quantity update
- [ ] Collection search
- [ ] Collection filters
- [ ] ManaBox CSV upload
- [ ] CSV parsing
- [ ] ImportJob model
- [ ] Partial import error handling
- [ ] Import results screen
- [ ] Demo user

Definition of Done:

```text
User can log in
User can import ManaBox CSV
User can browse their collection
```

---

## Phase 3 — Deck Builder

Goal:

Build the first complete deck-building experience.

Initial formats:

```text
Commander
Freeform
```

Tasks:

- [ ] Deck entity
- [ ] DeckCard entity
- [ ] Create deck
- [ ] Edit deck
- [ ] Delete deck
- [ ] Add cards
- [ ] Remove cards
- [ ] Change quantities
- [ ] Card search inside deck builder
- [ ] Commander support
- [ ] Deck analysis endpoint
- [ ] Mana curve
- [ ] Card-type counts
- [ ] Average mana value
- [ ] Color distribution
- [ ] Mana-source analysis
- [ ] Collection ownership indicators

Definition of Done:

```text
User can create and analyze a Commander deck.
```

---

## Phase 4 — Rules Engine

Goal:

Add format legality and deck-building guidance.

Tasks:

- [ ] Create DeckRule abstraction
- [ ] Create RuleResult
- [ ] Implement severity levels
- [ ] Commander size validation
- [ ] Singleton validation
- [ ] Color identity validation
- [ ] Commander validity
- [ ] Ban-list validation
- [ ] Recommendation rules
- [ ] Display warnings in Angular
- [ ] Keep editing possible even when deck is illegal
- [ ] Unit test rule classes

Definition of Done:

```text
Deck shows:

LEGAL
or
NOT FORMAT LEGAL

plus separate guidance warnings.
```

---

## Phase 5 — Prerelease / Sealed

Goal:

Support prerelease deck building.

Tasks:

- [ ] SealedPool entity
- [ ] PoolCard entity
- [ ] Create sealed pool
- [ ] Add cards manually
- [ ] Import pool
- [ ] Build deck from pool
- [ ] Basic-land exception
- [ ] 40-card minimum validation
- [ ] Pool color statistics
- [ ] Limited mana curve
- [ ] Limited guidance warnings

Definition of Done:

```text
User can create a prerelease pool
and build/analyze a Sealed deck from it.
```

---

## Phase 6 — Card Scanner

Goal:

Allow physical-card scanning.

Tasks:

- [ ] Camera UI
- [ ] Capture image
- [ ] OCR proof of concept
- [ ] Extract probable card name
- [ ] Search local card DB
- [ ] External fallback
- [ ] Candidate confirmation screen
- [ ] Add confirmed card to collection
- [ ] Improve printing detection
- [ ] Mobile UX testing

Definition of Done:

```text
Scan card
   ->
identify candidate
   ->
confirm
   ->
add to collection
```

---

## Phase 7 — Prices

Goal:

Add financial information for cards and collections.

Tasks:

- [ ] CardPrice entity
- [ ] Price sync
- [ ] EUR support
- [ ] Foil/non-foil pricing
- [ ] Collection valuation
- [ ] Deck valuation
- [ ] Missing-card cost
- [ ] Price history
- [ ] Price charts

Definition of Done:

```text
Collection and decks have estimated values.
```

---

## Phase 8 — More Formats

Add additional format support incrementally.

Possible order:

- [ ] Standard
- [ ] Modern
- [ ] Pioneer
- [ ] Pauper
- [ ] Legacy
- [ ] Vintage

Each format must define:

```text
Deck constraints
Card legality
Ban / restriction rules
Format-specific validation
```

---

## Phase 9 — Deployment

Goal:

Deploy the application publicly.

Tasks:

- [ ] Backend Docker image
- [ ] Frontend production build
- [ ] Production PostgreSQL
- [ ] Environment configuration
- [ ] CI pipeline
- [ ] CD pipeline
- [ ] HTTPS
- [ ] Demo environment
- [ ] Health checks
- [ ] Basic monitoring

---

# 28. MVP Definition

The first meaningful MVP does **not** include everything.

MVP:

```text
Card Search
+
Local Scryfall caching
+
Authentication
+
ManaBox Import
+
Collection
+
Commander Deck Builder
+
Mana Curve
+
Card Type Statistics
+
Basic Rules Engine
+
Dockerized PostgreSQL
```

Not MVP:

```text
Scanner
Price History
Every Magic Format
Complex recommendations
Advanced card tagging
Social features
Trading
Marketplace
AI deck building
```

---

# 29. Possible Future Features

These are intentionally out of scope initially.

## Deck improvements

- Sideboards
- Maybe boards
- Deck versions
- Deck snapshots
- Compare deck versions
- Import decklists
- Export decklists
- Shareable decks
- Public / private decks

## Collection improvements

- Binders
- Locations
- Wishlist
- Trade list
- Duplicate detection
- Collection history
- Card condition
- Purchase tracking

## Scanner improvements

- Continuous scanning
- Batch scanning
- Printing detection
- Foil detection
- Set-symbol detection

## Analytics

- Price history
- Collection growth
- Most-owned sets
- Color statistics
- Deck statistics history

## Social

- Public profiles
- Shared deck links
- Likes
- Comments
- Deck cloning

Do not implement these until the core application is stable.

---

# 30. Engineering Principles

## Keep the project understandable

Do not introduce technology only because it is fashionable.

Avoid premature use of:

```text
Microservices
Kubernetes
Kafka
Redis
Event sourcing
CQRS
Complex distributed architecture
```

These may be explored later only if a real project requirement appears.

---

## Prefer clear domain boundaries

Examples:

```text
Card
Collection
Deck
Rules
Pricing
Authentication
```

Avoid one giant service that controls the entire application.

---

## External APIs must be isolated

Use a dedicated adapter/client.

Example:

```text
CardService
    |
CardProvider
    |
ScryfallCardProvider
```

This makes it possible to change providers later.

---

## Do not expose database entities directly

Use DTOs between backend and frontend.

Example:

```text
Entity
  |
Mapper
  |
DTO
  |
REST
```

---

## Database changes use migrations

Use Flyway.

Never rely on manually modifying production database tables.

---

## Learn by building

When a new concept is required, understand why it is required before adding it.

Examples:

```text
Interfaces        -> Rules Engine
Streams           -> Card analysis
JPA relationships -> Collections / Decks
Exceptions        -> API error handling
RxJS switchMap    -> Card search
Signals           -> Derived deck state
Docker            -> Local infrastructure
```

---

# 31. First Concrete Milestone

The first development milestone should remain intentionally small.

## Backend

```text
Card
CardPrinting
Scryfall Client
Local Card Search
PostgreSQL
```

Endpoint:

```http
GET /api/cards/search?q=sol+ring
```

Behavior:

```text
1. Search PostgreSQL.
2. If found, return local results.
3. If missing, query Scryfall.
4. Normalize the result.
5. Save the card locally.
6. Return it.
```

## Frontend

Create:

```text
/card-search
```

Features:

```text
Search input
Loading state
Error state
Card results
Card image
Card name
Mana cost
Type
Set
```

## Infrastructure

```text
PostgreSQL via Docker Compose
```

This milestone proves the most important architectural loop:

```text
Angular
   |
Spring Boot
   |
PostgreSQL
   |
Scryfall
```

Once that works reliably, the rest of the application can grow around it.

---

# 32. Project Success Criteria

This project is successful if it becomes:

1. A useful Magic application.
2. A real Angular learning project.
3. A real Java / Spring learning project.
4. A place to practice database design.
5. A place to practice Docker.
6. A portfolio project that can be demonstrated publicly.
7. A project whose architecture the developer can explain confidently in an interview.

The priority is not feature count.

The priority is:

> Build features that are understood, tested, maintainable, and explainable.
