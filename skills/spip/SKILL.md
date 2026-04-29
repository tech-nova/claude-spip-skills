---
name: spip
description: Use when working in a SPIP squelettes folder, writing BOUCLE loops,
  balises, critères or filtres, or when the user asks about building a SPIP website
  template. Not for plugin development — use spip-plugins for that.
---

# SPIP — Template Reference (SPIP 4.1+)

SPIP generates pages from **squelettes** — `.html` files mixing static markup with BOUCLE loops and `#BALISE` tags. All template work lives in `squelettes/`. No PHP needed.

## BOUCLE Syntax

```html
<BOUCLE_name(TABLE){critère1}{critère2}>
  ...repeated for each result...
</BOUCLE_name>
<//B_name>  <!--[ optional: shown when loop returns 0 results ]-->
```

**Examples:**

```html
<!-- Last 10 articles in current rubrique, newest first, paginated -->
<BOUCLE_articles(ARTICLES){id_rubrique}{par date}{inverse}{pagination 10}>
  <a href="#URL_ARTICLE">#TITRE</a> — #DATE
</BOUCLE_articles>
[(#PAGINATION)]

<!-- Nested: all top-level rubriques then their articles -->
<BOUCLE_rubriques(RUBRIQUES){racine}>
  <h2>#TITRE</h2>
  <BOUCLE_arts(ARTICLES){id_rubrique}{par titre}>
    <p><a href="#URL_ARTICLE">#TITRE</a></p>
  </BOUCLE_arts>
</BOUCLE_rubriques>
```

## Balises & Conditions

```html
#TITRE                            <!-- direct output -->
[(#TITRE)]                        <!-- optional: nothing if empty -->
[(#TITRE|majuscules)]             <!-- with filter -->
[<b>(#TITRE)</b>]                 <!-- surrounding markup only if tag non-empty -->
[(#LOGO_ARTICLE|image_reduire{200})]
```

## Key Critères

| Critère | Effect |
|---|---|
| `{id_rubrique}` | Articles in current rubrique (inherited from context) |
| `{id_rubrique=3}` | Articles in rubrique #3 |
| `{par titre}` | Sort ascending by title |
| `{par date}{inverse}` | Sort newest first |
| `{limit 5}` | Max 5 results |
| `{pagination 10}` | 10 per page — add `[(#PAGINATION)]` for page links |
| `{statut=publie}` | Published only |
| `{tout}` | All statuses (including drafts) |
| `{lang}` | Current language only |
| `{doublons}` | Skip items already displayed by another loop |

Full reference: `references/balises-criteres.md` · `references/filtres.md`
