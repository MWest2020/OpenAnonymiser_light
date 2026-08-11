# OpenAnonymiser Light

Slanke API voor detectie en anonimisering van privacygevoelige informatie (PII) in Nederlandse tekst. Gebaseerd op [Microsoft Presidio](https://github.com/microsoft/presidio) met SpaCy NER (`nl_core_news_lg`) en Nederlandse pattern recognizers.

**Productie:** https://api.openanonymiser.commonground.nu/api/v1/docs
**Staging:** https://api.openanonymiser.accept.commonground.nu/api/v1/docs

## Quickstart

```bash
uv venv && uv sync                 # incl. GLiNER (+ torch)
uv run api.py
```

Swagger UI: [http://localhost:8080/api/v1/docs](http://localhost:8080/api/v1/docs)

## Detectie-engine (GLiNER-only)

OpenAnonymiser draait **één** detectie-pad: spaCy als NER-engine + **GLiNER**
(zero-shot transformer) voor contextuele PII, plus presidio-recognizers. Er zijn
geen classic/gpu/contextual-flavors meer — GLiNER staat in de base-dependencies
(trekt `torch` mee) en `plugins.yaml` is de enige config.

```bash
# één image (GLiNER + spaCy + mdeberta-tokenizer gebakken, ~3GB)
docker build -t openanonymiser-light:dev .
```

GPU geeft de beste productie-latency; op CPU werkt het maar traag.

## Endpoints

| Endpoint | Beschrijving |
|----------|-------------|
| `GET /api/v1/health` | Liveness check |
| `POST /api/v1/analyze` | Detecteer PII — geeft entiteiten + posities terug |
| `POST /api/v1/anonymize` | Anonimiseer tekst — vervangt PII door placeholders |

## Pre-push gate (optioneel)

Snelle checks vóór `git push` — uv-lock-sync + bandit HIGH severity. Geen API of containers nodig, draait in <1 s.

```bash
# Eenmalig installeren als git pre-push hook
ln -sf ../../scripts/pre-push.sh .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

Bypass tijdelijk met `git push --no-verify`. Volledige test-suite + container builds blijven CI's verantwoordelijkheid.

## Documentatie

- [01 Getting Started](docs/01-getting-started.md) — installatie, eerste verzoek, entiteittypes
- [02 API Reference](docs/02-api-reference.md) — alle endpoints met curl-voorbeelden
- [03 Configuration](docs/03-configuration.md) — env vars, modellen, pattern recognizers
- [04 Deployment](docs/04-deployment.md) — container, Kubernetes/Helm, CI/CD
- [Contributing](CONTRIBUTING.md) — branching, code standards, tooling

## Stack

| Component | Technologie |
|-----------|------------|
| Framework | FastAPI + Presidio |
| NER | SpaCy `nl_core_news_lg` (overal — lokaal, container, K8s) |
| Patronen | Custom Dutch regex recognizers |
| Package manager | uv |
| Container | Docker |
| Deployment | Helm + ArgoCD |
