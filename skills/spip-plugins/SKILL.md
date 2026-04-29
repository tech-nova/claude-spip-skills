---
name: spip-plugins
description: Use when developing a SPIP plugin (paquet.xml present, _pipelines.php,
  spip_* table references in PHP) or when the user asks about plugin architecture,
  pipelines, hooks, or the SPIP PHP/SQL API.
---

# SPIP Plugins — Developer Reference (SPIP 4.1+)

A SPIP plugin is a directory in `plugins/` with a `paquet.xml` manifest. Plugins extend SPIP via pipelines (event hooks), add squelettes, define CVT forms, and version the DB schema via `_administrations.php`.

## Plugin File Map

| File | Purpose |
|---|---|
| `paquet.xml` | Manifest — required |
| `options.php` | Loaded on every page — constants, global config |
| `fonctions.php` | Custom balises/filtres available in squelettes |
| `monplugin_pipelines.php` (or `_pipelines.php`) | Pipeline handler functions |
| `_administrations.php` | DB schema creation/upgrades (versioned by `schema=` in paquet.xml) |
| `lang/monplugin_fr.php` | i18n strings |
| `formulaires/mon_form.html` + `.php` | CVT form (template + logic) |
| `squelettes/` | Templates bundled with the plugin |

## Minimal paquet.xml

```xml
<paquet prefix="monplugin"
        categorie="outil"
        version="1.0.0"
        etat="stable"
        compatibilite="[4.1.0;4.99.99]"
        schema="1">
  <nom>Mon Plugin</nom>
  <auteur>Nom Auteur</auteur>
  <licence>GNU/GPL</licence>
  <necessite nom="spip" compatibilite="[4.1.0;]" />
  <pipeline nom="post_edition" inclure="monplugin_pipelines.php" />
</paquet>
```

## Pipeline Pattern

```php
// In monplugin_pipelines.php
// Handler name = {prefix}_{pipeline_nom}
function monplugin_post_edition($flux) {
    if ($flux['args']['table'] === 'spip_articles') {
        $id = $flux['args']['id_objet'];
        // act on article #$id after save
    }
    return $flux; // always return $flux
}
```

Common hooks: `pre_edition` `post_edition` `pre_insertion` `post_insertion` `affichage_final` `formulaire_charger` `formulaire_verifier` `formulaire_traiter`

## CVT Form Skeleton

```php
// formulaires/mon_form.php
function formulaires_mon_form_charger_dist() {
    return ['champ' => _request('champ') ?: ''];
}
function formulaires_mon_form_verifier_dist() {
    $erreurs = [];
    if (!_request('champ')) {
        $erreurs['champ'] = _T('monplugin:champ_requis');
    }
    return $erreurs;
}
function formulaires_mon_form_traiter_dist() {
    // sql_insert / sql_updateq here
    return ['editable' => false];
}
```

Full reference: `references/paquet-xml.md` · `references/pipelines-catalog.md` · `references/api.md`

For squelette/template work inside a plugin, see the `spip` skill.
