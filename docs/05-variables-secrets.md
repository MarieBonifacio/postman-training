---

title: "Variables & Secrets Management"
sidebar_position: 4
slug: /postman/variables-secrets
--------------------------------

# 4. Variables & Secrets Management

Utiliser correctement les **variables** fait la différence entre un fichier Postman bricolé et une suite de tests professionnelle. Nous allons :

1. Comprendre les **quatre scopes** (Global, Environment, Collection, Local) et leur résolution.
2. Créer trois environnements (`local`, `staging`, `prod`) et commuter entre eux.
3. Apprendre à sécuriser vos secrets (tokens, clés API).
4. Automatiser la mise à jour des variables via scripts Pre‑request et Post‑response.

---

## 4.1 Pourquoi des variables ?

* **Réutilisabilité** : changez une URL de base, toute votre collection suit.
* **Sécurité** : stockez les secrets hors du code et évitez de pousser des tokens dans Git.
* **Paramétrage** : lancez la même collection contre `staging` puis `prod` sans modifier chaque requête.

---

## 4.2 Les quatre scopes et l’algorithme de résolution

| Scope           | Durée de vie      | Exemple typique                    | Priorité de résolution |
| --------------- | ----------------- | ---------------------------------- | ---------------------- |
| **Local**       | Requête actuelle  | `pm.variables.set("trace", $guid)` | 1 (le plus haut)       |
| **Data**        | Runner (CSV/JSON) | `{{email}}` en data‑driven         | 2                      |
| **Environment** | Workspace courant | `{{baseUrl}}`, `{{token}}`         | 3                      |
| **Global**      | Tous workspaces   | `{{company}}`, `{{timezone}}`      | 4                      |

> **Algorithme** : Postman cherche `Local → Data → Env → Global`. Si aucun scope ne contient la variable, elle reste littérale.

### Check‑list compréhension

* [ ] Vous pouvez expliquer l’ordre de résolution.
* [ ] Vous utilisez `Local` pour les valeurs éphémères (uuid).
* [ ] Vos secrets ne sont **pas** en Globals.

---

## 4.3 Créer et basculer entre trois environnements

1. **Environments ▸ +** → `local`.
2. Variables :

   | Key       | Initial Value           | Current Value |
   | --------- | ----------------------- | ------------- |
   | `baseUrl` | `http://localhost:8080` | —             |
   | `token`   | `dev‑token‑123`         | —             |
3. Répétez pour **staging** et **prod**.
4. Sélecteur en haut‑droite : choisissez `staging` avant d’envoyer une requête.

> **Pourquoi séparer Initial/Current ?** `Current` reste local à votre poste, `Initial` est partagé quand vous exportez l’environnement.

### Script Pre‑request auto‑token

```js
if (!pm.environment.get("token") || isExpired()) {
  pm.sendRequest({
    url: pm.environment.get("baseUrl") + "/auth",
    method: "POST",
    body: {
      mode: "raw",
      raw: JSON.stringify({ username: "qa", password: "qa" })
    }
  }, (err, res) => {
    pm.environment.set("token", res.json().access_token);
    pm.environment.set("token_exp", Date.now() + 50 * 60 * 1000);
  });
}
```

---

## 4.4 Sécuriser les secrets

| Méthode                     | Plan Postman  | Avantages                                     | Limites                        |
| --------------------------- | ------------- | --------------------------------------------- | ------------------------------ |
| **Secure Variables** (v10+) | Pro, Business | Chiffrées au repos, masquées UI               | Non exportables ↔ CI difficile |
| **Postman Vault**           | Business      | Intégration HashiCorp/Azure Key Vault         | Payant, Cloud only             |
| **CLI Secrets push**        | Gratuit       | `postman vars push --env prod --from envfile` | Pas d’UI, script requis        |

### Masquage manuel via variable Convention

```js
// Dans Pre‑request, cachez toute valeur commençant par « secret_ »
if (pm.request.url.toString().includes("secret_")) {
  throw new Error("Secret leaked in URL !");
}
```

#### Anti‑patterns secrets

| Mauvaise pratique                                           | Risque                 | Correction                          |
| ----------------------------------------------------------- | ---------------------- | ----------------------------------- |
| `token=abcd1234` en dur dans l’URL                          | Loggué dans Proxy, Git | Utiliser `{{token}}` variable d’Env |
| Token dans Tests (`console.log(token)`)                     | Fuite Console export   | Supprimer logs secrets              |
| Commit d’un fichier `.postman_environment` plein de secrets | Stockage GitHub public | Fichier `.gitignore` + Secure Vars  |

---

## 4.5 Importer/Exporter des environnements – *Pourquoi versionner ?*

* **Audit** : savoir quand un endpoint de staging a changé.
* **Rollback** : revenir à un token précédent si nouveau scope invalide.

```bash
# Export JSON propret, sans Current Value
postman export env "staging" --output envs/staging.postman_environment.json --no-current

git add envs/staging.postman_environment.json && git commit -m "chore(env): bump api version"
```

---

## 4.6 Visualizer rapide des variables

Ajoutez dans Tests :

```js
const table = Object.entries(pm.environment.toObject())
  .map(([key, value]) => `<tr><td>${key}</td><td>${value}</td></tr>`)  
  .join("");
pm.visualizer.set(`<table><tr><th>Var</th><th>Val</th></tr>${table}</table>`);
```

*Pourquoi ?* Afficher à l’écran les valeurs actives évite d’envoyer un token vide.

---

## 4.7 Check‑list finale Variables & Secrets

* [ ] Trois environnements `local`, `staging`, `prod` créés.
* [ ] `baseUrl`, `token` paramétrés par env.
* [ ] Script auto‑refresh token fonctionne (< 50 minutes).
* [ ] Aucun secret en Global ou dans le code source.
* [ ] Exports JSON commités **sans Current Value**.

> 👉 **Page suivante** – Authentification avancée (Bearer, OAuth 2.0, Hawk, AWS SigV4) : comprendre les flux, scripts de refresh et tests de sécurité.
