# §10 — Matrice de traçabilité

## Introduction

Cette matrice croise les 21 exigences du cahier des charges SupEvents (13 exigences fonctionnelles EF + 8 exigences non-fonctionnelles ENF) avec les éléments produits dans la DCT au fil des TP 1.4 à 1.6. Elle constitue le document de référence pour vérifier qu'aucune exigence n'est orpheline et qu'aucun composant de la DCT n'existe sans justification dans le CDC. La lecture se fait dans les deux sens : horizontalement pour s'assurer qu'une exigence est bien couverte, verticalement pour vérifier qu'un module ou ADR sert au moins une exigence. Toute exigence non encore couverte est explicitement signalée — une matrice honnête vaut mieux qu'une matrice optimiste.

---

## §10.1 — Matrice principale

| Réf. | Intitulé (court) | Module(s) | Section(s) DCT | ADR |
|---|---|---|---|---|
| EF-01 | Création d'événements | EventModule | §6.4 (Event, Organizer), §8 (`POST /api/v1/events`, `POST /api/v1/events/{id}/publish`) | — |
| EF-02 | Catalogue public d'événements | EventModule | §6.4 (Event, Category), §8 (`GET /api/v1/events`, `GET /api/v1/events/search`, `GET /api/v1/events/{id}`) | — |
| EF-03 | Inscription à un événement | TicketModule | §6.4 (Ticket, WaitingListEntry), §7.1, §8 (`POST /api/v1/tickets`), §9.1.1 (STRIDE flux inscription) | ADR-002 |
| EF-04 | Paiement en ligne | PaymentModule | §6.4 (Payment), §8 (`POST /api/v1/payments/initiate`, `POST /api/v1/payments/webhook`), §9.1.1 (STRIDE flux inscription) | — |
| EF-05 | Annulation de ticket | TicketModule | §6.4 (Ticket), §7.1 (cas limite annulation après paiement), §8 (`DELETE /api/v1/tickets/{id}`) | — |
| EF-06 | Authentification SSO | AuthModule | §6.4 (User, RefreshToken), §7.3, §8 (`GET /api/v1/auth/callback`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/logout`), §9.1.2 (STRIDE flux SSO) | ADR-001 |
| EF-07 | Gestion des rôles | AuthModule | §6.4 (User, Organizer), §7.3 (provisioning, mapping rôles), §8 (colonne Auth du tableau synoptique) | ADR-001 |
| EF-08 | Notifications utilisateur | NotificationModule | §6.4 (Notification), §7.2, §8 (événements async `ticket.confirmed`, `payment.failed`, `event.cancelled`) | ADR-003 |
| EF-09 | Liste d'attente | TicketModule | §6.4 (WaitingListEntry), §8 (`POST /api/v1/events/{id}/waitlist`, `DELETE /api/v1/events/{id}/waitlist`) | — |
| EF-10 | Tableau de bord organisateur | EventModule | §6.4 (Ticket, Payment), §8 (`GET /api/v1/organizer/events/{id}/participants`, `GET /api/v1/organizer/events/{id}/kpi`) | — |
| EF-11 | Export CSV participants | EventModule | §8 (`GET /api/v1/organizer/events/{id}/participants/export`), §6.4 (stratégie stockage S3) | — |
| EF-12 | Validation organisateur | AuthModule, AdminService | §6.4 (Organizer), §8 (`POST /api/v1/admin/organizers/{id}/validate`, `POST /api/v1/admin/organizers/{id}/revoke`) | — |
| EF-13 | Génération QR code billet | TicketModule | §6.4 (stratégie stockage S3), §7.1 (QRCodeAdapter, cas limite QR generation failed) | ⚠️ Non couvert par ADR — à traiter en TP 1.8 (choix signature symétrique vs asymétrique) |
| ENF-01 | Performance p95 < 500 ms | TicketModule, EventModule | §9.2 (SLO-01 latence p95 < 200 ms catalogue, SLO-03 pic inscriptions), §6.4 (index PostgreSQL) | ADR-002 |
| ENF-02 | Disponibilité 99,5 % | AuthModule, TicketModule, NotificationModule | §9.2 (SLO-02 disponibilité, budget d'erreur 3h36/mois) | — |
| ENF-03 | Sécurité authentification | AuthModule | §7.3, §9.1.2 (STRIDE SSO), §9.3.b (mécanismes protection) | ADR-001 |
| ENF-04 | Conformité PCI-DSS paiements | PaymentModule | §6.4 (Payment — rappel conformité, pas de données bancaires), §8 (webhook HMAC), §9.1.1 (STRIDE tampering) | — |
| ENF-05 | Conformité RGPD | AuthModule, UserModule | §6.4 (marquage RGPD dictionnaire), §9.3 (registre traitements, mécanismes, procédures droits) | — |
| ENF-06 | 500 utilisateurs simultanés | TicketModule | §9.2 (SLO-03 pic concurrent), §7.1 (verrou pessimiste, pool PgBouncer) | ADR-002 |
| ENF-07 | Idempotence des opérations | PaymentModule, TicketModule | §8 (clé idempotence webhook Stripe, Redis `idempotency:stripe:{id}`), §7.1 (double inscription UNIQUE constraint), §7.2 (idempotence NotificationModule par ticket_id) | ADR-003 |
| ENF-08 | Résilience asynchrone | NotificationModule | §7.2 (retry exponentiel, DLQ), §8 (événements async — garantie at-least-once) | ADR-003 |

---

## §10.2 — Synthèse

**Couverture EF : 13 / 13** — toutes les exigences fonctionnelles sont tracées dans la DCT.

**Couverture ENF : 8 / 8** — toutes les exigences non-fonctionnelles sont tracées dans la DCT.

**Exigences partiellement couvertes (zone à compléter) :**

| Réf. | Motif | Traitement prévu |
|---|---|---|
| EF-13 | La stratégie de génération et validation du QR code (choix signature symétrique vs asymétrique vs lookup base) n'est pas documentée en ADR | Traiter en TP 1.8 — rédiger ADR-004 sur ce choix |
| EF-10 / EF-11 | L'EventModule n'a pas de fiche §7 détaillée (seuls TicketModule, NotificationModule et AuthModule ont été développés en TP 1.5) | Fiche EventModule à produire en TP 1.8 si temps disponible |
| EF-04 | Le PaymentModule n'a pas de fiche §7 détaillée | Fiche PaymentModule à produire en TP 1.8 si temps disponible |

**Lecture par module (vérification de complétude) :**

| Module | Nombre d'apparitions dans la matrice | Statut |
|---|---|---|
| TicketModule | 7 lignes (EF-03, EF-05, EF-09, EF-13, ENF-01, ENF-06, ENF-07) | ✅ Couverture normale |
| NotificationModule | 3 lignes (EF-08, ENF-07, ENF-08) | ✅ Couverture normale |
| AuthModule | 6 lignes (EF-06, EF-07, EF-12, ENF-02, ENF-03, ENF-05) | ✅ Couverture normale |
| EventModule | 4 lignes (EF-01, EF-02, EF-10, EF-11) | ⚠️ Fiche §7 non produite — à compléter |
| PaymentModule | 3 lignes (EF-04, ENF-04, ENF-07) | ⚠️ Fiche §7 non produite — à compléter |

Aucun module ne dépasse 8 lignes — pas de module fourre-tout détecté. Aucun module n'est absent de la matrice — pas de module superflu.
