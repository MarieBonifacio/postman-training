---

title: "Mock Servers, Service Virtualization & Chaos Testing"
sidebar\_position: 9
slug: /postman/mock-chaos
-------------------------

# 9. Mock Servers, Service Virtualization & Chaos Testing 🧪

> **Pourquoi ce chapitre ?** Vos front‑end devs n’attendent plus que l’API soit prête ; vos tests QA tournent même quand le back‑end est down ; et vous simulez les pires conditions réseau **avant** la production.

Objectifs :

1. Créer des **Mock Servers** Postman pour front‑end et intégration continue.
2. Utiliser la **Service Virtualization** (matching rules, dynamic responses).
3. Introduire du **Chaos Testing** : latence, erreurs 5xx, pannes régionales.
4. Intégrer les mocks dans les tests automatisés et CI.
5. Publier un **Sandbox public** pour partenaires externes.

---

## 9.1 Pourquoi mocker ?

| Besoin                             | Sans mock                 | Avec mock Postman                 |
| ---------------------------------- | ------------------------- | --------------------------------- |
| Débloquer front‑end avant back‑end | Sprint bloqué             | URL Mock renvoie JSON contractuel |
| Démonstration à un client          | Installer backend local   | Simple URL publique, zéro infra   |
| Tests QA sur data stable           | Data Prod évolue          | Réponses figées, versionnées      |
| Tests de résilience                | Doit mettre Prod en panne | Mock renvoie pannes contrôlées    |

---

## 9.2 Créer un Mock Server – pas‑à‑pas

1. Sélectionner la collection **Postman Basics** ➜ **Mock Servers ▸ + New Mock**.
2. **Environment** : `staging`.
3. **Name** : `Echo‑Mock`.
4. **Mock server type** : *Private* (Auth API Key) ou *Public* pour partenaires.
5. Cocher **“Save example responses”** → Postman persistera les bodies que vous envoyez.
6. **Create Mock** → URL générée `https://<uuid>.mock.pstmn.io`.

> *Tips :* ajoutez le header `x-mock-match-request-body: true` si vous avez plusieurs exemples `POST /user`.

### Facturation Mock

* Free : 10 Mock Servers / 1 000 req/mois.
* Payant : 100 Mock / 100 000 req/mois (\~0,00005 \$/req au‑delà).

---

## 9.3 Service Virtualization – matching & dynamic

| Feature             | Description                       | Exemple                           |
| ------------------- | --------------------------------- | --------------------------------- |
| **Path** matching   | URL exacte ou templating `:id`    | `/users/:id`                      |
| **Query** matching  | Différencie `?status=active`      | `/users?status=active`            |
| **Header** matching | `x-api-version: 2`                | Versioning header                 |
| **Body** matching   | `x-mock-match-request-body: true` | `POST /orders` différents schémas |
| **Dynamic tokens**  | Injecter valeur de requête        | `{{request.body.userId}}`         |

#### Exemple de réponse dynamique

```json
{
  "id": "{{request.body.userId}}",
  "createdAt": "{{$isoTimestamp}}"
}
```

*Pourquoi ?* Permet au front-end de voir sa propre saisie reflétée.

---

## 9.4 Chaos Testing avec Mock Servers

### Latence artificielle

```js
if (pm.mock) {
  pm.mock.setDelay(3000); // 3 s
}
```

### Erreur aléatoire 30 %

```js
if (Math.random() < 0.3) {
  pm.mock.setStatus(500);
}
```

### Coupe‑circuit régional

*Créer un mock clone* nommé `Echo‑Mock‑eu‑down` et renvoyer **503** fixe → configurez DNS failover pour tester fallback CDN.

| Scénario chaos   | Impact testé      | Action correctrice attendue     |
| ---------------- | ----------------- | ------------------------------- |
| Latence 3 s      | Timeout front‑end | Affichage spinner, retry client |
| 30 % erreurs 500 | Circuit Breaker   | Fallback cache, alerte SRE      |
| 503 EU only      | DNS Failover      | Route 53 bascule vers US‑East   |

---

## 9.5 Intégrer mock dans CI

### CLI substitution baseUrl

```yaml
postman_collection_run:
  script:
    - postman collection run "Postman Basics" \
        --env "mock-env" \
        --export-environment envs/mock-env.json
```

`mock-env.json` contient `baseUrl` = `https://<uuid>.mock.pstmn.io`.

### Contract parity test

Comparez mock vs Swagger :

```bash
spectral lint openapi.yaml --mockBase https://<uuid>.mock.pstmn.io
```

---

## 9.6 Publier un Sandbox public

1. Passer le mock en **Public** → URL accessible sans clé API.
2. Ajouter README : description, exemples `curl`.
3. Lister dans **Postman Public API Network** pour visibilité.

> **Astuce sécurité :** jamais de données sensibles dans les payloads exemples.

---

## 9.7 Check‑list Mock & Chaos

* [ ] Mock Server `Echo‑Mock` créé, URL copiée.
* [ ] Matching path + query + header testé.
* [ ] Réponse dynamique reflète `userId`.
* [ ] Scénario Chaos latence + erreur 30 % configuré.
* [ ] CI utilise `mock-env` pour tests isolés.
* [ ] Sandbox public documenté, pas de données sensibles.

> 👉 **Page suivante** – Génération de documentation & Publication : doc auto Postman, Docusaurus, badges, Postman API Network, versioning.
