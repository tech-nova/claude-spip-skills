---
name: spip-plugins
description: Use when developing a SPIP plugin (paquet.xml present, _pipelines.php,
  spip_* table references in PHP) or when the user asks about plugin architecture,
  pipelines, hooks, or the SPIP PHP/SQL API. SPIP 4.1+. Not for template/squelettes
  work — use the spip skill for that.
---

# SPIP plugin development

## SPIP-specific terms (always kept in original form)

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
| **objet éditorial** | First-class content type with CRUD, i18n, revisions (article, rubrique…) |
| **genie** | SPIP scheduled task (cron), declared via `<genie>` in paquet.xml |
| **action** | Secured script entry point (`?action=nom&arg=...&hash=...`) |
| **exec** | Private-space page handler (`?exec=nom`) |
| **inc** | Helper function file, lazy-loaded via `charger_fonction('nom', 'inc')` |
| **prefix** | Short plugin identifier — prefixes all PHP filenames and function names |

---

## Quick-start: minimal plugin skeleton

A SPIP plugin needs at minimum these files:

```
myplugin/
├── paquet.xml
├── myplugin_pipelines.php   ← pipeline handlers
└── lang/
    └── myplugin_fr.php      ← reference language strings
```

**paquet.xml** (no schema, no DB):
```xml
<plugin
  nom="Mon Plugin"
  prefix="myplugin"
  version="1.0.0"
  compatibilite="[4.1;5.0["
  etat="dev"
  categorie="divers">

  <auteur>Prénom Nom</auteur>
  <licence>GNU/GPL</licence>

  <pipeline nom="mon_pipeline" inclure="myplugin_pipelines.php" />

  <traduire module="myplugin" reference="fr" />
</plugin>
```

**paquet.xml** (with a database table):
```xml
<plugin ... schema="1.0.0" ...>
  ...
  <install inclure="myplugin_administrations.php" />

  <!-- declare pipeline handlers for table declarations -->
  <pipeline nom="declarer_tables_objets_sql" inclure="base/myplugin.php" />
  <pipeline nom="declarer_tables_interfaces"  inclure="base/myplugin.php" />
</plugin>
```

**myplugin_pipelines.php**:
```php
<?php
if (!defined('_ECRIRE_INC_VERSION')) { return; }

function myplugin_mon_pipeline($flux) {
    // modify $flux['data'], return $flux
    return $flux;
}
```

---

## Decision tree — I want to…

| Goal | Read |
|---|---|
| Create a plugin manifest (`paquet.xml`) | `references/paquet-xml.md` |
| Understand the directory layout and file loading order | `references/arborescence.md` |
| Hook into SPIP events (content published, user logged in, etc.) | `references/pipelines.md` |
| Add a contact/subscription/edit form | `references/cvt-formulaires.md` |
| Read or write database rows | `references/sql-api.md` |
| Create a new database table (not an objet éditorial) | `references/declarer-table.md` |
| Create a new objet éditorial (with statut, edition form, search) | `references/declarer-objet.md` |
| Define a custom `#BALISE`, `|filtre`, or `{critère}` | `references/balises-filtres-criteres.md` |
| Add translated strings to a plugin | `references/i18n.md` |
| Write install/upgrade/uninstall logic (`_administrations.php`) | `references/cycle-de-vie.md` |

---

## Workflow index

### Starting a new plugin

1. `references/paquet-xml.md` — write the manifest
2. `references/arborescence.md` — choose the right file for each function
3. `references/pipelines.md` — pick which pipeline(s) to plug into

### Adding a database table

1. `references/declarer-table.md` — declare schema via `declarer_tables_principales` or
   `declarer_tables_objets_sql`
2. `references/cycle-de-vie.md` — write `_administrations.php` with `maj_tables()`
3. `references/sql-api.md` — read/write the table at runtime

### Building a form

1. `references/cvt-formulaires.md` — implement `charger`, `verifier`, `traiter`
2. `references/sql-api.md` — write data in `traiter()`
3. `references/pipelines.md` → `formulaire_charger` / `formulaire_verifier` / `formulaire_traiter`
   if you need to hook into other plugins' forms

### Creating an objet éditorial

1. `references/declarer-objet.md` — full `declarer_tables_objets_sql` descriptor
2. `references/declarer-table.md` — add `declarer_tables_interfaces` for compiler config
3. `references/cycle-de-vie.md` — install/uninstall the table
4. `references/cvt-formulaires.md` — use `formulaires_editer_objet_charger/verifier/traiter()`
5. `references/pipelines.md` → `pre_edition`, `post_edition` to react to changes

### Extending squelettes

1. `references/balises-filtres-criteres.md` — add `#BALISE`, `|filtre`, or `{critère}`
2. `references/declarer-table.md` → `declarer_tables_interfaces` for `table_des_traitements`

---

## Source of truth

All examples in this skill are extracted from:
- SPIP core: `/src/spip/spip/ecrire/`
- plugins-dist: `/src/spip/spip/plugins-dist/spip/`

When in doubt, read the source. The spip.net documentation is incomplete for SPIP 4.x.
