# Atelier — Construire un agent ReAct multi-outils avec Claude

*Document de préparation — sera transformé en slides*

---

## 1. Objectif de l'atelier

Faire comprendre à un groupe d'étudiants, par la pratique, ce qu'est un agent
LLM basé sur le pattern **ReAct** (Reasoning + Acting), en partant du papier
de recherche original et en le réappliquant sur un nouveau cas d'usage, avec
du vrai code testé pas à pas.

**À la fin de l'atelier, les étudiants doivent être capables de :**
- Expliquer la boucle Thought → Action → Observation
- Comprendre pourquoi un parsing de texte brut est fragile
- Utiliser le tool use structuré d'une API LLM pour fiabiliser un agent
- Construire et tester un agent multi-outils, brique par brique

---

## 2. Rappel du concept : qu'est-ce que ReAct

Le papier *"ReAct: Synergizing Reasoning and Acting in Language Models"*
(Yao et al., ICLR 2023) répond à un problème simple : un LLM seul, soit
**raisonne** (chain-of-thought, mais peut halluciner car il n'a que sa
mémoire interne), soit **agit** (déclenche des outils, mais sans ajuster
son raisonnement à chaque étape).

ReAct combine les deux dans une boucle :

```
Thought  → je réfléchis à ce qu'il me manque
Action   → j'appelle un outil
Observation → je récupère un résultat réel
   ↓ (on recommence jusqu'à la réponse finale)
```

Ce pattern est aujourd'hui la base de nombreux outils que les étudiants
connaissent déjà sans le savoir : les agents LangChain/LangGraph, le
comportement agentique de Claude Code, ou l'API Assistants d'OpenAI
suivent tous une variante de cette boucle.

---

## 3. Choix du sujet de l'atelier

**Sujet retenu : un agent qui répond à des questions nécessitant une
recherche encyclopédique *et* un calcul.**

Exemple de question type : *"Quelle est la population de la ville de
naissance d'Albert Einstein, divisée par 1000 ?"*

### Pourquoi ce sujet

- **Reproductible sans compte payant ni clé tierce** — la recherche utilise
  l'API publique Wikipédia (aucune inscription nécessaire), seule la clé
  Claude est requise.
- **Multi-outils, contrairement au papier original** — le repo `ysymyth/ReAct`
  n'utilise qu'un seul outil (`Search` sur Wikipédia). Ici on en ajoute un
  second (`calculer`), ce qui illustre un vrai enjeu de conception : comment
  un agent choisit *quel* outil utiliser, dans quel ordre.
- **Neutre et transposable** — ne dépend d'aucun métier particulier ; chaque
  étudiant peut ensuite remplacer les outils par ceux de son propre projet
  (support client, analyse de données, veille, etc.) sans changer la
  mécanique de la boucle.
- **Sécurité pédagogique intégrée** — le calcul est fait avec un évaluateur
  restreint (`ast`), pas un `eval()` brut, ce qui permet d'aborder un vrai
  sujet de sécurité : ne jamais exécuter aveuglément du texte généré par un
  LLM.

---

## 4. Ce qu'on réutilise du papier original, ce qu'on remplace

| Élément du papier original (`ysymyth/ReAct`) | Ce qu'on garde | Ce qu'on remplace et pourquoi |
|---|---|---|
| Boucle Thought/Action/Observation | ✅ Le principe, identique | — |
| Un seul outil (`Search` Wikipédia) | ✅ La recherche Wikipédia | ➕ Ajout d'un second outil (`calculer`) pour illustrer l'orchestration multi-outils |
| Parsing par `.split("\nAction 1: ")` | ❌ Abandonné | Remplacé par le **tool use natif** de l'API Claude — élimine la classe de bugs liée au format de texte (rencontrée concrètement lors des tests préparatoires : réponses mal formatées, erreurs de `stop_sequences`) |
| Appels `openai.Completion.create` | ❌ Abandonné | Remplacé par `client.messages.create` (API Claude), avec gestion des différences de paramètres (`temperature`/`top_p` non cumulables, pas de `frequency_penalty`) |
| `env.step()` avec `gym` | ❌ Abandonné | Remplacé par de simples fonctions Python (`search_wikipedia`, `safe_calculate`) — pas besoin de la couche `gym`/RL pour ce cas d'usage |

---

## 5. Déroulement du notebook, bloc par bloc

Le notebook (`atelier_react_agent.ipynb`) est structuré pour que **chaque
bloc soit testable indépendamment** avant d'être assemblé — c'est la
méthode à transmettre aux étudiants : ne jamais tester une boucle complexe
avant d'avoir validé chaque brique séparément.

| Bloc | Contenu | Ce qu'on teste | Résultat attendu |
|---|---|---|---|
| 1 | Setup (imports, client Claude) | La cellule s'exécute sans erreur | Aucune sortie visible, pas d'exception |
| 2 | Définition des outils (`search_wikipedia`, `safe_calculate`) | Rien encore — définitions seules | — |
| 3 | **Tests unitaires** des outils, hors LLM | Recherche Wikipédia réelle, calcul, rejet d'une expression dangereuse | Un extrait de texte Wikipédia, un résultat numérique, un message de rejet de sécurité |
| 4 | `TOOLS_SCHEMA` + `execute_tool()` (le routeur) | Définition seule | Message de confirmation (nombre d'outils déclarés) |
| 5 | **Test du routeur**, sans Claude | Appels manuels à `execute_tool(...)` | Les bons résultats renvoyés pour chaque outil, y compris un outil inconnu |
| 6 | La boucle `run_react_tool_use(...)` | Définition seule | — |
| 7 | **Test de bout en bout** | Question nécessitant recherche + recherche + calcul | Plusieurs tours Thought/Action/Observation affichés, réponse finale cohérente |
| 8 | Exercices pour aller plus loin | — | Pistes pour les étudiants (ajouter un outil, casser la sécurité volontairement, réduire `max_steps`) |

---

## 6. Message clé pour les étudiants

> Le pattern ReAct n'est pas un algorithme figé à copier-coller — c'est une
> **méthode de conception** : décider quelles actions un agent peut faire,
> les décrire clairement, et laisser le modèle choisir quand les utiliser.
> La partie la plus fragile n'est presque jamais le modèle lui-même, mais
> **l'interface entre le texte généré et le code qui l'exécute** — d'où
> l'intérêt du tool use structuré plutôt que du parsing de texte libre.

---

## 7. Structure du dépôt Git à pousser

```
atelier-react-agent/
├── README.md                     # présentation, installation, usage
├── requirements.txt               # dépendances Python
├── .gitignore                     # exclut .venv, .env, checkpoints Jupyter
├── atelier_react_agent.ipynb      # le notebook de l'atelier
└── doc_atelier.md                 # ce document (base des slides)
```

**Rappel avant de pousser** : ne jamais committer la clé API — elle doit
rester dans une variable d'environnement ou un fichier `.env` non versionné
(exclu via `.gitignore`).

---

## 8. Conclusion de l'atelier

Les étudiants repartent avec :
- Un notebook fonctionnel et commenté, à réutiliser comme template
- Une compréhension pratique (pas seulement théorique) du pattern ReAct
- Une méthode de travail : tester chaque brique isolément avant d'assembler
- Un repo Git propre, prêt à être adapté à leurs propres projets futurs
