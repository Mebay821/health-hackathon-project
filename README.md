# health-hackathon-project
AI diabetes risk screening prototype for community health workers. Enter age, BMI, blood pressure and glucose to get a risk level and a refer-for-testing flag, tuned to minimize missed cases. Built for the Veritas AI Hackathon 2026.  
 # SugarSpot: Diabetes Risk Screening for Community Health Workers

**Track:** AI + Health | **Team:** WIM (Wings of Innovation in Medicine): Wada Mirkana, Meba Yonas, Inas Khalid
**Veritas AI International High School Hackathon 2026**

> This project is a prototype and is not medical advice.

## The problem
Many people have diabetes without knowing it. In many communities the first person they meet is a community health worker, who has limited time and often no lab access. They need a fast way to decide who should be sent for a blood test first.

## What we built
A screening tool for a community health worker. They enter four simple measurements (age, BMI, blood pressure, glucose) and get back a risk level (Low / Medium / High) with a recommendation such as "refer for a blood test". Because a missed case is worse than an unnecessary test, we tuned the model to catch as many real cases as possible, and we report missed cases and false alarms separately.

## What we added
- **Our own question:** does the model work equally well across age groups? We split the test patients into under 30, 30-45 and over 45 and compared recall.
- **Our fix:** we found the model missed more cases in patients under 30, so we tried a lower threshold (0.2) for patients aged 30 and under, chosen on validation data, then tested it once on the held-out test set.
- **Our user:** the risk levels and wording are designed for a health worker deciding who to refer.

## How it works
- **Data:** Pima Indians Diabetes dataset, 768 patients (about 35% with diabetes). Zero values for glucose, blood pressure and BMI are really missing, so we replaced them with the median. Split: 460 training, 154 validation, 154 test. The test set was used once, at the end.
- **Baseline:** predict "no diabetes" for everyone.
- **Model:** logistic regression with balanced class weights, using age, BMI, blood pressure and glucose.
- **Threshold:** we lowered the cutoff from 0.5 to 0.3 (chosen on the validation set) because a missed case costs more than a false alarm.

## Results (held-out test set: 54 sick, 100 healthy patients)
| | Accuracy | Recall (cases caught) | Missed cases | False alarms |
|---|---|---|---|---|
| Baseline: always "no diabetes" | 64.9% | 0% | 54 | 0 |
| Model, threshold 0.3 | 66.9% | 87.0% | 7 | 44 |
| Model, per-age threshold | 61.7% | 90.7% | 5 | 54 |

**Fairness check by age (threshold 0.3):**
| Age group | Patients | Sick | Caught | Missed | Recall |
|---|---|---|---|---|---|
| Under 30 | 89 | 18 | 14 | 4 | 78% |
| 30-45 | 40 | 26 | 23 | 3 | 88% |
| Over 45 | 25 | 10 | 10 | 0 | 100% |

The per-age threshold raised recall for under-30s from 78% to 89% (16 of 18), at the cost of 10 extra false alarms in that group (37 vs 27).

**Where it fails:**
- It still missed 5 sick patients, and flags about half of healthy patients, which means many unnecessary blood tests. We accepted this trade-off because a missed diagnosis is worse than an extra test, but a clinic with limited testing capacity might choose differently.
- Older patients get the most false alarms (about 60% of healthy patients over 45 were flagged). We think this is because age and glucose both raise risk, but we have not proven the cause.
- Example of a wrong prediction: [add one patient from the missed-cases cell and why we think it happened]

## Limitations
- The age-group results use very few sick patients per group (10 to 26), so they are suggestive, not proven.
- The dataset covers one population only (Pima women, 21 and older), so results may not transfer to other groups.
- The test set is small (154 patients).

## What we'd do next
- Test on a second dataset such as CDC BRFSS to see whether the model works on a different population.
- Try the tool with a real health worker to see whether the risk levels are useful.

## How to run
1. Open the notebook in Google Colab: [diabetes_risk_screening](https://colab.research.google.com/drive/1ocrT_IZ-r8z_3XjVv_V8LoEHOg0rs9aA?usp=sharing)
2. Click **Runtime, then Run all**. It takes under a minute and needs no API keys or uploads.
3. The notebook loads the Pima Indians Diabetes data, trains the model, and prints the results.
4. To try the tool yourself, edit the numbers in the last cell and run it: `risk_check(age, bmi, blood_pressure, glucose)`. For example, `risk_check(52, 34, 85, 160)` returns a risk level and a recommendation.

## Sources
- Dataset: Pima Indians Diabetes Database (UCI / NIDDK), loaded from a public copy at github.com/jbrownlee/Datasets
- Libraries: scikit-learn, pandas, NumPy
- AI tools used: Claude (help with planning )
   
