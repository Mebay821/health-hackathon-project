 # SugarSpot

**Track:** Health
**Team:** [Member 1 name], [Member 2 name], [Member 3 name]

> **This project is a prototype and is not medical advice.**

*Wings to lift, insight to guide, SugarSpot to heal.*

# The problem

In many communities in Ethiopia, people with raised blood sugar are found only after complications appear, and Health Extension Workers (HEWs) have far more people to see than time. SugarSpot helps one **HEW on a home or clinic visit** (and a **client** screening themselves) decide who should get a glucose test first, using four routine numbers (age, BMI, blood pressure, glucose), even when the person does not know some of them.

# What we built

**Goes in:** age, BMI, diastolic blood pressure and fasting glucose, typed or spoken. If the client does not know their BMI, blood pressure or glucose, a smart wizard (height and weight) works out BMI and the app estimates the other two.
**Comes out:** a risk level (Low / Moderate / High), a probability, plain-language advice and a spoken result in the chosen language.

**How it is used**
- **Client portal:** sign up (with a guardian's details if under 18), receive an ID, screen, save medicines (type the name or photograph the label) and habits, download data as CSV/PDF, message a HEW, and find nearby facilities by GPS.
- **HEW portal:** log in with institution name and access code, see records of clients who agreed to share (sorted by risk), ask clients questions, send medical suggestions, post updates, and upload a **local dataset** so the model is retrained on a country's own data.

Run it: open `SugarSpot_app.ipynb` in Google Colab, run the install cell, restart the session, then run the app cell and open the `gradio.live` link. The HEW demo access code is given in the submission notes (not published here).

# What we added

**Our own idea** (same pattern as suggested project 4.1.2, tabular risk from checkup data, but for diabetes). On top of the baseline we added:
- **Our own user:** a design built around the HEW workflow (consent-based sharing, two-way messaging, community risk list).
- **Our own data path:** HEWs can upload a local country dataset (needs at least 30 valid rows); the model for that country is retrained, to reduce bias from the base data.
- **Our own question:** we report how the model does for age groups and where it misses cases.
- Estimators that fill in blood pressure and glucose when they are unknown, voice input/output, 17-language page translation, medicine-label photo reading, and an OpenStreetMap facility finder.

# How it works

**Data:** Pima Indians Diabetes dataset (768 rows, women aged 21+, public). Zeros in glucose, blood pressure and BMI are treated as missing and replaced with the median. We use 4 features: Age, BMI, BloodPressure, Glucose. 80/20 train/test split, stratified, fixed random seed.

**Model:** logistic regression with feature scaling (scikit-learn), chosen because it is simple, gives a probability and can be explained to a client. The alert threshold is 0.30 (0.20 for age 30 or younger) so it misses fewer real cases, at the cost of more false alarms. Blood pressure and glucose estimates come from two small linear regressions (age + BMI) trained on the same data.

# Results

All numbers are on the 20% test set the model never saw. *(Copy them from the "EVALUATION" block printed when the app cell starts.)*

| Measure | Baseline ("always no diabetes") | Our model |
|---|---|---|
| Accuracy | [X]% | [X]% |
| Recall (real cases caught) | 0% | [X]% |
| Precision (alerts that were right) | n/a | [X]% |
| **Missed cases (false negatives)** | all [X] real cases | [X] of [X] |
| **False alarms (false positives)** | 0 | [X] |

By age group: age 30 or younger: recall [X]%, missed [X], false alarms [X]; age over 30: recall [X]%, missed [X], false alarms [X].

**Where it fails, and why we think so:** [paste one or two of the "MISSED" rows printed by the app and describe them, e.g. a person with normal-looking glucose and BMI but real diabetes]. The model only sees four numbers, so people whose risk comes from other things (family history, pregnancies, insulin levels) can look "low". Lowering the threshold catches more cases but raises false alarms (see the numbers above).

**Known problems**
- **Less accurate risk results when there is no data for the user's country.** The base data come from one group (Pima women, 21+) and may be less accurate for men, children and other populations. A country model exists only after a HEW uploads a local dataset, so for most countries (including Ethiopia today) results are less accurate.
- **Estimated blood pressure and glucose are guesses** from age and BMI. The result card says "Estimated (not measured)" and confidence is lower.
- **Local languages are under development.** Amharic, Afaan Oromoo and Tigrinya **voice input and voice output are under development**: Afaan Oromoo and Tigrinya have no voice yet, and Amharic voice is experimental. Their on-screen text uses machine translation that has not been checked by a native speaker.
- Voice, translation, facility search and medicine-photo reading need internet; voice input needs Chrome or Edge.
- Medicine-photo reading knows only about 20 common medicines and struggles with blurry photos.
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
2. Work with HEWs to collect a consented, de-identified Ethiopian dataset so the model is trained on local patterns, then re-test missed cases and false alarms by age and sex.

# Sources

- Pima Indians Diabetes Database (Smith et al., 1988), CSV from https://github.com/jbrownlee/Datasets (original: UCI / Kaggle)
- scikit-learn https://scikit-learn.org · pandas · NumPy · Requests
- Gradio https://www.gradio.app · Google Colab
- gTTS (Google text-to-speech) https://gtts.readthedocs.io · browser Web Speech API
- Google Website Translator https://translate.google.com
- Tesseract OCR / pytesseract (medicine label reading) · ReportLab (PDF export)
- OpenStreetMap and the Overpass API https://www.openstreetmap.org (facility search)
- AI tools used: Claude (help debugging and plan). All code was run by the team.
