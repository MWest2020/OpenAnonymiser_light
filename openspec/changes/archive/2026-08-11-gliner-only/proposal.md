# gliner-only

## Why

Besluit Mark (2026-08-11): OpenAnonymiser draait **één** detectie-pad — GLiNER —
in plaats van de drie flavors (classic/gpu/contextual) die `split-into-3-flavors`
voorstelde. "Getest wat werkt": GLiNER (zero-shot transformer-NER) is het pad dat
de gewenste PII-dekking geeft; de classic (SpaCy+regex-only) text-only-default uit
`strip-to-text-only` en de contextual/LLM-flavor vervallen. Dit vervangt zowel de
3-flavor-opzet (`split-into-3-flavors`, nooit gebouwd DRAFT) als de "geen
transformers/torch"-eis uit `strip-to-text-only`.

## What Changes

- **`src/api/plugins.yaml`** wordt de enige config: spaCy als NER-engine + GLiNER
  als recognizer (`urchade/gliner_multi_pii-v1`) + MAC-recognizer; de overige
  pattern-recognizers blijven uit (GLiNER dekt de PII). `plugins.classic.yaml` en
  `plugins.gpu.yaml` verwijderd.
- **`pyproject.toml`**: `gliner` verhuist van de optionele `gpu`-extra naar de
  base-dependencies (trekt `torch` mee); de `gpu`-extra vervalt. `uv.lock`
  hergeresolved.
- **Docker**: één `Dockerfile` (GLiNER + spaCy + gebakken HF-model/tokenizer,
  offline runtime). `Dockerfile.classic`/`Dockerfile.gpu` verwijderd;
  `docker-compose.yaml` wees al naar `Dockerfile`.
- **Docs**: README-flavorsectie → GLiNER-only; `docs/architecture/flavors.md`
  gemarkeerd als superseded.

## Impact

- Vervangt `split-into-3-flavors` (verwijderd) en superseded de NLP-engine-/
  pattern-requirements uit de `text-only-api`-spec (`strip-to-text-only`).
- De REST-API (`/health`, `/analyze`, `/anonymize`) en het stateless-karakter
  wijzigen niet; alleen de detectie-engine en de dependency-footprint.
- Base-image is nu ~3GB (torch + GLiNER-model) i.p.v. de slanke text-only-build.
