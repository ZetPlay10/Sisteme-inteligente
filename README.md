# ⚽ Analiza Jucătorilor de Fotbal – Sezon 2025/2026

## Sursa datelor
- **Dataset:** Football Players Stats 2025-2026
- **Link:** https://www.kaggle.com/datasets/hubertsidorowicz/football-players-stats-2025-2026
- **Data descărcării:** 24 martie 2026

## Obiectivul proiectului
Proiectul analizează statisticile fotbaliștilor din top 5 campionate europene pentru a evalua performanța acestora pe baza datelor reale. Scopul principal este identificarea celor mai buni 5 jucători pentru fiecare poziție (Atacant, Mijlocaș, Fundaș, Portar) folosind algoritmi de machine learning.

M-am orientat spre acest subiect pentru că fotbalul modern depinde tot mai mult de date obiective, iar întrebarea dacă un algoritm poate evalua un jucător mai corect decât ochiul uman mi s-a părut interesantă de explorat.

## Ce fel de învățare automată am aplicat?
Am ales **regresia supervizată** deoarece scopul este să estimez scoruri continue de performanță, nu să clasific jucătorii în categorii fixe. Am antrenat și comparat 10 algoritmi pentru fiecare poziție:

Random Forest, Gradient Boosting, Extra Trees, AdaBoost, Ridge, Lasso, ElasticNet, SVR, KNN și Decision Tree – toți optimizați prin GridSearchCV cu 5-fold cross-validation.

## Tehnologii utilizate
- Python 3
- Pandas, NumPy, Matplotlib, Seaborn
- Scikit-learn
- Jupyter Notebook

## Structura proiectului
```
📁 proiect-fotbal/
├── 📓 fotbal.ipynb                  # Codul principal
├── 📄 players_data-2025_2026.csv    # Datele brute
├── 📄 players-curatat.csv           # Date după curățare
└── 📖 README.md
```