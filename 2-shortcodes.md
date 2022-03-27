# 2. De verjaardagen-plugin maken

In deze pagina ga je een pagina maken waarop je later de verjaardagen zult tonen. Je maakt kennis met de basis van een plugin, en je eerste action hooks.

## A. De plugin registreren

1. Ga naar de folder die je hebt gemaakt voor de verjaardagen-plugin (`jio-birthdays`) en maak daarin een nieuw bestand genaamd `jio-birthdays.php`. Begin de plugin met de volgende _plugin header_:

```php
<?php
/**
 * Plugin Name: JIO Birthdays
 * Author: <Jouw naam hier>
 * Version: 0.0.1
 */
```

> Zie [Header Requirements](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/) voor meer informatie over de plugin header.

2. In de Wordpress Admin Panel, ga naar de pagina [Plugins](http://localhost:8888/wp-admin/admin.php?page=plugins). Activeer jouw plugin.

**Checkpoint**: Controleer dat de plugin is geactiveerd.

## B. Een shortcode aanmaken

**Doel**: maak een _shortcode_ aan waarmee je de verjaardagen kunt tonen op de website.

> Een _shortcode_ is een soort tag die je kunt plaatsen binnen een pagina op je website. Bij het renderen van de pagina wordt de shortcode dan gerenderd door jouw functie. Een voorbeeld van een shortcode is `[jio-birthdays]`.

1. Maak een functie `jio_render_shortcode`. Laat deze functie een stuk tekst returnen, zodat je kunt testen dat je shortcode straks werkt.
2. Maak een functie `jio_register_shortcode`, waarin je een shortcode `jio-birthdays` toevoegt aan Wordpress met [`add_shortcode`](https://developer.wordpress.org/reference/functions/add_shortcode/). Deze moet de functie `jio_render_shortcode` aanroepen.
3. Tot slot, zorg dat `jio_register_shortcode` wordt aangeroepen gedurende de `init` action.

> Een _action_ is een gebeurtenis in Wordpress, waarop je kunt inhaken m.b.v. de functie [`add_action`](https://developer.wordpress.org/reference/functions/add_action/). Zo'n functie noem je een _action hook_.

**Checkpoint**: Maak een pagina aan met de inhoud `[jio-birthdays]`. Als alles goed is gegaan, zie je jouw test-output als je deze pagina bezoekt.

<details>
    <summary>Oplossing (klik om te openen)</summary>
    
```php
...

function jio_render_shortcode() {
    return "Hello world!";
}

function jio_register_shortcode() {
    add_shortcode("jio_birthdays", "jio_render_shortcode");
}

add_action("init", "jio_register_shortcode");
```
    
</details>

## C. Enkele verjaardagen tonen.

**Doel**: zorg dat de shortcode een lijst met verjaardagen toont.

1. Maak een functie genaamd `jio_get_birthdays` die alvast wat dummy-data teruggeeft. Gebruik de class hieronder.

```php

// Create with: new JioBirthday('name', 'date-of-birth');
class JioBirthday {
    public $name;
    public $birthday;
    function __construct($name, $birthday) {
        $this->name = $name;
        $this->birthday = $birthday;
    }
}
```

2. Zorg dat `jio_render_shortcode` de verjaardagen weergeeft in een lijst.
3. Zorg dat je de naam en verjaardag "sanitized" door middel van de `esc_html` functie. 

> **Let op**: Wanneer je in PHP data van gebruikers op het scherm toont, moet je deze *altijd* sanitizen met bijpassende functies! Doe je dat niet, dan open je de deur voor cross-site scripting (XSS) aanvallen. In de meeste gevallen gebruik je `esc_html`, maar er bestaan ook `esc_attr` (voor html attributes), `esc_url` en `esc_textarea`.

**Checkpoint**: Controleer de output van de shortcode.

<details>
    <summary>Oplossing (klik om te openen)</summary>

```php
...

class JioBirthday {
    public $name;
    public $birthday;
    function __construct($name, $birthday) {
        $this->name = $name;
        $this->birthday = $birthday;
    }
}

function jio_get_birthdays() {
    return [
        new JioBirthday("Theo", "1990-01-01"),
        new JioBirthday("Netty", "1972-03-12")
    ];
}

function jio_render_shortcode() {
    $html = "<ul>";
    foreach (jio_get_birthdays() as $record) {
        $html .= sprintf(
            "<li>%s: %s<li>",
            esc_html($record->name),
            esc_html($record->birthday)
        );
    }
    $html .= "</ul>";
    return $html;
}
```
    
</details>
