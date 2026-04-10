# Instagram Trend Jacking — Agent IA de Veille Automatisée
![Badge Production](https://img.shields.io/badge/Status-Production%20v2.1-green) ![Badge Cost](https://img.shields.io/badge/Coût-%3C5%24%2Fmois-yellow) ![Badge Reliability](https://img.shields.io/badge/Fiabilité-Error%20Handling-blueviolet) ![Version](https://img.shields.io/badge/Version-2.1-lightgrey)

Agent IA qui scrape, analyse et livre 3 templates de contenu Instagram prêts à reproduire (Reels, Carrousels, Photos) — 3x/semaine, 0h de veille manuelle, < 5$/mois.

---

## 🎯 Problème & Solution

**Problème :** Les créateurs de contenu dev perso perdent 3-5h/semaine en veille manuelle Instagram — scroller, noter, analyser, sans méthodologie. Résultats inconstants.  
**Solution IA :** Workflow N8N + Apify + Gemini multimodal qui scrape les comptes concurrents et hashtags SEO, score chaque contenu, analyse par type (Reel/Carrousel/Photo), et envoie un mail prescriptif avec 3 templates clé en main.  
**Impact :** 0h de veille (vs 3-5h), 3 formats analysés (vs Reels seuls), 4 appels LLM (vs 50 en v1), < 5$/mois.

---

## 🚀 Quick Start

> Ce projet est un workflow N8N — pas un repo à cloner. Le PRD complet est dans `/docs`.

1. Créer un compte [N8N Cloud](https://n8n.io) + [Apify](https://apify.com) (plan free)
2. Importer le workflow JSON dans N8N
3. Configurer : Apify Token, Google Sheets, Gmail, OpenRouter
4. Ajuster les comptes ciblés et hashtags SEO dans les nœuds Set/Code
5. Activer le Schedule Trigger → le workflow tourne tout seul

---

## 🛠 Tech Stack

| Composant | Outil | Raison Choisi |
|-----------|-------|---------------|
| Orchestration | N8N Cloud | No-code, scheduling, branching natif |
| Scraping | Apify | API Instagram fiable, plan free 5$/mois |
| Analyse IA | Google Gemini | Multimodal (vidéo + texte), gratuit |
| Agent IA | OpenRouter + Gemini | Synthèse prescriptive, coût minimal |
| Stockage | Google Sheets | Accessible au client, 0 coût |
| Livraison | Gmail | Intégration native N8N |

---

## ⚠️ Difficultés Rencontrées & Leçons PM

- **Défi 1 : Agent IA qui boucle** — 50 appels LLM, 10 lectures Sheet, 4 mails envoyés. Cause racine : N8N exécute l'agent 1x par item (10 items = 10 exécutions). Fix : nœud Aggregate en amont + Max Iterations = 3. Résultat : 4 appels. *Leçon : contraindre un agent structurellement, pas seulement par le prompt.*
- **Défi 2 : Structure Apify instable** — Données en profils imbriqués (latestPosts) OU posts individuels selon le mode. Fix : code dual-case qui détecte et s'adapte. *Leçon : ne jamais hardcoder une structure d'API tierce.*
- **Défi 3 : Déduplication qui bloque tout** — Le nœud comparait le Sheet contre lui-même ($input = sortie Get Existing URLs). Tout était "déjà analysé". Fix : référence explicite $('Top 10').all(). *Leçon : dans N8N, $input n'est pas "les données" — c'est "la sortie du nœud d'avant".*
- **Défi 4 : Hashtags sans Reels** — Le scraper en mode hashtag ne retourne que carrousels et photos. Pivot : accepter les 3 formats + adapter l'analyse Gemini par type. *Leçon : un "problème technique" peut devenir un avantage produit.*
- **Défi 5 : Contenus trop anciens** — Sans filtre temporel, analyse de posts vieux de plusieurs mois. Fix : filtre adaptatif (3/4/7 jours selon le jour d'exécution). *Leçon : le scraping sans fenêtre temporelle = du bruit, pas du signal.*

**Key Takeaway :** 60% du temps en debugging et fiabilisation — un workflow "qui marche" n'est pas un workflow "en production".

---

## 🧠 Focus PM : Conception de l'Agent IA

L'agent a traversé 4 versions — chacune a résolu un problème découvert en production :

| Version | Architecture | Appels LLM | Problème résolu |
|---------|-------------|------------|-----------------|
| v1 | Agent libre (3 tools) | 50 | — |
| v2 | Prompt blindé + descriptions tools | ~20 | Réduit les relectures |
| v3 | Max Iterations = 5 | ~20 | Découverte : 10 items = 10 exécutions |
| **v4** | **Aggregate + 2 tools + Max 3** | **4** | **Production stable** |

**Les 3 leviers de contrôle d'un agent IA (par ordre de fiabilité) :**
1. **Structurel** — Aggregate en amont → 1 exécution garantie. Mécanique, pas probabiliste.
2. **Paramétrique** — Max Iterations = 3 → coupe l'agent après 3 appels. Garde-fou absolu.
3. **Prompt** — Budget d'actions numéroté (1/3, 2/3, 3/3) → guide le comportement. Dernière ligne de défense.

> *Un agent IA n'est fiable que s'il est contraint structurellement. Le prompt complète les contraintes mécaniques, il ne les remplace pas.*

---

## 🛡️ Fiabilisation (5 mécanismes de production)

| Mécanisme | Rôle | Impact |
|-----------|------|--------|
| Error Trigger | Alerte mail au PM si un nœud plante | 0 erreur silencieuse |
| Déduplication | Compare URLs scrapées vs Sheet existant | 0 analyse en double |
| Validation | Vérifie URL, créateur, métriques, shortCode | 0 donnée corrompue |
| Métriques/Logs | Onglet Logs dans le Sheet | Visibilité totale |
| Sticky Notes | 6 notes dans le canvas N8N | Reprise par un autre PM |

---

## 📈 Résultats & Métriques

| Métrique | Avant (v0) | Après (v2.1) | Source |
|----------|-----------|-------------|--------|
| Temps de veille | 3-5h/semaine | 0h | Feedback client |
| Formats analysés | Reels seuls | Reels + Carrousels + Photos | Workflow |
| Appels LLM/run | 50 | 4 | Logs OpenRouter |
| Mails envoyés/run | 3-4 | 1 | Logs Gmail |
| Coût mensuel | ~15$ | < 5$ | Dashboards |
| Pertinence contenu | "Gadget" | Score moyen > 7/10 | Google Sheet |

---

## ✅ Critères d'Acceptation (Pass/Fail)

| Critère | Condition | Statut |
|---------|-----------|--------|
| Déclenchement scheduling | Workflow démarre Lun/Mer/Ven 9h ± 5 min | PASS |
| Error Trigger | Erreur simulée → alerte mail PM < 5 min | PASS |
| Agent bridé | Max 4 appels LLM par run sur 10 runs | PASS |
| 1 seul mail par run | 10 runs consécutifs → 1 mail chacun | PASS |
| Déduplication | URL existante → non réinsérée ni réanalysée | PASS |
| Filtre temporel | Contenu > daysBack → non inclus | PASS |
| Pertinence niche | Score moyen > 7/10 sur 10 analyses | PASS |
| Feedback loop | Formulaire de notation dans le mail | À TESTER (v2.1) |

---

## 🎯 Succès Client — Critères Qualitatifs

Au-delà des métriques techniques, ces critères mesurent l'impact réel sur la stratégie de contenu :

| Critère | Objectif M+3 | Méthode de mesure |
|---------|-------------|-------------------|
| Taux de reproduction des templates | > 50% des templates reproduits | Feedback mensuel + vérif compte |
| Satisfaction qualitative Rym | ≥ 4/5 sur formulaire feedback | Formulaire intégré mail (v2.1) |
| Engagement des posts inspirés | +30% vs baseline | Instagram Insights |
| Pertinence niche perçue | > 80% réponses "Oui" | Formulaire feedback |

---

## 🗺 Roadmap

| Version | Évolution | Horizon |
|---------|-----------|---------|
| v2.1 | Feedback loop + formulaire notation + Dashboard Looker Studio | T2 2026 |
| v3.0 | Architecture multi-clients + génération scripts Reels | T3 2026 |

---

## 👨‍💼 Compétences PM Démontrées

- **Vision → Exécution :** Audit v0 → PRD → POC → MVP → Production (v1.0 → v2.1 en itérations).
- **Debugging systémique :** Cause racine agent (10 items = 10 exécutions), pas du symptôme (prompt trop vague).
- **Arbitrages documentés :** 8 décisions de conception avec justifications.
- **Fiabilisation production :** 5 mécanismes (error handling, dédup, validation, logs, documentation inline).
- **Impact client mesuré :** KPIs qualitatifs (taux de reproduction, satisfaction) en plus des métriques techniques.
- **Outils :** N8N, Apify, Google Gemini, OpenRouter, Google Sheets, Gmail.

---

## 📁 Structure du projet

```
├── README.md
├── docs/
│   ├── PRD_v2.1.docx
│   └── decisions-log.md
├── prompts/
│   ├── gemini-reel.md
│   ├── gemini-carrousel.md
│   ├── gemini-photo.md
│   └── agent-phase6.md
└── scripts/
    ├── normalize-a.js / normalize-b.js
    ├── score-composite.js / parse-json.js
    ├── validate-data.js / dedup.js
    └── metrics.js / error-formatter.js
```

---

## 🤝 Contact

Portfolio : [github.com/yanisse-devai] | LinkedIn : [linkedin.com/in/yanisse-kemel] | ✉️ yanissekemel@gmail.com

*Projet client — Usage privé.*
