# Balises & Critères Reference

---

## Critères

Critères control what a BOUCLE selects and how results are ordered. They are placed in `{…}` braces after the table name. Multiple critères combine with AND logic.

```html
<BOUCLE_a(ARTICLES){id_rubrique}{statut=publie}{par date}{inverse}{pagination 10}>
```

---

### Filtering Critères

#### By identifier

| Critère | Effect |
|---|---|
| `{id_article}` | Article whose `id_article` matches the context (URL or parent loop) |
| `{id_article=5}` | Exactly article #5 |
| `{id_rubrique}` | Articles in the rubrique matching context |
| `{id_rubrique=3}` | Articles in rubrique #3 |
| `{id_secteur}` | All articles in a top-level secteur (rubrique at root) |
| `{id_secteur=2}` | Articles in secteur #2 |
| `{id_mot}` | Articles tagged with a given mot-clé |
| `{id_parent}` | Sub-rubriques of the parent rubrique in context |
| `{racine}` | Top-level rubriques only (equivalent to `{id_parent=0}`) |

```html
<!-- All articles in rubrique 3 -->
<BOUCLE_arts(ARTICLES){id_rubrique=3}>
  <a href="#URL_ARTICLE">#TITRE</a>
</BOUCLE_arts>
```

**Gotcha:** `{id_rubrique}` without a value reads `id_rubrique` from the current context (URL or enclosing loop). If neither provides one, it returns all articles. Always verify the context is what you expect.

---

#### Scope Critères

| Critère | Effect |
|---|---|
| `{branche}` | Articles in the current rubrique AND all its sub-rubriques |
| `{branche?}` | Same as `{branche}` but optional — applies only if a rubrique is in context; otherwise returns all |
| `{!branche}` | Articles NOT in the current rubrique or its sub-rubriques |

```html
<!-- All articles in current section tree -->
<BOUCLE_section(ARTICLES){branche}>
  #TITRE<br>
</BOUCLE_section>
```

**Gotcha:** `{branche}` is expensive on large sites because SPIP must compute the full subtree of rubrique IDs. Use sparingly; prefer `{id_secteur}` when you only need the top-level scope.

---

#### Statut Critère

| Critère | Effect |
|---|---|
| `{statut=publie}` | Published articles only (default for most boucles) |
| `{statut=prepa}` | Draft articles (private space only) |
| `{statut=prop}` | Articles submitted for validation |
| `{tout}` | All statuses — published, draft, submitted, refused |

```html
<!-- All articles including drafts (admin view) -->
<BOUCLE_all(ARTICLES){tout}>
  [#STATUT] - #TITRE<br>
</BOUCLE_all>
```

**Gotcha:** By default, BOUCLE ARTICLES only returns `statut=publie` articles on the public site. You only need to write `{statut=publie}` explicitly if you have mixed contexts. `{tout}` is needed to show drafts.

---

#### Language Critères

| Critère | Effect |
|---|---|
| `{lang}` | Articles in the current site language |
| `{lang_select}` | Articles in the language selected by the visitor |
| `{lang=fr}` | Articles in French specifically |

```html
<!-- Articles in visitor's chosen language -->
<BOUCLE_traduits(ARTICLES){id_rubrique}{lang_select}>
  #TITRE
</BOUCLE_traduits>
```

---

#### Doublons (deduplication)

`{doublons}` prevents an item already displayed by another `{doublons}` boucle from appearing again. Both boucles must carry `{doublons}`.

```html
<!-- Featured article -->
<BOUCLE_vedette(ARTICLES){id_rubrique}{par date}{inverse}{limit 1}{doublons}>
<div class="featured"><h2>#TITRE</h2>#DESCRIPTIF</div>
</BOUCLE_vedette>

<!-- Remaining articles (excludes the featured one) -->
<BOUCLE_reste(ARTICLES){id_rubrique}{par date}{inverse}{doublons}>
<p><a href="#URL_ARTICLE">#TITRE</a></p>
</BOUCLE_reste>
```

Named doublons: `{doublons rouge}` and `{doublons bleu}` are independent sets — articles excluded from one set are not excluded from the other.

`{exclus}` is the local equivalent: it excludes the current article (the one whose page is being rendered) from the boucle result.

---

#### Recherche (full-text search)

| Critère | Effect |
|---|---|
| `{recherche}` | Filter to articles matching the search query in `?recherche=` URL param |

```html
<!-- Search results page -->
<BOUCLE_resultats(ARTICLES){recherche}{par points}{inverse}>
  <a href="#URL_ARTICLE">#TITRE</a> (#POINTS points)<br>
</BOUCLE_resultats>
<//B_resultats>Aucun résultat.

Use `#POINTS` (the relevance score) to sort results. Only valid inside a `{recherche}` boucle.

---

#### Jointure

`{jointure table}` forces a SQL JOIN with another table when the automatic join resolution isn't enough.

```html
<!-- Articles that have at least one document -->
<BOUCLE_avec_docs(ARTICLES){id_rubrique}{jointure documents}>
  #TITRE
</BOUCLE_avec_docs>
```

---

#### Date Critères

| Critère | Effect |
|---|---|
| `{age<30}` | Published less than 30 days ago |
| `{age>365}` | Published more than 365 days ago |
| `{age<0}` | Post-dated articles (future publication date) |
| `{age_redac<30}` | Authored (first redaction) less than 30 days ago |
| `{annee=2024}` | Published in year 2024 |
| `{mois=1}` | Published in January (of any year) |
| `{mois_redac=6}` | Authored in June |
| `{annee<=2000}` | Published before end of year 2000 |

```html
<!-- Articles from this month -->
<BOUCLE_recents(ARTICLES){age<31}{par date}{inverse}>
  #TITRE - [(#DATE|affdate)]<br>
</BOUCLE_recents>
```

```html
<!-- Articles published in 2023 -->
<BOUCLE_archive(ARTICLES){annee=2023}{par date}>
  #TITRE<br>
</BOUCLE_archive>
```

---

### Sorting Critères

| Critère | Effect |
|---|---|
| `{par titre}` | Alphabetical by title |
| `{par date}` | Chronological (oldest first) |
| `{par date}{inverse}` | Newest first |
| `{par date_redac}` | By authoring date |
| `{par points}` | By search relevance score (only with `{recherche}`) |
| `{par hasard}` | Random order |
| `{inverse}` | Reverse any preceding `{par …}` sort |

Multiple `{par …}` critères can be combined for secondary sort.

```html
<!-- Random featured article from rubrique 5 -->
<BOUCLE_rand(ARTICLES){id_rubrique=5}{par hasard}{limit 1}>
  <h2>#TITRE</h2>
</BOUCLE_rand>
```

---

### Pagination & Limit Critères

| Critère | Effect |
|---|---|
| `{limit N}` | At most N results |
| `{limit A,B}` | Skip A results, then show B (e.g., `{limit 3,5}` = results 4–8) |
| `{pagination N}` | N results per page; enables `#PAGINATION` and `debut_X` URL param |
| `{0, n-10}` | All results except the last 10 |
| `{n-5, 5}` | The last 5 results |

**Gotcha:** `{a, n}`, `{a, n-b}` and `{n-a, b}` do NOT work together with `{pagination}` — combine them and you get inconsistent results.

```html
<!-- Paginated list of 10 articles per page -->
<B_articles>
<ul>
</B_articles>

<BOUCLE_articles(ARTICLES){id_rubrique}{par date}{inverse}{pagination 10}>
  <li><a href="#URL_ARTICLE">#TITRE</a></li>
</BOUCLE_articles>

<BB_articles>
</ul>
[(#PAGINATION)]
</BB_articles>

<//B_articles>
<p>Aucun article dans cette rubrique.</p>
```

The `debut_boucle` URL parameter (auto-generated by `#PAGINATION`) controls the offset. Its exact name depends on the boucle name: `debut_articles` for `BOUCLE_articles`.

---

## Balises

Balises output content from the database, from context, or computed by SPIP. They appear inside or outside boucles.

**Syntax recap:**
- `#BALISE` — bare, fails silently if empty
- `[(#BALISE)]` — optional: outputs nothing if balise is empty
- `[before(#BALISE)after]` — wraps with text only when balise has content
- `[(#BALISE|filtre1|filtre2)]` — filter chain

---

### Balises inside BOUCLE ARTICLES

| Balise | Description |
|---|---|
| `#ID_ARTICLE` | Unique identifier |
| `#TITRE` | Title |
| `#TEXTE` | Body text (SPIP markup rendered) |
| `#DESCRIPTIF` | Short description field |
| `#CHAPO` | Introduction / chapeau text |
| `#PS` | Post-scriptum |
| `#SURTITRE` | Surtitle (pre-title) |
| `#SOUSTITRE` | Subtitle |
| `#DATE` | Publication date |
| `#DATE_REDAC` | Authoring date |
| `#DATE_MODIF` | Last modification date |
| `#URL_ARTICLE` | Public URL of the article |
| `#LANG` | Language code of the article (`fr`, `en`, etc.) |
| `#STATUT` | Status: `publie`, `prepa`, `prop`, `refuse` |
| `#ID_RUBRIQUE` | ID of the parent rubrique |
| `#ID_SECTEUR` | ID of the top-level secteur |
| `#POPULARITE` | Popularity score (percentage) |

```html
<article>
  <h1>#TITRE</h1>
  [<p class="intro">(#CHAPO)</p>]
  <div class="body">#TEXTE</div>
  <p class="date">[(#DATE|affdate_court)]</p>
  <a href="#URL_ARTICLE">Lire la suite</a>
</article>
```

#### Logo balises (ARTICLES)

| Balise | Description |
|---|---|
| `#LOGO_ARTICLE` | Logo of the article (with hover support if two logos uploaded) |
| `#LOGO_ARTICLE_NORMAL` | Logo without hover |
| `#LOGO_ARTICLE_SURVOL` | Hover logo only |
| `#LOGO_ARTICLE_RUBRIQUE` | Article logo; falls back to the rubrique logo if no article logo |
| `#LOGO_RUBRIQUE` | Logo of the parent rubrique |

```html
<!-- Fallback logo: article logo if set, else rubrique logo -->
[(#LOGO_ARTICLE_RUBRIQUE{#URL_ARTICLE})]

<!-- Conditionally show logo only when present -->
[(#LOGO_ARTICLE)]
```

**Gotcha:** `#LOGO_ARTICLE` returns nothing if no logo has been uploaded. Use `#LOGO_ARTICLE_RUBRIQUE` to get automatic fallback to the rubrique's logo. Always wrap logo balises in `[(…)]` so no empty `<img>` tags appear.

---

### Balises inside BOUCLE RUBRIQUES

| Balise | Description |
|---|---|
| `#ID_RUBRIQUE` | Unique identifier |
| `#TITRE` | Title |
| `#TEXTE` | Body text of the rubrique description |
| `#DESCRIPTIF` | Short descriptif |
| `#ID_PARENT` | ID of the parent rubrique (0 if at root) |
| `#ID_SECTEUR` | ID of the top-level secteur |
| `#PROFONDEUR` | Depth level in the rubrique tree (0 = root) |
| `#LANG` | Language of the rubrique |
| `#URL_RUBRIQUE` | Public URL |
| `#DATE` | Date of the most recent publication in this rubrique |
| `#INTRODUCTION` | First 600 characters of `#TEXTE`, no markup |
| `#NOTES` | Footnotes generated from the text |
| `#LOGO_RUBRIQUE` | Logo of the rubrique |
| `#LOGO_RUBRIQUE_NORMAL` | Logo without hover |
| `#LOGO_RUBRIQUE_SURVOL` | Hover logo |

```html
<BOUCLE_rubriques(RUBRIQUES){id_parent=0}{par titre}>
<li>
  [(#LOGO_RUBRIQUE{#URL_RUBRIQUE})]
  <a href="#URL_RUBRIQUE">#TITRE</a>
</li>
</BOUCLE_rubriques>
```

**Gotcha:** A BOUCLE RUBRIQUES by default only returns **active** rubriques — ones that contain published articles, documents, or sub-rubriques. Use `{tout}` to include empty rubriques.

---

### Balises inside BOUCLE DOCUMENTS

| Balise | Description |
|---|---|
| `#ID_DOCUMENT` | Unique identifier |
| `#TITRE` | Title of the document |
| `#DESCRIPTIF` | Description |
| `#CREDITS` | Credits (e.g. photographer name) |
| `#ALT` | Alternative text for images |
| `#FICHIER` | Relative URL of the file (path from site root) |
| `#URL_DOCUMENT` | Absolute URL for linking |
| `#EXTENSION` | File extension: `pdf`, `jpg`, `mp4`, etc. |
| `#MEDIA` | Media type: `image`, `audio`, `video`, `file` |
| `#TYPE_DOCUMENT` | Human-readable type label |
| `#LARGEUR` | Image width in pixels |
| `#HAUTEUR` | Image height in pixels |
| `#LOGO_DOCUMENT` | Thumbnail/preview image |
| `#EMBED_DOCUMENT` | Embeds the document (player for audio/video, viewer for PDF) |

```html
<!-- Image gallery -->
<BOUCLE_gallery(DOCUMENTS){id_article}{mode=image}{doublons}>
<figure>
  [(#FICHIER|image_reduire{300})]
  [<figcaption>(#TITRE)</figcaption>]
</figure>
</BOUCLE_gallery>

<!-- Clickable thumbnail linking to full file -->
[(#LOGO_DOCUMENT{#URL_DOCUMENT})]

<!-- PDF download link -->
<BOUCLE_pdf(DOCUMENTS){id_article}{extension=pdf}>
<a href="#URL_DOCUMENT">#TITRE (PDF)</a>
</BOUCLE_pdf>
```

**Gotcha:** Use `{mode=document}` to get attached documents, `{mode=image}` for inline images, or omit `mode` to get both. The `{doublons}` critère prevents images already embedded in `#TEXTE` from appearing twice in the document list.

---

### Global Balises (available outside any boucle)

These balises work anywhere in a squelette, inside or outside a boucle.

#### Site configuration

| Balise | Description |
|---|---|
| `#NOM_SITE_SPIP` | Site name (from admin configuration) |
| `#URL_SITE_SPIP` | Site URL, no trailing slash |
| `#DESCRIPTIF_SITE_SPIP` | Site description |
| `#EMAIL_WEBMASTER` | Webmaster email |
| `#LOGO_SITE_SPIP` | Site logo |
| `#CHARSET` | Character encoding (default: `utf-8`) |
| `#LANG` | Current language of the site |
| `#LANG_DIR` | Text direction: `ltr` or `rtl` |

```html
<meta charset="[(#CHARSET)]">
<html lang="#LANG" dir="#LANG_DIR">
<title>#NOM_SITE_SPIP</title>
```

#### Environment & variables

| Balise | Description |
|---|---|
| `#ENV{key}` | Value of URL parameter or INCLURE argument named `key` |
| `#ENV{key, default}` | With fallback default value |
| `#SET{key, value}` | Sets a local variable in the current squelette |
| `#GET{key}` | Reads a variable set with `#SET` |

```html
<!-- Read URL param: spip.php?page=foo&id=42 -->
[(#ENV{id})]

<!-- Set and reuse a value -->
#SET{couleur, bleu}
La couleur est : #GET{couleur}
```

**Gotcha:** `#ENV` and `#GET`/`#SET` do not cross INCLURE boundaries. Variables set with `#SET` inside an INCLURE are not visible in the parent squelette, and vice versa. Pass values explicitly as INCLURE arguments.

#### Pagination

| Balise | Description |
|---|---|
| `#PAGINATION` | Renders the pagination navigation links |
| `#PAGINATION{modele}` | Renders pagination using a custom modèle |
| `#TOTAL_BOUCLE` | Total number of results in the boucle (before pagination) |
| `#COMPTEUR_BOUCLE` | Current row index within the loop (1-based) |

```html
<!-- Full paginated list with count -->
<p>[(#TOTAL_BOUCLE) articles au total</p>

<BOUCLE_arts(ARTICLES){par date}{inverse}{pagination 10}>
  <p>#COMPTEUR_BOUCLE. <a href="#URL_ARTICLE">#TITRE</a></p>
</BOUCLE_arts>

<BB_arts>
[(#PAGINATION)]
</BB_arts>
```

**Gotcha:** `#PAGINATION` must be placed in the **post-section** (`<BB_…>`) or after the loop, not inside the loop body. It renders as nothing when pagination is not active.

#### Cache

| Balise | Description |
|---|---|
| `#CACHE{seconds}` | Sets squelette cache duration in seconds |
| `#CACHE{0}` | Disables cache — squelette recomputed on every request |

```html
<!-- Cache for 1 hour -->
#CACHE{3600}

<!-- Cache for 1 day -->
#CACHE{24*3600}

<!-- Never cache (e.g., real-time search results) -->
#CACHE{0}
```

**Gotcha:** `#CACHE` affects the whole squelette, not individual boucles. Place it at the top of the file. If omitted, SPIP uses a default cache duration (typically 24h).

#### Navigation & URLs

| Balise | Description |
|---|---|
| `#SELF` | URL of the current page, stripped of SPIP internal params |
| `#URL_PAGE{name}` | URL of a squelette page: `spip.php?page=name` |
| `#URL_PAGE{name, key=val}` | URL with additional params |
| `#MENU_LANG` | Language switcher menu |
| `#REM` | Comment — not rendered in output: `[(#REM) comment here]` |
| `#SQUELETTE` | Path of the current squelette file |

```html
<form action="#SELF" method="get">
  <input type="search" name="recherche">
</form>

[(#REM) This section is the main article body ]
```

---

### Other Loop-Specific Balises (quick reference)

#### BOUCLE AUTEURS

| Balise | Description |
|---|---|
| `#ID_AUTEUR` | Author ID |
| `#NOM` | Author name |
| `#BIO` | Biography |
| `#EMAIL` | Email |
| `#URL_AUTEUR` | Public author page URL |
| `#LOGO_AUTEUR` | Author avatar/logo |

#### BOUCLE MOTS

| Balise | Description |
|---|---|
| `#ID_MOT` | Mot-clé ID |
| `#TITRE` | Mot-clé label |
| `#URL_MOT` | Public URL |
| `#ID_GROUPE` | Group ID this mot belongs to |

#### BOUCLE HIERARCHIE

| Balise | Description |
|---|---|
| `#ID_RUBRIQUE` | Rubrique ID at this level |
| `#TITRE` | Title |
| `#URL_RUBRIQUE` | URL |
| `#PROFONDEUR` | Depth (0 = root) |
| `#RANG` | Position of this node in the hierarchy path |

```html
<!-- Breadcrumb -->
<BOUCLE_hier(HIERARCHIE){self}>
<span><a href="#URL_RUBRIQUE">#TITRE</a> &rsaquo; </span>
</BOUCLE_hier>
#TITRE
```

---

## Common Patterns & Gotchas Summary

| Issue | Solution |
|---|---|
| Logo outputs empty `<img>` | Wrap in `[(…)]`: `[(#LOGO_ARTICLE)]` |
| `#LOGO_ARTICLE` shows nothing | Use `#LOGO_ARTICLE_RUBRIQUE` for automatic rubrique fallback |
| `{pagination}` + `{0, n-5}` broken | Never combine `{pagination}` with `{n-a, b}` variants |
| `#ENV{key}` not available in INCLURE | Pass explicitly: `<INCLURE{fond=frag, key=#ENV{key}}>` |
| RUBRIQUES boucle skips empty rubriques | Add `{tout}` to include empty ones |
| `#PAGINATION` appears inside loop | Move it to `<BB_name>` post-section |
| `{doublons}` not working | Both boucles must carry the `{doublons}` critère |
| `#TEXTE` shows raw SPIP markup | Apply `|propre` filter: `[(#TEXTE|propre)]` — but in ARTICLES boucle, `#TEXTE` is already processed |
