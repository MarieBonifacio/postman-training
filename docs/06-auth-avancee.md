---

title: "Authentification avancée (Bearer, OAuth 2.0, Hawk, AWS SigV4)"
sidebar\_position: 5
slug: /postman/auth-avancee
---------------------------

# 5. Authentification avancée – Bearer, OAuth 2.0, Hawk, AWS SigV4

La sécurité est le **cœur** de toute API. Ce chapitre détaille les schémas d’authentification courants :

* **Bearer JWT** – transmission simple de token opaque ou signé.
* **OAuth 2.0** – flux *client‑credentials*, *authorization‑code* (PKCE) et *refresh*.
* **Hawk** – signatures HMAC horodatées (anti‑replay).
* **AWS SigV4** – signature canonique pour services AWS / API Gateway.

Chaque section inclut : *Pourquoi*, configuration Postman, scripts de rafraîchissement, pièges fréquents et check‑list.

---

## 5.1 Bearer Token (JWT)

### Pourquoi choisir Bearer ?

* **Simplicité** : une seule chaîne base64 dans `Authorization: Bearer <token>`.
* **Stateless** : le serveur n’a pas à stocker de session (si JWT auto‑contenu).

### Mise en place Postman

1. Onglet **Auth ▸ Type ▸ Bearer Token**.
2. Collez `{{access_token}}` comme valeur.
3. Sauvegardez la requête.

### Script Pre‑request de rafraîchissement (client‑credentials)

```js
const exp = pm.environment.get("token_exp");
if (!pm.environment.get("access_token") || Date.now() > exp) {
  pm.sendRequest({
    url: pm.environment.get("auth_url"),
    method: "POST",
    header: {"Content-Type": "application/json"},
    body: {
      mode: "raw",
      raw: JSON.stringify({
        client_id: pm.environment.get("client_id"),
        client_secret: pm.environment.get("client_secret"),
        audience: "api://backend",
        grant_type: "client_credentials"
      })
    }
  }, (err, res) => {
    const json = res.json();
    pm.environment.set("access_token", json.access_token);
    pm.environment.set("token_exp", Date.now() + json.expires_in * 1000 - 60_000); // marge 1 min
  });
}
```

#### Check‑list Bearer

* [ ] Header `Authorization` présent sur chaque requête.
* [ ] Token découle du script auto‑refresh.
* [ ] Décodage rapide : `console.log(JSON.parse(atob(token.split('.')[1])))`.

---

## 5.2 OAuth 2.0 – flux principaux

| Flux                         | Cas d’usage                  | Avantages                       | Limites                                    |
| ---------------------------- | ---------------------------- | ------------------------------- | ------------------------------------------ |
| *Client‑credentials*         | Backend‑to‑Backend (machine) | Simplicité, pas de user consent | Pas de scopes utilisateurs                 |
| *Authorization‑code* (+PKCE) | Apps SPA / mobile            | Sûr (secret‑less)               | Nécessite navigateur intégré Postman       |
| *Refresh Token*              | Renouvellement long terme    | Persiste session                | Stockage sécurisé du refresh indispensable |

### Assistant OAuth 2.0 de Postman

1. Onglet **Auth ▸ OAuth 2.0**.
2. **Configure New Token** : remplir *Auth URL*, *Token URL*, *Client ID* …
3. Choisir **Grant Type**.
4. Postman ouvre un navigateur (flow *auth‑code*) ou appelle la *token URL*.

### Cas PKCE – Pourquoi ?

*Elimine l’usage d’un client secret* ; idéal pour front‑end public. Postman génère le `code_verifier` et `code_challenge`.

#### Script Test pour vérifier les scopes

```js
const decoded = JSON.parse(atob(pm.environment.get("access_token").split(".")[1]));
pm.test("Scope orders:read présent", () => {
  pm.expect(decoded.scope.split(" ")).to.include("orders:read");
});
```

#### Pièges courants OAuth

| Problème              | Symptôme                            | Correctif                                     |
| --------------------- | ----------------------------------- | --------------------------------------------- |
| Mauvais redirect\_uri | Erreur 400 « Mismatching redirect » | Copier la valeur exacte depuis Postman config |
| Heure PC décalée      | `invalid_grant`                     | Sync NTP, timezone correcte                   |
| Port 443 bloqué       | Timeout token URL                   | Ouvrir proxy/pare‑feu                         |

---

## 5.3 Hawk HMAC (clock‑skew tolerant)

### Pourquoi Hawk ?

* Protection **anti‑replay** : timestamp + nonce.
* Pas de token à stocker, juste clé/secret (HMAC‑SHA‑256).

### Configuration

1. Onglet **Auth ▸ Hawk Authentication** (desktop agent requis).
2. Remplir `Hawk ID`, `Hawk Key`, `Algorithm`.
3. Postman signe automatiquement chaque requête.

#### Test anti‑replay

```js
pm.test("Réponse MAC match", () => {
  pm.expect(pm.response.headers.get("Server-Authorization")).to.exist;
});
```

---

## 5.4 AWS Signature Version 4 (SigV4)

### Pourquoi SigV4 ?

* Obligatoire pour toutes les API Gateway sécurisées, S3, etc.
* Signature canonique incluant date, région, service.

### Pas‑à‑pas Postman

1. **Auth ▸ AWS Signature**.
2. `AccessKey`, `SecretKey`, `AWS Region`, `Service Name` (`execute-api`).
3. Postman calcule la signature et ajoute headers `Authorization`, `x‑amz‑date`.

#### Script de test latence AWS

```js
pm.test("Latency < 300 ms", () => {
  pm.expect(pm.response.responseTime).to.be.below(300);
});
```

#### Pièges courants SigV4

| Cause                     | Erreur AWS              | Solution                                |
| ------------------------- | ----------------------- | --------------------------------------- |
| Horloge PC décalée >5 min | `RequestTimeTooSkewed`  | Resync NTP                              |
| Service Name erroné       | `SignatureDoesNotMatch` | `execute-api`, `s3`, etc.               |
| Headers canonique oubliés | idem                    | Laisser Postman gérer 100 % des headers |

---

## 5.5 Comparatif rapide des schémas

| Critère         | Bearer                | OAuth 2.0 PKCE    | Hawk                    | AWS SigV4           |
| --------------- | --------------------- | ----------------- | ----------------------- | ------------------- |
| **Cas d’usage** | micro‑service interne | SPA mobile        | API internes horodatées | Cloud AWS           |
| **Complexité**  | ★☆☆                   | ★★☆               | ★★☆                     | ★★★                 |
| **Anti‑replay** | Non                   | Oui (state param) | Oui (nonce)             | Oui (canonical req) |
| **Expiration**  | `exp` claim           | Refresh token     | Timestamp ±60 s         | 15 min par défaut   |

---

## 5.6 Check‑list finale Auth

* [ ] Token Bearer auto‑refresh opérationnel.
* [ ] Assistant OAuth 2.0 testé (PKCE ou client‑cred).
* [ ] Requêtes Hawk signées sans erreur 401.
* [ ] Requête SigV4 validée (`200 OK`).
* [ ] Tests de scope, anti‑replay ou latence ajoutés.

> 👉 **Page suivante** – JavaScript Scripting : du basique (`pm.test`) au Ninja (Ajv, Lodash, modules externes, retry backoff).
