# TP 1.7 — Audit qualité & Pipeline CI


## Catégorie 1 — Structure

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| S1 | Page de garde présente avec titre, date, version, auteurs | 2 | Page de garde complète, version et auteurs identifiés. | — |
| S2 | Table des matières à jour et cohérente avec les sections | 2 | TDM générée automatiquement, tous les liens fonctionnent. | — |
| S3 | Numérotation des sections cohérente (§1 à §11) | 1 | La numérotation saute de §6.3 à §6.5 — §6.4 absent de la TDM bien que le contenu existe. | Renommer le fichier `06-4-donnees.md` et l'ajouter explicitement dans la TDM. |
| S4 | Historique des révisions présent et renseigné | 0 | Aucun historique de révisions trouvé — ni en page de garde ni en fichier dédié. | Créer une section `## Historique des révisions` en page de garde avec au minimum les 3 dernières versions. |
| S5 | Glossaire présent avec au moins 10 termes définis | 2 | 14 termes définis, bien structurés en tableau. | — |
| S6 | Format des fichiers .md homogène (encodage, saut de ligne) | 1 | Deux fichiers utilisent CRLF au lieu de LF (détecté via git diff). | Lancer `git config core.autocrlf input` et normaliser via `.gitattributes`. |

---

## Catégorie 2 — Complétude

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| C1 | §6.1 — Diagramme C4 Context présent et complet | 2 | Les 3 acteurs et les 4 systèmes tiers sont représentés avec labels de protocole. | — |
| C2 | §6.2 — Diagramme C4 Containers avec technologies et ports | 2 | Technologies et protocoles/ports présents sur chaque lien. Flux sync/async distingués. | — |
| C3 | §6.2 — Diagramme de composants API Backend | 1 | Les 6 modules sont présents mais `UserModule` n'a aucune dépendance vers PostgreSQL alors qu'elle est évidente. | Ajouter le lien `UserModule → PostgreSQL (TCP :5432)` dans le diagramme B1. |
| C4 | §6.2 — Séquence inscription nominale complète | 2 | Tous les participants, fragments alt et opt présents, flèches sync/async correctes. | — |
| C5 | §6.2 — Séquence échec de paiement avec alt timeout/refus | 2 | Les deux cas sont traités avec libération de réservation et notification à l'étudiant. | — |
| C6 | §8 — Spécification des endpoints API (routes principales) | 1 | Les endpoints CRUD des événements sont documentés mais les routes webhook Stripe et auth SSO sont absentes. | Ajouter `POST /webhooks/stripe` et `GET /auth/callback` dans §8. |
| C7 | §10 — Matrice de traçabilité exigences ↔ composants renseignée | 0 | La matrice existe mais 6 exigences sur 18 n'ont aucun composant associé (cellules vides). | Compléter les lignes EF-07 à EF-12 avec les modules responsables. |

---

## Catégorie 3 — Cohérence

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| CO1 | Terminologie homogène entre §6, §7 et §8 | 1 | Le concept de réservation est appelé `Reservation` en §6.2, `Booking` en §7.3 et `TicketDraft` dans §8. Trois noms pour la même entité. | Choisir `Reservation` (terme du glossaire) et faire un find/replace dans §7.3 et §8. |
| CO2 | Les modules cités en §10 existent tous en §7 | 2 | Vérification croisée effectuée — les 6 modules sont présents dans les deux sections. | — |
| CO3 | Les diagrammes sont cohérents entre eux | 2 | Le diagramme containers et le diagramme de composants représentent bien la même architecture. | — |
| CO4 | Les acteurs du diagramme Context correspondent aux personas du §3 | 1 | Le diagramme Context inclut un acteur "Organisateur" mais §3 ne définit que "Étudiant" et "Administrateur". | Ajouter le persona Organisateur dans §3, ou retirer l'acteur si non pertinent. |
| CO5 | Les exigences du §5 sont toutes référencées dans §10 | 1 | 3 exigences non fonctionnelles (ENF-04, ENF-07, ENF-09) citées en §5 sont absentes de §10. | Ajouter ces exigences à la matrice même sans composant unique associé. |
| CO6 | Les ADR référencent des décisions cohérentes avec l'architecture | 2 | Les 4 ADR présents sont alignés avec les choix technologiques visibles en §6. | — |

---

## Catégorie 4 — Lisibilité

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| L1 | Phrases courtes (< 25 mots en moyenne dans les sections techniques) | 1 | §7.2 contient plusieurs phrases dépassant 40 mots dans la description du `PaymentModule`. | Découper les phrases longues de §7.2 en deux phrases courtes. |
| L2 | Voix active privilégiée dans les descriptions de comportement | 1 | §9 utilise majoritairement la voix passive : "est géré par", "est validé par". | Reformuler en actif : "Le middleware JWT valide le token" plutôt que "Le token est validé par le middleware". |
| L3 | Mots-clés en gras dans les paragraphes critiques | 2 | Les termes importants sont mis en évidence dans §6 et §7. | — |
| L4 | Diagrammes accompagnés d'un paragraphe introductif et d'un paragraphe de lecture | 2 | Chaque diagramme dispose des deux paragraphes. Bonne qualité rédactionnelle. | — |
| L5 | Pas d'abréviations non définies au premier usage | 1 | "AMQP", "OIDC" et "SPA" utilisées sans définition au premier usage en §6.1 — elles ne sont définies qu'en glossaire. | Ajouter une parenthèse explicative au premier usage : "OIDC (OpenID Connect)". |
| L6 | Titres de sections informatifs | 2 | Tous les titres sont spécifiques et porteurs de sens. | — |

---

## Catégorie 5 — Maintenabilité

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| M1 | Tous les liens internes fonctionnent | 1 | 2 liens internes morts : `[voir §6.4](#section-64)` pointe vers une ancre incorrecte. | Corriger l'ancre en `(#64--modèle-de-données)` selon la syntaxe GitHub Markdown. |
| M2 | Tous les liens externes répondent (HTTP 200) | 1 | Le lien vers la doc Stripe dans §8 retourne une 404 (URL versionnée obsolète). | Remplacer par l'URL stable `https://stripe.com/docs/api`. |
| M3 | Métadonnée "dernière mise à jour" présente dans chaque fichier .md | 0 | Aucun fichier ne porte de métadonnée de date. Impossible de savoir quelle section est à jour sans consulter git log. | Ajouter `> Dernière mise à jour : YYYY-MM-DD` en pied de chaque fichier .md. |
| M4 | Le dépôt dispose d'un fichier CODEOWNERS | 0 | Fichier CODEOWNERS absent au moment de l'audit. | Créer `.github/CODEOWNERS` avec une règle par section principale. |
| M5 | Un pipeline CI valide automatiquement les fichiers .md | 0 | Aucun workflow présent dans `.github/workflows/`. | Créer `docs-quality.yml` avec lint Markdown, vérification des liens et rendu Mermaid. |

---

## Synthèse

### Top 5 des défauts les plus impactants

1. **[M3 + M4 + M5] Absence totale d'outillage de maintenabilité** — Pas de date de mise à jour, pas de CODEOWNERS, pas de CI. Il est impossible de savoir qui est responsable d'une section et si elle est à jour. Toute modification future peut dégrader la qualité sans détection. *Impact : critique sur la durée du projet.*

2. **[C7] Matrice de traçabilité incomplète** — 6 exigences sur 18 sans composant associé. Lors de la soutenance, impossible de démontrer que toutes les exigences sont couvertes par l'implémentation. *Impact : fort sur l'évaluation du §10.*

3. **[CO1] Terminologie incohérente Reservation/Booking/TicketDraft** — Un même concept nommé différemment selon les sections. Un développeur arrivant dans l'équipe ne peut pas savoir si ces trois termes désignent la même entité ou des concepts distincts. *Impact : fort sur la compréhension du modèle de données.*

4. **[S4] Absence d'historique des révisions** — Sans historique, la DCT ne permet pas de comprendre ce qui a changé entre deux versions. Lors d'un audit externe, l'absence de traçabilité est un signal de manque de rigueur. *Impact : modéré mais visible immédiatement.*

5. **[CO4] Persona Organisateur absent de §3** — Le diagramme Context mentionne un acteur non documenté dans les personas. Source de confusion pour un lecteur qui lit les sections dans l'ordre. *Impact : modéré sur la cohérence globale.*

### Plan de correction (priorité et traçage)

| Priorité | Critère | Défaut | Impact | Coût | Action |
|----------|---------|--------|--------|------|--------|
| 1 | M4 | CODEOWNERS absent | Élevé | Faible | Créer `.github/CODEOWNERS` |
| 2 | M5 | Pipeline CI absent | Élevé | Faible | Créer `docs-quality.yml` |
| 3 | CO1 | Terminologie incohérente | Élevé | Faible | Find/replace `Booking` → `Reservation` dans §7.3 et §8 |
| 4 | S4 | Historique révisions absent | Modéré | Faible | Ajouter tableau historique en page de garde |
| 5 | M3 | Pas de date de mise à jour | Modéré | Faible | Ajouter pied de page dans chaque .md |
| 6 | C7 | Matrice traçabilité incomplète | Élevé | Élevé | Reporter en TP 1.8 — dette documentée |
| 7 | CO4 | Persona Organisateur absent | Modéré | Moyen | Reporter en TP 1.8 |
| 8 | M1/M2 | Liens morts | Modéré | Faible | Corriger les 3 liens identifiés |

---

## .github/CODEOWNERS

```
# CODEOWNERS — DCT SupEvents
#
# Répartition basée sur les contributions réelles (git log) :
#   - @alice-dupont  : architecture (§6), modules (§7), ADR
#   - @bob-martin    : exigences (§5), API (§8), traçabilité (§10)
#   - @clara-nguyen  : page de garde, glossaire, transverses (§9), audit
#
# Mis à jour : 2026-05-13

# Fallback global — tout fichier non couvert spécifiquement
*                                           @alice-dupont @bob-martin @clara-nguyen

# Fichiers de configuration et CI
.github/workflows/                          @alice-dupont @bob-martin
.markdownlint.json                          @clara-nguyen

# Page de garde et README
README.md                                   @clara-nguyen
docs/00-page-de-garde.md                    @clara-nguyen

# §1 — Glossaire
docs/01-glossaire.md                        @clara-nguyen

# §5 — Exigences fonctionnelles et non fonctionnelles
docs/05-exigences.md                        @bob-martin

# §6 — Architecture
docs/06-architecture.md                     @alice-dupont
docs/06-1-context.md                        @alice-dupont
docs/06-2-containers.md                     @alice-dupont
docs/06-3-composants.md                     @alice-dupont
docs/06-4-donnees.md                        @alice-dupont @bob-martin

# §7 — Modules fonctionnels
docs/07-modules.md                          @alice-dupont

# §8 — API
docs/08-api.md                              @bob-martin

# §9 — Aspects transverses
docs/09-transverses.md                      @clara-nguyen

# §10 — Traçabilité
docs/10-tracabilite.md                      @bob-martin

# §11 — ADR (chaque ADR à son auteur principal)
docs/adr/                                   @alice-dupont
docs/adr/ADR-001-choix-framework-backend.md @alice-dupont
docs/adr/ADR-002-choix-broker-messages.md   @alice-dupont
docs/adr/ADR-003-strategie-auth-oidc.md     @bob-martin
docs/adr/ADR-004-stockage-objets.md         @clara-nguyen

# Audit et CI
docs/audit-qualite.md                       @clara-nguyen @bob-martin
docs/ci-validation.md                       @alice-dupont
```

---

## .markdownlint.json

```json
{
  "MD013": false,
  "MD033": false,
  "MD041": false,
  "MD024": false,
  "MD036": false
}
```

---

## .github/workflows/docs-quality.yml

```yaml
name: DCT Quality Checks

on:
  pull_request:
    paths:
      - '**/*.md'
      - '.markdownlint.json'
  push:
    branches:
      - main
    paths:
      - '**/*.md'

jobs:
  markdown-lint:
    name: Markdown Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install markdownlint-cli
        run: npm install -g markdownlint-cli
      - name: Lint Markdown files
        run: |
          markdownlint '**/*.md' \
            --ignore node_modules \
            --config .markdownlint.json

  link-check:
    name: Check Links
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install markdown-link-check
        run: npm install -g markdown-link-check
      - name: Create link-check config
        run: |
          cat > .mlc-config.json << 'EOF'
          {
            "timeout": "5000ms",
            "retryOn429": true,
            "retryCount": 2,
            "aliveStatusCodes": [200, 206],
            "ignorePatterns": [
              { "pattern": "^https://localhost" },
              { "pattern": "^http://localhost" }
            ]
          }
          EOF
      - name: Check links
        run: |
          find . -name '*.md' -not -path './node_modules/*' \
            | xargs -I {} markdown-link-check {} \
              --config .mlc-config.json --quiet

  mermaid-validate:
    name: Validate Mermaid Diagrams
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install mermaid-cli
        run: npm install -g @mermaid-js/mermaid-cli
      - name: Install Chromium
        run: npx puppeteer browsers install chrome
      - name: Extract and validate Mermaid blocks
        run: |
          FAIL=0
          while IFS= read -r mdfile; do
            python3 - "$mdfile" <<'PYEOF'
          import sys, re, os, subprocess, tempfile
          mdfile = sys.argv[1]
          with open(mdfile, 'r', encoding='utf-8') as f:
              content = f.read()
          blocks = re.findall(r'```mermaid\n(.*?)```', content, re.DOTALL)
          fail = 0
          for i, block in enumerate(blocks):
              with tempfile.NamedTemporaryFile(suffix='.mmd', mode='w', delete=False) as tmp:
                  tmp.write(block)
                  tmp_path = tmp.name
              out_path = tmp_path.replace('.mmd', '.svg')
              result = subprocess.run(
                  ['mmdc', '-i', tmp_path, '-o', out_path],
                  capture_output=True, text=True
              )
              if result.returncode != 0:
                  print(f"Diagram {i+1} in {mdfile} FAILED")
                  print(result.stderr)
                  fail = 1
              else:
                  print(f"Diagram {i+1} in {mdfile} OK")
              os.unlink(tmp_path)
              if os.path.exists(out_path):
                  os.unlink(out_path)
          sys.exit(fail)
          PYEOF
            [ $? -ne 0 ] && FAIL=1
          done < <(find . -name '*.md' -not -path './node_modules/*')
          [ $FAIL -ne 0 ] && echo "Mermaid validation failed" && exit 1
          echo "All Mermaid diagrams OK"
```

---

## ci-validation.md

### Procédure

Conformément aux instructions du TP 1.7 (§C.4), une PR de test a été ouverte pour vérifier le pipeline dans les trois cas attendus.

**Étape 1 — Défauts volontaires introduits dans `docs/08-api.md` :**

- Lien mort : `[documentation Stripe](https://stripe.com/docs/api/v2-obsolete-test)`
- Bloc Mermaid invalide (flèche incomplète) :

```
flowchart TD
    A --> B
    B --> C -->
```

**Étape 2 — Résultats CI observés :**

| Job | Statut | Détail |
|-----|--------|--------|
| Markdown Lint | Passed | Aucune violation de linting |
| Check Links | Failed | 1 lien mort dans `docs/08-api.md` |
| Validate Mermaid Diagrams | Failed | Erreur de syntaxe dans `docs/08-api.md` |

La PR était bloquée (merge impossible). `@bob-martin` (CODEOWNER de `docs/08-api.md`) a été automatiquement ajouté comme reviewer.

**Étape 3 — Correction et passage au vert :**

- URL corrigée : `https://stripe.com/docs/api`
- Bloc Mermaid corrigé (flèche complétée)

| Job | Statut |
|-----|--------|
| Markdown Lint | Passed |
| Check Links | Passed |
| Validate Mermaid Diagrams | Passed |

La PR a pu être mergée après correction.

### Références commits

| Rôle | Hash |
|------|------|
| Commit introduisant le défaut | `b1d3e9f` |
| Commit corrigeant le défaut | `c72a881` |
