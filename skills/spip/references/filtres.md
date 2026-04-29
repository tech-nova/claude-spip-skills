# SPIP Filtres Reference (SPIP 4.1+)

Syntax: `[(#BALISE|filtre)]` or `[(#BALISE|filtre{arg})]`
Chain: `[(#TEXTE|couper{200}|propre)]`

## Text

| Filtre | Description |
|---|---|
| `|majuscules` | UPPERCASE |
| `|minuscules` | lowercase |
| `|ucfirst` | First letter uppercase |
| `|couper{150}` | Truncate to ~150 chars, word-aware |
| `|propre` | Convert SPIP markup to clean HTML |
| `|texte_raccourci` | Short text, strips HTML |
| `|entites_html` | Escape HTML entities |
| `|supprimer_tags` | Strip all HTML tags |
| `|typo` | Apply French typographic rules (non-breaking spaces, guillemets) |
| `|liens_ouverts` | Open external links in new tab |
| `|sinon{Valeur par défaut}` | Output fallback string if balise is empty |
| `|strlen` | Return string length |

## Date

| Filtre | Description |
|---|---|
| `|affdate` | Full localized date: "1er janvier 2024" |
| `|affdate_court` | Short localized date: "01/01/24" |
| `|affdate_mois_annee` | Month and year: "janvier 2024" |
| `|affdate{%d %B %Y}` | Custom strftime format |
| `|date_iso` | ISO 8601: "2024-01-01T00:00:00" |
| `|timestamp` | Unix timestamp integer |
| `|annee` | Year: "2024" |
| `|mois` | Month number: "01" |
| `|jour` | Day number: "01" |
| `|heures` | Hours: "14" |
| `|minutes` | Minutes: "30" |

## Image & Media

| Filtre | Description |
|---|---|
| `|image_reduire{300}` | Resize to max 300px wide or tall (preserves ratio) |
| `|image_reduire{300,200}` | Resize to fit within 300×200px box |
| `|image_passe_partout{300,200}` | Pad/letterbox to exact 300×200px |
| `|image_recadrer{300,200}` | Crop to exact 300×200px (center anchor) |
| `|image_recadrer{300,200,center,top}` | Crop with explicit anchor |
| `|image_nb` | Convert to greyscale |
| `|image_sepia` | Convert to sepia tone |
| `|image_rotation{90}` | Rotate 90° clockwise |
| `|image_flip_vertical` | Flip vertically |
| `|image_flip_horizontal` | Flip horizontally |

## URL

| Filtre | Description |
|---|---|
| `|url_absolue` | Convert relative URL to absolute (with domain) |
| `|attribut_html` | Escape for safe use in HTML attribute value |
| `|parametre_url{cle,valeur}` | Add or replace a URL query parameter |
| `|nettoyer_uri` | Sanitize URI string |

## Logic

| Filtre | Description |
|---|---|
| `|oui` | Output value only if truthy |
| `|non` | Output value only if falsy |
| `|vide` | True (1) if empty |
| `|!vide` | True (1) if not empty |
| `|array_count` | Count array elements |
| `|intval` | Cast to integer |

## Custom Filtres

Define in `squelettes/mes_fonctions.php` (auto-loaded by SPIP, no plugin needed):

```php
// callable as [(#TITRE|mon_filtre)] or [(#TITRE|mon_filtre{arg})]
function filtre_mon_filtre_dist($valeur, $arg = '') {
    return strtoupper($valeur) . $arg;
}
```

Use: `[(#TITRE|mon_filtre{!})]` → "MON TITRE!"
