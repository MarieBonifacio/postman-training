---

title: "Génération de documentation & Publication"
sidebar\_position: 10
-----------------------------

# 10. Génération de documentation & Publication 📚

> **Pourquoi ce chapitre ?** Une API sans documentation est invisible : les devs la comprennent mal, les partenaires l’ignorent, vos propres équipes ré‑inventent la roue. Postman génère une doc vivante à partir de vos collections ; Docusaurus (ou Readme, GitBook) la rend encore plus riche et versionnée.

Objectifs :

1. Publier automatiquement une **doc Postman** publique ou privée à partir d’une collection.
2. Personnaliser la doc (Markdown, exemples code, variables dynamiques).
3. Exporter vers **Docusaurus** et intégrer dans votre site dev portal.
4. Mettre en place le **versioning** (SemVer, branches, tags).
5. Ajouter des **badges** (uptime, test coverage) et un **changelog** auto.

---

## 10.1 Pourquoi documenter via Postman ?

| Besoin                  | Avant          | Avec doc Postman                      |
| ----------------------- | -------------- | ------------------------------------- |
| Décrire chaque endpoint | Wiki à la main | Description Markdown inline           |
| Fournir snippet cURL/JS | Copier‑coller  | Auto‑généré en 12 langages            |
| Tenir doc à jour        | Oubli fréquent | Mise à jour dès que la requête change |
| Laisser feedback        | Mail/Slack     | Commentaires directement sur la page  |

---

## 10.2 Génération Postman – pas‑à‑pas

1. Ouvrez la collection **Postman Basics** ▸ **View Documentation**.
2. **Edit** ➜ ajoutez un **Overview** Markdown (titre, diagramme, flow OAuth).
3. Pour chaque requête, vérifiez :

   * **Name** clair (`GET Users – list`).
   * **Description** : but, param, réponse exemple.
   * **Example responses** sauvegardées (`Save Response`) → affichées dans la doc.
4. **Publish** ➜ choisir *Private* (limité aux Workspace) ou *Public* (répertoire API Network).
5. URL générée : `https://documenter.getpostman.com/view/<uid>`.

### Enrichir avec variables dynamiques

Utilisez `{{baseUrl}}` → la doc affiche `https://api.example.com`.
Ajoutez un tableau *Environments* pour que l’utilisateur copie directement la bonne URL.

---

## 10.3 Exemples code auto‑générés

Postman propose :

* **cURL**, **Node (Axios, Fetch)**, **Python (Requests)**, **Go**, **Java**, **C#**, **Swift**, **Ruby**.

> **Tip** : Dans les **Docs Settings**, cochez « Include SDK tabs » pour Go/JS SDK si vous avez généré un SDK via Postman Generation (Plan Enterprise).

---

## 10.4 Exporter pour Docusaurus

### 10.4.1 Export Markdown

1. `...` menu collection ▸ **Export ▸ Documentation ▸ Markdown**.
2. Postman télécharge `Postman Basics.md` (front‑matter minimal).

### 10.4.2 Script de conversion multi‑fichiers (PowerShell)

```powershell
# split-doc.ps1
param($Source="Postman Basics.md", $TargetDir="docs/api")
$content = Get-Content -Raw $Source
$content -split '\n# ' | ForEach-Object -Begin {$i=0} -Process {
  if ($_ -ne ""){
    $title = ($_ -split '\n')[0]
    $slug  = $title.ToLower() -replace '[^a-z0-9]+','-'
    $file  = "{0:d2}-$slug.md" -f ++$i
    "---`ntitle: '$title'`nsidebar_position: $i`n---`n# $_" | Set-Content "$TargetDir/$file"
  }
}
```

Lancez `pwsh split-doc.ps1` → fichiers prêts pour Docusaurus sidebar autogen.

### 10.4.3 CI GitHub Actions

```yaml
name: Export Postman Docs
on:
  push:
    paths: ["collections/*.json"]
jobs:
  export:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm i -g @postman/cli
      - run: postman login --with-api-key ${{ secrets.POSTMAN_KEY }}
      - run: postman collection export "Postman Basics" --format markdown --output docs/api
      - run: node ./scripts/split-doc.js
      - run: git add docs/api && git commit -m "chore(doc): sync postman" || echo "no changes"
```

---

## 10.5 Versioning & changelog

| Stratégie      | Outil            | Implémentation                       |
| -------------- | ---------------- | ------------------------------------ |
| **SemVer**     | Git tag `v1.2.0` | Branch `release/1.x` ↔ docs `/v1/`   |
| **Date‑based** | `2025-06`        | Doc sous `/docs/2025-06/`            |
| **Channel**    | `beta`, `stable` | Deux collections, deux docs publiées |

### Postman Docs versioning

* Plan Business : créez un **Documentation Version** dans *Publish settings*.

### Changelog auto (Script)

```bash
git log --pretty="* %s (%h)" v1.1.0..HEAD > docs/changelog.md
```

Intégrez dans la page d’accueil Docusaurus via `import changelog from './changelog.md'`.

---

## 10.6 Badges & health indicators

| Badge              | Markdown                                                                    | Propriété mesurée |
| ------------------ | --------------------------------------------------------------------------- | ----------------- |
| **Docs build**     | `![docs](https://github.com/org/repo/actions/workflows/docs.yml/badge.svg)` | build site OK     |
| **Coverage tests** | `![coverage](https://api.getpostman.com/collections/<uid>/badge)`           | ratio tests pass  |
| **Uptime monitor** | `![uptime](https://uptime.com/badge/<id>.svg)`                              | 30‑d uptime       |

---

## 10.7 Check‑list Documentation

* [ ] Overview Markdown rédigé (but, contact, SLA).
* [ ] Chaque requête nommée clairement + description + example response.
* [ ] Documentation Postman publiée (*Public* ou *Private*).
* [ ] Export Markdown -> Docusaurus automatisé en CI.
* [ ] Badge coverage tests & build dans README.
* [ ] Changelog auto mis à jour.

> 👉 **Page suivante** – Industrialisation & Governance : style guide API, lint Spectral, workflow PR avec ReviewBot.
