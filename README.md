# HealthSetu AI

**Voice-first triage and public-scheme navigator for Goa, built as a single web page.**

You speak or tap where it hurts. HealthSetu returns a colour-coded urgency token, a 0–100 score, nearby care, and the government schemes that may cover it. A 108 button stays on every screen.

Built by **Team Titans** for the Sankalp Setu AI Hackathon (Health & Wellbeing track). It is a hackathon prototype: decision support and navigation, **not a diagnosis tool**.

**Live demo:** `https://<your-username>.github.io/healthsetu-ai/`

> **Medical disclaimer.** HealthSetu AI does not diagnose, treat or replace a doctor. The triage rules are not clinically validated. In an emergency, call **108**.

## The problem

Suburban and rural Goa faces three stacked problems:

- Triage delays at Primary Health Centres
- Benefits such as DDSSY and Ayushman Bharat that go unclaimed because people don't know which one fits
- Little support in Konkani, Marathi or Hindi

## What it does

- **Voice or tap intake** in English, Hindi, Marathi and Konkani, plus a body map with 25+ regions, front and back, with severity, depth and pain type
- **Rules-first triage** that returns a green, yellow, orange or red token and a 0–100 urgency score, with the reasons shown
- **Optional on-device AI** that reads free text and adds warning flags
- **Scheme matching** for PM-JAY, DDSSY, eSanjeevani, JSY and Jan Aushadhi, with links to the official sites
- **Nearby care** and a **specialist section** (eSanjeevani, place search, and an editable summary you can share)
- **Private health locker**: profile, saved checks and notes, encrypted on the device
- **Support contact**: save a relative and message them in one tap
- One **Call 108** button, always visible

## How severity is decided

No AI model decides severity. This is the main design choice.

1. Free text, body-map choices and toggles become features such as `chest_pain`, `sweating` or `severe`.
2. A **hand-written rules table** (`RULES` in `index.html`) maps features to a tier. The highest tier that fires wins. For example, chest pain with sweating gives red.
3. The optional **Qwen2.5-0.5B** model returns tags from a fixed list. Tags are added to the same feature set, so they can **raise** urgency but never lower it.
4. A **score** places the case inside the tier's band: green 5–39, yellow 40–59, orange 60–74, red 75–100. Position within the band comes from severity (45%), duration (20%), age (15%) and the number of rules that fired (20%). The weights are our own choices and are not validated.

The colour is set by the rules, not the number, so a wrong weight cannot hide an emergency.

## Architecture

| Part | Tool | Role |
|---|---|---|
| Triage | Rules table and scoring formula in plain JavaScript | Decides tier and score |
| Language model | Transformers.js + Qwen2.5-0.5B-Instruct (ONNX), in a Web Worker | Optional, escalate-only flags |
| Embeddings | paraphrase-multilingual-MiniLM-L12-v2 | Re-ranks scheme matches across languages |
| Retrieval | MiniSearch (BM25) | Finds scheme entries. Shows the stored text, generates nothing |
| Locker | Web Crypto: AES-256-GCM, PBKDF2-SHA-256 (310,000 rounds) | Encrypts records on the device |
| Places and map | Mapbox Search Box API and Mapbox GL JS | Finds facilities and specialists |
| Speech | Browser Web Speech API | Voice input, and the Listen button |

There is no backend. It is one static `index.html`.

## Run it

Needs HTTPS for voice, location and the encrypted locker (GitHub Pages provides this).

1. Fork or clone this repo.
2. Create a free [Mapbox](https://www.mapbox.com) account and copy a **public** token (starts with `pk.`).
3. Either paste it into `MAPBOX_TOKEN` near the top of the script in `index.html`, or leave it empty and the app asks on first use.
4. Restrict the token to your site's URL in the Mapbox dashboard. Never commit a secret token.
5. Serve the repo root. On GitHub: **Settings → Pages → Deploy from branch → main / root**.

Local testing: `python3 -m http.server`, then open `http://localhost:8000` (browsers treat localhost as secure).

The on-device AI is optional and off by default. Enabling it downloads about 500 MB once, then runs offline. Without it, the rules, score, scheme search, locker and 108 button all still work.

## Privacy

- AI inference, symptom text, records and contacts stay in the browser. Records are encrypted with a key derived from your passphrase. There is no account and no recovery: a forgotten passphrase means the data cannot be read.
- Two disclosed exceptions: some browsers send speech audio to their own service, and Mapbox receives your approximate location and search words to find places.
- Clearing browser data deletes the locker. Use the encrypted backup export.
- This is not a certified medical-record system.

## Data

- `data/schemes.sample.json`: the format for loading your own scheme entries. Text is **starter wording and unverified**. Replace it with official sources: [pmjay.gov.in](https://pmjay.gov.in), [esanjeevani.mohfw.gov.in](https://esanjeevani.mohfw.gov.in), [nhm.gov.in](https://nhm.gov.in), [janaushadhi.gov.in](https://janaushadhi.gov.in), and the Goa Directorate of Health Services.
- `data/facilities.template.csv`: the header for loading an official facility directory (`name,tier,lat,lon,phone`).
- Both load from the Results screen.

## Known limits

- Triage rules are **not clinically validated** and need clinician review.
- Scheme text is starter data and must be verified against official sources.
- Konkani voice uses Hindi recognition. The Marathi and Konkani keyword list is small, and the translated result headlines need native-speaker review.
- Mapbox coverage of Goa health facilities is unverified, so place results may be thin.
- Not benchmarked on low-end phones.
- Hackathon feedback: the symptom-checker flow was judged not yet usable enough for people with low digital literacy. A simpler guided flow is the main open problem.

## Roadmap

1. Verify scheme and facility data, get clinician and native-speaker review, add PWA install.
2. Pilot at selected Goa PHCs with eSanjeevani integration and a clinician summary view.
3. Guided voice-only flow that asks one question at a time, tested with real senior users.

## Repo layout

```
index.html                  the app (HTML, CSS and JS in one file)
data/                       sample formats for scheme and facility data
docs/                       pitch deck
LICENSE                     MIT
```

## Credits

[Transformers.js](https://github.com/huggingface/transformers.js), [Qwen2.5-0.5B-Instruct](https://huggingface.co/onnx-community/Qwen2.5-0.5B-Instruct) (Apache 2.0), [paraphrase-multilingual-MiniLM-L12-v2](https://huggingface.co/Xenova/paraphrase-multilingual-MiniLM-L12-v2) (Apache 2.0), [MiniSearch](https://github.com/lucaong/minisearch) (MIT), [Mapbox](https://www.mapbox.com). Place data © Mapbox and its suppliers.

## License

MIT. See `LICENSE`. The disclaimer above applies to all use.
