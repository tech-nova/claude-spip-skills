# BOUCLE Types Reference

Source: programmer.spip.net, www.spip.net (fetched 2026-04-30). SPIP 4.1+.

---

## Anatomy of a BOUCLE

```html
<B_name>                         <!-- pre-section: shown before results, only if ≥1 result -->
<BOUCLE_name(TABLE){critère1}{critère2}>
  ... template for each row ...
</BOUCLE_name>
<BB_name>                        <!-- post-section: shown after results, only if ≥1 result -->
</BB_name>
<//B_name>                       <!-- zero-result alternate: shown only when loop returns 0 rows -->
```

- **`<B_name>` / `</B_name>`** — pre-section. Wraps HTML that should only appear when the loop has at least one result (e.g. an opening `<ul>`).
- **`<BB_name>` / `</BB_name>`** — post-section. Appears after loop output, only when ≥1 result (e.g. a closing `</ul>` and pagination link).
- **`<//B_name>`** — zero-result alternative. Shown when the loop finds nothing (e.g. "No articles yet.").

### Annotated example

```html
<B_articles>
<ul class="article-list">
</B_articles>

<BOUCLE_articles(ARTICLES){id_rubrique}{par date}{inverse}{pagination 10}>
  <li><a href="#URL_ARTICLE">#TITRE</a> — [(#DATE|affdate)]</li>
</BOUCLE_articles>

<BB_articles>
</ul>
[(#PAGINATION)]
</BB_articles>

<//B_articles>
  <p>Aucun article dans cette rubrique.</p>
</BOUCLE_articles>
```

> **Gotcha:** `<B_name>` and `<BB_name>` are often confused. `<B_name>` is the pre-section (before the loop body) and `<BB_name>` is the post-section (after). Both require at least one result to display. If you put `</ul>` in `<B_name>` you get broken HTML.

---

## Native BOUCLE Types

| Type | Table | Primary use |
|---|---|---|
| `ARTICLES` | `spip_articles` | Published articles |
| `RUBRIQUES` | `spip_rubriques` | Sections / categories |
| `AUTEURS` | `spip_auteurs` | Authors |
| `FORUMS` | `spip_forum` | Forum messages |
| `MOTS` | `spip_mots` | Keywords / tags |
| `DOCUMENTS` | `spip_documents` | Files, images, media |
| `SITES` | `spip_syndic` | Linked external sites |
| `SYNDIC_ARTICLES` | `spip_syndic_articles` | Syndicated feed items |
| `HIERARCHIE` | `spip_rubriques` | Ancestor rubriques (breadcrumb) |
| `DATA` | *(any source)* | External data: JSON, CSV, XML, YAML, etc. |
| `CONDITION` | *(none)* | Conditional block, no SQL query |

### ARTICLES

Lists articles from the SPIP database.

```html
<!-- Last 5 articles in rubrique #3, newest first -->
<BOUCLE_arts(ARTICLES){id_rubrique=3}{par date}{inverse}{0,5}>
  <li><a href="#URL_ARTICLE">#TITRE</a></li>
</BOUCLE_arts>
```

Key balises: `#TITRE`, `#TEXTE`, `#CHAPO`, `#DESCRIPTIF`, `#DATE`, `#DATE_REDAC`, `#URL_ARTICLE`, `#LOGO_ARTICLE`, `#ID_ARTICLE`, `#ID_RUBRIQUE`, `#LANG`, `#STATUT`.

### RUBRIQUES

Lists sections (categories). Context automatically passes `{id_rubrique}` from a parent BOUCLE.

```html
<!-- Top-level sections, sorted alphabetically -->
<BOUCLE_rubs(RUBRIQUES){racine}{par titre}>
  <h2><a href="#URL_RUBRIQUE">#TITRE</a></h2>
</BOUCLE_rubs>
```

Key balises: `#TITRE`, `#TEXTE`, `#URL_RUBRIQUE`, `#LOGO_RUBRIQUE`, `#ID_RUBRIQUE`, `#ID_PARENT`, `#PROFONDEUR`.

### AUTEURS

Lists authors. Can be scoped to an article via `{id_article}`.

```html
<BOUCLE_auteurs(AUTEURS){id_article}>
  <span class="author">#NOM</span>
</BOUCLE_auteurs>
```

### FORUMS

Lists forum messages. Scope with `{id_article}` or `{id_parent}` for thread nesting.

```html
<BOUCLE_msgs(FORUMS){id_article}{par date}>
  <p><strong>#NOM_EMAIL</strong>: #TEXTE</p>
</BOUCLE_msgs>
```

### MOTS

Lists keywords/tags. Scope with `{id_article}` to get tags for a specific article, or `{type=Categorie}` to filter by keyword group.

```html
<BOUCLE_tags(MOTS){id_article}>
  <span class="tag">#TITRE</span>
</BOUCLE_tags>
```

### DOCUMENTS

Lists files and media attached to articles or rubriques.

```html
<!-- Images attached to current article -->
<BOUCLE_docs(DOCUMENTS){id_article}{extension IN jpg,png,webp}>
  [(#FICHIER|image_reduire{600})]
</BOUCLE_docs>
```

Key balises: `#FICHIER`, `#URL_DOCUMENT`, `#TITRE`, `#EXTENSION`, `#MIME_TYPE`, `#TAILLE`, `#LARGEUR`, `#HAUTEUR`.

### SITES and SYNDIC_ARTICLES

`SITES` lists linked external sites (from SPIP's syndication system). `SYNDIC_ARTICLES` lists individual items from those feeds.

```html
<BOUCLE_flux(SYNDIC_ARTICLES){id_syndic}{par date}{inverse}{0,5}>
  <li><a href="#URL_SYNDIC_ARTICLE">#TITRE</a></li>
</BOUCLE_flux>
```

---

## BOUCLE HIERARCHIE

`HIERARCHIE` returns the ancestor rubriques of the current page, from root down to the direct parent. It is used to build breadcrumbs.

```html
<BOUCLE_hier(HIERARCHIE){}>
  <a href="#URL_RUBRIQUE">#TITRE</a> &gt;
</BOUCLE_hier>
```

> **Invariant:** HIERARCHIE always includes the rubrique of the current page. It walks upward from root to the current rubrique. The final `>` separator is always output (consider using the pre/post sections or CSS to style or trim the last one).

**Breadcrumb with current page title:**

```html
<nav class="breadcrumb">
  <a href="#URL_SITE_SPIP">#NOM_SITE_SPIP</a>
  <BOUCLE_fil(HIERARCHIE){}>
    &rsaquo; <a href="#URL_RUBRIQUE">#TITRE</a>
  </BOUCLE_fil>
  &rsaquo; <span>#TITRE_PAGE</span>
</nav>
```

> **Gotcha:** HIERARCHIE does not include the article itself, only its ancestor rubriques. Add the article's `#TITRE` manually after the loop if you want it in the trail.

---

## Nested BOUCLEs and Context Inheritance

Inner loops automatically inherit the context (current identifiers) of their parent loop. A critère like `{id_rubrique}` inside a child loop means "use the `id_rubrique` of the current parent loop iteration."

```html
<BOUCLE_rubs(RUBRIQUES){racine}{par titre}>
  <h2>#TITRE</h2>
  <B_arts>
    <ul>
    <BOUCLE_arts(ARTICLES){id_rubrique}{par titre}>
      <li><a href="#URL_ARTICLE">#TITRE</a></li>
    </BOUCLE_arts>
    </ul>
  </B_arts>
</BOUCLE_rubs>
```

`{id_rubrique}` in the inner `BOUCLE_arts` automatically resolves to the `#ID_RUBRIQUE` of the current `BOUCLE_rubs` iteration — no explicit value needed.

### Accessing parent loop values

Use `#_parentname:BALISE` to read a balise from a named parent loop:

```html
<BOUCLE_rubs(RUBRIQUES)>
  <ul>
    <BOUCLE_arts(ARTICLES){id_rubrique}>
      <!-- Shows "SectionTitle — ArticleTitle" -->
      <li>#_rubs:TITRE — #TITRE</li>
    </BOUCLE_arts>
  </ul>
</BOUCLE_rubs>
```

> **Gotcha:** You must use the exact loop name (without `BOUCLE_` prefix) with an underscore prefix: `#_rubs:TITRE`, not `#rubs:TITRE`.

---

## Recursive BOUCLEs

A recursive boucle calls itself to traverse tree-structured data. The inner boucle references the outer boucle's name as its type.

```html
<BOUCLEx(TABLE){id_parent}>
  ...
  <BOUCLEn(BOUCLEx) />   <!-- calls BOUCLEx again with new context -->
  ...
</BOUCLEx>
```

### Full rubrique tree (nested `<ul>`)

```html
<B_rubs>
<ul>
</B_rubs>

<BOUCLE_rubs(RUBRIQUES){id_parent}{par num titre, titre}>
  <li>
    <a href="#URL_RUBRIQUE">#TITRE</a>
    <BOUCLE_sous(BOUCLE_rubs) />
  </li>
</BOUCLE_rubs>

<BB_rubs>
</ul>
</BB_rubs>
```

- The first call uses `{id_parent}` = 0 (root rubriques).
- Each recursive call passes the current rubrique's `id_rubrique` as the new `id_parent`.
- Recursion stops automatically when a rubrique has no children.

> **Gotcha:** Recursive loops have no depth limit guard in SPIP itself. On a very deep or circular tree (e.g. broken `id_parent` data), they can loop indefinitely. Always ensure the tree data is sound.

**Forum thread tree (recursive replies):**

```html
<BOUCLE_forum(FORUMS){id_article}>
  <p>#TEXTE</p>
  <B_replies>
    <ul class="replies">
    <BOUCLE_replies(FORUMS){id_parent}>
      <li>#TEXTE
        <BOUCLE_deep(BOUCLE_replies) />
      </li>
    </BOUCLE_replies>
    </ul>
  </B_replies>
</BOUCLE_forum>
```

---

## BOUCLE DATA

`DATA` iterates over any data source, not just SQL tables. It was introduced in SPIP 3.0 and remains central in SPIP 4.x.

### Syntax

```html
<BOUCLE_name(DATA){source format, données}>
  #VALEUR
</BOUCLE_name>
```

The `{source format, données}` critère requires:
- **format** — one of: `table` / `array` / `tableau`, `csv`, `json`, `yaml`, `xml`, `file`, `glob`, `rss` / `atom`, `sql`, `plugins`
- **données** — a data path: file path, URL, PHP array expression, or inline string

### Examples

```html
<!-- JSON from external API -->
<BOUCLE_events(DATA){source json, https://api.example.com/events.json}>
  <li>#VALEUR{titre} — [(#VALEUR{date}|affdate)]</li>
</BOUCLE_events>

<!-- CSV file on disk -->
<BOUCLE_csv(DATA){source csv, local/data/produits.csv}>
  <tr><td>#VALEUR{nom}</td><td>#VALEUR{prix}</td></tr>
</BOUCLE_csv>

<!-- Inline list -->
<BOUCLE_sizes(DATA){liste S,M,L,XL}{" | "}>
  #VALEUR
</BOUCLE_sizes>
```

### Cache for DATA loops

By default, DATA loops on a URL are cached for 24h. Override with `{datacache N}` (N in seconds), or disable with `{datacache 0}`:

```html
<!-- Refresh every hour -->
<BOUCLE_feed(DATA){source rss, https://blog.example.com/feed}{datacache 3600}>
  <li><a href="#VALEUR{link}">#VALEUR{title}</a></li>
</BOUCLE_feed>
```

> **Gotcha:** DATA loops on a URL incur an HTTP request during page compilation (cached). They do not make a live HTTP call on every page load when the cache is active. Remove `{datacache 0}` in production to avoid latency.

---

## BOUCLE CONDITION

`CONDITION` executes its body if a condition evaluates to true. It performs no SQL query.

### Syntax

```html
<BOUCLE_name(CONDITION){si expression}>
  ... shown when condition is true ...
</BOUCLE_name>
<//B_name>
  ... shown when condition is false ...
</BOUCLE_name>
```

### Use cases

```html
<!-- Show content only to logged-in users -->
<BOUCLE_auth(CONDITION){si #SESSION{id_auteur}}>
  <p>Bienvenue, [(#SESSION{nom})] !</p>
</BOUCLE_auth>
<//B_auth>
  <p>Veuillez vous <a href="#URL_PAGE{login}">connecter</a>.</p>
</BOUCLE_auth>

<!-- Show a block only if an ENV variable is set -->
<BOUCLE_show(CONDITION){si #ENV{mode_debug}}>
  <div class="debug">Mode debug actif</div>
</BOUCLE_show>
```

> **Gotcha:** `{si ...}` tests for truthiness: an empty string, `"0"`, or `null` evaluates to false. Be careful with balises that return `"0"` as a legitimate value — use explicit comparisons where needed: `{si #ENV{count}|>{0}}`.

---

## Optional Table (Plugin-dependent)

When a loop targets a table that may not exist (e.g. from an optional plugin), append `?` to avoid a visible error for administrators:

```html
<BOUCLE_events(EVENEMENTS ?){id_article}{!par date}>
  ...
</BOUCLE_events>
```

Without `?`, SPIP shows a warning on the page to site administrators when the table is missing.

---

## Summary: When to Use Which BOUCLE

| Need | Use |
|---|---|
| List articles, filter by rubrique/date/statut | `ARTICLES` |
| Navigate section tree | `RUBRIQUES` |
| Breadcrumb trail | `HIERARCHIE` |
| Full site tree (nested menus) | `RUBRIQUES` recursive |
| Tags/keywords | `MOTS` |
| Attached files/images | `DOCUMENTS` |
| External JSON/CSV/RSS data | `DATA` |
| Show/hide based on condition | `CONDITION` |
| Forum threads | `FORUMS` recursive |
