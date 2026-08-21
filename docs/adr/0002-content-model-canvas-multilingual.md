# ADR-0002: Content model, Canvas boundary and multilingual presentation

- Status: Accepted
- Date: 2026-08-21

## Context

E+E Master needs unique corporate pages and repeatable, searchable domain
content in RU and EN. Mixing business facts into free-form Canvas composition
would weaken Views, Search API, SEO, translation and future API use.

## Decision

Technical Article, Service and Asset are separate Content Types. Asset Status
is a fixed Options enum. Unique corporate pages use Canvas Page; repeatable
entities use fields plus one Canvas Content Template per bundle. Patterns and
global regions are governed centrally; per-node override is exceptional.

RU is default without prefix; EN uses `/en`. Translations are optional. Text
and SEO inputs translate; facts and references are shared. Canvas layout is
symmetric between translations. When a translation is missing, the switcher
opens the selected-language homepage. Translation freshness is manual in v1.

## Alternatives considered

- Model services only as Canvas Pages.
- Store Asset Status in Taxonomy.
- Permit unrelated RU/EN Canvas trees.
- Require every material to have both translations.

## Consequences

Structured content remains queryable and reusable while Canvas owns
presentation. Editors cannot use taxonomy to invent statuses. Language variants
share layout and facts, reducing drift, but genuinely different campaigns need
separate pages.

## Deferred triggers

Review custom components/theme only after a documented platform gap. Review
translation-freshness automation only when manual control becomes unreliable.

