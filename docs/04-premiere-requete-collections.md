---

title: "Première requête HTTP & Collections"
sidebar\_position: 3
-------------------------------------------

# 3. Première requête HTTP & Collections

Comprendre **POURQUOI** chaque étape compte vous aidera à prendre des habitudes solides dès le départ. Ce chapitre illustre les opérations CRUD de base et, surtout, explique la raison d’être de chaque action.

---

## 3.1 Créer une requête `GET` – *Pourquoi commencer par GET ?*

`GET` est **lecture seule**, donc sans effet de bord : parfait pour la découverte d’une API.

1. **New ▸ HTTP Request**.
2. URL : `https://postman-echo.com/get?course=postman`.
3. Vérifiez que le **verbe** est **GET**.
4. **Send** (`Ctrl + Enter`).

### Décryptage de la réponse

| Onglet           | Ce qu’il affiche              | Pourquoi c’est utile                   | Tips rapides                               |
| ---------------- | ----------------------------- | -------------------------------------- | ------------------------------------------ |
| **Body**         | Corps (`JSON`, `HTML`, `XML`) | Voir la **charge utile** réelle        | *Pretty* colore JSON ; *Preview* rend HTML |
| **Headers**      | Métadonnées HTTP              | Vérifier cache, CORS, version de l’API | Double‑clic pour copier clé/valeur         |
| **Cookies**      | Cookies Set‑Cookie            | Auth session, trace utilisateur        | Éditables pour debug front‑end             |
| **Test Results** | Résultats JS                  | Assurance qualité **dès maintenant**   | Vide tant que pas de tests                 |

> **Pourquoi ajouter un header custom ?** Visualiser sur Echo vous montre le trajet complet client→serveur et votre capacité à injecter des infos.

#### Exercice

`X‑Training: true` → voyez‑le remonter dans `headers` du corps ; vous savez maintenant que votre requête peut transporter des indicateurs custom.

---

## 3.2 Sauvegarder dans une Collection – *Pourquoi persister ?*

*Les requêtes en **History** disparaissent ; les collections, elles, se versionnent et se partagent.*

1. **Save ▸ Save as**.
2. Nom : `GET Echo – course`.
3. **Create collection** `Postman Basics`.
4. Validez.

**Avantage** : la collection agit comme un « projet » ; vous pourrez l’exécuter en lot, la documenter, la versionner Git.

---

## 3.3 Construire un `POST` – *Pourquoi POST ?*

Vous envoyez des **données** côté serveur : utile pour tester formulaires, endpoints de création.

Corps JSON :

```json
{
  "message": "Hello Postman!"
}
```

`Content‑Type` est ajouté auto → gain de temps.

#### Tests : votre premier filet de sécurité

```js
pm.test("Statut 200", () => pm.response.to.have.status(200));
pm.test("Message retourné", () => {
  pm.expect(pm.response.json().data.message).to.eql("Hello Postman!");
});
```

*Pourquoi ?* Vous documentez la **contrainte métier** (« le service renvoie la même chaîne ») et serez alerté si un dev casse l’API.

---

## 3.4 Scripts Pre‑request & Variables – *Pourquoi dynamiques ?*

Des **ids uniques** évitent conflits (ex. création d’utilisateur). Exemple :

```js
pm.variables.set("uuid", pm.guid());
```

Puis dans le corps : `"id": "{{uuid}}"`.

> **Tip** : variable **Local** (`pm.variables`) dure le temps de la requête ; évite de polluer l’Environment.

---

## 3.5 Étendre au CRUD complet

| Verbe      | Endpoint Echo         | Objectif pédagogique                                                      |
| ---------- | --------------------- | ------------------------------------------------------------------------- |
| **PUT**    | `/put`                | Comprendre idempotence : relancer la même requête ne crée pas de doublons |
| **DELETE** | `/delete?id={{uuid}}` | Vérifier la suppression et la reprise de variable                         |

```js
// Test idempotence PUT
pm.test("idempotent", () => {
  pm.expect(pm.response.headers.get("x-forwarded-method")).to.eql("PUT");
});
```

---

## 3.6 Visualizer – *Pourquoi présenter ?*

Le Visualizer transforme la réponse en **mini‑dashboard HTML** ; idéal pour montrer un Proof of Concept à un Product Owner sans exporter vers Excel.

```js
pm.visualizer.set(`<h3>Réponse</h3><pre>{{json}}</pre>`, {
  json: JSON.stringify(pm.response.json(), null, 2)
});
```

---

## 3.7 Checklist à cocher dans Postman

* [ ] Collection **Postman Basics** avec dossier *Echo* (CRUD complet).
* [ ] Tests statut + message + idempotence.
* [ ] Génération `{{uuid}}` via Pre‑request (Local scope).
* [ ] Visualizer affichant le JSON.
* [ ] Descriptions Markdown renseignées sur chaque requête.

---

## 3.8 Bonnes pratiques vs Anti‑patterns

| Anti‑pattern                                    | Pourquoi c’est un piège         | Correction                                      |
| ----------------------------------------------- | ------------------------------- | ----------------------------------------------- |
| Secrets (*token*) stockés en Globals            | Fuite si export/partage         | Variables d’**Environment** (Cryptage plan Pro) |
| `Untitled Request`                              | Incompréhensible en CI          | `VERB Resource – intent`                        |
| Tests fourre‑tout                               | Dur à déboguer                  | 1 assertion = 1 test                            |
| Pas de description                              | Aucun contexte pour un collègue | Markdown dans **Description**                   |
| Variables dans l’URL *hardcodées* (`/user/123`) | Pas réutilisable                | `{{userId}}` via Runner CSV                     |

---

## 3.9 Pourquoi cette page est capitale

* Vous venez de **fiabiliser** vos appels (tests) plutôt que de cliquer sans filet.
* Les collections deviennent vos **spécifications vivantes** : un dev back les exécute, un QA les vérifie, un PO les lit.
* La rigueur prise ici (naming, scope variable) vous évitera des heures de debug quand les appels seront des centaines.

> 👉 **Page suivante :** Variables & Secrets Management – maîtriser les scopes (Global, Environment, Collection, Local) et sécuriser tokens/API‑keys.
