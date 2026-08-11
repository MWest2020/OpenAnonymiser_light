## ADDED Requirements

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

### Requirement: NLP-engine is SpaCy, zonder Transformers/torch
De API SHALL SpaCy (`nl_core_news_lg` in dev, `nl_core_news_md` in productie) gebruiken voor NER en SHALL GEEN Transformers/torch-dependency in de base-installatie hebben.

#### Scenario: SpaCy-engine beschikbaar bij startup
- **WHEN** de API start met de default plugin-config
- **THEN** laadt de SpaCy-engine zonder errors en logt de modelnaam
- **AND** wordt er geen GLiNER/transformer/torch geïmporteerd

### Requirement: Pattern recognizers actief
De API SHALL de NL-specifieke pattern recognizers `PHONE_NUMBER`, `IBAN`, `BSN`, `DATE_TIME`, `EMAIL`, `ID_NO`, `DRIVERS_LICENSE` en `CASE_NO` standaard actief hebben.

#### Scenario: Regex-PII wordt gedetecteerd
- **WHEN** een client tekst met een telefoonnummer, IBAN en BSN naar `/api/v1/analyze` POST
- **THEN** bevat het antwoord `PHONE_NUMBER`-, `IBAN`- en `BSN`-entiteiten uit de pattern recognizers

### Requirement: Stateless deployment
De API SHALL geen persistente opslag vereisen (geen SQLite, geen file-upload, geen crypto-keys voor PDF).

#### Scenario: Pod restart
- **WHEN** een Kubernetes-pod herstart
- **THEN** is de API direct beschikbaar zonder data-migratie of volume-mounts
