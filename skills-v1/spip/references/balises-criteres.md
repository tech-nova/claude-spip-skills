# SPIP Balises & Critères Reference (SPIP 4.1+)

## BOUCLE Table Types

| Alias | Content |
|---|---|
| `ARTICLES` | Articles |
| `RUBRIQUES` | Sections/categories |
| `AUTEURS` | Authors |
| `DOCUMENTS` | Attached files and media |
| `FORUMS` | Forum messages |
| `MOTS` | Keywords |
| `GROUPES_MOTS` | Keyword groups |
| `SYNDICATION` | Syndicated news items |
| `SITES` | Syndicated sites |
| `SIGNATURES` | Petition signatures |
| `REFERENCES` | Bibliographic references |
| `VISITES` | Visit statistics |

Real table name also works: `(spip_articles)` instead of `(ARTICLES)`.

## Critères Catalog

### Filtering

| Critère | Description |
|---|---|
| `{id_article=5}` | Specific article by ID |
| `{id_rubrique}` | In current rubrique (inherited from context) |
| `{id_rubrique=3}` | In rubrique #3 |
| `{id_secteur}` | In current sector |
| `{id_auteur}` | By current author |
| `{id_mot=7}` | Tagged with keyword #7 |
| `{statut=publie}` | Published only |
| `{statut=prop}` | Proposed only |
| `{tout}` | All statuses |
| `{lang}` | Current language |
| `{lang=fr}` | French only |
| `{doublons}` | Exclude items already displayed by any loop |
| `{doublons articles}` | Exclude using named group "articles" |

### Sorting

| Critère | Description |
|---|---|
| `{par titre}` | Alphabetical by title |
| `{par date}` | By publication date |
| `{par date_redac}` | By editorial date |
| `{par date_modif}` | By modification date |
| `{par id_article}` | By ID |
| `{par hasard}` | Random order |
| `{inverse}` | Reverse current sort order |
| `{par num titre}` | Numeric sort on leading number in title (e.g. "3. Mon titre") |

### Pagination & Limit

| Critère | Description |
|---|---|
| `{limit 10}` | First 10 results |
| `{limit 5,10}` | Skip 5, take next 10 |
| `{pagination 10}` | 10 per page — use `[(#PAGINATION)]` to render page links |
| `{debut_articles}` | Use URL parameter `debut_articles` as offset |

### Context & Joins

| Critère | Description |
|---|---|
| `{id_rubrique?}` | Use rubrique from context if available, otherwise all |
| `{0,1}` | Return exactly 1 result without iterating |
| `{strict}` | Strict matching for joins |

## Built-in Balises by Table

### ARTICLES
`#ID_ARTICLE` `#TITRE` `#DESCRIPTIF` `#CHAPO` `#TEXTE` `#PS` `#NOTES`
`#DATE` `#DATE_REDAC` `#DATE_MODIF` `#DATE_NOUVEAUTE`
`#URL_ARTICLE` `#LOGO_ARTICLE` `#LOGO_ARTICLE_NORMAL`
`#ID_RUBRIQUE` `#ID_SECTEUR` `#NOM_SITE` `#URL_SITE`
`#LANG` `#STATUT` `#VU`
`[(#INTRODUCTION)]` — auto-generated from `#CHAPO` or start of `#TEXTE`

### RUBRIQUES
`#ID_RUBRIQUE` `#TITRE` `#DESCRIPTIF` `#TEXTE`
`#URL_RUBRIQUE` `#LOGO_RUBRIQUE` `#LOGO_RUBRIQUE_NORMAL`
`#ID_SECTEUR` `#ID_PARENT` `#LANG` `#STATUT`

### AUTEURS
`#ID_AUTEUR` `#NOM` `#BIO` `#EMAIL`
`#URL_AUTEUR` `#LOGO_AUTEUR` `#STATUT` `#LANG`

### DOCUMENTS
`#ID_DOCUMENT` `#TITRE` `#DESCRIPTIF`
`#FICHIER` `#URL_DOCUMENT` `#MIME_TYPE` `#TAILLE`
`#LARGEUR` `#HAUTEUR` `#EXTENSION` `#DATE`
`#LOGO_DOCUMENT`

### MOTS
`#ID_MOT` `#TITRE` `#TEXTE` `#URL_MOT` `#ID_GROUPE` `#TYPE`

## Loop Patterns

### Alternative section (0 results)

```html
<BOUCLE_arts(ARTICLES){id_rubrique}>
  <li><a href="#URL_ARTICLE">#TITRE</a></li>
</BOUCLE_arts>
<p>Aucun article dans cette rubrique.</p>
<//B_arts>
```

Content between `</BOUCLE_arts>` and `<//B_arts>` shows only when loop returns 0 results.

### Recursive loop (full rubrique tree)

```html
<!-- inc-rubriques.html — called recursively -->
<BOUCLE_sousrub(RUBRIQUES){id_parent}{tout}>
  <li>#TITRE
    <ul><INCLURE{fond=inc-rubriques,id_rubrique=#ID_RUBRIQUE} /></ul>
  </li>
</BOUCLE_sousrub>
```

### Avoiding duplicates across loops

```html
<BOUCLE_vedette(ARTICLES){id_rubrique}{limit 1}>
  <h2>#TITRE</h2>  <!-- featured article -->
</BOUCLE_vedette>

<BOUCLE_reste(ARTICLES){id_rubrique}{doublons}{limit 9}>
  <p>#TITRE</p>   <!-- remaining articles, skips the one above -->
</BOUCLE_reste>
```
