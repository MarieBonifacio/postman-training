---

title: "Data‑Driven & Contract Testing"
sidebar\_position: 7
-----------------------------------

# 7. Data‑Driven & Contract Testing 📊

Objectifs :

1. Exécuter une collection en boucle sur des jeux de données **CSV/JSON** avec le **Collection Runner**.
2. Générer automatiquement des données réalistes via **Faker.js**.
3. Industrialiser l’exécution en CI avec **Newman** et **Postman CLI**.
4. Garantir la conformité du contrat API (OpenAPI + Spectral + Ajv).
5. Produire des rapports lisibles pour les équipes (htmlextra, JUnit).

---

## 7.1 Pourquoi le data‑driven ?

| Cas d’usage                            | Bénéfice concret                              |
| -------------------------------------- | --------------------------------------------- |
| Créer 1 000 utilisateurs test          | Charge API réaliste, détection de limites 429 |
| Tester toutes combinaisons de rôle     | Couverture fonctionnelle exhaustive           |
| Vérifier edge‑cases (champ vide, null) | Détection régressions validation              |

---

## 7.2 Collection Runner UI

1. Ouvrez votre collection **Postman Basics** ➜ **Run Collection**.
2. Sélectionnez le fichier `users.csv` :

```csv
email,password,role
alice@example.com,Passw0rd!,ADMIN
bob@example.com,S3cr3t!,USER
```

3. Options : *Delay* 100 ms, *Iterations* = nombre de lignes.
4. **Run** ➜ onglet *Test Results* affiche un tableau par itération.

### Pourquoi Runner UI ?

* Debug immédiat (logs, cookies).
* Affiche latence moyenne, p95.

---

## 7.3 Faker.js – générer bulk data

```js
// Pre‑request – faker chargé automatiquement dans sandbox v11
const faker = require("faker");
const row = {
  email: faker.internet.email(),
  password: faker.internet.password(12),
  role: faker.helpers.randomize(["ADMIN","USER","GUEST"])
};
pm.variables.set("row", JSON.stringify(row));
```

Dans le corps :

```json
{{row}}
```

> **Pourquoi ?** Plus besoin de maintenir un CSV, données uniques à chaque run.

---

## 7.4 Runner CLI : Newman vs Postman CLI

| Critère           | **Newman** (Node)          | **Postman CLI** (2024+)            |
| ----------------- | -------------------------- | ---------------------------------- |
| Installation      | `npm i -g newman`          | `npm i -g @postman/cli`            |
| Source Collection | Fichier JSON               | Stockée dans Postman Cloud ou JSON |
| Reporters         | `cli,json,junit,htmlextra` | `cli,junit,html` + Dashboard Cloud |
| Auth Postman      | N/A                        | `postman login --with-api-key`     |
| Performance       | Node 18 dépendant          | \~20 % plus rapide C++ runtime     |

### Exemple GitHub Actions – Postman CLI

```yaml
name: API‑tests
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm i -g @postman/cli
      - run: postman login --with-api-key ${{ secrets.POSTMAN_KEY }}
      - run: postman collection run "Postman Basics" -e "staging" --reporters junit,html
      - uses: actions/upload-artifact@v4
        with:
          name: postman-report
          path: newman
```

---

## 7.5 Contract testing – Spectral + Ajv

1. **Spectral** linter OpenAPI :

```bash
spectral lint openapi.yaml --fail-severity error
```

Ajoutez dans CI avant la step Postman.

2. Test de schéma à l’exécution :

```js
const schema = pm.collectionVariables.get("OrderSchema");
const data = pm.response.json();
pm.test("Contrat Order", () => pm.expect(ajv.validate(JSON.parse(schema),data)).to.be.true);
```

### Pourquoi deux niveaux ?

* **Design First** : Spectral bloque un swagger invalide avant qu’il n’atteigne Postman.
* **Runtime** : Ajv garantit que le backend respecte le contrat.

---

## 7.6 Cas edge & performance

### 7.6.1 CSV volumineux (> 10 000 lignes)

*Problème :* Runner UI consomme toute la RAM, freeze.

**Solutions** :

1. **Chunk** le fichier :

   ```bash
   split -l 1000 users.csv chunk_
   ```
2. Utiliser l’option `--iteration-count` avec Newman pour limiter le jeu (ex. 5000) et lancer plusieurs jobs parallèles (matrix CI).
3. Ajouter `--delay-request 50` pour éviter 429.

### 7.6.2 JSON bulk (200 Mo)

*Limite UI :* taille max 50 Mo.

**Astuce** : streamer via script Pre‑request qui lit ligne par ligne depuis un repo Git brut :

```js
if(!pm.variables.get("bulk")){
  pm.sendRequest("https://raw.githubusercontent.com/org/bulk.json",(e,r)=>{
    pm.variables.set("bulk", JSON.stringify(r.json()));
  });
}
```

### 7.6.3 Random seed contrôlé

Pour des tests **répétables** :

```js
const faker = require("faker");
faker.seed(pm.info.iteration); // seed = numéro d’itération
```

Ainsi, la CI reproduit exactement les mêmes données.

### 7.6.4 Détection drift de contrat

Lorsqu’une réponse diverge du schéma Ajv :

```js
if(!ajv.validate(schema,data)){
  console.error("DRIFT", ajv.errors);
  postman.setNextRequest(null); // stop run pour inspecter
}
```

### 7.6.5 Runner parallèle (Newman)

```bash
npx newman run col.json -n 4 --iteration-count 3000 --delay-request 10 \
  --folder Echo --env staging.json &
```

Quatre processus parallèle pour simuler charge.

---

## 7.7 Rapports htmlextra & badges

```bash
newman run collection.json -e staging.json \
  -r cli,htmlextra --reporter-htmlextra-export newman/report.html
```

*Rapport* ➜ graph succès/échecs, stats latence, logs assertions.
Ajoutez un **badge** Postman CLI dans README :

```
![API coverage](https://api.getpostman.com/collections/<uid>/badge)
```

---

## 7.7 Check‑list Data‑Driven & Contract

* [ ] Runner UI exécuté sur CSV de 2 lignes.
* [ ] Faker.js génère données uniques (email/pwd).
* [ ] Pipeline CI (Newman ou Postman CLI) en place.
* [ ] Spectral lint sans erreur sur OpenAPI.
* [ ] Tests Ajv de contrat intégrés.
* [ ] Rapport htmlextra ou Dashboard Cloud archivé.

> 👉 **Page suivante** – Monitoring, Orchestration & Alerting : Monitors multi‑régions, SLI/SLO, intégrations Slack/Datadog.
