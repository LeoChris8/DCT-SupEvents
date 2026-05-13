# Index des Architecture Decision Records — SupEvents

Ce dossier contient les décisions d'architecture structurantes du projet SupEvents, au format Nygard.

## Règle d'usage

Un ADR documente une décision prise, pas une question ouverte. Une fois accepté, un ADR ne se modifie pas : il se déprécie ou se remplace par un nouvel ADR. Toute évolution de décision donne lieu à un nouvel ADR numéroté à la suite.

---

## Liste des ADR

| N° | Titre | Statut | Date | Section DCT |
|---|---|---|---|---|
| [ADR-001](./ADR-001.md) | Les tokens JWT stateless sont utilisés comme mécanisme d'authentification applicatif principal | ✅ Accepté | 2025-05-13 | §7.3 — AuthModule |
| [ADR-002](./ADR-002.md) | Le verrou pessimiste SELECT FOR UPDATE est utilisé pour gérer la concurrence sur la jauge de places | ✅ Accepté | 2025-05-13 | §7.1 — TicketModule |
| [ADR-003](./ADR-003.md) | Le backoff exponentiel avec dead letter queue est la stratégie de retry des événements asynchrones RabbitMQ | ✅ Accepté | 2025-05-13 | §7.2 — NotificationModule |

---

## Template

Le template utilisé pour tous les ADR est disponible dans [template.md](./template.md).
