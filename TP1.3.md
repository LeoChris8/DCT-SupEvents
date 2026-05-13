# §6 — Architecture SupEvents

## §6.1 — Vue logique

---

### Diagramme C4 Context

Ce diagramme présente une vue d'ensemble du système SupEvents dans son environnement. Il positionne le système au centre des interactions avec ses utilisateurs humains et les services tiers dont il dépend. Il est destiné à toute l'équipe projet — développeurs, architectes et décideurs métier — afin de partager une vision commune du périmètre applicatif.

```mermaid
flowchart TD
    actor_etudiant(["👤 Étudiant\n[Personne]"])
    actor_orga(["👤 Organisateur\n[Personne]"])
    actor_admin(["👤 Administrateur\n[Personne]"])

    system_supevents["🖥️ SupEvents\n[Système logiciel]"]

    ext_stripe["Stripe\n[Système externe]"]
    ext_sendgrid["SendGrid\n[Système externe]"]
    ext_sso["SSO École\n[Système externe]"]
    ext_cdn["CDN\n[Système externe]"]

    actor_etudiant -- "HTTPS/REST\nConsulte et s'inscrit" --> system_supevents
    actor_orga -- "HTTPS/REST\nCrée et gère les événements" --> system_supevents
    actor_admin -- "HTTPS/REST\nAdministre la plateforme" --> system_supevents

    system_supevents -- "HTTPS/REST\nInitie le paiement" --> ext_stripe
    ext_stripe -- "Webhook HTTPS\nConfirme la transaction" --> system_supevents

    system_supevents -- "SMTP\nEnvoie les emails transactionnels" --> ext_sendgrid

    system_supevents -- "OIDC/OAuth2\nDélègue l'authentification" --> ext_sso
    ext_sso -- "OIDC/OAuth2\nRetourne le token d'identité" --> system_supevents

    system_supevents -- "HTTPS\nPublie les médias" --> ext_cdn
    actor_etudiant -- "HTTPS\nTélécharge les visuels" --> ext_cdn

    style system_supevents fill:#1168bd,color:#fff,stroke:#0b4884
    style actor_etudiant fill:#08427b,color:#fff,stroke:#052e56
    style actor_orga fill:#08427b,color:#fff,stroke:#052e56
    style actor_admin fill:#08427b,color:#fff,stroke:#052e56
    style ext_stripe fill:#999,color:#fff,stroke:#666
    style ext_sendgrid fill:#999,color:#fff,stroke:#666
    style ext_sso fill:#999,color:#fff,stroke:#666
    style ext_cdn fill:#999,color:#fff,stroke:#666
```

Le diagramme met en évidence quatre dépendances externes critiques : Stripe pour la gestion sécurisée des paiements (flux aller-retour via webhook), SendGrid pour les notifications email, le SSO École comme fournisseur d'identité unique (OIDC), et le CDN pour la distribution des médias. Le découplage entre SupEvents et ces systèmes tiers est un choix architectural fort qui limite l'impact en cas de panne d'un fournisseur.

---

### Diagramme C4 Containers

Ce diagramme zoome à l'intérieur du système SupEvents et expose les différents blocs déployables qui le composent, avec leurs technologies respectives. Il est principalement destiné aux développeurs et aux architectes pour comprendre comment les responsabilités sont distribuées entre les containers et quels protocoles sont utilisés pour leur communication.

```mermaid
flowchart TD
    actor_user(["👤 Utilisateurs\n[Personne]"])

    subgraph supevents ["SupEvents [Système logiciel]"]
        frontend["Frontend Web\n[SPA — React]\nInterface utilisateur"]
        gateway["API Gateway\n[nginx / Kong]\nPoint d'entrée unique"]
        backend["Service Backend\n[API REST — NestJS]\nLogique métier"]
        db["Base de données\n[PostgreSQL]\nDonnées persistantes"]
        cache["Cache\n[Redis]\nSessions & cache"]
        broker["Broker de messages\n[RabbitMQ]\nFile d'attente asynchrone"]
        notif["Service Notifications\n[Worker — Node.js]\nEnvoi des emails"]
        storage["Stockage objets\n[S3 / MinIO]\nMédias & tickets PDF"]
    end

    ext_stripe["Stripe\n[Système externe]"]
    ext_sendgrid["SendGrid\n[Système externe]"]
    ext_sso["SSO École\n[Système externe]"]
    ext_cdn["CDN\n[Système externe]"]

    actor_user -- "HTTPS :443" --> frontend
    frontend -- "HTTPS :443" --> gateway
    gateway -- "HTTPS :443" --> backend

    backend -- "TCP :5432" --> db
    backend -- "Redis :6379" --> cache
    backend -- "AMQP :5672\n[async]" --> broker
    backend -- "HTTPS :443" --> ext_stripe
    backend -- "OIDC/OAuth2 :443" --> ext_sso
    backend -- "HTTPS :443\n[PUT objects]" --> storage

    broker -- "AMQP :5672\n[async]" --> notif
    notif -- "SMTP :587" --> ext_sendgrid

    storage -- "HTTPS :443" --> ext_cdn

    ext_stripe -- "Webhook HTTPS :443\n[async]" --> gateway

    style frontend fill:#1168bd,color:#fff
    style gateway fill:#1168bd,color:#fff
    style backend fill:#1168bd,color:#fff
    style db fill:#2d6a4f,color:#fff
    style cache fill:#2d6a4f,color:#fff
    style broker fill:#e07b00,color:#fff
    style notif fill:#1168bd,color:#fff
    style storage fill:#2d6a4f,color:#fff
    style ext_stripe fill:#999,color:#fff
    style ext_sendgrid fill:#999,color:#fff
    style ext_sso fill:#999,color:#fff
    style ext_cdn fill:#999,color:#fff
```

On notera la séparation nette entre les flux synchrones (HTTPS, TCP, Redis) et les flux asynchrones (AMQP via RabbitMQ). Le broker découple le service Backend du service de notifications : le backend n'a pas besoin d'attendre la confirmation de l'envoi email pour répondre à l'utilisateur. L'API Gateway centralise l'authentification et le routage, ce qui simplifie la logique de sécurité dans le backend.

---

## §6.2 — Vue des processus

---

### Diagramme de composants — API Backend

Ce diagramme décompose l'intérieur du container "Service Backend" en modules fonctionnels NestJS. Il représente les interfaces exposées par chaque module, leurs dépendances internes et leurs connexions aux systèmes tiers. Il est destiné aux développeurs backend pour comprendre les responsabilités de chaque module et éviter les couplages non intentionnels.

```mermaid
flowchart TD
    subgraph backend ["Service Backend [NestJS]"]
        auth["AuthModule\nIAuthService\nVérification tokens OIDC"]
        user["UserModule\nIUserService\nGestion des profils"]
        event["EventModule\nIEventService\nGestion des événements"]
        ticket["TicketModule\nITicketService\nRéservations & QR codes"]
        payment["PaymentModule\nIPaymentService\nPaiements & webhooks"]
        notif["NotificationModule\nINotificationService\nPublication d'événements"]
    end

    db[("PostgreSQL\n:5432")]
    cache[("Redis\n:6379")]
    broker["RabbitMQ\n:5672"]
    ext_stripe["Stripe\nAPI externe"]
    ext_sso["SSO École\nOIDC"]

    auth -- "OIDC validation" --> ext_sso
    auth -- "Lit/écrit sessions" --> cache

    user -- "TCP :5432" --> db
    user --> auth

    event -- "TCP :5432" --> db
    event -- "Redis :6379\nCache listes" --> cache

    ticket --> event
    ticket --> payment
    ticket -- "TCP :5432" --> db

    payment -- "HTTPS :443" --> ext_stripe
    payment -- "TCP :5432" --> db
    payment --> notif

    notif -- "AMQP :5672\nPublie événements" --> broker

    style auth fill:#1168bd,color:#fff
    style user fill:#1168bd,color:#fff
    style event fill:#1168bd,color:#fff
    style ticket fill:#1168bd,color:#fff
    style payment fill:#1168bd,color:#fff
    style notif fill:#1168bd,color:#fff
    style db fill:#2d6a4f,color:#fff
    style cache fill:#2d6a4f,color:#fff
    style broker fill:#e07b00,color:#fff
    style ext_stripe fill:#999,color:#fff
    style ext_sso fill:#999,color:#fff
```

Les dépendances sont intentionnellement limitées : `TicketModule` dépend de `EventModule` (vérification des places disponibles) et de `PaymentModule` (initiation du paiement), mais pas directement de `NotificationModule`. C'est `PaymentModule` qui publie dans RabbitMQ via `NotificationModule`, assurant ainsi le découplage entre la logique de paiement et l'envoi des emails. `AuthModule` est transversal et peut être utilisé par tous les modules via un guard NestJS.

---

### Diagramme de séquence — Inscription nominale

Ce diagramme couvre le parcours complet d'un étudiant depuis le clic "S'inscrire" jusqu'à la réception du ticket par email. Il détaille les interactions entre tous les composants du système, en distinguant flux synchrones et asynchrones. Il est destiné aux développeurs pour implémenter et tester le flux de bout en bout.

```mermaid
sequenceDiagram
    actor Étudiant
    participant Frontend
    participant Gateway as API Gateway
    participant Auth as AuthModule
    participant Event as EventModule
    participant Ticket as TicketModule
    participant Payment as PaymentModule
    participant Notif as NotificationModule
    participant PG as PostgreSQL
    participant Stripe
    participant MQ as RabbitMQ

    Étudiant->>Frontend: Clic "S'inscrire" à l'événement
    Frontend->>Gateway: POST /events/{id}/register (JWT)
    Gateway->>Auth: Valide le token JWT
    Auth-->>Gateway: Token valide, userId extrait

    Gateway->>Event: Vérifie disponibilité des places
    Event->>PG: SELECT places_disponibles WHERE event_id = ?
    PG-->>Event: 12 places restantes
    Event-->>Gateway: Disponible

    opt Événement complet
        Gateway-->>Frontend: 409 - Ajout en liste d'attente
        Frontend-->>Étudiant: "Vous êtes sur liste d'attente"
    end

    Gateway->>Ticket: Crée réservation pending
    Ticket->>PG: INSERT reservation (status='pending')
    PG-->>Ticket: reservation_id = 42
    Ticket-->>Gateway: Réservation créée

    Gateway->>Payment: Initie paiement Stripe
    Payment->>Stripe: POST /payment_intents (montant, devise)
    Stripe-->>Payment: {client_secret, payment_intent_id}
    Payment-->>Gateway: URL de paiement
    Gateway-->>Frontend: Redirect vers Stripe Checkout
    Frontend-->>Étudiant: Page de paiement Stripe

    Étudiant->>Stripe: Saisit ses coordonnées bancaires

    alt Paiement accepté
        Stripe->>Gateway: Webhook POST /webhooks/stripe (payment_intent.succeeded)
        Gateway->>Payment: Traite le webhook
        Payment->>PG: UPDATE reservation SET status='confirmed'
        Payment->>Ticket: Génère QR code
        Ticket->>PG: UPDATE ticket SET qr_code = ?
        Payment->>Notif: Notifie confirmation paiement
        Notif->>MQ: Publish ticket.confirmed {userId, eventId, ticketId}
        MQ-->>Notif: ACK
        Note over MQ: Message asynchrone
        Notif-->>MQ: [Worker] Consume ticket.confirmed
        Notif->>Étudiant: Email avec ticket PDF + QR code (via SendGrid)
        Gateway-->>Frontend: 200 - Inscription confirmée
        Frontend-->>Étudiant: "Votre inscription est confirmée !"

    else Paiement refusé
        Stripe->>Gateway: Webhook POST /webhooks/stripe (payment_intent.payment_failed)
        Gateway->>Payment: Traite l'échec
        Payment->>PG: UPDATE reservation SET status='cancelled'
        Payment->>Notif: Notifie échec paiement
        Notif->>MQ: Publish payment.failed {userId, reason}
        Notif-->>Étudiant: Email "Paiement refusé"
        Gateway-->>Frontend: 402 - Paiement échoué
        Frontend-->>Étudiant: "Votre paiement a échoué"
    end
```

Le fragment `alt` distingue clairement le chemin nominal (paiement accepté) du chemin d'erreur. Le fragment `opt` gère le cas de la liste d'attente avant même l'initiation du paiement. La confirmation Stripe arrive via webhook (flux asynchrone entrant), ce qui implique que le frontend ne doit pas attendre une réponse synchrone du backend sur cette étape — une contrainte importante pour l'implémentation du polling ou des WebSockets côté client.

---

### Diagramme de séquence — Échec de paiement

Ce diagramme reprend le flux d'inscription jusqu'à l'étape de confirmation Stripe, mais modélise les deux cas d'échec possibles : le timeout (absence de réponse dans les délais) et le refus explicite (carte refusée). Il est destiné aux développeurs et aux équipes de test pour s'assurer que les cas d'erreur sont correctement gérés et tracés.

```mermaid
sequenceDiagram
    actor Étudiant
    participant Frontend
    participant Gateway as API Gateway
    participant Payment as PaymentModule
    participant Notif as NotificationModule
    participant PG as PostgreSQL
    participant Stripe
    participant MQ as RabbitMQ

    Note over Étudiant,Stripe: Flux identique jusqu'à l'étape de confirmation Stripe

    Étudiant->>Frontend: Clic "S'inscrire"
    Frontend->>Gateway: POST /events/{id}/register (JWT)
    Gateway->>Payment: Initie paiement Stripe
    Payment->>Stripe: POST /payment_intents
    Stripe-->>Payment: {client_secret}
    Gateway-->>Frontend: URL de paiement
    Étudiant->>Stripe: Tente le paiement

    alt Refus explicite (carte refusée)
        Stripe->>Gateway: Webhook payment_intent.payment_failed\n{error: "card_declined"}
        Gateway->>Payment: Traite le webhook d'échec
        Payment->>PG: UPDATE reservation SET status='cancelled'
        PG-->>Payment: OK
        Payment->>PG: INSERT error_log (type='card_declined', timestamp, userId)
        Payment->>Notif: Émet événement d'échec
        Notif->>MQ: Publish payment.failed {reason: 'card_declined', userId}
        MQ-->>Notif: ACK
        Notif-->>Étudiant: Email "Votre paiement a été refusé\n(carte bancaire refusée)"
        Gateway-->>Frontend: 402 Payment Required
        Frontend-->>Étudiant: Message d'erreur explicatif

    else Timeout Stripe (pas de réponse dans les 30s)
        Note over Payment: Job planifié vérifie les reservations pending > 30min
        Payment->>Stripe: GET /payment_intents/{id} (polling)
        Stripe-->>Payment: status='requires_payment_method' (expiré)
        Payment->>PG: UPDATE reservation SET status='cancelled'
        PG-->>Payment: OK
        Payment->>PG: INSERT error_log (type='stripe_timeout', timestamp, userId)
        Payment->>Notif: Émet événement de timeout
        Notif->>MQ: Publish payment.failed {reason: 'timeout', userId}
        MQ-->>Notif: ACK
        Notif-->>Étudiant: Email "Votre session de paiement a expiré"
        Note over Frontend: L'utilisateur peut retenter l'inscription
    end
```

Les deux cas d'échec aboutissent systématiquement à une libération de la réservation (`status='cancelled'`) et à un log d'incident dans PostgreSQL — garantissant la traçabilité. La distinction principale réside dans le déclencheur : le refus explicite est traité immédiatement via webhook, tandis que le timeout nécessite un job planifié (cron) qui interroge Stripe en polling. La notification à l'étudiant est dans les deux cas différente afin de lui communiquer une raison précise et lui permettre de réagir (retenter, contacter sa banque, etc.).
