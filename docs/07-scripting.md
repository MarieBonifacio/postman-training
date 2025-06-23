---

title: "JavaScript Scripting : du basique au Ninja"
sidebar\_position: 6
------------------------------

# 6. JavaScript Scripting – du basique au Ninja 👾

Les scripts **Pre‑request** et **Tests** transforment Postman en véritable moteur d’automatisation. Cette page couvre :

1. Syntaxe de base `pm.*`, assertions Chai.
2. Utilitaires intégrés (Crypto, Lodash, Moment).
3. Ajv pour la validation de schéma JSON.
4. Patterns avancés : retry exponentiel, requêtes chaînées, modules réutilisables.

---

## 6.1 Pourquoi scripter ?

| Besoin                 | Sans script                  | Avec script Postman                 |
| ---------------------- | ---------------------------- | ----------------------------------- |
| Changer un header dyn. | À la main avant chaque envoi | `pm.request.headers.add()`          |
| Chaîner 2 appels       | Plusieurs clics              | `pm.sendRequest()` dans Pre‑request |
| Vérifier un schéma     | Lecture œil humain           | Ajv + test automatique              |
| Retry sur 500          | Relancer manuellement        | Boucle `while` + `setTimeout`       |

---

## 6.2 Écosystème JavaScript Postman

| Lib dispo par défaut | Version | Exemple rapide                 |
| -------------------- | ------- | ------------------------------ |
| **Chai**             | 4.x     | `pm.expect(val).to.be.true;`   |
| **Lodash** (`_`)     | 4.17    | `_.uniq([1,1,2])`              |
| **Moment**           | 2.x     | `moment().add(7,'days')`       |
| **CryptoJS**         | 4.x     | `CryptoJS.HmacSHA256(msg,key)` |

> **Tip** : Postman Sandbox ≈ Chromium ; fonctions ES2021 (optional chaining, nullish coalescing) sont supportées.

---

## 6.3 Assertions basiques avec Chai

```js
pm.test("Statut 200", () => pm.response.to.have.status(200));
pm.test("Array non vide", () => {
  pm.expect(pm.response.json().items).to.be.an("array").that.is.not.empty;
});
```

### Pourquoi 1 assertion = 1 test ?

* Localiser le bug plus vite.
* Rapport Newman clair (1 test = 1 ligne JUnit).

---

## 6.4 Ajv + JSON Schema – valider la structure

```js
import Ajv from "ajv";
const ajv = new Ajv({allErrors:true});
const schema = {
  type:"object",
  required:["id","email"],
  properties:{
    id:{type:"string",pattern:"^[0-9a-f-]{36}$"},
    email:{type:"string",format:"email"}
  }
};
const data = pm.response.json();
pm.test("Schéma valide", () => pm.expect(ajv.validate(schema,data)).to.be.true);
```

*Pourquoi ?* Prévient les régressions structurelles invisibles à l’œil nu.

---

## 6.5 Retry exponentiel (pattern résilient)

```js
const maxRetry = 3;
const retryKey = `retry_${pm.info.requestId}`;
let attempt = pm.variables.get(retryKey) || 0;

if (pm.response.code >= 500 && attempt < maxRetry) {
  attempt++;
  pm.variables.set(retryKey, attempt);
  const delay = 2 ** attempt * 100; // 200ms, 400ms, 800ms
  setTimeout(() => postman.setNextRequest(pm.info.requestName), delay);
} else {
  pm.variables.unset(retryKey);
}
```

*Pourquoi ?* Requêtes robustes face aux flaps réseau.

---

## 6.6 Requêtes chaînées (get‑then‑use)

```js
// Pre‑request – récup ID utilisateur avant l’appel principal
pm.sendRequest(`${pm.environment.get("baseUrl")}/users?email={{email}}`, (err,res) => {
  pm.variables.set("userId", res.json().id);
});
```

Dans l’URL suivante : `/orders/{{userId}}`.

---

## 6.7 Modules réutilisables

1. Créez un fichier `utils.js` :

```js
exports.randomPwd = () => `Pw-${Math.random().toString(36).slice(2,10)}`;
```

2. Stockez le contenu minifié dans `pm.globals.set("lib_utils", <code>)`.
3. Dans Pre‑request :

```js
eval(pm.globals.get("lib_utils"));
pm.variables.set("pwd", exports.randomPwd());
```

*Pourquoi ?* Factoriser la logique, éviter le copier‑coller.

---

## 6.12 Check‑list Ninja ++

* [ ] Assertions **Chai** distinctes (1 assertion = 1 test).
* [ ] Validation **Ajv** du schéma JSON.
* [ ] **Retry exponentiel** ≤ 3 tentatives.
* [ ] Variables **chaînées** entre requêtes (`pm.sendRequest`).
* [ ] **Utils.js** chargé via Globals / minifié.
* [ ] **Visualizer Chart.js** opérationnel (latence ou autre métrique).
* [ ] **Pagination automatique** cursor/offset sans boucle infinie.
* [ ] **gRPC streaming** : 10 messages reçus et testés.
* [ ] **Agrégation des erreurs** Newman affichée en fin de run.

> 👉 **Page suivante** – Data‑Driven & Contract Testing : Runner CSV/JSON, Faker.js, rapports Newman / Postman CLI.
