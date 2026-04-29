# SPIP Skill — Baseline Test (RED)

Answers given using training knowledge only, before any skill file exists.

---

## Q1: BOUCLE listing last 5 published articles from rubrique #3, sorted by date descending

**Answer:**
```html
<BOUCLE_articles(ARTICLES){id_rubrique=3}{statut=publie}{par date}{inverse}{limit 5}>
  <a href="#URL_ARTICLE">#TITRE</a>
</BOUCLE_articles>
```

**Assessment:** ✅ Correct BOUCLE syntax, correct table alias, correct critères, correct ordering.

---

## Q2: Display #TITRE only when not empty

**Answer:**
```html
[(#TITRE)]
```

**Assessment:** ✅ Used correct `[(#BALISE)]` optional syntax.

---

## Q3: Critère for pagination and balise for page links

**Answer:**
- Critère: `{pagination 10}` (where 10 is items per page)
- Balise: `[(#PAGINATION)]`

**Assessment:** ✅ Both `{pagination N}` and `[(#PAGINATION)]` mentioned correctly.

---

## Q4: Apply image_reduire filter (max 300px) to #LOGO_ARTICLE

**Answer:**
```html
[(#LOGO_ARTICLE|image_reduire{300})]
```

**Assessment:** ✅ Correct filter syntax.

---

## Summary

All four questions answered correctly from training knowledge. No non-SPIP patterns (Twig, PHP, generic templating) were used. BOUCLE syntax, optional balise syntax, pagination pair, and filter syntax are all accurate.

**Baseline verdict:** Claude already has reasonable SPIP template knowledge without a dedicated skill. The skill will add value primarily for: complete critères catalog, full balise lists per table, and the full filtres reference — things that are hard to recall exhaustively from training alone.
