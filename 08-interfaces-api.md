# §8 — Interfaces & contrats d'API

## Tableau synoptique des endpoints REST

| Méthode | Chemin | Description | Auth | Codes retour | Dépendances aval |
|---|---|---|---|---|---|
| **AUTHENTIFICATION** | | | | | |
| GET | `/api/v1/auth/callback` | Callback OIDC — échange le code contre tokens JWT + refresh | Public | 302, 400, 401, 500 | AuthService, UserService |
| POST | `/api/v1/auth/refresh` | Émet un nouveau JWT à partir d'un refresh token valide | Public | 200, 401, 403 | AuthService, Redis |
| POST | `/api/v1/auth/logout` | Révoque le refresh token et invalide la session | JWT | 204, 401 | AuthService, Redis |
| **CATALOGUE D'ÉVÉNEMENTS** | | | | | |
| GET | `/api/v1/events` | Liste paginée des événements publiés, avec filtres | Public | 200, 400 | EventService, Redis (cache) |
| GET | `/api/v1/events/search` | Recherche full-text d'événements (titre, lieu, catégorie) | Public | 200, 400 | EventService |
| GET | `/api/v1/events/{id}` | Détail d'un événement | Public | 200, 404 | EventService |
| GET | `/api/v1/categories` | Liste de toutes les catégories | Public | 200 | EventService |
| **GESTION DES ÉVÉNEMENTS (ORGANISATEUR)** | | | | | |
| POST | `/api/v1/events` | Crée un nouvel événement (status: draft) | JWT + role:organizer | 201, 400, 403, 409 | EventService |
| PUT | `/api/v1/events/{id}` | Met à jour un événement (si draft ou published) | JWT + role:organizer | 200, 400, 403, 404, 409 | EventService |
| POST | `/api/v1/events/{id}/publish` | Publie un événement (draft → published) | JWT + role:organizer | 200, 403, 404, 422 | EventService, RabbitMQ |
| POST | `/api/v1/events/{id}/cancel` | Annule un événement ouvert | JWT + role:organizer | 200, 403, 404, 409 | EventService, RabbitMQ |
| POST | `/api/v1/events/{id}/banner` | Upload de la bannière (multipart) | JWT + role:organizer | 200, 400, 403, 404, 413 | EventService, S3/MinIO |
| **INSCRIPTIONS & TICKETS** | | | | | |
| POST | `/api/v1/tickets` | Crée un ticket (inscription à un événement) | JWT | 201, 400, 402, 409, 422 | TicketService, EventService, PaymentService |
| GET | `/api/v1/tickets/{id}` | Détail d'un ticket | JWT | 200, 403, 404 | TicketService |
| GET | `/api/v1/users/me/tickets` | Liste des tickets de l'utilisateur connecté | JWT | 200, 401 | TicketService |
| DELETE | `/api/v1/tickets/{id}` | Annule un ticket (si délai d'annulation respecté) | JWT | 200, 403, 404, 409 | TicketService, PaymentService, RabbitMQ |
| POST | `/api/v1/events/{id}/waitlist` | S'inscrit sur la liste d'attente | JWT | 201, 409, 404, 422 | TicketService |
| DELETE | `/api/v1/events/{id}/waitlist` | Se retire de la liste d'attente | JWT | 204, 404 | TicketService |
| **PAIEMENT** | | | | | |
| POST | `/api/v1/payments/initiate` | Crée un PaymentIntent Stripe et retourne le client_secret | JWT | 201, 400, 402, 409, 422 | PaymentService, Stripe |
| POST | `/api/v1/payments/webhook` | Réception des événements Stripe (payment_intent.succeeded, etc.) | **HMAC** | 200, 400, 401 | PaymentService, RabbitMQ |
| GET | `/api/v1/payments/{id}` | Statut d'un paiement | JWT | 200, 403, 404 | PaymentService |
| **TABLEAU DE BORD ORGANISATEUR** | | | | | |
| GET | `/api/v1/organizer/events/{id}/participants` | Liste paginée des participants inscrits | JWT + role:organizer | 200, 403, 404 | EventService, TicketService |
| GET | `/api/v1/organizer/events/{id}/participants/export` | Export CSV des participants | JWT + role:organizer | 200, 403, 404 | EventService, TicketService, S3/MinIO |
| GET | `/api/v1/organizer/events/{id}/kpi` | KPIs : taux de remplissage, revenus, annulations | JWT + role:organizer | 200, 403, 404 | EventService, PaymentService |
| **ADMINISTRATION PLATEFORME** | | | | | |
| GET | `/api/v1/admin/organizers` | Liste des organisateurs en attente de validation | JWT + role:admin | 200, 403 | AdminService |
| POST | `/api/v1/admin/organizers/{id}/validate` | Valide un organisateur | JWT + role:admin | 200, 403, 404, 409 | AdminService, RabbitMQ |
| POST | `/api/v1/admin/organizers/{id}/revoke` | Révoque un organisateur | JWT + role:admin | 200, 403, 404 | AdminService, RabbitMQ |
| GET | `/api/v1/admin/users` | Liste des utilisateurs (recherche, filtre) | JWT + role:admin | 200, 403 | AdminService |
| DELETE | `/api/v1/admin/users/{id}` | Désactive un compte utilisateur | JWT + role:admin | 200, 403, 404 | AdminService |

> **⚠️ Point critique** : `POST /api/v1/payments/webhook` n'est **pas** authentifié par JWT. L'authentification repose sur la vérification de la signature HMAC-SHA256 fournie dans l'en-tête `Stripe-Signature`, calculée avec le webhook secret Stripe. Toute requête dont la signature est invalide ou expirée (tolérance 300s) est rejetée avec `401 Unauthorized`.

---

## Conventions transverses

### Format des erreurs (RFC 7807 Problem Details)

Toutes les erreurs sont retournées au format [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807) :

```json
{
  "type": "https://supevents.io/errors/ticket-already-exists",
  "title": "Conflict",
  "status": 409,
  "detail": "L'utilisateur possède déjà un ticket actif pour cet événement.",
  "instance": "/api/v1/tickets"
}
```

Les codes d'erreur métier courants :

| Code HTTP | `type` | Cas d'usage |
|---|---|---|
| 400 | `.../invalid-input` | Champ manquant ou format invalide |
| 401 | `.../unauthorized` | Token JWT absent, expiré ou signature HMAC invalide |
| 403 | `.../forbidden` | Rôle insuffisant ou ressource appartenant à un autre user |
| 404 | `.../not-found` | Ressource inexistante |
| 409 | `.../conflict` | Violation de contrainte UNIQUE (double inscription, email déjà pris) |
| 422 | `.../unprocessable` | Règle métier violée (événement complet, annulation hors délai) |
| 429 | `.../rate-limited` | Quota dépassé |

### Stratégie de versioning

L'API est versionnée par préfixe de chemin (`/api/v1/`). Le passage à `v2` interviendra uniquement en cas de rupture de contrat (suppression de champ, changement de type). Les versions sont maintenues en parallèle au minimum 6 mois après la publication de la version suivante. L'en-tête `Deprecation` est ajouté aux réponses des endpoints obsolètes.

### Rate limiting et quotas

| Profil | Fenêtre | Limite |
|---|---|---|
| Public (non authentifié) | 1 min | 60 req/IP |
| Utilisateur authentifié | 1 min | 300 req/user |
| Organisateur | 1 min | 600 req/user |
| Admin | 1 min | illimité |
| Webhook Stripe | — | pas de limite côté SupEvents |

Les limites sont appliquées via Redis (compteur sliding window). Le dépassement retourne `429 Too Many Requests` avec l'en-tête `Retry-After`.

---

## Événements asynchrones

### `ticket.confirmed`

#### Fiche descriptive

| Champ | Valeur |
|---|---|
| **Nom** | `ticket.confirmed` |
| **Producteur** | **TicketService** — publié après confirmation atomique du ticket en base : soit immédiatement à la création (événement gratuit, `price = 0`), soit à la réception du webhook Stripe `payment_intent.succeeded` (événement payant) |
| **Topic / exchange** | `tickets.events.v1` (RabbitMQ, exchange type `topic`, durable) |
| **Routing key** | `ticket.confirmed` |
| **Consommateurs connus** | **NotificationService** : envoie un email de confirmation + notification in-app à l'utilisateur ; **StatsService** : incrémente les compteurs de KPI de l'organisateur (taux de remplissage, revenus) |
| **Garantie de livraison** | `at-least-once` — le message peut être redelivré en cas d'échec du consommateur. Les consommateurs doivent être idempotents : l'idempotence est assurée par la clé `ticket_id` (UUID) dans le payload |
| **Stratégie de retry** | Backoff exponentiel : 1s, 2s, 4s, 8s — max 5 tentatives. Après épuisement : routage vers la dead letter queue `tickets.events.v1.dlq` avec le header `x-death` pour analyse |

#### Schéma JSON du payload

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TicketConfirmedEvent",
  "type": "object",
  "required": ["event_name", "ticket_id", "user_id", "event_id", "confirmed_at", "is_paid", "metadata"],
  "properties": {
    "event_name": {
      "type": "string",
      "const": "ticket.confirmed",
      "description": "Nom canonique de l'événement asynchrone"
    },
    "ticket_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant unique du ticket confirmé (clé d'idempotence)"
    },
    "user_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'utilisateur détenteur du ticket"
    },
    "event_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'événement concerné"
    },
    "confirmed_at": {
      "type": "string",
      "format": "date-time",
      "description": "Horodatage ISO 8601 de la confirmation (UTC)"
    },
    "is_paid": {
      "type": "boolean",
      "description": "True si le ticket a nécessité un paiement, false si gratuit"
    },
    "metadata": {
      "type": "object",
      "required": ["event_title", "event_starts_at", "seat_ref"],
      "properties": {
        "event_title": {
          "type": "string",
          "description": "Titre de l'événement (dénormalisé pour éviter un appel synchrone côté consommateur)"
        },
        "event_starts_at": {
          "type": "string",
          "format": "date-time",
          "description": "Date/heure de début de l'événement (UTC)"
        },
        "seat_ref": {
          "type": ["string", "null"],
          "description": "Référence de siège si applicable, null sinon"
        },
        "amount_paid": {
          "type": ["number", "null"],
          "minimum": 0,
          "description": "Montant payé en euros, null si gratuit"
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

---

### `payment.failed`

#### Fiche descriptive

| Champ | Valeur |
|---|---|
| **Nom** | `payment.failed` |
| **Producteur** | **PaymentService** — publié à la réception du webhook Stripe `payment_intent.payment_failed` (carte refusée, fonds insuffisants) ou `payment_intent.canceled` (time-out 30 min sans confirmation) |
| **Topic / exchange** | `payments.events.v1` (RabbitMQ, exchange type `topic`, durable) |
| **Routing key** | `payment.failed` |
| **Consommateurs connus** | **NotificationService** : envoie un email à l'utilisateur l'invitant à relancer le paiement ou à utiliser un autre moyen de paiement ; **TicketService** : repasse le ticket en statut `pending` (ou `cancelled` si délai dépassé) et libère la place réservée pour permettre à d'autres de s'inscrire |
| **Garantie de livraison** | `at-least-once` — idempotence côté TicketService assurée par la clé `payment_intent_id` : le statut du ticket ne peut être muté plusieurs fois par le même `payment_intent_id` |
| **Stratégie de retry** | Backoff exponentiel : 2s, 4s, 8s, 16s — max 4 tentatives. Après épuisement : routage vers `payments.events.v1.dlq`. Une alerte PagerDuty est déclenchée si la DLQ dépasse 10 messages non traités |

#### Schéma JSON du payload

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "PaymentFailedEvent",
  "type": "object",
  "required": ["event_name", "payment_id", "ticket_id", "user_id", "payment_intent_id", "failed_at", "failure_reason", "metadata"],
  "properties": {
    "event_name": {
      "type": "string",
      "const": "payment.failed",
      "description": "Nom canonique de l'événement asynchrone"
    },
    "payment_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant interne du paiement"
    },
    "ticket_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant du ticket dont le paiement a échoué"
    },
    "user_id": {
      "type": "string",
      "format": "uuid",
      "description": "Identifiant de l'utilisateur concerné"
    },
    "payment_intent_id": {
      "type": "string",
      "description": "Référence Stripe du PaymentIntent (clé d'idempotence)"
    },
    "failed_at": {
      "type": "string",
      "format": "date-time",
      "description": "Horodatage ISO 8601 de l'échec (UTC)"
    },
    "failure_reason": {
      "type": "string",
      "enum": ["card_declined", "insufficient_funds", "expired_card", "timeout", "fraud_detected", "unknown"],
      "description": "Code de raison normalisé (dérivé du code Stripe)"
    },
    "metadata": {
      "type": "object",
      "required": ["event_id", "amount", "currency"],
      "properties": {
        "event_id": {
          "type": "string",
          "format": "uuid",
          "description": "Identifiant de l'événement concerné"
        },
        "amount": {
          "type": "number",
          "minimum": 0,
          "description": "Montant du paiement tenté en devise"
        },
        "currency": {
          "type": "string",
          "minLength": 3,
          "maxLength": 3,
          "description": "Code devise ISO 4217 (ex: EUR)"
        },
        "retryable": {
          "type": "boolean",
          "description": "Indique si l'utilisateur peut retenter le paiement (false pour fraud_detected)"
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```
