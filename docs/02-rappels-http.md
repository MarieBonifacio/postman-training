---

title: "Rappels HTTP & Architecture API"
sidebar\_position: 1
---------------------------

# 1. Rappels HTTP & Architecture API

Une courte remise à niveau sur le protocole **HTTP/HTTPS** et les formes d’API modernes : indispensable avant d’ouvrir Postman.

---

## 1.1 Méthodes HTTP & Idempotence

| Verbe      | Idempotent ? | Cacheable ? | Sémantique            | Exemple                |
| ---------- | ------------ | ----------- | --------------------- | ---------------------- |
| **GET**    | ✅            | ✅           | Lecture de ressource  | `/books?author=Asimov` |
| **POST**   | ❌            | ❌           | Création (ou RPC)     | `/books`               |
| **PUT**    | ✅            | ❌           | Remplacement complet  | `/books/42`            |
| **PATCH**  | ✅            | ❌           | Mise à jour partielle | `/books/42`            |
| **DELETE** | ✅            | ❌           | Suppression           | `/books/42`            |

> 💡 **Idempotence** : une requête idempotente peut être répétée sans effet de bord supplémentaire (utile pour retry). *En pratique : GET/PUT/PATCH/DELETE le sont, POST ne l’est pas.*

### Exercices Postman

1. Créez une collection `HTTP Basics`.
2. Ajoutez un `GET https://postman-echo.com/get?msg=hello` et observez le *status code*.
3. Dupliquez‑la, changez le verbe en `PUT` : résultat attendu **404** (endpoint inexistant) — l’intérêt est de distinguer syntaxe et sémantique.

---

## 1.2 Codes de statut – Lecture ascendante

| Catégorie | But principal  | Exemples utiles                   | Notes pratiques                              |
| --------- | -------------- | --------------------------------- | -------------------------------------------- |
| **1xx**   | Information    | 100, 101                          | Rare en REST ; 101 ↔ upgrade WebSocket       |
| **2xx**   | Succès         | 200, 201, 204, 206                | 206 = Range (downloads partiels)             |
| **3xx**   | Redirection    | 301, 302, 307, 308                | 307/308 conservent le verbe                  |
| **4xx**   | Erreur client  | 400, 401, 403, 409, 422, 429, 451 | 422 pour validations • 429 Too Many Requests |
| **5xx**   | Erreur serveur | 500, 502, 503, 504, 508           | 503 + Retry‑After pour maintenance           |

> 🔍 **Cas pratique Postman – Conditional GET**
>
> 1. Faites un premier appel GET pour récupérer l’en‑tête `ETag`.
> 2. Relancez le même GET mais ajoutez `If-None-Match: <etag>` dans l’onglet *Headers* : vous devriez recevoir **304 Not Modified**.
>
> *Utilité : économiser la bande passante et vérifier la mise en cache côté API.*

---

## 1.3 En‑têtes HTTP indispensables

| Header             | Usage terrain                             | Dans Postman                                                       |
| ------------------ | ----------------------------------------- | ------------------------------------------------------------------ |
| `Accept`           | Format réponse voulu (`application/json`) | Onglet **Headers** (auto‑complétion)                               |
| `Content-Type`     | Format corps envoyé                       | Sélectionné automatiquement quand vous changez *Body → raw → JSON* |
| `Authorization`    | Authentification (Bearer, Basic)          | Onglet **Auth** gère cet header pour vous                          |
| `Cache-Control`    | TTL / contrôle intermédiaires             | Testez différentes valeurs dans les requêtes GET                   |
| `X-Correlation-ID` | Trace transaction distrib.                | Générez `{{$guid}}` via Postman pour chaque call                   |

---

## 1.4 Modèles d’architecture d’API

| Paradigme     | Transport      | Schéma/Contrat        | Avantages                                   | Inconvénients                         |
| ------------- | -------------- | --------------------- | ------------------------------------------- | ------------------------------------- |
| **REST**      | HTTP           | OpenAPI / JSON Schema | Simple, cache HTTP natif                    | Multiplicité d’endpoints              |
| **GraphQL**   | HTTP (POST)    | SDL GraphQL           | Un endpoint unique, typage fort côté client | Complexité serveur & tooling          |
| **gRPC**      | HTTP/2 (proto) | Protobuf              | Perf, streaming bidirectionnel              | Certificats + tooling Desktop requis  |
| **WebSocket** | TCP (upgrade)  | —                     | Temps réel bidirectionnel                   | Pas de cache ni proxies HTTP standard |

### Quand choisir quoi ?

* **REST** : par défaut pour services publics, docs faciles.
* **GraphQL** : agrégation de données, clients mobiles.
* **gRPC** : micro‑services internes, faible latence.
* **WebSocket** : notifications temps réel, jeux.

---

## 1.5 Checklist pour vos projets

* [ ] Utiliser `201 Created` avec header `Location:` à chaque création.
* [ ] Documenter tous les codes d’erreur personnalisés.
* [ ] Ajouter des **tests contractuels** de code HTTP dans Postman Test tab.
* [ ] Générer votre spec OpenAPI (Swagger) dès le design.
* [ ] Configurer la **retry policy** (idempotence) dans vos clients.

> 👉 **Prochaine page :** Installation de Postman, création de workspace et tour complet de l’interface.
