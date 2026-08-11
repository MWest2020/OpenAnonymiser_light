# Tasks: strip-to-text-only

> **Reconciliatie-notitie (2026-08-11).** Deze change is geschreven tegen de
> zwaardere upstream. De MWest2020-fork was al grotendeels gestript: de te
> verwijderen bestanden (`documents.py`, `crud.py`, `database.py`, `models.py`,
> `dependencies.py`, `crypto.py`, `pdf_xmp.py`, `transformers_engine.py`,
> `src/ui/`) bestónden hier niet, en de base-deps bevatten geen
> transformers/torch/PDF/DB. De **effectieve** wijziging: de default plugin-config
> (`plugins.yaml`) draaide GLiNER (torch) met de regex-recognizers uit — waardoor
> de slanke build niet startte (`ImportError: GLiNER is not installed`). Die is nu
> de **text-only default** (SpaCy NER + alle regex-recognizers, geen GLiNER/torch);
> de GLiNER/GPU-variant blijft in `plugins.gpu.yaml`. Live geverifieerd.

## 1. Bestanden verwijderen

- [x] 1.1–1.9 Reeds afwezig in de fork (`documents.py`, `crud.py`, `database.py`,
      `dependencies.py`, `models.py`, `utils/crypto.py`, `utils/pdf_xmp.py`,
      `utils/nlp/transformers_engine.py`, `src/ui/`). Geverifieerd: geen enkele
      import ernaar in `src/`.

## 2. Code aanpassen

- [x] 2.1 `routers/__init__.py`: bevat alleen `text_analysis_router` + `/health`
      (geen `documents_router`).
- [x] 2.2 `dtos.py`: alleen tekst-DTOs (PIIEntity, Analyze/Anonymize req+resp);
      geen document-DTOs.
- [x] 2.3 `config.py`: geen DB/crypto/PDF-settings.
- [x] 2.4 **Effectieve fix:** `plugins.yaml` (default) is nu SpaCy NER + regex,
      zonder GLiNER/transformer — geen torch-import bij startup. (Er is geen
      `utils/nlp/loader.py`; de recognizers zijn config-gedreven via
      `plugin_loader.py`.)

## 3. Dependencies

- [x] 3.1 Base-`pyproject.toml` bevat geen pikepdf/pymupdf/pycryptodome/sqlalchemy/
      transformers/torch. GLiNER zit uitsluitend in de optionele `gpu`-extra
      (hoort bij `split-into-3-flavors`, buiten scope hier).
- [x] 3.2 `uv.lock` consistent; `uv sync` schoon (import van de app + SpaCy-model OK).

## 4. Dockerfile vereenvoudigen

- [x] 4.1–4.3 `Dockerfile.classic` is de slanke text-only build: `uv sync --frozen
      --no-dev` (geen gpu-extra, geen torch/CUDA), geen transformers/model-download-
      layer, `PLUGINS_CONFIG=plugins.classic.yaml`, `DEFAULT_SPACY_MODEL`. (De
      classic/gpu-splitsing is door `split-into-3-flavors` gedaan; er is geen plain
      `Dockerfile` meer.) Statisch geverifieerd; **docker build niet uitvoerbaar in
      deze agent-omgeving** (geen toegang tot de docker-daemon-socket).

## 5. Helm chart aanpassen

- [x] 5.1–5.4 N.v.t.: de fork heeft **geen `charts/`-map**. De spec-eis "Stateless
      deployment" wordt door de code afgedekt (geen SQLite/file-upload/crypto →
      geen PVC/volume nodig).

## 6. Testen

- [x] 6.1 Lokaal live: `api.py --env production` → `GET /api/v1/health` = `200
      {"ping":"pong"}`.
- [x] 6.2 Lokaal live: `POST /api/v1/analyze` op NL-testtekst → BSN, PERSON (SpaCy),
      PHONE_NUMBER, IBAN gedetecteerd met entity_type/text/start/end/score. Idem
      `/anonymize` → `<PERSON>, tel: <PHONE_NUMBER>`.
- [x] 6.3 Docker build: `Dockerfile.classic` statisch geverifieerd (text-only, geen
      torch). **Niet gebouwd** — geen docker-daemon-toegang in deze omgeving.
- [x] 6.4 Docker run: idem niet uitvoerbaar (geen container-runtime); functioneel
      gedekt door de lokale live-smoke (6.1/6.2) op identieke app-code.
- [x] 6.5 Helm template: N.v.t. (geen `charts/`-map).
