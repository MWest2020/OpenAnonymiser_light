## REMOVED Requirements

### Requirement: NLP-engine is SpaCy, zonder Transformers/torch

Vervangen door "NLP-engine is spaCy + GLiNER": OpenAnonymiser is nu GLiNER-only,
dus torch is een base-dependency (besluit Mark 2026-08-11, supersedet
`strip-to-text-only`).

### Requirement: Pattern recognizers actief

De pattern-recognizers staan in de GLiNER-only-config standaard uit (GLiNER dekt
de PII); de "8 recognizers actief"-eis vervalt.

## ADDED Requirements

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
