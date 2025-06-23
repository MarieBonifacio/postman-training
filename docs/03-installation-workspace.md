---

title: "Installation, Workspace & UI Deep‑Dive"
sidebar\_position: 2
-------------------------------------

# 2. Installation, Workspace & UI Deep‑Dive

Dans ce chapitre, nous installons Postman, configurons un premier **workspace** et explorons chaque zone de l’interface.

---

## 2.1 Télécharger & installer Postman

| Plate‑forme           | Méthode recommandée                              | Commande silencieuse (CI / MDM)             |
| --------------------- | ------------------------------------------------ | ------------------------------------------- |
| Windows               | EXE 64‑bit                                       | `Postman-win64-x.y.z-Setup.exe /S`          |
| macOS (Intel)         | ZIP + drag‑and‑drop                              | `brew install --cask postman`               |
| macOS (Apple Silicon) | DMG ARM64                                        | idem (brew gère la bonne arch)              |
| Linux (deb/rpm)       | Repo officiel                                    | `sudo snap install postman`                 |
| Navigateur            | [https://web.postman.co](https://web.postman.co) | Nécessite Postman Agent pour gRPC/WebSocket |

> 🛡️ **Proxy d’entreprise** : ouvrez les ports 80/443 et 5555 (Agent).
> 🔄 **Mises à jour automatiques** : Desktop se met à jour en tâche de fond ; désactivez‑les via `Settings > Update > Disable auto‑download` si votre DSI contrôle les versions.

### 2.1.1 Checklist après installation

* [ ] Lancer Postman Desktop et se connecter (compte gratuit suffisant).
* [ ] Vérifier la version (`Help > Check for updates`).
* [ ] Ouvrir la **Console** (`Ctrl + Alt + C`) pour s’assurer que les logs fonctionnent.

---

## 2.2 Créer un Workspace

| Type         | Cas d’usage                | Limites plan Free             |
| ------------ | -------------------------- | ----------------------------- |
| **Personal** | Tests locaux, brouillons   | 3 workspaces max              |
| **Team**     | Collaboration intra‑équipe | 5 collaborateurs / collection |
| **Public**   | API Network (open‑source)  | Révision manuelle par Postman |

### 2.2.1 Pas-à‑pas

1. `Workspaces ▸ Create Workspace`
2. Nom : **Backend Academy** ‑ Type : **Team** – Visibilité : *Private*
3. Ajoutez une description Markdown (affichée dans la liste).

> ✏️ **Astuce** : créez un fichier `README.md` dans chaque workspace pour le on‑boarding de nouveaux collègues.

---

## 2.3 Tour complet de l’interface Desktop

| Zone                | Raccourci          | Description rapide                         |
| ------------------- | ------------------ | ------------------------------------------ |
| **Sidebar gauche**  | `Ctrl + Shift + E` | Collections, Environments, APIs, Flows     |
| **Tabs centrale**   |  —                 | Requêtes, Tests, Visualizer, Examples      |
| **Console**         | `Ctrl + Alt + C`   | Logs runtime JS, équivalent DevTools       |
| **Status bar**      |  —                 | Indique l’environnement actif, sync status |
| **Search (global)** | `Ctrl + K`         | Rechercher requêtes, variables, workspaces |

### Exercices rapides

* Ouvrez le **Launchpad** ; lancez l’exemple *Postman Echo* pour envoyer votre premier GET.
* Activez le **Theme Dark** : `Settings > Theme` → *Solarized Night*.

---

## 2.4 Postman Web + Agent

| Avantage                                      | Limitation actuelle (en 2025)                |
| --------------------------------------------- | -------------------------------------------- |
| Accessible sans installation local            | gRPC, WebSocket nécessitent l’**Agent**      |
| Idéal pour Chromebook / machines verrouillées | Impossible d’importer des certificats client |

**Installation Agent :**

```
postman-agent-win64-x.y.z-Setup.exe /S
```

L’icône Postman apparaît dans la zone de notification ; l’onglet *Agent* de l’app Web doit lister « Connected ».

---

## 2.5 Configuration initiale recommandée

1. **Proxy** : `Settings > Proxy > Use System Proxy` (par défaut) ou proxy HTTP externe.
2. **Certificate Pinning** : ajoutez votre certificat interne (`.pem`) dans `Settings > Certificates` pour éviter `CERT_INVALID`.
3. **Telemetry** : désactivez l’envoi d’analytics si politique interne stricte (`Settings > Privacy`).

---

## 2.6 Checklist de fin de chapitre

* [ ] Postman Desktop installé et lancé.
* [ ] Workspace **Backend Academy** créé (Team ou Personal).
* [ ] Console ouverte sans erreur.
* [ ] Paramètres proxy/certificats vérifiés.

> 👉 **Page suivante :** Construire votre première requête HTTP, sauvegarder des collections et comprendre les onglets *Body/Headers/Tests*.
