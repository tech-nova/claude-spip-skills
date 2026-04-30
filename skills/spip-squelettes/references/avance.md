# Advanced Features Reference

Topics: INCLURE fragments, AJAX reloading, modèles, native formulaires, squelette variants, 404 pages, #CACHE, #ENV/#SET/#GET.

---

## INCLURE — Including Fragments

`INCLURE` composes a squelette from smaller reusable parts. The fond argument names the template file (without `.html`).

### Syntax forms

```html
<!-- Basic include — mandatory, errors if fond missing -->
<INCLURE{fond=inc-sidebar} />

<!-- Optional include — renders nothing if fond not found -->
[(#INCLURE{fond=inc-sidebar})]

<!-- Pass the full current environment -->
<INCLURE{fond=inc-sidebar,env} />

<!-- Pass specific arguments -->
<INCLURE{fond=inc-liste-articles,id_rubrique=#ID_RUBRIQUE,nb=5} />

<!-- Mix: pass env + override one key -->
<INCLURE{fond=inc-header,env,titre_page=Mon titre} />
```

### Argument passing

Arguments become `#ENV{key}` inside the included file:

```html
<!-- parent template -->
<INCLURE{fond=inc-card,id_article=#ID_ARTICLE,classe=featured} />

<!-- inc-card.html -->
<BOUCLE_a(ARTICLES){id_article=#ENV{id_article}}>
  <article class="[(#ENV{classe})]">
    <h2>#TITRE</h2>
  </article>
</BOUCLE_a>
```

### File resolution order

SPIP searches for the template in this order:

1. firstly in the list of folders specified in the `$dossier_squelettes` folder, if one has been defined;
2. then in the 'squelettes/' folder located at site root;
3. then in the list of folders in the `$plugins` variable;
4. then in the site root;
5. then in the 'squelettes-dist/' folder;
6. and finally in the 'ecrire/' directory.

This means you can override any default fond by creating a same-named file in `squelettes/`.

### Gotcha: env is not inherited automatically

Without `env` or explicit arguments, the included template starts with an empty environment. Always be explicit about what the fragment needs.

```html
<!-- WRONG: inc-nav.html has no access to #ID_RUBRIQUE -->
<INCLURE{fond=inc-nav} />

<!-- CORRECT: pass what the fragment needs -->
<INCLURE{fond=inc-nav,id_rubrique=#ID_RUBRIQUE} />
<!-- or pass everything -->
<INCLURE{fond=inc-nav,env} />
```

---

## AJAX — Partial Reload

Adding `{ajax}` to an INCLURE makes the block reloadable without a full page refresh. SPIP's built-in JavaScript intercepts links and form submissions inside the block and replaces only that block's HTML.

### Syntax

```html
<!-- INCLURE with ajax reloading -->
<INCLURE{fond=inc-liste-articles,env}{ajax} />

<!-- Custom container ID (useful for JS targeting) -->
<INCLURE{fond=inc-recherche,env}{ajax id=zone-resultats} />
```

### How it works

1. SPIP wraps the output in `<div id="spip_ancre_NNN">…</div>`
2. Any link or form inside that div gets intercepted by SPIP JS
3. SPIP fetches the new content for just that fond and replaces the div

### Requirements

SPIP's JavaScript must be loaded in the page. Add this in your main squelette:

```html
[(#CHEMIN{javascript/jquery.form.js}|oui)]
```

Or use the `spip_javascripts` plugin, which is included by default in SPIP 4.x distributions.

### Limitations

- The included fond must be a self-contained SPIP template (no inline PHP)
- The fond is re-executed server-side on each AJAX request — keep it cacheable
- Does not work across different domains (same-origin only)
- `{ajax}` is not inherited by nested INCLUREs — add it to each one explicitly

### Gotcha: CACHE and AJAX

If the included fond is cached (`#CACHE{3600}`), AJAX reloads serve cached content until the cache expires. Use `#CACHE{0}` for live data, or set a short TTL.

```html
<!-- inc-live-results.html -->
#CACHE{0}
<BOUCLE_arts(ARTICLES){recherche}>
  <p>#TITRE</p>
</BOUCLE_arts>
```

---

## Modèles — Inline Shortcodes

Modèles are small templates in `squelettes/modeles/` (or `modeles/` in a plugin). Editors insert them into article body text using a shortcode syntax; SPIP processes them when rendering `#TEXTE`.

### Calling a modèle from article text

```
<mymodel|param1=value1|param2=value2>
```

Example — embed a highlighted article:
```
<article_vedette|id_article=42|style=compact>
```

### Creating a modèle

Create `squelettes/modeles/article_vedette.html`:

```html
<BOUCLE_m(ARTICLES){id_article=#ENV{id_article}}>
  <div class="vedette [(#ENV{style}|sinon{full})]">
    [(#LOGO_ARTICLE|image_reduire{200})]
    <h3><a href="#URL_ARTICLE">#TITRE</a></h3>
    [(#DESCRIPTIF)]
  </div>
</BOUCLE_m>
```

Parameters from the shortcode arrive as `#ENV{param}`.

### Gotcha: modèles are processed inside #TEXTE only

Shortcodes in `#TEXTE` are processed when `propre` or `typo` is applied. They are **not** processed in `#CHAPO`, `#TITRE`, or custom fields unless you explicitly apply `|propre`.

---

## Native Formulaires

SPIP ships with several ready-to-use form balises. Drop them in any squelette; SPIP handles rendering, validation, and processing.

| Balise | Purpose | Typical placement |
|--------|---------|-------------------|
| `#FORMULAIRE_FORUM` | Post a comment on the current article | Inside `BOUCLE_ARTICLES` context |
| `#FORMULAIRE_SIGNATURE` | Sign a petition | Inside `BOUCLE_ARTICLES` with petition enabled |
| `#FORMULAIRE_INSCRIPTION` | New user registration | Any page, typically `login.html` |
| `#FORMULAIRE_LOGIN` | Log in / log out | Any page, typically `login.html` or header |
| `#FORMULAIRE_OUBLI` | Password reset by email | Any page |
| `#FORMULAIRE_RECHERCHE` | Site search form | Any page, typically header |

### Usage examples

```html
<!-- Comment form — context must be inside BOUCLE_ARTICLES -->
<BOUCLE_art(ARTICLES){id_article}>
  ...
  #FORMULAIRE_FORUM
</BOUCLE_art>

<!-- Login form on any page -->
#FORMULAIRE_LOGIN

<!-- Search form with custom button text -->
#FORMULAIRE_RECHERCHE

<!-- Registration form -->
#FORMULAIRE_INSCRIPTION
```

### Gotcha: #FORMULAIRE_FORUM context

`#FORMULAIRE_FORUM` requires `id_article` in context. Outside a `BOUCLE_ARTICLES`, it renders nothing. If you place it in a shared fragment, always pass `id_article` explicitly.

```html
<!-- inc-footer.html — no article context, so pass it explicitly -->
<INCLURE{fond=inc-commentaires,id_article=#ENV{id_article}} />

<!-- inc-commentaires.html -->
<BOUCLE_a(ARTICLES){id_article=#ENV{id_article}}>
  #FORMULAIRE_FORUM
</BOUCLE_a>
```

### PHP-level customization

To customize form validation or processing logic, use the `formulaire_charger`, `formulaire_verifier`, `formulaire_traiter` pipelines — that requires PHP and belongs in a plugin. See the `spip-plugins` skill.

---

## Custom page fonds

Any file in `squelettes/` is accessible as a page:

```
squelettes/plan.html          → accessible at ?page=plan
squelettes/contact.html       → accessible at ?page=contact
```

---

## 404 Page

Create `squelettes/404.html` to display a custom error page when a resource is not found.

```html
<!-- squelettes/404.html -->
<!DOCTYPE html>
<html>
<head><title>Page introuvable - [(#NOM_SITE_SPIP)]</title></head>
<body>
  <h1>Page introuvable</h1>
  <p>La page demandée n'existe pas ou a été déplacée.</p>

  <!-- Show a search form to help the visitor -->
  #FORMULAIRE_RECHERCHE

  <!-- Or list recent articles -->
  <BOUCLE_recent(ARTICLES){statut=publie}{par date}{inverse}{limit 5}>
    <li><a href="#URL_ARTICLE">#TITRE</a></li>
  </BOUCLE_recent>
</body>
</html>
```

SPIP automatically serves this page with an HTTP 404 status code — no PHP configuration required.

### Gotcha: 404.html is not triggered for private-space requests

The custom 404 page applies to the public site. Broken URLs in the private administration interface show SPIP's built-in error page.

---

## #CACHE — Cache Control

`#CACHE{seconds}` controls how long SPIP caches a page. Place it anywhere in the squelette (by convention, near the top).

```html
#CACHE{3600}        <!-- cache for 1 hour (default) -->
#CACHE{86400}       <!-- cache for 24 hours -->
#CACHE{0}           <!-- never cache — always regenerate -->
```

### Cache invalidation

SPIP automatically invalidates the cache when content in the page's context changes:
- An article in a `BOUCLE_ARTICLES` loop is published, modified, or unpublished
- A rubrique is updated

This means `#CACHE{3600}` does not mean visitors wait an hour to see a new article — the cache is cleared immediately when the article is published.

### Personalized cache

Pages with `#CACHE{3600}` are shared across all anonymous visitors. Logged-in users always bypass the cache and see live content. To serve different content to logged-in users, use `BOUCLE_CONDITION` with `#SESSION`.

### Gotcha: #CACHE{0} has a real cost

Every visit regenerates the page from scratch. On high-traffic sites, use the shortest non-zero TTL that makes sense for your data freshness requirements rather than defaulting to 0.

### Gotcha: #CACHE inside INCLURE

The `#CACHE` of the main page controls the outer cache. If an included fragment has its own `#CACHE{0}`, the fragment is regenerated but wrapped inside a cached outer page — the outer cache takes precedence for the full page delivery unless AJAX is used.

---

## #ENV, #SET, #GET — Variables

### #ENV — Read environment and URL parameters

`#ENV{key}` reads from:
1. Arguments passed in the current `INCLURE`
2. URL parameters (`?key=value`)
3. The current page's context

```html
<!-- URL: ?page=article&id_article=42 -->
#ENV{id_article}         → 42
#ENV{page}               → article

<!-- With a default value if key is absent -->
#ENV{mode, liste}        → "liste" if mode not in URL

<!-- Used in a critère -->
<BOUCLE_a(ARTICLES){id_article=#ENV{id_article}}>
  #TITRE
</BOUCLE_a>
```

### #SET and #GET — Template-scoped variables

`#SET` stores a value; `#GET` retrieves it. Scope is **limited to the current template file** — values do NOT cross INCLURE boundaries.

```html
<!-- Set a variable -->
#SET{compteur, 0}
#SET{titre_formate, [(#TITRE|maj|couper{50})]}

<!-- Read it back -->
[(#GET{titre_formate})]

<!-- Useful for avoiding double computation -->
<BOUCLE_a(ARTICLES){id_article}>
  #SET{url, #URL_ARTICLE}
  <a href="[(#GET{url})]">#TITRE</a>
  <link rel="canonical" href="[(#GET{url})]" />
</BOUCLE_a>
```

### Gotcha: #SET does not cross INCLURE

```html
<!-- parent.html -->
#SET{myvar, hello}
<INCLURE{fond=child} />

<!-- child.html -->
[(#GET{myvar})]   ← EMPTY — #SET does not cross INCLURE boundary
```

To pass a value into an INCLURE, use an explicit argument:

```html
<!-- CORRECT -->
<INCLURE{fond=child,myvar=#GET{myvar}} />

<!-- child.html -->
#ENV{myvar}   ← "hello" ✓
```

### #SESSION — Logged-in user data

`#SESSION{key}` reads the current logged-in user's session data:

```html
[(#SESSION{nom})]          <!-- username -->
[(#SESSION{email})]        <!-- email -->
[(#SESSION{statut})]       <!-- 0minirezo (admin), 1comite (editor), 6forum (visitor) -->

<!-- Conditional block for logged-in users only -->
<BOUCLE_connecte(CONDITION){si #SESSION{id_auteur}}> 
  Bonjour [(#SESSION{nom})] !
</BOUCLE_connecte>
<B_connecte>
  <a href="#URL_PAGE{login}">Connexion</a>
</B_connecte>
```

---

## Quick Decision Guide

| Need | Use |
|------|-----|
| Reuse a template block | `<INCLURE{fond=...}>` |
| Reusable block, optional (no error if missing) | `[(#INCLURE{fond=...})]` |
| Make a block reload without full page refresh | Add `{ajax}` to INCLURE |
| Insert a template from article body text | Modèle in `squelettes/modeles/` |
| Show a login/registration/search/comment form | `#FORMULAIRE_*` balise |
| Different template per article or rubrique | Variant files (`article-42.html`) |
| Custom "page not found" | `squelettes/404.html` |
| Control caching | `#CACHE{seconds}` |
| Read URL param or INCLURE argument | `#ENV{key}` |
| Store a computed value for reuse in same file | `#SET` / `#GET` |
