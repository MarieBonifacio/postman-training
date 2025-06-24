---

title: "Introduction à la formation Postman"
sidebar\_position: 0
--------------

# Bienvenue dans la formation **Postman** 📨

Cette série de pages va vous guider pas à pas : **débutant ➜ confirmé ➜ expert**.

## Objectifs pédagogiques

| En fin de formation vous saurez…                                                  | Pourquoi c’est clé ?                                   |
| --------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Envoyer n’importe quelle requête HTTP, GraphQL, gRPC ou WebSocket depuis Postman. | Interagir avec **toutes** les API de votre stack.      |
| Écrire des *tests automatisés* robustes (statut, schéma, règles métier).          | Fiabiliser vos livraisons et éviter les régressions.   |
| Paramétrer des **workflows CI/CD** (GitHub Actions, GitLab CI, Azure DevOps).     | Intégrer la qualité API dans votre pipeline DevOps.    |
| Mettre en place des **monitors** avec alerting et SLO.                            | Détecter les incidents avant vos utilisateurs.         |
| Gérer la **documentation** et la gouvernance API au sein de votre équipe.         | Partager un langage commun et accélérer l’on‑boarding. |


💡 À ce stade, tu n’as rien à faire sauf t’assurer de :
bien comprendre le scope de Postman (ce n’est pas juste un outil pour “tester une API à la main”)

savoir que tout ce que tu construis dedans peut être versionné, exécuté, et documenté automatiquement

## 🃏 Mythes fréquents sur Postman

:::tip Mythe #1 – “Postman c’est juste pour faire des appels GET à la main.”
**Faux.** Postman est un moteur d’automatisation complet : scripting JS, assertions, visualisation, data-driven, mock, chaos…  
Tu peux l’exécuter dans une CI comme un vrai testeur back.
:::

:::tip Mythe #2 – “Les tests Postman sont trop basiques.”
Postman embarque **Chai.js**, **Ajv**, **Lodash**, **Crypto**, etc.  
Tu peux tester la structure d’une réponse, faire du retry, ou valider des contraintes métier complexes.
:::

:::tip Mythe #3 – “C’est un outil pour devs, pas pour QA.”
**Erreur classique.** Postman est idéal pour les QA qui :
- valident le comportement métier via API,
- automatisent sans framework lourd,
- posent des moniteurs et alertes.
:::

:::tip Mythe #4 – “On ne peut pas versionner ce qu’on fait.”
Bien au contraire ! Tu peux :
- exporter les collections/environnements,
- les mettre dans Git,
- les exécuter via Newman ou `postman-cli`,
- documenter en Markdown/Docusaurus.
:::

:::tip Mythe #5 – “J’ai testé une fois, ça suffit.”
Tester une fois = une photo.  
Tester avec Postman = un **filet de sécurité permanent**, rejouable, vérifiable, diffable.  
Tu bâtis une **stratégie qualité vivante**.
:::




> 📝 **Méthode** : chaque chapitre alterne *concepts*, *démos pas‑à‑pas* et *exercices pratiques*.
>
> 👉 Commencez ! Passez à la première leçon : **Rappels HTTP & architecture API**.
