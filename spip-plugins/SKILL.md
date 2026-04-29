---
name: spip-plugins
description: Use when developing a SPIP plugin (paquet.xml present, _pipelines.php,
  spip_* table references in PHP) or when the user asks about plugin architecture,
  pipelines, hooks, or the SPIP PHP/SQL API. SPIP 4.1+. Not for template/squelettes
  work — use the spip skill for that.
---

# SPIP plugin development

## SPIP-specific terms (always kept in original form)

These terms appear untranslated throughout all reference files.

| Term | Meaning |
|---|---|
| **boucle** | `<BOUCLE_xxx(TABLE){critères}>...</BOUCLE_xxx>` — template loop |
| **squelette** | Template file (`.html` with SPIP syntax) |
| **rubrique** | Hierarchical content section (SPIP's primary tree node) |
| **paquet.xml** | Plugin manifest — declares metadata, dependencies, pipelines, menus |
| **pipeline** | Extension hook — a named chain of functions called in sequence |
| **balise** | `#TAG` — template marker compiled to PHP |
| **critère** | `{filter}` inside a boucle — constrains the SQL query |
| **filtre** | `|function` — post-processor applied to a value in a squelette |
| **formulaire CVT** | Charger/Vérifier/Traiter — SPIP's form pattern (load/validate/process) |
| **objet éditorial** | First-class content type with CRUD, i18n, revisions (article, rubrique, auteur…) |
| **genie** | SPIP scheduled task (cron), declared via `<genie>` in paquet.xml |
| **action** | Secured script entry point (`?action=nom&arg=...&hash=...`) |
| **exec** | Private-space page handler (`?exec=nom`) |
| **inc** | Helper function file, lazy-loaded via `charger_fonction('nom', 'inc')` |
| **prefix** | Short plugin identifier — prefixes all PHP filenames and function names |
