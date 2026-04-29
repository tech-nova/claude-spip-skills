# SPIP Skill Design

## Goal

Create a Claude Code skill (`spip`) that serves as a comprehensive fast-reference for experienced SPIP developers. **Target: SPIP 4.1+.** The skill activates automatically when Claude detects a SPIP project and is also manually invokable via `/spip`.

## File Structure

```
~/.claude/skills/spip/
  SKILL.md                  # Main index: overview, quick reference, core examples
  references/
    squelettes.md           # Full BOUCLE/balise/critère/filtre reference
    plugins.md              # Plugin anatomy: paquet.xml, file conventions, pipelines
    api.md                  # SQL API, utility functions, PHP patterns
```

## SKILL.md

### Frontmatter

```yaml
---
name: spip
description: Use when working in a SPIP project (paquet.xml present, squelette files
  with BOUCLE syntax, spip_* table references) or when the user asks about SPIP
  templates, plugins, pipelines, or PHP development with SPIP.
---
```

### Body Sections

1. **Overview** — 3–4 sentences covering: SPIP is a PHP CMS/framework with a squelette template engine, paquet.xml-based plugin system, and SQL abstraction layer. Key directories: `squelettes/`, `plugins/`, `plugins-dist/`, `tmp/cache/`.

2. **Template Quick Reference** — Inline BOUCLE syntax with 2 real examples (articles loop, nested rubrique+articles), common balises (`#TITRE`, `#TEXTE`, `#URL_ARTICLE`, etc.), filter syntax (`|filtre{arg}`), criteria syntax (`{id_rubrique}`, `{par titre}`, `{limit 10}`), and conditional tag syntax.

3. **Plugin Quick Reference** — Inline `paquet.xml` skeleton, list of conventional plugin files with one-line descriptions (`options.php`, `fonctions.php`, `_pipelines.php`, `lang/`, `formulaires/`, `squelettes/`).

4. **Pipeline Quick Reference** — How to declare in `paquet.xml`, how to write a handler function, 2–3 most common hooks (`pre_edition`, `affichage_final`, `formulaire_traiter`).

5. **Reference Pointers** — Explicit "For full X reference, read `references/X.md`" lines for each reference file.

## references/squelettes.md

- All BOUCLE table types (ARTICLES, RUBRIQUES, DOCUMENTS, AUTEURS, FORUMS, MOTS, etc.)
- Complete critères catalog: filtering (`{id_rubrique}`, `{statut}`), sorting (`{par champ}`, `{inverse}`), pagination (`{limit n}`, `{debut_articles}`), joins
- Built-in balises with descriptions and common filters
- Conditional tag syntax: `[(#BALISE|filtre)]`, `[texte(#BALISE)texte]`, optional content blocks
- Nested and recursive loop patterns

## references/plugins.md

- Full `paquet.xml` schema: all attributes and child elements with examples
- Complete plugin file convention map (what each optional file does, when to create it)
- Full pipeline catalog grouped by category: content display, forms, admin interface, DB events (pre/post insertion/edition)
- How to create a custom pipeline
- `_administrations.php` for DB schema upgrades (versionning)

## references/api.md

- SQL API functions: `sql_select`, `sql_fetch`, `sql_allfetsel`, `sql_insert`, `sql_updateq`, `sql_delete` — signatures + examples
- Common utility functions: `_request()`, `include_spip()`, `find_in_path()`, `pipeline()`
- Cache management: `spip_clear_cache()`, cache invalidation patterns
- Security helpers: `_request()` vs raw superglobals, `securiser_acces_low_sec()` (SPIP 4.1+, defined in `inc/acces`; replaces deprecated `securiser_acces()`)

## Trigger Conditions

Auto-triggers when Claude sees any of:
- `paquet.xml` at project root
- `.html` files containing `<BOUCLE_` syntax
- PHP files referencing `spip_articles`, `spip_rubriques`, or other `spip_*` tables
- User invokes `/spip` explicitly

## Success Criteria

- An experienced SPIP developer can ask about any SPIP concept (template syntax, plugin structure, pipeline hooks, SQL API) and get a correct, actionable answer without Claude hallucinating non-SPIP patterns
- The main SKILL.md stays under 500 words for fast auto-load
- Reference files provide deep lookup without burdening every conversation
