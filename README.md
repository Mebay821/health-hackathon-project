 # SugarSpot

**Track:** Health
**Team:** WADA MIRKANA, MEBA YONAS , INAS KHALID

> **This project is a prototype and is not medical advice.**

*Wings to lift, insight to guide, SugarSpot to heal.*

# The problem

In many communities like our country Ethiopia, people with raised blood sugar are found only after complications appear, and Health Extension Workers (HEWs) have far more people to see than time. SugarSpot helps one **HEW on a home or clinic visit** (and a **client** screening themselves) decide who should get a glucose test first, using four routine numbers (age, BMI, blood pressure, glucose), even when the person does not know some of them.

# What we built

**Goes in:** age, BMI, diastolic blood pressure and fasting glucose, typed or spoken. If the client does not know their BMI, blood pressure or glucose, a smart wizard (height and weight) works out BMI and the app estimates the other two.
**Comes out:** a risk level (Low / Moderate / High), a probability, plain-language advice and a spoken result in the chosen language.

**How it is used**
- **Client portal:** sign up (with a guardian's details if under 18), receive an ID, screen, save medicines (type the name or photograph the label) and habits, download data as CSV/PDF, message a HEW, and find nearby facilities by GPS.
- **HEW portal:** log in with institution name and access code, see records of clients who agreed to share (sorted by risk), ask clients questions, send medical suggestions(⚠️MEDICAL SUGGESTIONS ARE STRICTELY NOT GIVEN BY THE AI.), post updates, and upload a **local dataset** so the model is retrained on a country's own data.

Run it: open the notebook in Google Colab (link under *How it works*) and choose Runtime > Run all. The last cell prints a `gradio.live` link: open it to use the app. The HEW demo access code is given in the submission notes (not published here).

# What we added

**Our own idea** (same pattern as suggested project 4.1.2, tabular risk from checkup data, but for diabetes). On top of the baseline we added:
- **Our own user:** a design built around the HEW workflow (consent-based sharing, two-way messaging, community risk list).
- **Our own data path:** HEWs can upload a local country dataset (needs at least 30 valid rows); the model for that country is retrained, to reduce bias from the base data.
- **Our own question:** who does the model fail for? We tested age groups, found it misses more people aged 30 or younger, and fixed part of that with a lower alert threshold for them chosen on a validation set (not on the test set).
- Estimators that fill in blood pressure and glucose when they are unknown, voice input/output, 17-language page translation, medicine-label photo reading, and an OpenStreetMap facility finder.

# How it works

**Code:** [Open the notebook in Google Colab](https://colab.research.google.com/github/Mebay821/health-hackathon-project/blob/main/diabetes_risk_screening.ipynb) (`diabetes_risk_screening.ipynb`). Choose Runtime > Run all. The notebook has these sections, in order:

| Section | What it does |
|---|---|
| 0. Install | Installs the app's libraries (restart if Colab asks, then Run all) |
| 1. Data | Loads the Pima dataset and replaces impossible zeros with the median |
| 2. Split, baseline, first model | 60/20/20 split, "always no diabetes" baseline, balanced logistic regression |
| 3. Thresholds | Tries alert thresholds 0.5, 0.4, 0.3, 0.2 on the validation set |
| 4. Test results | Scores the model once on the test set |
| 5. Age groups | Missed cases and false alarms for ages 30 or younger, 31 to 45, over 45 |
| 6. Final rule | Threshold 0.2 for age 30 or younger, 0.3 for everyone else |
| 7. Where it fails | Prints the real people the model missed, with their scores |
| 8. Risk check | The function that turns a score into Low, Moderate or High |
| 9. App | The SugarSpot Gradio app; it prints an evaluation summary and a `gradio.live` link |

**Data:** Pima Indians Diabetes dataset (768 rows, women aged 21+, public; 34.9% have diabetes). Zeros in glucose, blood pressure and BMI are treated as missing and replaced with the median. We use 4 features: Age, BMI, BloodPressure, Glucose.

**Split:** 60% train (460 rows) / 20% validation (154) / 20% test (154), stratified, fixed random seed. The validation set is used to choose alert thresholds; the test set is scored once at the end.

**Model:** logistic regression (scikit-learn) with balanced class weights, so the smaller "diabetes" group is not ignored. We chose it because it is simple, gives a probability and can be explained to a client. A risk score at or above the **alert threshold** is High: **0.30**, or **0.20 for age 30 or younger** (lowered because the validation set showed the model missed more young people). Blood pressure and glucose estimates for people who do not know them come from two small linear regressions (age + BMI) trained on the same data. The app uses exactly this model and the same split.

# Results

All numbers are on the 154-person test set the model never saw (54 real cases). The baseline says "no diabetes" for everyone.

| Measure | Baseline (always "no diabetes") | One threshold (0.3) | **Our final rule** (0.2 if age 30 or younger, else 0.3) |
|---|---|---|---|
| Accuracy | 64.9% | 66.9% | 61.7% |
| Recall (real cases caught) | 0% | 87.0% | **90.7%** |
| Precision (alerts that were right) | n/a | 52% | 48% |
| **Missed cases (false negatives)** | all 54 | 7 | **5** |
| **False alarms (false positives)** | 0 | 44 | **54** |

**By age group (final rule):**

| Age group | People | Real cases | Caught | Missed | False alarms |
|---|---|---|---|---|---|
| 30 or younger | 89 | 18 | 16 | 2 | 37 |
| 31 to 45 | 40 | 26 | 23 | 3 | 8 |
| over 45 | 25 | 10 | 10 | 0 | 9 |

With one threshold, the 30-or-younger group had 4 missed cases and 27 false alarms; the lower threshold cut the misses to 2 but added 10 false alarms.

**Honest reading of the results:** accuracy is not the right measure here. Our final rule is *less* accurate than the baseline (61.7% vs 64.9%), because we chose to catch more real cases (49 of 54, where the baseline catches none) and accept more false alarms: more than half of the alerts (54 of 103) are wrong. For a HEW, a false alarm means an extra glucose test, which we judged better than a missed case, but it costs time and supplies. We could only test age groups, not men vs women, because the dataset has only women.

**Where it fails, and why we think so:** young people are the hardest group: even with the lower threshold, 2 of 18 young cases were missed and there were 37 false alarms for them, and we think this is because younger people with diabetes often have only mildly raised numbers that look like healthy people's (we have not tested this explanation). Three of 26 cases aged 31 to 45 were missed too. Section 7 of the notebook prints all 5 missed people with the score the model gave each. The model only sees four numbers, so people whose risk comes from other things (family history, pregnancies, insulin levels) can look "low".

**Known problems**
- **Less accurate risk results when there is no data for the user's country.** The base data come from one group (Pima women, 21+) and may be less accurate for men, children and other populations. A country model exists only after a HEW uploads a local dataset, so for most countries (including Ethiopia today) results are less accurate.
- **Estimated blood pressure and glucose are guesses** from age and BMI. The result card says "Estimated (not measured)" and confidence is lower.
- **Local languages are under development.** Amharic, Afaan Oromoo and Tigrinya **voice input and voice output are under development**: Afaan Oromoo and Tigrinya have no voice yet, and Amharic voice is experimental. Their on-screen text uses machine translation that has not been checked by a native speaker.
- Voice, translation, facility search and medicine-photo reading need internet; voice input needs Chrome or Edge.
- Medicine-photo reading knows only about  common medicines and struggles with blurry photos.
- Facility results come from OpenStreetMap, which is incomplete in rural areas; if it is unreachable a short approximate sample list is shown.
- Accounts are stored in a simple file on the Colab machine (lost when the session resets). Emails are **not** sent: the ID and the password-reset code are shown on screen (demo mode). The HEW access code is a shared demo code, not real authentication.
- The model is not clinically validated. Demo accounts use made-up test data only; no real patient records were used.

# Languages available

| Feature | Languages |
|---|---|
| Page translation (Google Website Translator, internet needed) | English, አማርኛ Amharic, Afaan Oromoo, ትግርኛ Tigrinya, العربية Arabic, Français, Español, Português, Deutsch, Kiswahili, हिन्दी Hindi, 中文 Chinese, 日本語, 한국어, Русский, Türkçe, Italiano (17) |
| Voice output (spoken risk result) | English, Arabic, French, Spanish, Portuguese, German, Swahili, Hindi, Chinese, Japanese, Korean, Russian, Turkish, Italian. **Amharic: experimental.** **Afaan Oromoo, Tigrinya: under development** |
| Voice input (browser speech recognition) | The same languages as the menu in the app. **Amharic: experimental. Afaan Oromoo, Tigrinya: under development** |

# What we'd do next

1. Test the Amharic, Afaan Oromoo and Tigrinya text and voice with native speakers, and build proper voice support for them.
2. Work to collect a consented, de-identified dataset for other countries so the model is trained on local patterns, then re-test missed cases and false alarms by age and sex.

# Sources

- Pima Indians Diabetes Database (Smith et al., 1988), CSV from https://github.com/jbrownlee/Datasets (original: UCI / Kaggle)
- scikit-learn https://scikit-learn.org · pandas · NumPy · Requests
- Gradio https://www.gradio.app · Google Colab
- gTTS (Google text-to-speech) https://gtts.readthedocs.io · browser Web Speech API
- Google Website Translator https://translate.google.com
- Tesseract OCR / pytesseract (medicine label reading) · ReportLab (PDF export)
- OpenStreetMap and the Overpass API https://www.openstreetmap.org (facility search)
- AI tools used:  Claude (to build out the Gradio UI, PDF generator, code debugging, and Whisper integration). All code was run by the team.
