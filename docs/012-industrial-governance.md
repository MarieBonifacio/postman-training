---

title: "Industrialisation & Governance"
sidebar\_position: 11
slug: /postman/industrialisation-governance
-------------------------------------------

# 11. Industrialisation & Governance 🛠️📏

> **Pourquoi ce chapitre ?** Une API en production n’est pas qu’un code qui tourne : c’est un contrat public, des conventions de nommage, un workflow de validation, des outils de lint, et des garde‑fous automatiques. Sans gouvernance, chaque équipe crée ses propres règles ➜ chaos.

Objectifs :

1. Mettre en place un **Style Guide** API (naming, statuts, pagination, version header).
2. Linter automatiquement avec **Spectral** et **ReviewBot** sur chaque Pull Request.
3. Gérer le **workflow PR** (Design First ➜ Code ➜ Tests ➜ Doc).
4. Appliquer **Semantic Release** et **Versioning** (SemVer).
5. Partager un **Playbook incident** & **Change Policy**.

---

## 11.1 Style Guide API – pourquoi et quoi ?

| Catégorie       | Convention               | Exemple                          |
| --------------- | ------------------------ | -------------------------------- |
| HTTP Verbs      | CRUD mapping             | `GET /users` `POST /users`       |
| Resource naming | Kebab‑case pluriel       | `/user‑profiles`                 |
| Id format       | UUID v4                  | `id="550e8400‑e29b"`             |
| Status codes    | 2xx / 4xx / 5xx ‑ précis | `422` validation, `409` conflict |
| Error payload   | JSON Problem Details     | `{type,title,detail}`            |
| Pagination      | Cursor (link headers)    | `Link: <...cursor>; rel=next`    |
| Versioning      | Header `API‑Version`     | `API‑Version: 2`                 |

### Outil : Spectral ruleset

```yaml
extends: spectral:oas
rules:
  acme‑resource‑naming:
    description: "Resources must be kebab plural."
    message: "{{property}} is not kebab plural"
    given: $.paths[*]~
    then:
      function: pattern
      functionOptions:
        match: ^/[a‑z]+(-[a‑z]+)*s$
```

---

## 11.2 Lint continue avec Spectral & ReviewBot

### 11.2.1 Spectral CLI

```bash
spectral lint openapi.yaml --fail-severity error
```

Ajoutez dans CI :

```yaml
- name: API Lint
  run: spectral lint openapi.yaml
```

### 11.2.2 ReviewBot (Postman)

* Plan Business : activez ReviewBot ➜ protégé branch `main`.
* ReviewBot commente les PR (missing description, untagged examples).

> **Pourquoi ?** Les erreurs sont bloquées **avant** d’atteindre QA/Prod.

---

## 11.3 Workflow PR – Design First

1. **Designer** propose spec OpenAPI dans `/spec/openapi.yaml`.
2. PR ouvre → Spectral Lint + Tests Postman CLI contre *mock server*.
3. ReviewBot ajoute commentaires style guide.
4. Merge ➜ pipeline génère SDK + documentation.

| Étape          | Outil       | Gate        |
| -------------- | ----------- | ----------- |
| Lint OAS       | Spectral    | `must‑pass` |
| Unit tests     | Jest        | ≥ 90 % cov  |
| Contract tests | Postman CLI | 0 error     |
| Docs build     | Docusaurus  | build OK    |

---

## 11.4 Semantic Release & Versioning

### 11.4.1 Conventional Commits → release notes auto

```bash
feat(auth): add PKCE flow
fix(user): return 422 on invalid email
chore(ci): add ReviewBot lint
```

`semantic‑release` génère v1.3.0 + changelog.

### 11.4.2 Header de version + Deprecation window

* `API‑Version: 3` requis ⇒ clients refusent v2.
* Banner Deprecation 6 mois avant removal via docs.

---

## 11.5 Playbook incident & change policy

| Document          | Contenu clé                      | Lien                         |
| ----------------- | -------------------------------- | ---------------------------- |
| **Runbook**       | Pas‑à‑pas rollback, feature flag | `/docs/runbooks/rollback.md` |
| **Change policy** | Fenêtre freeze, PR template      | `/docs/policy/change.md`     |
| **SLO doc**       | p95, error budget, alert routing | `/docs/slo.md`               |

---

## 11.6 Check‑list Governance

* [ ] Style guide publié (Markdown) + ruleset Spectral.
* [ ] Spectral lint dans CI bloque severity≥error.
* [ ] ReviewBot activé (Business Plan).
* [ ] Workflow PR Design First appliqué.
* [ ] Semantic‑Release crée tags + changelog.
* [ ] Docs Deprecation banner 6 mois avant break change.
* [ ] Runbook rollback disponible.

> 🎉 Fin du parcours « De débutant à expert Postman ». Vous disposez maintenant d’un workflow complet, reproductible, gouverné et documenté.

