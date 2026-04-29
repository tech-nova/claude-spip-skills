# SPIP PHP/SQL API Reference (SPIP 4.1+)

Never use raw `mysqli_*` or PDO directly. Always go through SPIP's SQL abstraction layer.

## SQL API

### `sql_select` — query returning a resource

```php
$res = sql_select(
    $select,   // string or array: 'id_article, titre' or ['id_article', 'titre']
    $from,     // string: 'spip_articles'
    $where,    // string or array: "statut='publie'" or ["statut='publie'", "lang='fr'"]
    $groupby,  // string: '' or 'id_rubrique'
    $orderby,  // string: 'date DESC'
    $limit,    // string: '10' or '5,10'
    $having,   // string: ''
    $serveur   // string: '' for default DB
);
while ($row = sql_fetch($res)) {
    echo $row['titre'];
}
```

### `sql_fetch` — next row from resource

```php
$row = sql_fetch($res); // array or false when exhausted
```

### `sql_allfetsel` — select + fetch all rows at once

```php
$rows = sql_allfetsel(
    'id_article, titre, date',
    'spip_articles',
    "statut='publie'",
    '',         // groupby
    'date DESC',
    '10'        // limit
);
// Returns: [['id_article'=>1,'titre'=>'...'], ...]
```

### `sql_fetsel` — fetch single row

```php
$row = sql_fetsel('titre, texte, date', 'spip_articles', "id_article=" . intval($id));
// Returns: ['titre'=>'...', 'texte'=>'...'] or false
```

### `sql_getfetsel` — fetch single field value

```php
$titre = sql_getfetsel('titre', 'spip_articles', "id_article=" . intval($id));
// Returns: 'Mon titre' or false
```

### `sql_insert` — INSERT a row

```php
$new_id = sql_insert('spip_articles', [
    'titre'   => 'Nouveau titre',
    'statut'  => 'prepa',
    'lang'    => 'fr',
    'id_rubrique' => 3,
]);
```

### `sql_updateq` — UPDATE with auto-quoting

```php
sql_updateq('spip_articles',
    ['titre' => 'Titre modifié', 'statut' => 'publie'],
    "id_article=$id"
);
```

### `sql_delete` — DELETE rows

```php
sql_delete('spip_articles', "id_article=$id");
```

### `sql_count` — count rows

```php
$n = sql_count(sql_select('id_article', 'spip_articles', "statut='publie'"));
```

### `sql_quote` — quote a value for WHERE clause

```php
$safe = sql_quote($user_input);
$where = "titre=" . sql_quote($titre);
```

### `sql_in` — generate IN(...) clause

```php
$where = sql_in('id_article', [1, 2, 3]); // "id_article IN (1,2,3)"
$where = sql_in('id_article', [1, 2, 3], 'NOT'); // "id_article NOT IN (1,2,3)"
```

## Utility Functions

### `_request($key, $default = '')`

Safe GET+POST accessor. Always use instead of `$_GET`/`$_POST`.

```php
$id    = intval(_request('id_article'));
$titre = _request('titre', 'Sans titre');
```

### `include_spip($fichier)`

Include a SPIP file found via `find_in_path`. Returns true on success.

```php
include_spip('inc/securiser_action');
include_spip('inc/acces'); // required before using securiser_acces_low_sec()
```

### `find_in_path($fichier, $dossier = '', $all = false)`

Locate a file across SPIP's load path (plugins, squelettes, SPIP core).

```php
$path = find_in_path('mes_fonctions.php');
$all  = find_in_path('style.css', 'squelettes/', true); // array of all matches
```

### `pipeline($nom, $valeur)`

Execute a pipeline and return the (possibly modified) value.

```php
$html = pipeline('affichage_final', $html);
```

### `_T($cle)` — translate

```php
echo _T('monplugin:message_confirmation');
// Reads from lang/monplugin_fr.php: $GLOBALS['i18n']['message_confirmation']
```

## Cache Invalidation

```php
include_spip('inc/invalideur');

// Invalidate cache for a specific object
suivre_invalideur("id_article=" . intval($id));

// List current invalideurs (see inc/invalideur for full invalidation API)
lister_invalideurs();
```

## Security

### `securiser_acces_low_sec()` (SPIP 4.1+)

Defined in `inc/acces`. Creates a fixed token for an action/author pair. The URL is accessible even when the author is disconnected — anyone with the URL can trigger the action. Use only for low-security actions.

```php
include_spip('inc/acces');
$token_url = securiser_acces_low_sec($id_auteur, 'mon_action', 'param=val');
```

> `securiser_acces()` from SPIP <4.1 is **deprecated**. A `filtre_securiser_acces_dist()` shim exists in `inc/filtres` for backward compatibility only — do not use it in new code.

### Always use `_request()` over raw superglobals

`_request()` merges GET+POST and applies SPIP input filtering. Direct access to `$_GET`/`$_POST` bypasses this and is a security risk in plugin code.

### CVT forms handle CSRF automatically

SPIP's built-in CVT form system (`formulaires/`) includes CSRF token verification. Use it for all user-facing forms instead of rolling your own nonce logic.
