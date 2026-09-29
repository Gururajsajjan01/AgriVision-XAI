# AgriVision-XAI

An explainable agriculture decision-support platform with crop recommendation, plant image analysis, crop product classification, crop references, food nutrition lookup, weather context, farmer feedback, and a research results dashboard.

## Truthful status

The code and integrations are provided, but no datasets are bundled and no models have been trained. Prediction endpoints return a clear model-unavailable response until you obtain datasets and run the training scripts. Crop reference records start empty because unverified agronomic values are unsafe. Evaluation numbers are only displayed when actual training scripts produce metric manifests.

## Stack and architecture

- Backend: Python 3.11+, FastAPI, Pydantic, SQLAlchemy, SQLite (PostgreSQL URL supported)
- Tabular ML: pandas, scikit-learn Random Forest, joblib, measured holdout metrics
- Image DL: optional TensorFlow/Keras transfer learning (MobileNetV2, EfficientNetB0, ResNet50), Pillow, Grad-CAM
- Explainability: feature importance by default; optional SHAP (`requirements-xai.txt`)
- Frontend: React + TypeScript + Vite, responsive English/Kannada interface
- External data: Open-Meteo forecast; USDA FoodData Central (API key required)

## Install and run on Windows

Install Python 3.11+ and Node.js 20+ first.

```powershell
# From the project root
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r backend\requirements.txt
Copy-Item backend\.env.example backend\.env

# Backend (first terminal)
Set-Location backend
..\.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

In a second terminal:

```powershell
Set-Location frontend
npm install
npm run dev
```

Visit `http://localhost:5173`, API docs `http://localhost:8000/docs`, health `http://localhost:8000/api/health`.

macOS/Linux: make `.venv`, install `backend/requirements.txt`, copy `backend/.env.example` to `backend/.env`, start from `backend/` with `../.venv/bin/python -m uvicorn app.main:app --reload`, then run `npm install && npm run dev` from `frontend/`.

## Environment

`backend/.env.example` documents `DATABASE_URL`, `FRONTEND_ORIGINS`, image upload cap, model path, USDA FDC API key, provider timeout, and the admin key. To enable the Admin panel, set a long random `ADMIN_API_KEY` in `backend/.env` and restart the API; the panel asks for this key and holds it in page memory only. Admin endpoints are disabled when the key is unset. Use HTTPS and a private deployment secret store outside local development. Add a USDA API key privately to `backend/.env` to enable nutrition lookups. Never commit `.env`. Weather uses Open-Meteo and requires network access. For PostgreSQL configure `DATABASE_URL` and install a matching SQLAlchemy driver.

## Database and migrations

The API creates missing SQLite tables on startup. To manage schema through Alembic from `backend/`, activate the project environment and run `python -m alembic upgrade head`. Crop knowledge can be imported from a CSV with required source name, URL, citation, and license columns using `python scripts/import_crop_catalog.py path/to/crops.csv`.

## Train the models

Read [dataset guidance](data/README.md) and document source/license, counts, splits, and limitations before training.

- Soil crop recommendation: `cd backend`; `python scripts/train_crop_model.py ../data/raw/crop_recommendation.csv`
- Disease images: install optional stack `python -m pip install -r requirements-vision.txt`; arrange class folders and run `python scripts/train_disease_model.py ../data/raw/plant_disease`
- Crop product images: `python scripts/train_crop_classifier.py ../data/raw/crop_images`
- Print saved evaluation manifests: `python scripts/evaluate_models.py`

The image trainer reports image-level stratified splits. Near-duplicate or same-plant images may leak across splits, so grouped/site-held-out evaluation is needed before field claims. ImageNet pretrained weights may need downloading on first image-model training. The app does not claim model reliability from a benchmark alone.

## API

- `GET /api/health`
- `GET /api/crops`, `GET /api/crops/{crop_name}`
- `POST /api/crop/recommend`
- `POST /api/disease/predict` (multipart `file`)
- `POST /api/crop-classification/predict` (multipart `file`)
- `GET /api/nutrition/{food_name}`
- `GET /api/weather?latitude=...&longitude=...`
- `POST /api/feedback`
- `GET /api/models/status`

The Admin panel is linked in the site navigation. It can add/delete source records, add/update/delete crop guide records tied to saved sources, and view model evaluation status. Admin endpoints under `/api/admin/*` require the `X-Admin-Key` header. The panel does not start long-running training jobs; train models with the documented backend scripts.

Interactive OpenAPI docs are at `/docs`.

## Privacy and limitations

Images are validated and processed in memory; uploads are not persisted. Soil inputs are not stored, but model outputs (task, predicted label, model score, timestamp) are recorded without an account ID or source image. Feedback stores module, yes/no response, free text, and timestamp without an account ID; comments can still identify a person, so don't enter personal details. Weather coordinates go to Open-Meteo. Food search terms go to USDA only when configured, and successful nutrient lookups are cached locally with their FDC source ID and URL. Set a retention policy before public deployment. Add authentication/rate limits for write APIs and review privacy policy, security headers, backups, and provider terms.

Disease messages do not recommend a specific pesticide or chemical. Outputs are informational and not a substitute for an agricultural professional. Nutrition data is educational, not medical advice. Kannada labels are translated; scientific values remain in their sourced units and wording.

## Research files

See `research/methodology.md`, `research/dataset_documentation.md`, and `research/experiments.md`. No model result exists until an actual run is documented. See `docs/deployment.md` for deployment setup.

Source-backed disease facts can be imported with python scripts/import_disease_reference.py path/to/disease-reference.csv from ackend/. Templates are in data/templates/.
