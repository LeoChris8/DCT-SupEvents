# Revue croisée TP 1.4 — Cohérence §6.4 ↔ §8

**Groupe relecteur :** LeoChris8  
**Groupe relu :** *(à compléter)*  
**Lien repo relu :** *(à compléter)*  
**Date :** *(à compléter)*

---

## Checklist de cohérence

| Point de contrôle | OK | À corriger |
|---|---|---|
| Toutes les entités référencées dans les payloads d'événements existent dans le dictionnaire de données | ☐ | ☐ |
| Tous les `id` côté API sont du même type que les `id` côté base (UUID des deux côtés ou cohérence explicite) | ☐ | ☐ |
| Chaque contrainte `UNIQUE` en base a un code d'erreur `409 Conflict` documenté côté API | ☐ | ☐ |
| Les champs marqués sensibles RGPD ne fuitent pas dans les payloads d'événements (sauf chiffrement explicite) | ☐ | ☐ |
| Le webhook Stripe est bien authentifié par HMAC, pas par JWT | ☐ | ☐ |
| Chaque événement asynchrone a un consommateur identifié — pas d'événement orphelin | ☐ | ☐ |

---

## Commentaires

*(À remplir en classe — note ici ce qui est bien fait et ce qui est à corriger pour chaque point)*

---

## Conclusion

*(À remplir en classe — 2-3 lignes de synthèse générale)*
