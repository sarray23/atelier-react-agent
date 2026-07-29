# Atelier — Agent ReAct multi-outils avec Claude
paper source https://paperswithcode.co/paper/2210.03629
Atelier pédagogique reprenant le pattern **ReAct** (Reasoning + Acting,
Yao et al. 2023) et l'adaptant à un agent multi-outils (recherche
Wikipédia + calcul), avec l'API Claude et son tool use natif.

## Contenu

- `atelier_react_agent.ipynb` — le notebook de l'atelier, testable bloc par bloc
- `doc_atelier.md` — document explicatif complet (base des slides de présentation)
- `requirements.txt` — dépendances Python

## Installation

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows : .venv\Scripts\activate
pip install -r requirements.txt
export ANTHROPIC_API_KEY="votre_clé_ici"
jupyter notebook atelier_react_agent.ipynb
```

## Prérequis

- Python 3.9+
- Une clé API Claude (aucune autre clé nécessaire — la recherche utilise
  l'API publique Wikipédia)
- Une connexion internet (appels réels à Wikipédia)

## Structure pédagogique

Le notebook est conçu pour que chaque brique (recherche, calcul, routeur,
boucle complète) soit testée isolément avant d'être assemblée. Voir
`doc_atelier.md` pour le déroulement détaillé bloc par bloc.

## Exercices proposés

Voir la dernière section du notebook et de `doc_atelier.md` pour des pistes
d'extension (ajouter un outil, tester les limites de sécurité, réduire le
nombre d'étapes autorisées).
