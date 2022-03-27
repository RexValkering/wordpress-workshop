# 4. Een plugin-menupagina maken

Je hebt nu een pagina die een lijst met verjaardagen toont, en waarvanuit de verjaardagen uit de database worden geladen.
Hoe zorg je ervoor dat een beheerder deze kan bijwerken?

## A. Een nieuwe menu-pagina maken.

Ga verder in het bestand `jio-birthdays.php`.

1. Maak een functie `jio_render_admin_page`. Dit is waar je straks je formulieren plaatst. Zorg voor nu dat je een simpele test-tekst print (`echo "hello world";`).
2. Maak een functie `jio_register_menu_page` en zorg dat deze aangeroepen wordt gedurende de action `admin_menu`.
3. Zorg dat `jio_register_menu_page` een pagina toevoegt aan het Wordpress admin menu met de functie [`add_menu_page`](https://developer.wordpress.org/reference/functions/add_menu_page/).

**Checkpoint**: Controleer dat er een pagina is in het admin menu met als inhoud jouw test-tekst.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_render_admin_page() {
    echo "Hello world!";
}

function jio_register_menu_page() {
  add_menu_page( 'JIO Birthdays', 'JIO Verjaardagen', 'manage_options', 'jio-birthdays', 'jio_render_admin_page');
}
add_action('admin_menu', 'jio_register_menu_page');
```

</details>
    
## B. Maak een formulier aan.

1. In de functie `jio_render_admin_page`, voeg een simpel formulier toe met twee velden: `name` en `birthday`. Voeg ook een submit-knop toe.
2. Wanneer het formulier gesubmit wordt, zorg dat er een POST wordt gedaan naar de huidige pagina.

**Checkpoint**: Controleer dat je het formulier kunt submitten, en dat je dan terecht komt op de huidige pagina.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_render_admin_page() {
    ?>
    <form action="?page=jio-birthdays" method="POST">
        <label id="jio-name-label" for="jio-name">Name *</label>
        <input id="jio-name" name="name" aria-labelledby="jio-name-label" />

        <label id="jio-birthday-label" for="jio-birthday">Birthday *</label>
        <input id="jio-birthday" name="birthday" aria-labelledby="jio-birthday-label" />

        <input name="submit" type="submit" />
    </form>
    <?php
}
```

</details>

## C. Het formulier verwerken.

1. Voeg bovenaan de functie `jio_render_admin_page` een stuk logica toe dat controleert of er een POST-actie is uitgevoerd.
2. Gebruik [`$wpdb->insert`](https://developer.wordpress.org/reference/classes/wpdb/insert/) om een nieuwe entry toe te voegen aan de database tabel.
3. Aan het eind van de functie, redirect de gebruiker naar de huidige pagina.

> Een gebruiker redirecten nadat een wijziging is doorgevoerd staat bekent als [Post/Redirect/Get](https://en.wikipedia.org/wiki/Post/Redirect/Get). Deze best-practice voorkomt dat een actie nogmaals wordt uitgevoerd als de gebruiker de pagina herlaadt.

**Checkpoint**: Controleer dat je een verjaardag kunt toevoegen via deze admin-pagina.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_render_admin_page() {
    if (isset($_POST["submit"])) {
        $name = $_POST["name"] ?? null;
        $birthday = $_POST["birthday"] ?? null;
        if (!$name || !$birthday) {
            wp_redirect("?page=jio-birthdays&status=error");
            exit;
        }

        $result = $wpdb->insert("{$wpdb->prefix}jio_birthdays", ["name" => $name, "birthday" => $birthday], ["%s", "%s"]);

        $redirect_page = "?page=jio-birthdays&status=" . ($result ? "success" : "error");
        wp_redirect($redirect_page);
        exit;
    }
    ?>

    <form action="?page=jio-birthdays" method="POST">
        <label id="jio-name-label" for="jio-name">Name *</label>
        <input id="jio-name" name="name" aria-labelledby="jio-name-label" />

        <label id="jio-birthday-label" for="jio-birthday">Birthday *</label>
        <input id="jio-birthday" name="birthday" aria-labelledby="jio-birthday-label" />

        <input name="submit" type="submit" />
    </form>
    <?php
}
```

</details>
