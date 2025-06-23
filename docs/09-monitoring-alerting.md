---

title: "Monitoring, Orchestration & Alerting"
sidebar\_position: 8
----------------------------------

# 8. Monitoring, Orchestration & Alerting ⏰

> **Pourquoi ce chapitre ?** Mettre en production sans monitoring, c’est comme conduire de nuit sans phares : vous ne verrez ni les pannes, ni les dégradations de latence, ni les erreurs régionales. Les **Monitors Postman** sont l’option « low‑ops » la plus rapide pour suivre la santé fonctionnelle de votre API sans déployer de nouvelle stack.

Objectifs :

1. Créer des **Monitors Cloud** multi‑régions et comprendre leur facturation.
2. Définir des **SLI/SLO** (latence, disponibilité, taux d’erreur, budget d’erreurs).
3. Orchestrer l’exécution : CRON, Webhook post‑déploiement, Flows conditionnels.
4. **Alerter** efficacement (Slack, Datadog, PagerDuty) avec payload enrichi.
5. Réduire les *faux positifs* et optimiser le coût → équilibre Réactivité / Prix.

---

## 8.1 Pourquoi monitorer ? (scénarios réels)

| Risque réel                                                      | Impact business                          | Valeur d’un monitor Postman                          |
| ---------------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------- |
| **Régression fonctionnelle** : `/checkout` renvoie 500 depuis EU | Chiffre d’affaires –30 %/h               | Alerte instantanée + rollback Canary                 |
| **SLO latence p95 > 1 s**                                        | SLA pénalités / churn utilisateur        | Stats p95/p99 export Datadog → capacity planning     |
| **CORS manquant** après nouvelle release                         | Front SPA inutilisable                   | Monitor exécute scénario complet navigateur via Echo |
| **Failover CDN** rompu US‑West uniquement                        | Incident régional invisible aux tests EU | Multi‑région détecte BGP/Edge/Country issues         |

---

## 8.2 Créer un Monitor Postman – tutoriel complet

> *Prérequis :* la collection contient déjà **tests SLI** (statut, latence).

1. **Open** collection `Echo Prod`.
2. **Monitors ▸ + New Monitor**.
3. **Name** : `Echo‑Prod‑us‑east‑1`.
4. **Environment** : `prod`.
5. **Region** : *US East (N. Virginia)*.
6. **Schedule** : dropdown `Every 5 minutes` → traduit en cron `*/5 * * * *`.
7. **Advanced** → *Request timeout* 10 000 ms, *Retries* 2, *Max failures before alert* 1.
8. **Integrations** → Slack ➜ coller le webhook channel `#api-alerts`.
9. **Create Monitor** → Postman lance un premier run « baseline ».

### Facturation & limites Free tier

| Fréquence | Runs/mois/monitor   | Runs mensuel (1 monitor) | Quota Free (10 000) | Coût payant (\~0,0001 \$/run) |
| --------- | ------------------- | ------------------------ | ------------------- | ----------------------------- |
| 1 min     | 43 200              | 43 200                   | 🚫 dépasse quota    | 4,3 \$ / monitor / mois       |
| 5 min     | 8 640               | 8 640                    | ✔️ OK               | 0,86 \$                       |
| 15 min    | 2 880               | 2 880                    | ✔️ large marge      | 0,28 \$                       |
| Webhook   | dépend déploiements | \~300                    | ✔️ négligeable      | < 0,03 \$                     |

> **Astuce coût :** combinez CRON 15 min + Webhook « on=deploy » pour garder un œil régulier tout en validant chaque release.

### Check‑list création monitor

* [ ] Collection attachée avec tests SLI.
* [ ] Environment `prod` chargé de vrais tokens.
* [ ] 3 régions créées (`us-east-1`, `eu-west-1`, `ap-southeast-1`).
* [ ] Slack webhook OK ➜ test message envoyé.

---

## 8.3 Tests SLI/SLO en détail

#### Script générique avec variables SLO

```js
const allowedLatency = +(pm.environment.get("slo_latency") || 800);
const allowedCodes   = [200,201,204];

pm.test("LAT p95 < " + allowedLatency + " ms", () => {
  pm.expect(pm.response.responseTime).below(allowedLatency);
});

pm.test("Code OK", () => pm.expect(pm.response.code).oneOf(allowedCodes));
```

*Note :* Vous pouvez stocker les **SLO** dans l’environnement, voire les importer dynamiquement depuis un doc YAML de référence.

#### Budget d’erreurs glissant

```js
const budgetKey = `errBudget_${pm.info.requestName}`;
let used = +(pm.globals.get(budgetKey) || 0);
used += (pm.response.code >= 500) ? 1 : 0;
pm.globals.set(budgetKey, used);

pm.test("Budget erreurs 24h < 5", () => pm.expect(used).below(5));
```

Un `Pre-request` réinitialise les compteurs every 24 h.

---

## 8.4 Alerting enrichi : Slack, Datadog, PagerDuty

### Slack (format BlockKit)

```json
{
  "blocks": [
    {"type":"section","text":{"type":"mrkdwn","text":"*:red_circle: API Echo Prod FAILURE*"}},
    {"type":"section","fields":[
       {"type":"mrkdwn","text":"*Status*: 500"},
       {"type":"mrkdwn","text":"*Latency*: 1350 ms"}]},
    {"type":"context","elements":[{"type":"mrkdwn","text":"Region: us‑east‑1 | Run ID: {{runId}}"}]}
  ]
}
```

> **Avantage :** plus lisible qu’un simple texte brut.

### Datadog Logs ➜ Metric ➜ Monitor

1. Settings ▸ Integrations ▸ **API Key**.
2. Postman envoie un log JSON (service, status, latency, region).
3. Créez `avg:postman.latency{service:echo,region:us-east-1}`.
4. Datadog Monitor : alerte > 800 ms p95 sur 5 min.

### PagerDuty – mapping de sévérité

| Event Postman | Routing PagerDuty           |
| ------------- | --------------------------- |
| `major`       | Escalation Tier‑1 (on‑call) |
| `minor`       | Tier‑2 (biz hours)          |
| `info`        | Pas d’alerte                |

---

## 8.5 Orchestration avancée

### CRON avancé

* `0 7 * * MON-FRI` ➜ run à 07:00 UTC jours ouvrés.
* `*/2 9-18 * * *` ➜ toutes les 2 min en heures de bureau (9‑18 h).

### Webhook après déploiement – GitLab CI

```yaml
deploy_prod:
  script:
    - ./run-deploy.sh
    - curl -X POST $MONITOR_URL -H "x-api-key: $POSTMAN_KEY" -d '{}'
```

### Flows 2.0 conditional

* **Node** *Run Monitor Prod* → **If** `tests.success == true` → *Run Monitor Staging* (smoke tests) → *If success* → *Slack message « Deployment OK »*.

---

## 8.6 Réduire faux positifs & optimiser coûts

| Technique                     | Effet                  | Exemple                              |
| ----------------------------- | ---------------------- | ------------------------------------ |
| **Retry interne**             | Absorbe blip réseau    | Exponential backoff 100‑800 ms       |
| **Jitter délai**              | Évite hitting pattern  | `delay = 3000 + $randomInt`          |
| **Alert threshold > 2 fails** | Ignore burp isolé      | Monitor setting « Alert after 2 »    |
| **Split Critical / Info**     | Slack #alerts vs #info | 2 Collections, 2 Monitors            |
| **Webhook on‑deploy**         | Réduit runs mensuels   | 12 déploiements/jour → 360 runs/mois |

---

## 8.7 Check‑list Monitoring

* [ ] Monitors multi‑régions (`us-east‑1`, `eu-west‑1`, `ap-southeast‑1`).
* [ ] SLO latence p95 < 800 ms, taux d’erreur < 0.1 %.
* [ ] Alertes Slack (BlockKit) + dashboard Datadog + escalade PagerDuty.
* [ ] Budget erreurs glissant réinitialisé chaque 24 h.
* [ ] Aucun faux positif sur 7 jours, coût mensuel < 10 \$.

> 👉 **Page suivante** – Mock Servers, Service Virtualization & Chaos Testing : débloquez vos front‑end dev avant que l’API réelle n’existe, simulez des pannes pour tester la résilience.
