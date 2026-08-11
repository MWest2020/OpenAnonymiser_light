# text-only-api Specification

## Purpose
TBD - created by archiving change strip-to-text-only. Update Purpose after archive.
## Requirements
### Requirement: Health endpoint
De API SHALL een `GET /api/v1/health` endpoint bieden dat `{"ping": "pong"}` retourneert zonder authenticatie.

#### Scenario: Health check
- **WHEN** een client `GET /api/v1/health` aanroept
- **THEN** retourneert de API `200 OK` met body `{"ping": "pong"}`

### Requirement: Tekst analyseren
De API SHALL een `POST /api/v1/analyze` endpoint bieden dat gedetecteerde PII-entiteiten in de tekst teruggeeft.

#### Scenario: Analyze met default engine
- **WHEN** een client `{"text": "Jan Janssen woont in Amsterdam"}` POST naar `/api/v1/analyze`
- **THEN** retourneert de API een lijst van gedetecteerde PII-entiteiten met `entity_type`, `text`, `start`, `end` en `score`

#### Scenario: Analyze met expliciete entities-filter
- **WHEN** een client `{"text": "...", "entities": ["PERSON", "PHONE_NUMBER"]}` POST
- **THEN** retourneert de API alleen entiteiten van de gevraagde types

### Requirement: Tekst anonimiseren
De API SHALL een `POST /api/v1/anonymize` endpoint bieden dat de gevonden PII vervangt door placeholders.

#### Scenario: Anonymize tekst
- **WHEN** een client `{"text": "Jan Janssen, tel: 0612345678"}` POST naar `/api/v1/anonymize`
- **THEN** retourneert de API `original_text`, `anonymized_text` (met `<PERSON>`- en `<PHONE_NUMBER>`-placeholders) en `entities_found`

### Requirement: Stateless deployment
De API SHALL geen persistente opslag vereisen (geen SQLite, geen file-upload, geen crypto-keys voor PDF).

#### Scenario: Pod restart
- **WHEN** een Kubernetes-pod herstart
- **THEN** is de API direct beschikbaar zonder data-migratie of volume-mounts

### Requirement: NLP-engine is spaCy + GLiNER

De API SHALL spaCy (`nl_core_news_lg` in dev, `nl_core_news_md` in productie) als
NER-engine gebruiken én GLiNER (`urchade/gliner_multi_pii-v1`, zero-shot
transformer) als recognizer voor contextuele PII. GLiNER (en daarmee `torch`)
SHALL een base-dependency zijn; er is één detectie-config (`plugins.yaml`), geen
classic/gpu/contextual-flavors.

#### Scenario: GLiNER-engine beschikbaar bij startup
- **WHEN** de API start met de default plugin-config
- **THEN** laadt de spaCy-engine én de GLiNER-recognizer zonder errors
- **AND** logt het GLiNER-model (`urchade/gliner_multi_pii-v1`)

#### Scenario: Contextuele PII wordt gedetecteerd
- **WHEN** een client `{"text": "Jan Janssen woont in Amsterdam"}` POST naar `/api/v1/analyze`
- **THEN** bevat het antwoord een `PERSON`- en een `LOCATION`-entiteit uit GLiNER

