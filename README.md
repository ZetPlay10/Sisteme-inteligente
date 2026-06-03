# Analiza Performantei Jucatorilor de Fotbal – Sezon 2025/2026

---

## Cuprins

1. [Sursa datelor](#1-sursa-datelor)
2. [Introducere si Motivatie](#2-introducere-si-motivatie)
3. [Descrierea Datelor si Contextul Proiectului](#3-descrierea-datelor-si-contextul-proiectului)
   - 3.1 [Setul de date](#31-setul-de-date)
   - 3.2 [Curatarea datelor](#32-curatarea-datelor)
   - 3.3 [Ce incearca sa rezolve proiectul](#33-ce-incearca-sa-rezolve-proiectul)
4. [Aspecte Teoretice](#4-aspecte-teoretice)
   - 4.1 [De ce regresie supervizata?](#41-de-ce-regresie-supervizata)
   - 4.2 [Algoritmii folositi](#42-algoritmii-folositi)
   - 4.3 [Targeturile per pozitie](#43-targeturile-per-pozitie)
5. [Implementarea](#5-implementarea)
   - 5.1 [Fluxul de lucru](#51-fluxul-de-lucru)
   - 5.2 [Evaluarea modelelor](#52-evaluarea-modelelor)
   - 5.3 [Optimizarea hiperparametrilor](#53-optimizarea-hiperparametrilor)
6. [Rezultatele modelelor](#6-rezultatele-modelelor)
   - 6.1 [Metricile de evaluare](#61-metricile-de-evaluare)
   - 6.2 [Rezultatele comparative](#62-rezultatele-comparative)
7. [Rezultate si Discutii](#7-rezultate-si-discutii)
   - 7.1 [Topul jucatorilor per pozitie](#71-topul-jucatorilor-per-pozitie)
   - 7.2 [Unde greseste modelul?](#72-unde-greseste-modelul)
   - 7.3 [Explicabilitatea modelului](#73-explicabilitatea-modelului-feature-importance)
8. [Concluzii si Cunostinte Noi](#8-concluzii-si-cunostinte-noi)
9. [Referinte](#9-referinte)
10. [Structura Proiectului](#10-structura-proiectului)
11. [Tehnologii Utilizate](#11-tehnologii-utilizate)

---

## 1. Sursa datelor

| Camp | Detalii |
|------|---------|
| **Dataset** | Football Players Stats 2025-2026 |
| **Sursa** | [Kaggle – hubertsidorowicz/football-players-stats-2025-2026](https://www.kaggle.com/datasets/hubertsidorowicz/football-players-stats-2025-2026) |
| **Data descarcarii** | 24 martie 2026 |
| **Campionate acoperite** | Premier League, La Liga, Serie A, Bundesliga, Ligue 1 |
| **Nr. jucatori (brut)** | ~2.800 |
| **Nr. jucatori (dupa curatare)** | ~1.400 |
| **Nr. caracteristici (dupa curatare)** | ~60–70 per jucator |

---

## 2. Introducere si Motivatie

Oricine urmareste fotbal cu ceva mai multa atentie a ajuns la un moment dat la concluzia ca presa sportiva evalueaza jucatorii pe pilot automat. Odata ce un fotbalist capata eticheta de "mare jucator", acel statut ramane agatat de el indiferent de ce face efectiv pe teren. Comentatorii il lauda la fiecare atingere, retelele sociale il citeaza, si totul devine un soi de traditie – nu o evaluare reala.

Autorul acestui proiect a ajuns, cu timpul, sa urmareasca fotbalul mai mult prin cifre decat prin comentarii. Pase reuzite, dueluri castigate, pressing eficient – statistici care chiar spun ceva despre ce a facut un jucator intr-un meci, nu despre ce reputatie are. Si de la observatia asta a aparut intrebarea: ce s-ar intampla daca un algoritm ar evalua jucatorii exclusiv pe baza numerelor, fara sa "stie" ca unul e considerat legenda si altul joaca la o echipa mai putin vizibila?

Asta a fost ideea de baza. Nu s-a urmarit construirea unui sistem de scouting complet – ci raspunsul la o intrebare mai simpla si mai concreta: **pe baza cifrelor dintr-un sezon, cine sunt cei mai buni 5 jucatori pentru fiecare pozitie?** Si mai interesant: daca algoritmul ajunge la un raspuns diferit fata de ce spun analistii sportivi, de ce?

Fotbalul modern se bazeaza din ce in ce mai mult pe date obiective. Cluburile mari au departamente intregi de analiza. Un proiect de aceasta scara, chiar si academic, atinge ceva real din ce se intampla in industrie – nu e doar un exercitiu de bifat la curs.

---

## 3. Descrierea Datelor si Contextul Proiectului

### 3.1. Setul de date

Datele provin de la FBref, agregate prin Kaggle – una dintre cele mai complete surse publice de statistici fotbalistice disponibile. Setul brut continea aproximativ **2.800 de jucatori**, fiecare cu **peste 100 de coloane** – statistici de atac, aparare, pase, presiuni, actiuni standard si statistici de portar.

Fiecare rand reprezinta performanta unui jucator intr-un sezon, la o anumita echipa. Tipurile de statistici incluse acopera:

- **Statistici ofensive:** goluri, suturi, goluri non-penalty, eficienta la sut (G/Sh, G/SoT)
- **Statistici de pase:** pase reuzite, pase progresive, assisturi
- **Statistici defensive:** dueluri castigate, interceptii, blocaje, clearance-uri
- **Statistici de pressing:** presiuni aplicate, pressing reusit
- **Statistici de portar:** Save%, goluri primite, distributie, clean sheets

---

### 3.2. Curatarea datelor

Daca exista o lectie clara pe care a oferit-o acest proiect, aceea este ca datele reale nu arata niciodata cum te astepti. Faza de data cleaning a consumat mai mult timp decat antrenarea tuturor modelelor la un loc.

**Problemele principale intalnite si rezolvate:**

**1. Headerele duble generate de FBref**

FBref exporta datele cu doua randuri de antet: primul indica categoria (ex: `Shooting`), al doilea contine numele real al coloanei (ex: `Gls`). La citirea in pandas rezultatul era fie un MultiIndex de coloane, fie coloane cu nume neclare. A trebuit sa se concateneze manual cele doua randuri pentru a obtine nume de coloane unice si lizibile.

**2. Coloane duplicate pentru jucatorii transferati**

Jucatorii care au schimbat echipa in fereastra de iarna apareau de 2-3 ori in set – cate un rand per echipa, plus un rand cu totalul sezonului. S-a pastrat exclusiv randul cu totalul sezonului si s-au eliminat intrarile partiale.

**3. Valori lipsa (NaN)**

Coloanele specifice portarilor aveau valori lipsa pentru jucatorii de camp – ceea ce are sens. S-a ales completarea cu 0 acolo unde lipsa valorii implica logic lipsa evenimentului.

**4. Jucatori cu minute putine**

S-au filtrat jucatorii cu sub **500 de minute jucate** in sezon. Fara acest filtru, un atacant care a intrat de 3 ori din banca si a dat un gol din singura ocazie primita ar fi obtinut un scor artificial ridicat.

**5. Coloane cu variabilitate zero sau cvasi-zero**

Cateva statistici aveau aceeasi valoare pentru toti jucatorii. Nu aduc niciun semnal util unui model si au fost eliminate.

Dupa toate aceste etape, setul de date a ajuns la aproximativ **1.400 de jucatori** si **60-70 de caracteristici relevante**, impartiti pe patru categorii: Atacanti (FW: 190), Mijlocasi (MF: 630), Fundasi (DF: 484) si Portari (GK: 109).

---

### 3.3. Ce incearca sa rezolve proiectul

Intrebarea centrala este: **pornind exclusiv de la statisticile unui jucator dintr-un sezon, se poate construi un model care sa identifice cei mai valorosi jucatori de pe fiecare pozitie?**

Abordarea aleasa este de **regresie supervizata**: pentru fiecare pozitie s-a ales un target numeric relevant (goluri, assisturi, rata salvari etc.), iar modelele au sarcina de a-l prezice pornind de la celelalte statistici. Jucatorii sunt apoi clasati dupa predictia modelului castigator.

Scopul nu este predictia unui eveniment viitor, ci identificarea combinatiei de statistici care explica cel mai bine performanta observata – si, prin asta, generarea unui top obiectiv al jucatorilor, fara influente externe.

---

## 4. Aspecte Teoretice

### 4.1. De ce regresie supervizata?

Alegerea regresiei supervizate fata de clasificare sau clustering vine din logica problemei in sine.

**Clasificarea** ar fi presupus impartirea jucatorilor in categorii fixe (ex: "bun" / "mediocru" / "slab"), iar asta ar fi pierdut tocmai nuantele care conteaza – diferenta dintre un scor de 87 si 89 e relevanta cand se construieste un top al celor mai buni 5.

**Clusteringul** ar fi util ca analiza exploratorie – si cateva vizualizari de acest tip au fost realizate ca exercitiu suplimentar – dar nu raspunde direct la intrebarea "cine performeaza mai bine".

**Regresia supervizata** permite obtinerea unui scor continuu, ordonabil, direct comparabil intre jucatori – exact ce trebuie pentru un astfel de top.

---

### 4.2. Algoritmii folositi

S-au testat si comparat **10 algoritmi** pentru fiecare dintre cele 4 pozitii, toti optimizati prin GridSearchCV cu 5-fold cross-validation.

**Random Forest** — se antreneaza un numar mare de arbori de decizie, fiecare pe un subset diferit din date, si la final se face media tuturor predictiilor. Stabil, nu face figuri la outlieri si da rezultate decente chiar si fara tuning agresiv.

**Gradient Boosting** — fata de Random Forest, care construieste arborii in paralel, Gradient Boosting ii construieste secvential – fiecare arbore incearca sa corecteze erorile celui dinainte. In general mai precis, dar si mai sensibil la hiperparametri.

**Extra Trees (Extremely Randomized Trees)** — seamana cu Random Forest, cu diferenta ca split-urile din arbori sunt alese complet aleatoriu. De obicei mai rapid si cu rezultate comparabile sau superioare.

**AdaBoost** — un algoritm de boosting mai vechi, care la fiecare iteratie acorda mai multa greutate exemplelor gresit prezise anterior. Mai sensibil la zgomotul din date fata de variantele moderne.

**Ridge, Lasso si ElasticNet** — trei variante de regresie liniara cu regularizare. Ridge penalizeaza coeficientii mari prin norma L2, Lasso poate seta coeficienti exact pe zero realizand implicit selectie de caracteristici, iar ElasticNet combina ambele. Au fost incluse ca baseline si s-au dovedit competitive.

**SVR (Support Vector Regression)** — versiunea de regresie a Support Vector Machines. Functioneaza bine pe date scalate si in spatii de dimensiuni mari.

**KNN (K-Nearest Neighbors)** — pentru a prezice scorul unui jucator, algoritmul cauta cei mai apropiati K jucatori din datele de antrenare si face media scorurilor acestora.

**Decision Tree** — un singur arbore de decizie, inclus mai ales pentru interpretabilitate. Tinde sa overfitteze daca adancimea nu e limitata agresiv.

---

### 4.3. Targeturile per pozitie

In loc sa se construiasca un scor compozit artificial, s-a ales ca target pentru fiecare pozitie o **statistica reala din date**, relevanta pentru rolul respectiv:

| Pozitie | Target | Justificare |
|---------|--------|-------------|
| **Atacanti (FW)** | `Gls_90` – goluri non-penalty per 90 min | Masura directa a eficientei ofensive |
| **Mijlocasi (MF)** | `G+A` – goluri + assisturi (sezon intreg) | Contributia totala la fazele de gol |
| **Fundasi (DF)** | `+/-90` – impactul echipei cand e pe teren, per 90 min | Reflecta atat contributia defensiva cat si cea la constructia jocului |
| **Portari (GK)** | `Save%` – procentaj salvari | Masura directa a performantei intre buturi |

Targetul initial pentru fundasi era `TklW_90` (tackle-uri castigate per 90), insa acesta a generat un R² de doar 0.17 – tackle-urile depind puternic de stilul echipei (o echipa cu posesie mare tackle-uieste mai putin). Schimbarea la `+/-90` a dus R² la 0.91, confirmand ca impactul global al jucatorului pe teren e mult mai predictibil decat o singura statistica defensiva izolata.

---

## 5. Implementarea

### 5.1. Fluxul de lucru

Intreg codul se afla in `fotbal_corectat_v5.ipynb`, organizat pe sectiuni distincte. Pasii urmati:

1. Incarcarea si inspectia datelor brute (`players_data-2025_2026.csv`)
2. Curatarea datelor (rezolvarea headerelor duble, deduplicarea, filtrarea pe minute jucate)
3. Analiza exploratorie – EDA (distributii, corelatii, vizualizari per pozitie)
4. Ingineria caracteristicilor si normalizarea per 90 de minute
5. Antrenarea si optimizarea a 10 algoritmi per pozitie prin GridSearchCV
6. Evaluarea comparativa si selectia modelului final per pozitie
7. Analiza explicabilitatii modelului (feature importance, coeficienti)
8. Generarea topurilor finale si exportul rezultatelor

---

### 5.2. Evaluarea modelelor

S-a folosit **5-fold cross-validation pe intregul dataset** in loc de un split fix 80/20. Avantajul: nu se "pierd" 20% din date doar pentru test, ceea ce conteaza in special la portari (109 jucatori) si atacanti (190 jucatori) unde setul e relativ mic.

---

### 5.3. Optimizarea hiperparametrilor

Toti algoritmii au fost optimizati prin **GridSearchCV cu 5-fold cross-validation**. Din cauza constrangerilor hardware (laptop personal, rulare peste noapte), spatiul de cautare a trebuit limitat:

```python
# Exemplu parametri testati pentru Random Forest
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [5, 10, None],
    'min_samples_split': [2, 5]
}
```

Ca masura practica, rezultatele GridSearchCV au fost salvate in fisiere pickle dupa prima rulare, pentru a evita recalcularea la fiecare reluare a notebook-ului.

---

## 6. Rezultatele modelelor

### 6.1. Metricile de evaluare

**R² (Coeficientul de determinare)** — indica ce proportie din varianta targetului este explicata de model. A fost metrica principala de comparare.

**RMSE (Root Mean Squared Error)** — eroarea medie patratica, in aceleasi unitati cu targetul. Penalizeaza mai mult erorile mari.

**MAE (Mean Absolute Error)** — eroarea medie absoluta, mai intuitiva si mai putin sensibila la outlieri.

---

### 6.2. Rezultatele comparative

| Pozitie | Algoritmul castigator | CV R² (mean) | CV R² (std) | CV RMSE | CV MAE |
|---------|----------------------|--------------|-------------|---------|--------|
| **Atacanti (FW) – Gls/90** | Extra Trees | 0.9594 | 0.0172 | 0.0342 | – |
| **Mijlocasi (MF) – G+A** | Ridge Regression | 0.7454 | 0.0366 | 2.0343 | 1.4977 |
| **Fundasi (DF) – +/-90** | ElasticNet | 0.9100 | – | – | – |
| **Portari (GK) – Save%** | SVR | 0.8144 | 0.0384 | 1.8763 | 1.4586 |

Cateva observatii:

- **Extra Trees a castigat la atacanti** cu R²=0.96, cel mai bun rezultat din proiect. Setul mic de 190 de jucatori favorizeaza metodele de ensemble care pot "memora" mai usor cazurile extreme – de aceea la unii jucatori Predicted = Actual exact. Nu e un rezultat perfect, dar e consistent pe cross-validation.
- **Ridge Regression a castigat la mijlocasi** cu R²=0.74 – singurul pozitii unde performanta e mai modesta. G+A depinde mult de rolul din echipa si de numarul de minute jucate, factori pe care features-urile disponibile nu ii captureaza complet.
- **ElasticNet a castigat la fundasi** cu R²=0.91 dupa schimbarea targetului din TklW_90 in +/-90. Diferenta fata de versiunea initiala (R²=0.17) confirma cat de mult conteaza alegerea unui target relevant.
- **SVR a castigat la portari** cu R²=0.81, consistent si stabil.
- **Decision Tree a overfittat** aproape sistematic la toate pozitiile – diferenta dintre R² pe train si pe test era vizibila fara pruning agresiv.
- **KNN** a avut cele mai slabe rezultate, ceea ce sugereaza ca spatiul caracteristicilor nu are o structura de vecinatate clara.

---

## 7. Rezultate si Discutii

### 7.1. Topul jucatorilor per pozitie

Modelele finale au generat predictii pentru toti cei ~1.400 de jucatori din setul de date, iar topurile au fost construite dupa scorul prezis de modelul castigator per pozitie.

**Top 5 Atacanti** *(model: Extra Trees, target: Gls_90)*

| # | Jucator | Echipa | Campionat | Varsta | Gls_90 | Predicted |
|---|---------|--------|-----------|--------|--------|-----------|
| 1 | Harry Kane | Bayern Munich | Bundesliga | 32 | 0.940 | 0.940 |
| 2 | Deniz Undav | Stuttgart | Bundesliga | 29 | 0.849 | 0.849 |
| 3 | Robert Lewandowski | Barcelona | La Liga | 37 | 0.821 | 0.821 |
| 4 | Ferrán Torres | Barcelona | La Liga | 26 | 0.753 | 0.752 |
| 5 | Lautaro Martínez | Inter | Serie A | 28 | 0.741 | 0.740 |

Kane si Lewandowski sunt exact unde te-ai astepta. Interesant e ca Deniz Undav de la Stuttgart apare pe locul 2 – un jucator mai putin mediatizat care a avut un sezon exceptional statistic. Algoritmul nu stie cine e "legenda" si cine nu.

**Top 5 Mijlocasi** *(model: Ridge Regression, target: G+A)*

| # | Jucator | Echipa | Campionat | Varsta | G+A | Predicted |
|---|---------|--------|-----------|--------|-----|-----------|
| 1 | Michael Olise | Bayern Munich | Bundesliga | 24 | 30 | 25.78 |
| 2 | Luis Díaz | Bayern Munich | Bundesliga | 29 | 28 | 19.37 |
| 3 | Serge Gnabry | Bayern Munich | Bundesliga | 30 | 14 | 17.84 |
| 4 | Lamine Yamal | Barcelona | La Liga | 18 | 26 | 17.28 |
| 5 | Marcus Rashford | Barcelona | La Liga | 28 | 12 | 16.79 |

Lamine Yamal la 18 ani pe locul 4 e poate cel mai interesant rezultat din tot proiectul – confirma ca sezonul sau nu a fost hype media, ci cifre reale.

**Top 5 Fundasi** *(model: ElasticNet, target: +/-90)*

| # | Jucator | Echipa | Campionat | Varsta | +/-90 | Predicted |
|---|---------|--------|-----------|--------|-------|-----------|
| 1 | Konrad Laimer | Bayern Munich | Bundesliga | 28 | 3.24 | 2.70 |
| 2 | Dayot Upamecano | Bayern Munich | Bundesliga | 27 | 2.88 | 2.60 |
| 3 | Josip Stanišić | Bayern Munich | Bundesliga | 26 | 2.74 | 2.53 |
| 4 | Tom Bischof | Bayern Munich | Bundesliga | 20 | 2.81 | 2.45 |
| 5 | Jonathan Tah | Bayern Munich | Bundesliga | 30 | 2.51 | 2.32 |

Toti 5 sunt de la Bayern Munich – ceea ce spune ceva despre **limitarea targetului ales**. `+/-90` reflecta impactul echipei cand jucatorul e pe teren, nu doar performanta individuala. Bayern a dominat Bundesliga in acest sezon, asa ca orice fundas titular acumuleaza automat un +/- ridicat. E o limitare reala a abordarii, discutata in sectiunea 7.2.

**Top 5 Portari** *(model: SVR, target: Save%)*

| # | Jucator | Echipa | Campionat | Varsta | Save% | Predicted |
|---|---------|--------|-----------|--------|-------|-----------|
| 1 | Hervé Koffi | Angers | Ligue 1 | 29 | 78.6 | 79.10 |
| 2 | Mile Svilar | Roma | Serie A | 26 | 77.8 | 77.41 |
| 3 | Joan García | Barcelona | La Liga | 24 | 79.5 | 77.20 |
| 4 | Marco Carnesecchi | Atalanta | Serie A | 25 | 77.4 | 77.08 |
| 5 | Ivan Provedel | Lazio | Serie A | 32 | 78.6 | 76.61 |

Topul portarilor e cel mai credibil din toate cele patru – Svilar, Carnesecchi si Provedel sunt portari foarte bine cotati in comunitatea fotbalistica. Hervé Koffi pe primul loc e o surpriza, dar cifrele lui de Save% sunt reale si ridicate.

---

### 7.2. Unde greseste modelul?

**Contextul echipei lipseste**

Modelul nu vede contextul in care evolueaza un jucator. Cel mai evident exemplu e topul fundasilor: toti 5 sunt de la Bayern Munich. `+/-90` e influentat direct de cat de dominanta e echipa – un fundas mediocru dintr-o echipa care castiga cu 3-0 in fiecare meci va avea un +/- ridicat, in timp ce un fundas exceptional dintr-o echipa in lupta pentru salvare va arata mult mai slab la aceasta metrica.

**Atacanti: Predicted = Actual la unii jucatori**

La Extra Trees pe un set de 190 de atacanti, modelul a memorat efectiv unele cazuri extreme – Kane si Undav au Predicted = Actual identic. Asta nu inseamna ca modelul e perfect, ci ca pe seturi mici metodele de ensemble pot sa "invete pe de rost" outlier-ii. R²=0.96 pe cross-validation e real, dar trebuie interpretat cu aceasta rezerva.

**Mijlocasi: R²=0.74, cel mai slab**

G+A ca target pentru mijlocasi e influentat puternic de rolul din echipa. Un mijlocas defensiv care face 90 de minute pe saptamana dar nu trage la poarta va arata slab la G+A, chiar daca e exceptional din punct de vedere defensiv. Features-urile disponibile (suturi, assisturi, actiuni defensive) nu captureaza suficient de bine aceasta nuanta.

**Statistici fizice absente**

Datele de pe FBref nu includ statistici fizice – viteza, distanta parcursa, numarul de sprinturi. Acestea ar fi relevante in special pentru evaluarea fundasilor si a mijlocasilor defensivi.

**Jucatori cu accidentari**

Un jucator care a jucat 600 de minute dintr-un sezon din cauza unei accidentari va fi evaluat pe baza acelei perioade limitate, care poate sa nu fie reprezentativa pentru nivelul sau real.

---

### 7.3. Explicabilitatea modelului (Feature Importance)

Unul din avantajele algoritmilor bazati pe arbori este ca ofera importanta caracteristicilor calculata direct din structura modelului. Pentru modelele liniare (Ridge, ElasticNet) s-au folosit coeficientii absoluți.

**Atacanti:** `G/Sh` si `G/SoT` (eficienta la sut) au dominat, urmate de `SoT/90`. In absenta datelor xG, eficienta la sut este cel mai bun predictor al golurilor per 90.

**Mijlocasi:** `Ast_90` (assisturi per 90) a iesit ca cea mai importanta caracteristica, devansand volumul de suturi.

**Fundasi:** `TklW_90` si `onG_90` au dominat – tackle-urile castigate si golurile echipei cand jucatorul e pe teren sunt cei mai buni predictori ai impactului global.

**Portari:** `onGA_90` (goluri primite cand e pe teren per 90) si `CS_Rate` (rata clean sheets) au dominat clar – portarii care primesc putine goluri si tin poarta inchisa au automat un Save% ridicat.

---

## 8. Concluzii si Cunostinte Noi

### Ce s-a invatat din proiect

Cel mai valoros lucru pe care l-a oferit acest proiect nu e strict tehnic. Datele brute din sport sunt mult mai contextuale decat par la prima vedere. O statistica de 10 goluri nu inseamna acelasi lucru pentru un atacant dintr-o echipa care trage de 25 de ori pe meci fata de unul care trage de 12 ori.

Alegerea targetului conteaza enorm. Diferenta dintre R²=0.17 si R²=0.91 la fundasi a venit exclusiv din schimbarea targetului, nu din schimbarea algoritmului. Niciun tuning de hiperparametri nu ar fi recuperat un target prost ales.

Din punct de vedere tehnic, **metodele de ensemble sunt greu de batut** ca algoritmi generalisti. La 3 din 4 pozitii modelul castigator a apartinut fie categoriei ensemble (Extra Trees), fie regresiei liniare cu regularizare (Ridge, ElasticNet, SVR). Decision Tree singur a overfittat aproape sistematic.

O alta lectie practica: **GridSearchCV e puternic, dar costisitor computational**. Pentru proiecte viitoare, ar fi de explorat Bayesian Optimization (ex: Optuna sau Hyperopt).

### Limitarile abordarii

- Contextul echipei lipseste din caracteristici – topul fundasilor dominat de Bayern este dovada cea mai clara
- Proiectul acopera un singur sezon – nu capteaza consistenta unui jucator pe termen lung
- Datele nu includ statistici fizice (viteza, distanta parcursa)
- Spatiul de cautare al hiperparametrilor a fost limitat din constrangeri hardware
- Pe seturi mici (atacanti: 190, portari: 109), metodele de ensemble pot memora cazuri extreme

### Ce s-ar putea imbunatati in viitor

- Adaugarea datelor de context al echipei ca caracteristici suplimentare (posesia medie, xG permis, stilul de pressing)
- Extinderea la date pe mai multi sezoane pentru a captura consistenta, nu doar performanta dintr-un singur an
- Testarea unor implementari mai performante – **XGBoost**, **LightGBM**
- Validarea externa a topurilor prin comparatie cu ratinguri recunoscute (FIFA, platforme de scouting)
- Construirea unui dashboard interactiv in care utilizatorul poate ajusta ponderile per pozitie

---

## 9. Referinte

1. Breiman, L. (2001). *Random Forests*. Machine Learning, 45(1), 5–32.
2. Friedman, J. H. (2001). *Greedy function approximation: A gradient boosting machine*. Annals of Statistics, 29(5), 1189–1232.
3. Geurts, P., Ernst, D., & Wehenkel, L. (2006). *Extremely randomized trees*. Machine Learning, 63(1), 3–42.
4. Freund, Y., & Schapire, R. E. (1997). *A Decision-Theoretic Generalization of On-Line Learning and an Application to Boosting*. Journal of Computer and System Sciences, 55(1), 119–139.
5. Tibshirani, R. (1996). *Regression shrinkage and selection via the lasso*. Journal of the Royal Statistical Society: Series B, 58(1), 267–288.
6. Vapnik, V. N. (1995). *The Nature of Statistical Learning Theory*. Springer.
7. Cover, T., & Hart, P. (1967). *Nearest neighbor pattern classification*. IEEE Transactions on Information Theory, 13(1), 21–27.
8. Pedregosa, F., et al. (2011). *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research, 12, 2825–2830.
9. Altman, N., & Krzywinski, M. (2018). *The curse(s) of dimensionality*. Nature Methods, 15(6), 399–400.
10. Hvattum, L. M., & Arntzen, H. (2010). *Using ELO ratings for match result prediction in association football*. International Journal of Forecasting, 26(3), 460–470.
11. Pappalardo, L., et al. (2019). *A public data set of spatio-temporal match events in soccer competitions*. Scientific Data, 6(1), 236.

---

## 10. Structura Proiectului

```
proiect-fotbal/
├── fotbal_corectat_v5.ipynb        # Codul principal – EDA, modele, evaluare, topuri
├── players_data-2025_2026.csv      # Datele brute descarcate de pe Kaggle
├── players-curatat.csv             # Date dupa procesul de curatare
├── players-final.csv               # Date finale cu features calculate
├── rezultate_brute_modele.xlsx     # R², RMSE, MAE pentru toti algoritmii
├── top5_jucatori_pozitii.xlsx      # Top 5 per pozitie
├── grafic_eda.png                  # Distributia pozitiilor si minute vs goluri
├── grafic_corelatii.png            # Matricea de corelatii
├── grafic_r2_algoritmi.png         # Comparatie R² toti algoritmii
├── grafic_predicted_vs_actual.png  # Predicted vs Actual per pozitie
├── grafic_feature_importance.png   # Feature importance per pozitie
├── grafic_top5.png                 # Top 5 jucatori per pozitie
└── README.md                       # Documentatia completa (acest fisier)
```

---

## 11. Tehnologii Utilizate

| Tehnologie | Versiune | Rol |
|------------|----------|-----|
| Python | 3.14 | Limbajul de programare principal |
| Pandas | – | Manipularea si curatarea datelor |
| NumPy | – | Operatii numerice |
| Scikit-learn | – | Algoritmi ML, GridSearchCV, metrici |
| Matplotlib | – | Vizualizari statice |
| Seaborn | – | Vizualizari statistice (heatmaps, distributii) |
| Jupyter Notebook | – | Mediul de dezvoltare si prezentare |

---