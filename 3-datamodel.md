# 3. Een databasetabel maken

In deze sectie ga je een tabel maken in de database om de verjaardagen in op te slaan. Hierbij maak je gebruik van de `$wpdb` en `dbDelta` om je tabellen te benaderen en aan te maken.

## A. Een nieuw bestand maken

Het is verstandig om verschillende onderdelen van je plugin in aparte bestanden te zetten, zodat het overzichtelijk blijft.

1. Maak een bestand genaamd `datamodel.php` aan.
2. Voeg hier alvast de volgende snippet aan toe. Bestudeer hem goed zodat je begrijpt wat er gebeurt.

```php
<?php

class JioDatamodel {
    public static $version = 0.00;
    public static $version_key = "jio_db_version";

    public static function check_database() {
        if (static::$version != get_option(static::$version_key)) {
            static::update_database();
        }
    }

    private static function update_database() {
        global $wpdb;

        // Get the default collate
        $charset_collate = '';
        if (!empty($wpdb->charset)) {
            $charset_collate = "DEFAULT CHARACTER SET {$wpdb->charset}";
        }
        if (!empty($wpdb->collate)) {
            $charset_collate .= " COLLATE {$wpdb->collate}";
        }

        // TODO: Define new SQL table
        $table_name = $wpdb->prefix() . "jio_birthdays";
        $sql = "";

        // Let dbDelta update our database definitions
        require_once ABSPATH . 'wp-admin/includes/upgrade.php';
        dbDelta($sql);

        // Update the stored plugin database version
        update_option(static::$version_key, static::$version);
    }
}

add_action('plugins_loaded', 'JioDatamodel::check_database');
```

3. Roep het bestand aan bovenaan je hoofd-pluginfile (`jio-birthdays.php`).

```php
<?php
require_once("./datamodel.php");
```

## B. Een databasetabel maken.

1. Vul de parameter `$sql` met geldige SQL voor het definieren van een databasetabel voor verjaardagen. Zorg dat deze tabel drie kolommen heeft: `id` van type `integer(11)`, `name` van type `varchar(100)`, birthday van type `date`. Een voorbeeld van een geldige `$sql` is als volgt:

```php
$sql = "CREATE TABLE $table_name (
    id integer(11) NOT NULL AUTO_INCREMENT,
    <second column definition>,
    <third column definition>,
    PRIMARY KEY  id
) $charset_collage;";
```

> Let op: De 2 spaties tussen `KEY` en `id` zijn verplicht.

2. Hoog de versie op van `0.00` naar `0.01` en ververs een pagina binnen je Wordpress om de upgrade te doen.

```sql
        ...
        $table_name = $wpdb->prefix() . "jio_birthdays";
        $sql = "CREATE TABLE $table_name (
            id integer(11) NOT NULL AUTO_INCREMENT,
            name varchar(100),
            birthday date,
            PRIMARY KEY  id
        ) $charset_collage;";
```

**Checkpoint**: Controleer in phpmyadmin (`http://localhost:8889`) dat de tabel `wp_jio_birthdays` is aangemaakt.

## C. De databasetabel vullen met data.

1. Ga naar phpmyadmin en ga naar de tabel die je zojuist hebt aangemaakt.
2. Voeg enkele verjaardagen toe via de phpmyadmin interface.
3. Ga terug naar `jio_birthdays.php`. Vervang de inhoud van de functie `jio_get_birthdays` door de verjaardagen uit je database. Gebruik hiervoor de `$wpdb` global. Je mag de class `JioBirthday` weggooien.

**Checkpoint**: Controleer dat de verjaardagen op je pagina overeenkomen met wat je in de database hebt gezet.

```php
function jio_get_birthdays() {
    global $wpdb;
    return $wpdb->get_results("SELECT * FROM {$wpdb->prefix}jio_birthdays");
}
```
