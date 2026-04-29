# SPIP Pipelines Catalog (SPIP 4.1+)

Declare in `paquet.xml`: `<pipeline nom="name" inclure="file.php" />`
Handler name: `{prefix}_{name}($flux)` — always `return $flux`.

## Content Display

| Pipeline | `$flux['data']` | Use for |
|---|---|---|
| `affichage_final` | Final HTML string of the page | Modify or inject into full page output |
| `affichage_entetes_final` | HTTP headers array | Add/modify response headers |
| `recuperer_fond` | Rendered HTML of a single fond (template) | Modify a specific template's output |
| `styliser` | Template/style selection data | Override which squelette is used |

```php
function monplugin_affichage_final($flux) {
    $flux['data'] = str_replace('</body>', '<p>injected</p></body>', $flux['data']);
    return $flux;
}
```

## Database Events

| Pipeline | Key `$flux` keys | Use for |
|---|---|---|
| `pre_insertion` | `$flux['data']` = field array; `$flux['args']['table']` | Modify data before INSERT |
| `post_insertion` | `$flux['args']['id_objet']`; `$flux['args']['table']` | Act after INSERT (send email, create related object) |
| `pre_edition` | `$flux['data']` = changed fields; `$flux['args']['table']` | Modify data before UPDATE |
| `post_edition` | `$flux['args']['id_objet']`; `$flux['args']['table']`; `$flux['args']['action']` | Act after UPDATE |
| `post_edition_lien` | `$flux['args']['objet']`, `$flux['args']['id_objet']` | Act after a link between objects is created/removed |

`$flux['args']['action']` values for `post_edition`: `modifier` (field edit), `instituer` (status change).

```php
function monplugin_post_edition($flux) {
    if ($flux['args']['table'] === 'spip_articles'
        && $flux['args']['action'] === 'instituer') {
        // article status changed — notify, clear cache, etc.
        $id = $flux['args']['id_objet'];
    }
    return $flux;
}
```

## CVT Forms

| Pipeline | `$flux['args']['form']` | Use for |
|---|---|---|
| `formulaire_charger` | Form name string | Add fields to any existing CVT form |
| `formulaire_verifier` | Form name string | Add validation rules to any CVT form |
| `formulaire_traiter` | Form name string | Add processing to any CVT form |
| `formulaire_fond` | Form name string | Override which template is used for a form |

## Admin Interface

| Pipeline | Use for |
|---|---|
| `ajouter_boutons` | Add buttons to admin top navigation bar |
| `ajouter_onglets` | Add tabs to admin object edit pages |
| `ajouter_menus` | Add entries to admin menus |
| `affiche_gauche` | Add content to left column in admin |
| `affiche_droite` | Add content to right column in admin |
| `affiche_milieu` | Add content to main area in admin |
| `affiche_enfants` | Add content to child-objects panel |

## Head & Assets

| Pipeline | Use for |
|---|---|
| `insert_head` | Add CSS/JS to public `<head>` |
| `insert_head_css` | Add CSS only to public `<head>` |
| `header_prive` | Add CSS/JS to admin `<head>` |

## Database Schema

| Pipeline | Use for |
|---|---|
| `declarer_tables_objets_sql` | Declare a new SPIP object type with its table |
| `declarer_tables_principales` | Declare additional primary tables |
| `declarer_tables_auxiliaires` | Declare auxiliary/join tables |

## Cron, Authorization & Other

| Pipeline | Use for |
|---|---|
| `taches_generales_cron` | Register background cron tasks |
| `autoriser` | Add/modify authorization rules for actions |
| `rechercher_objet_type` | Extend SPIP search to custom object types |
| `objet_compte_enfants` | Count child objects for admin display |

## Custom Pipelines

Create your own pipeline callable by other plugins:

```php
// In your plugin code
$result = pipeline('mon_pipeline_custom', $initial_data);
```

Other plugins declare in their `paquet.xml`:
```xml
<pipeline nom="mon_pipeline_custom" inclure="their_pipelines.php" />
```
And define `theirplugin_mon_pipeline_custom($flux)`.
