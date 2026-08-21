# ADR-0003: Search backend and retrieval policy

- Status: Accepted
- Date: 2026-08-21

## Context

The launch needs bilingual site search and a structured Asset catalogue without
adding infrastructure before corpus and relevance requirements are measurable.

## Decision

Use Search API with its Database backend for v1. Index published,
access-allowed content with structured fields and curated rendered text. The
default Search View returns the current language without translation duplicates.
Asset catalogue filtering remains a Views concern. Future machine retrieval may
consume an explicit Search API View, while JSON:API remains closed in v1.

## Alternatives considered

- Deploy Solr at launch.
- Use Drupal core search directly.
- Treat the Asset catalogue as global full-text search.
- Expose unrestricted JSON:API as the retrieval layer.

## Consequences

Launch is operationally simple and portable to qualified shared hosting. RU
morphology, protected technical tokens, facets and autocomplete are not promised
until tested against a golden query set.

## Deferred triggers

Consider Solr when required morphology, vocabulary, facets,
autocomplete/spellcheck, relevance, p95 latency or DB load fail acceptance.

