# spip-plugins Skill — Conception Spec

> **For agentic workers:** This is the authoritative design spec for the `spip-plugins` skill.
> Before editing any reference file or the SKILL.md, read this document in full.
> Design decisions below override intuition — don't re-litigate them.

**Goal:** Produce a Claude skill that lets a PHP developer build SPIP 4.1+ plugins correctly without consulting the incomplete/outdated spip.net documentation.

**Audience:** PHP developers who know SPIP's editorial model (rubriques, articles, auteurs) but need authoritative API reference for plugin development. Not frontend integrators (→ `spip` skill).

**Source of truth:** `/src/spip/spip` — composer-installed SPIP 4.4 core + plugins-dist. All examples, signatures, and conventions must be extracted from this codebase, not from spip.net.

---

## Skill identity

```yaml
name: spip-plugins
triggers:
  - paquet.xml present in the project
  - files matching *_pipelines.php
  - spip_* table references in PHP
  - user asks about pipelines, hooks, plugin architecture, SPIP PHP/SQL API
not-for:
  - squelettes / template work → use the `spip` skill
  - SPIP internals / compiler → use the `spip-expert` skill
```

---

## Language policy

All prose (descriptions, explanations, table headers, section titles) is in **English**.

SPIP-specific terms are **always kept in their original French/SPIP form**, never translated:

| Keep as-is | Never write |
|---|---|
| `boucle` | ~~"loop"~~ (when referring to SPIP's `<BOUCLE_>`) |
| `squelette` | ~~"template"~~ (in SPIP context) |
| `pipeline` | ~~"hook"~~ |
| `balise` | ~~"tag"~~ |
| `critère` | ~~"criterion/filter"~~ |
| `filtre` | ~~"filter function"~~ |
| `formulaire CVT` | ~~"CVT form"~~ |
| `objet éditorial` | ~~"editorial object"~~ |
| `genie` | ~~"cron task"~~ |
| `rubrique` | ~~"section"~~ |
| `paquet.xml` | (always verbatim) |

Code examples keep their original comments even when in French, **unless** the comment was written for this skill (then use English).

---

## File structure

```
spip-plugins/
├── SKILL.md                          # entry point loaded by Claude — glossary + workflow index
└── references/
    ├── paquet-xml.md                 # Bloc 1 ✅
    ├── arborescence.md               # Bloc 1 ✅
    ├── pipelines.md                  # Bloc 2 — large, needs TOC
    ├── cvt-formulaires.md            # Bloc 3
    ├── sql-api.md                    # Bloc 4
    ├── declarer-table.md             # Bloc 4
    ├── declarer-objet.md             # Bloc 4
    ├── balises-filtres-criteres.md   # Bloc 4
    ├── i18n.md                       # Bloc 4
    └── cycle-de-vie.md               # Bloc 5
```

**Size constraint:** no reference file beyond ~350 lines without a TOC at the top.

---

## Extraction methodology

Each reference file is produced by reading real source files, not by recalling general knowledge:

1. `find` the relevant files in `/src/spip/spip` (excluding `vendor/`)
2. Cross-reference multiple examples to identify invariants vs. plugin-specific choices
3. Synthesise: write the rule, then show a minimal real example with file path in a comment
4. Never invent examples — if a real pattern can't be found in the corpus, say so

Corpus hierarchy (prefer in this order):
1. SPIP core (`ecrire/`, `ecrire/public/`, `ecrire/inc/`)
2. plugins-dist with schema + administrations (forum, medias, mots, revisions, urls…)
3. plugins-dist without schema (aide, compresseur, tw…)

---

## Progress tracker

### Bloc 1 — Structure ✅

- [x] `references/paquet-xml.md` — all elements and attributes with observed values
- [x] `references/arborescence.md` — directory layout, loading order, special root files
- [x] `SKILL.md` stub — frontmatter + glossary table

### Bloc 2 — Pipelines ✅

- [x] Catalogue exhaustif with TOC
- [x] For each pipeline: signature, `$flux` shape (what enters, what must exit), example
- [x] Group by domain: édition, affichage, déclaration, formulaires, authentification, cron
- [x] Cover: pipelines déclarés vides (extension points) vs. pipelines avec implémentation core
- [x] Source: `ecrire/paquet.xml` (master list) + cross-ref each plugin's usage
- Note: 824 lines — TOC present, compliant with size rule

### Bloc 3 — CVT formulaires

- [ ] `charger` / `verifier` / `traiter` signatures and contracts
- [ ] `$flux` shape for the formulaire pipelines (`formulaire_charger`, `formulaire_verifier`, `formulaire_traiter`)
- [ ] Multi-step formulaires (`cvt_multietapes`)
- [ ] autosave pattern (`cvt_autosave`)
- [ ] File: `inc/cvt_multietapes.php`, `inc/cvt_autosave.php` in core; `formulaires/` across plugins-dist
- [ ] Real example: `formulaires/editer_forum.php` (forum plugin)

### Bloc 4 — Auxiliaires

- [ ] **`sql-api.md`** — `sql_select`, `sql_insert`, `sql_update`, `sql_delete`, `sql_alter`, `sql_drop_table`; transaction helpers; `sql_quote`; quand utiliser `objet_modifier()` plutôt que SQL direct
- [ ] **`declarer-table.md`** — `declarer_tables_principales`, `declarer_tables_auxiliaires`, `declarer_tables_interfaces`; champ structure; `$tables_principales` format
- [ ] **`declarer-objet.md`** — `declarer_tables_objets_sql`; what makes an objet éditorial; `objet_modifier`, `objet_inserer`, `objet_instituer`; the `spip_objets_permanents` mechanism
- [ ] **`balises-filtres-criteres.md`** — `balise_X_dist(&$p)` signature; `Champ` object API; custom critère signature; filtre registration in `_fonctions.php`
- [ ] **`i18n.md`** — `_T('module:cle')`, `_T_ou_typo()`, `lang_select()`; `lang/module_fr.php` format; `traduire` pipeline; `paquet-module.xml` vs `module.xml`

### Bloc 5 — Cycle de vie

- [ ] **`cycle-de-vie.md`** — `_administrations.php` in depth: `maj_plugin()`, `maj_tables()`, migration step format; `_vider_tables()`; when `schema` bumps are mandatory vs. safe to defer; `install_plugin_actif()` hook

### Rédaction finale SKILL.md

- [ ] Workflow section: when to load which reference
- [ ] Quick-start: minimal plugin skeleton (paquet.xml + 3 files)
- [ ] Decision tree: "I want to do X → read Y"
- [ ] Keep under 300 lines total

---

## Quality bar per reference file

A reference file is **done** when:

1. Every function/pipeline/element has: exact signature, what it receives, what it must return
2. At least one real example with source path in a comment
3. All invariants (always true) separated from conventions (common practice)
4. Edge cases documented (e.g. `schema` without `_administrations.php` → nothing called)
5. Cross-references to other reference files where relevant

---

## Decisions log

| Date | Decision | Rationale |
|---|---|---|
| 2026-04-29 | English prose, French SPIP terms | Skill triggered by English queries; SPIP terms are proper nouns with no adequate translation |
| 2026-04-29 | Extract from source, not spip.net | spip.net doc is incomplete and sometimes wrong for 4.x |
| 2026-04-29 | SKILL.md glossary as first section | Readers need the term mapping before any reference makes sense |
| 2026-04-29 | `pipelines.md` gets its own TOC | Catalogue will exceed 350 lines; TOC mandatory per size constraint |
| 2026-04-29 | `<pipeline action="">` documented as extension point | Non-obvious: many devs assume empty action = bug |
