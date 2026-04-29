# spip Skill — GREEN Verification

## Answers

### Q1: BOUCLE listing last 5 published articles from rubrique #3, date descending

```html
<BOUCLE_articles(ARTICLES){id_rubrique=3}{statut=publie}{par date}{inverse}{limit 5}>
  <a href="#URL_ARTICLE">#TITRE</a> — #DATE
</BOUCLE_articles>
```

Sourced from: SKILL.md Key Critères table (`{id_rubrique=3}`, `{statut=publie}`, `{par date}{inverse}`, `{limit 5}`)

### Q2: Display #TITRE only when not empty

```html
[(#TITRE)]
```

Sourced from: SKILL.md Balises & Conditions section — `[(#TITRE)]` optional syntax.

### Q3: Pagination critère + page links balise

- Critère: `{pagination 10}` (or any N)
- Balise: `[(#PAGINATION)]` placed after the closing BOUCLE tag

Sourced from: SKILL.md Key Critères table — `{pagination 10}` row explicitly notes "add `[(#PAGINATION)]` for page links"

### Q4: image_reduire at 300px on #LOGO_ARTICLE

```html
[(#LOGO_ARTICLE|image_reduire{300})]
```

Sourced from: SKILL.md Balises & Conditions (shows `image_reduire{200}` pattern); filtres.md Image section (`|image_reduire{300}` described as "Resize to max 300px wide or tall").

## Verification Table

| Expected | Pass? |
|---|---|
| BOUCLE uses `(ARTICLES){id_rubrique=3}{statut=publie}{par date}{inverse}{limit 5}` | ✅ |
| Optional tag uses `[(#TITRE)]` syntax | ✅ |
| Mentions both `{pagination N}` AND `[(#PAGINATION)]` | ✅ |
| Filter syntax is `[(#LOGO_ARTICLE|image_reduire{300})]` | ✅ |

## Conclusion

**PASS** — All 4 checks pass. The skill correctly surfaces SPIP 4.1+ template syntax for all test questions. The SKILL.md quick reference table was sufficient for 3 of 4 questions; filtres.md confirmed the image filter syntax.
