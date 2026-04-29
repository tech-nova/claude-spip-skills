# paquet.xml Reference (SPIP 4.1+)

Required at the root of every plugin directory.

## `<paquet>` Attributes

| Attribute | Required | Values / Notes |
|---|---|---|
| `prefix` | Yes | Unique identifier, lowercase, no spaces (e.g. `monplugin`) |
| `categorie` | Yes | `outil` `contenu` `communication` `navigation` `statistique` `spam` `maintenance` `squelette` `dist` |
| `version` | Yes | Semver string: `1.2.3` |
| `etat` | Yes | `dev` `test` `stable` |
| `compatibilite` | Yes | SPIP version range: `[4.1.0;4.99.99]` or `[4.1.0;]` for open-ended |
| `schema` | No | Integer — increment to trigger `_administrations.php` on next activation |
| `logo` | No | Relative path to logo image (e.g. `monplugin_logo.png`) |
| `documentation` | No | URL to online documentation |
| `nom_fichier_lang` | No | Override default lang file prefix (default: `{prefix}`) |

## Child Elements

### `<nom>`
Human-readable name shown in the plugin manager.
```xml
<nom>Mon Plugin</nom>
```

### `<auteur>`
Repeatable. Optional `lien` attribute for author URL.
```xml
<auteur lien="https://example.com">Jean Dupont</auteur>
```

### `<licence>`
```xml
<licence lien="https://www.gnu.org/licenses/gpl.html">GNU/GPL</licence>
```

### `<necessite>`
Hard dependency — plugin will not activate without it.
```xml
<necessite nom="spip" compatibilite="[4.1.0;]" />
<necessite nom="saisies" compatibilite="[4.0.0;]" />
```

### `<utilise>`
Soft dependency — activates the named plugin if installed, but not required.
```xml
<utilise nom="mots" compatibilite="[3.0.0;]" />
```

### `<pipeline>`
Declares a pipeline hook. `inclure` is the file containing the handler function.
Handler name must be: `{prefix}_{nom}` (e.g. `monplugin_post_edition`).
```xml
<pipeline nom="post_edition"    inclure="monplugin_pipelines.php" />
<pipeline nom="affichage_final" inclure="monplugin_pipelines.php" />
```

### `<credit>`
Third-party library attribution.
```xml
<credit lien="https://library.example.com">SomeLibrary v2.3</credit>
```

## Full Annotated Example

```xml
<paquet prefix="monplugin"
        categorie="outil"
        version="1.2.0"
        etat="stable"
        compatibilite="[4.1.0;4.99.99]"
        schema="3"
        logo="monplugin_logo.png"
        documentation="https://contrib.spip.net/monplugin">

  <nom>Mon Plugin</nom>
  <auteur lien="https://example.com">Jean Dupont</auteur>
  <licence lien="https://www.gnu.org/licenses/gpl.html">GNU/GPL</licence>

  <necessite nom="spip" compatibilite="[4.1.0;]" />
  <necessite nom="saisies" compatibilite="[4.0.0;]" />
  <utilise nom="mots" />

  <pipeline nom="pre_edition"     inclure="monplugin_pipelines.php" />
  <pipeline nom="post_edition"    inclure="monplugin_pipelines.php" />
  <pipeline nom="affichage_final" inclure="monplugin_pipelines.php" />

  <credit lien="https://vendor.example.com">Vendor Lib v1.0</credit>
</paquet>
```
