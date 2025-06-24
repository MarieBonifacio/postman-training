---
title: "Exécution automatisée avec Newman (avancé)"
sidebar_position: 12
---

# 12. Exécution automatisée avec Newman (avancé) ⚙️🧪

> Pourquoi ce chapitre ? Parce que **Newman devient un outil critique** dès que vous passez de l’exploration manuelle à l’automatisation en CI/CD. Ce chapitre approfondit les options les moins connues et les patterns efficaces.

---

## 12.1 Installation & rappel rapide

```bash
npm install -g newman newman-reporter-htmlextra
```

Check rapide :

```bash
newman run collection.json -e staging.json -r cli,htmlextra
```

## 12.2 Paramétrage complet de la ligne de commande


```bash
newman run collection.json \
  -e env/staging.json \
  -d data/users.csv \
  --iteration-count 500 \
  --delay-request 200 \
  --bail \
  --timeout 60000 \
  --reporters cli,htmlextra,junit,json \
  --reporter-htmlextra-export reports/report.html \
  --reporter-junit-export reports/report.xml
 ``` 

### Options utiles :

| Option                 | Utilité terrain                    |
| ---------------------- | ---------------------------------- |
| `--bail`               | Arrête au 1er échec critique       |
| `--timeout`            | Gestion réseau instable            |
| `--delay-request`      | Evite les 429                      |
| `--env-var key=value`  | Override dynamique                 |
| `--export-environment` | Snapshot d’environnement en sortie |


## 12.3 Intégration dans GitHub Actions

```yaml
name: API Tests Newman

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g newman newman-reporter-htmlextra
      - run: newman run collection.json -e env/staging.json -r cli,htmlextra
      - uses: actions/upload-artifact@v4
        with:
          name: report-html
          path: reports/report.html

```

## 12.4 Patterns robustes : retry, hooks, DRY

Ajoutez ce bloc en Post-request pour créer un log synthétique :

```javascript
console.log(`TEST ${pm.request.name} → ${pm.response.code} (${pm.response.responseTime}ms)`);
```

Avec le reporter json, vous pouvez agréger ces infos dans un tableau Excel.


## 12.6 Astuces de productivité

newman run col.json --folder "Login" → n’exécute qu’un sous-ensemble.

--global-var timestamp=$(date +%s) → horodatage sans polluer l’environnement.

--insecure si vous testez une API HTTP/SSL auto-signée.

--disable-unicode si logs bizarres dans GitLab.

## 12.7 Check-list Newman complète ✅

 Utilisation du bon reporter selon contexte (htmlextra, junit, json).

 Intégration CI validée (GitHub Actions, GitLab, Jenkins…).

 Gestion des secrets via --env-var ou Vault.

 Tests robustes aux erreurs serveur (retry, bail).

 Rapport HTML archivé pour lecture post-mortem.

 Exécution parallèle documentée si besoin de load-test.

👉 Retour au sommaire : Vous pouvez maintenant industrialiser vos tests Postman à grande échelle.

```yaml

---

## 🪄 3. À faire ensuite

- **Ajoute cette page dans le sommaire `intro.md`** :
```markdown
| Exécution avancée avec Newman | [/docs/newman-avance](/docs/newman-avance) | Paramétrer et industrialiser les runs CLI |
```

Lien croisé à placer dans :

08-data-driven-contract.md > section 7.4 : 👉 Voir aussi /docs/newman-avance pour aller plus loin.

11-industrial-governance.md > après le bloc CI.

(Optionnel) : créer un mini script de démo Newman local (newman-demo.sh) dans ton repo pour les collègues.