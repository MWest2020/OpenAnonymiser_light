# Tasks: gliner-only

## 1. Config

- [x] 1.1 `src/api/plugins.yaml` = de GLiNER-config (spaCy NER + GLiNER-recognizer
      + MAC; overige patterns uit). Enige config.
- [x] 1.2 `src/api/plugins.classic.yaml` en `plugins.gpu.yaml` verwijderd.

## 2. Dependencies

- [x] 2.1 `pyproject.toml`: `gliner>=0.1.13` naar base-dependencies; `gpu`-extra
      verwijderd.
- [x] 2.2 `uv lock` hergeresolved (torch + gliner nu in de base-resolutie).

## 3. Docker

- [x] 3.1 Eén `Dockerfile` (GLiNER + spaCy, `uv sync` zonder extra, gebakken
      HF-model/tokenizer, `HF_HUB_OFFLINE=1`, default `plugins.yaml`).
- [x] 3.2 `Dockerfile.classic` en `Dockerfile.gpu` verwijderd; `docker-compose.yaml`
      wees al naar `Dockerfile`.

## 4. Docs & spec

- [x] 4.1 README-flavorsectie → GLiNER-only; pyproject-comment bijgewerkt.
- [x] 4.2 `docs/architecture/flavors.md` gemarkeerd als superseded.
- [x] 4.3 Spec-delta `text-only-api`: NLP-engine-/pattern-requirements REMOVED,
      GLiNER-engine-requirement ADDED.

## 5. Verify

- [x] 5.1 Deps + startup gedeeltelijk geverifieerd: `uv sync` installeerde torch
      2.11.0 + gliner 0.2.26 (import ok); de app startte en laadde de GLiNER-config
      correct (haalde `urchade/gliner_multi_pii-v1` van HuggingFace — geen
      config/importfout). **De volledige runtime-smoke KAN NIET op de agent-host**:
      die heeft 4 GiB RAM / 2 vCPU / geen GPU; torch + spaCy-lg + het GLiNER-model
      lopen daar in OOM-thrash → **host-freeze** (bevestigd, reboot nodig). Runtime
      hoort op een echte box (zie proposal/README: GPU-pool voor productie; op CPU
      traag). Mark heeft de runtime elders getest ("getest wat werkt").
- [x] 5.2 Statisch: `plugins.yaml` geldige YAML, `pyproject.toml` geldig,
      `uv lock` resolved (142 packages, torch+gliner present), config = de door
      Mark geteste GLiNER-config.
