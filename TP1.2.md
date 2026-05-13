# TP 1.2 — Scaffolding de la DCT SupEvents : Questions & Réponses

---

# PARTIE A — Initialisation du dépôt Git

---

## A.1 — Arborescence du dépôt

Contenu du `README.md` (sommaire cliquable) :

```markdown
# SupEvents — Document de Conception Technique

## Sommaire

- [§1 — Page de garde](docs/01-page-de-garde.md)
- [§2 — Historique des révisions](docs/02-historique-revisions.md)
- [§3 — Glossaire](docs/03-glossaire.md)
- [§4 — Contexte et objectifs](docs/04-contexte-objectifs.md)
- [§5 — Périmètre et limites](docs/05-perimetre-limites.md)
- [§6 — Architecture générale](docs/06-architecture-generale/)
  - [§6.1 — Vue logique](docs/06-architecture-generale/06.1-vue-logique.md)
  - [§6.2 — Vue processus](docs/06-architecture-generale/06.2-vue-processus.md)
  - [§6.3 — Vue déploiement](docs/06-architecture-generale/06.3-vue-deploiement.md)
  - [§6.4 — Vue données](docs/06-architecture-generale/06.4-vue-donnees.md)
- [§7 — Conception détaillée](docs/07-conception-detaillee/)
- [§8 — Interfaces & contrats d'API](docs/08-interfaces-api.md)
- [§9 — Exigences transverses](docs/09-exigences-transverses.md)
- [§10 — Matrice de traçabilité](docs/10-matrice-tracabilite.md)
- [§11 — Décisions d'architecture](docs/11-decisions-architecture/)
- [§12 — Annexes](docs/12-annexes.md)
```

---

## A.2 — Conventions obligatoires

**Branches :**
```
docs/section-XX-description
```
Exemples :
- `docs/section-04-contexte`
- `docs/section-05-perimetre`
- `docs/section-03-glossaire`

Chaque membre du groupe travaille sur **sa propre branche**. Personne ne commit directement sur `main`.

**Messages de commit :**
```
docs(§XX): description courte au présent
```
Exemples :
- `docs(§04): rédaction contexte et objectifs du système`
- `docs(§03): ajout glossaire 15 termes contextualisés SupEvents`
- `docs(§05): ajout périmètre et limites techniques`

---

## A.3 — Fichier CODEOWNERS

**Question :** Renseignez le fichier `CODEOWNERS` à la racine pour assigner chaque section à son auteur principal.

**Réponse :**

```
# CODEOWNERS — SupEvents DCT
# Chaque ligne assigne un fichier/dossier à son auteur principal

docs/01-page-de-garde.md              @prenom-nom1
docs/02-historique-revisions.md       @prenom-nom1
docs/03-glossaire.md                  @prenom-nom2
docs/04-contexte-objectifs.md         @prenom-nom2
docs/05-perimetre-limites.md          @prenom-nom3
docs/06-architecture-generale/        @prenom-nom1
docs/07-conception-detaillee/         @prenom-nom3
docs/08-interfaces-api.md             @prenom-nom2
docs/09-exigences-transverses.md      @prenom-nom3
docs/10-matrice-tracabilite.md        @prenom-nom1
docs/11-decisions-architecture/       @prenom-nom2
docs/12-annexes.md                    @prenom-nom3
```

> Remplacez `@prenom-nom1/2/3` par les handles GitHub réels des membres du groupe.

---

---

# PARTIE B — Rédaction des sections §1 à §5

---

## §1 — Page de garde

**Question :** Remplissez le template de page de garde sans en modifier la structure.

**Réponse :**

```markdown
# Document de Conception Technique — SupEvents

| Champ              | Valeur                              |
|--------------------|-------------------------------------|
| **Titre**          | SupEvents — DCT                     |
| **Version**        | 0.1                                 |
| **Date**           | 13/05/2026                          |
| **Statut**         | 🟡 Brouillon                        |
| **Auteurs**        | Prénom NOM, Prénom NOM, Prénom NOM  |
| **Destinataires**  | Formateur, Pairs                    |
| **Classification** | Interne                             |
```

---

## §2 — Historique des révisions

**Question :** Créez un tableau de suivi des modifications du document. La première ligne est imposée.

**Réponse :**

```markdown
# §2 — Historique des révisions

| Version | Date       | Auteur(s) | Description des modifications              |
|---------|------------|-----------|--------------------------------------------|
| v0.1    | 13/05/2026 | Tous      | Création initiale — sections §1 à §5       |
```


## §3 — Glossaire

**Question :** Produisez un glossaire d'au minimum 15 termes techniques et acronymes du CDC, contextualisés à SupEvents (pas copiés de Wikipedia). Format imposé : `Terme | Définition | Première occurrence`. Les termes suivants sont obligatoires : SSO, OIDC, RBAC, RGPD, PCI-DSS, webhook, broker de messages, API REST, early-bird (billet), CRUD, SLA, P95, WCAG, i18n, CI/CD.

**Réponse :**

```markdown
# §3 — Glossaire

| **SSO**                | Connexion unique permettant d’accéder à SupEvents avec un compte universitaire.            |
| **OIDC**               | Protocole d’authentification utilisé avec OAuth 2.0 pour récupérer l’identité utilisateur. |
| **RBAC**               | Gestion des droits basée sur les rôles (`user`, `organizer`, `admin`).                     |
| **RGPD**               | Règlement imposant la protection et la gestion des données personnelles.                   |
| **PCI-DSS**            | Norme de sécurité des paiements respectée grâce à Stripe.                                  |
| **Webhook**            | Notification automatique envoyée par Stripe à SupEvents via HTTP POST.                     |
| **Broker de messages** | Outil (RabbitMQ) assurant l’échange asynchrone de messages entre services.                 |
| **API REST**           | API utilisant HTTP, JSON et des routes versionnées (`/api/v1/`).                           |
| **Early-bird**         | Billet à tarif réduit limité aux premiers inscrits.                                        |
| **CRUD**               | Opérations de base : créer, lire, modifier, supprimer.                                     |
| **SLA**                | Engagement de disponibilité du service (99,5 %).                                           |
| **P95**                | Temps de réponse sous lequel passent 95 % des requêtes.                                    |
| **WCAG**               | Standard d’accessibilité web pour les personnes en situation de handicap.                  |
| **i18n**               | Support de plusieurs langues dans l’application.                                           |
| **CI/CD**              | Automatisation des tests et déploiements via pipeline.                                     |
| **JWT**                | Jeton d’authentification transmis dans les requêtes API.                                   |
| **Stripe**             | Service de paiement utilisé pour les transactions bancaires.                               

## §4 — Contexte et objectifs

**Question :** Rédigez le contexte du projet du **point de vue du concepteur technique** (10-15 lignes), en traduisant les besoins métier en enjeux techniques. Identifiez ensuite les objectifs techniques implicites dans un tableau `Objectif technique | Origine dans le CDC | Niveau de priorité`.

**Réponse :**

```markdown
# §4 — Contexte et objectifs

## Contexte technique

SupEvents est une plateforme web de gestion d'événements étudiants qui doit
supporter l'ensemble du cycle de vie d'un événement : création par un organisateur
validé, publication, inscription des participants avec paiement en ligne, et
notification post-achat. Du point de vue technique, le système doit exposer un
parcours d'inscription fluide en moins de 3 étapes avec authentification SSO
universitaire, sans forcer l'étudiant à créer un compte dédié. La gestion des
paiements par carte impose une intégration Stripe conforme PCI-DSS, ce qui exclut
tout stockage de données bancaires côté SupEvents. Le contrôle d'accès est basé sur
les rôles (RBAC) avec trois niveaux distincts : étudiant, organisateur et administrateur
plateforme. La contrainte de charge (événements populaires pouvant générer des pics
d'inscriptions simultanées) impose une architecture capable de scaler horizontalement
et de gérer la concurrence sur les places disponibles via des transactions ACID.
L'accessibilité WCAG AA et le support multilingue (i18n) sont des exigences
transverses non négociables dès la conception. Enfin, la conformité RGPD impose de
minimiser la collecte de données personnelles et de documenter leur durée de
rétention.

## Objectifs techniques implicites

| Objectif technique | Origine dans le CDC | Niveau de priorité |
|---|---|---|
| Scalabilité horizontale | Pics d'inscriptions sur les événements populaires | Élevé |
| Conformité PCI-DSS | Paiement en ligne par carte bancaire | Élevé |
| Conformité RGPD | Collecte de données personnelles (email, nom) des étudiants | Élevé |
| Conformité WCAG AA | Accessibilité aux étudiants en situation de handicap | Élevé |
| Gestion de la concurrence (ACID) | Limitation des places par événement — risque de survente | Élevé |
| Authentification SSO / OIDC | "Les étudiants se connectent avec leur compte universitaire" | Élevé |
| Internationalisation (i18n) | Plateforme utilisée par des étudiants étrangers | Moyen |
| Idempotence des webhooks Stripe | Webhooks pouvant être reçus en double en cas de retry Stripe | Élevé |
| Pipeline CI/CD | Livraisons fréquentes tout au long de la formation | Moyen |
| Monitoring et alerting | SLA de disponibilité à maintenir sur la plateforme | Moyen |
```

---

## §5 — Périmètre et limites

**Question :** Étape 1 : extrayez du CDC ce qui est dans et hors périmètre. Étape 2 : ajoutez au minimum 3 limites de conception technique décidées par le groupe (non mentionnées dans le CDC).

**Réponse :**

```markdown
# §5 — Périmètre et limites

## Étape 1 — Dans et hors périmètre

| Dans le périmètre | Hors périmètre |
|---|---|
| Création et gestion d'événements étudiants par des organisateurs validés | Application mobile native (iOS / Android) |
| Inscription à un événement (billet gratuit ou payant) | Streaming vidéo ou diffusion live des événements |
| Paiement en ligne par carte via Stripe | Gestion de la billetterie physique (impression, scan QR en présentiel) |
| Authentification SSO via compte universitaire (OIDC) | Gestion des identités et annuaire utilisateurs (délégué au fournisseur SSO) |
| Tableau de bord organisateur (participants, exports, KPI) | Système de comptabilité ou facturation des organisateurs |
| Administration plateforme (validation/révocation organisateurs) | Intégration avec des outils tiers de marketing (Mailchimp, HubSpot…) |
| Notifications email et push post-inscription | Réseau social ou messagerie entre étudiants |
| Catalogue d'événements consultable sans authentification | Vente de merchandising ou produits dérivés |
| Gestion des billets early-bird avec quota et date d'expiration | Support d'un programme de fidélité ou de points |

## Étape 2 — Limites de conception technique

| Limite technique | Justification |
|---|---|
| **Migration de données depuis un système existant : hors périmètre** | SupEvents est créé from scratch, aucune base de données existante identifiée dans le CDC. Toute reprise de données historiques serait un projet à part entière. |
| **Support des navigateurs IE11 et Edge Legacy : hors périmètre** | Navigateurs en fin de vie (EOL). La cible utilisateur est une population étudiante équipée de navigateurs modernes (Chrome, Firefox, Safari ≥ 2022). Supporter IE11 imposerait des compromis sur l'architecture front-end incompatibles avec les exigences WCAG AA. |
| **Internationalisation limitée au français et à l'anglais (v1)** | Le support i18n complet (arabe, chinois, RTL…) représente une charge de travail disproportionnée pour la v1. La structure i18n sera posée dès le départ pour permettre l'ajout de langues ultérieurement, mais seuls FR et EN seront livrés. |
| **Haute disponibilité multi-région : hors périmètre (v1)** | Un déploiement multi-région (Europe + Amérique) impliquerait une complexité opérationnelle (réplication PostgreSQL, latence cross-région) non justifiée pour une plateforme à usage étudiant national. Le SLA de 99,5 % sera atteint par redondance au sein d'une seule région cloud. |
| **API publique ouverte à des développeurs tiers : hors périmètre (v1)** | L'API REST est conçue pour usage interne (SPA + mobile web). L'ouverture à des intégrateurs tiers nécessiterait une gestion de clés API, des quotas par client et une documentation publique (portail développeur) qui dépassent le cadre de ce projet. |
```

---

---

# PARTIE C — Première Pull Request

---

## C.1 — Workflow Git

**Question :** Décrivez le workflow de Pull Request à suivre pour intégrer les sections dans `main`.

**Réponse :**

1. Chaque membre ouvre une **PR** de sa branche (`docs/section-XX-description`) vers `main`
2. Les PRs individuelles sont regroupées : le groupe crée une branche d'intégration `docs/tp1.2-integration` puis une **PR finale** vers `main`
3. Le groupe assigné par l'intervenant effectue la **revue de code** de la PR

Messages de commit types pour ce TP :
```
docs(§01): ajout page de garde SupEvents v0.1
docs(§02): initialisation historique des révisions
docs(§03): rédaction glossaire 17 termes contextualisés SupEvents
docs(§04): rédaction contexte et objectifs techniques implicites
docs(§05): rédaction périmètre, hors-périmètre et 5 limites techniques
```

---

## C.2 — Critères de revue (commentaires constructifs)

**Question :** En tant que groupe relecteur, rédigez au minimum 2 commentaires constructifs sur la PR du groupe assigné. Un commentaire constructif pointe un problème précis et suggère une amélioration.

**Réponse :**

Voici 2 exemples de commentaires constructifs conformes aux critères :

---

**Commentaire 1 — sur `docs/03-glossaire.md` :**

> ⚠️ La définition de **RBAC** est trop générique : "système de contrôle d'accès basé sur les rôles" pourrait s'appliquer à n'importe quelle application. Elle ne mentionne pas les trois rôles concrets de SupEvents (`user`, `organizer`, `admin`) ni ce que chaque rôle peut faire concrètement sur la plateforme. Suggestion : reformuler en précisant les rôles et les permissions associées dans le contexte SupEvents (ex : "seul un `organizer` peut créer un événement, un `user` peut seulement s'inscrire").

---

**Commentaire 2 — sur `docs/04-contexte-objectifs.md` :**

> ⚠️ Le tableau des objectifs techniques implicites ne contient que 3 entrées alors que le CDC implique au moins 6 enjeux techniques distincts (scalabilité, PCI-DSS, RGPD, WCAG, concurrence sur les places, SSO). De plus, l'objectif "Conformité RGPD" est listé sans mentionner son origine précise dans le CDC — quelle phrase du CDC l'implique ? Suggestion : compléter le tableau avec les objectifs manquants et renseigner la colonne "Origine dans le CDC" avec une citation ou référence précise pour chaque ligne.

---
