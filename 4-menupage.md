# 4. Een plugin-menupagina maken

Je hebt nu een pagina die een lijst met verjaardagen toont, en waarvanuit de verjaardagen uit de database worden geladen.
Hoe zorg je ervoor dat een beheerder deze kan bijwerken?

## A. Een nieuwe menu-pagina maken.

Ga verder in het bestand `jio-birthdays.php`.

1. Maak een functie `jio_render_admin_page`. Dit is waar je straks je formulieren plaatst. Zorg voor nu dat je een simpele test-tekst print (`echo '<div class="wrap">hello world</div>';`).
2. Maak een functie `jio_register_menu_page` en zorg dat deze aangeroepen wordt gedurende de action `admin_menu`.
3. Zorg dat `jio_register_menu_page` een pagina toevoegt aan het Wordpress admin menu met de functie [`add_menu_page`](https://developer.wordpress.org/reference/functions/add_menu_page/).

**Checkpoint**: Controleer dat er een pagina is in het admin menu met als inhoud jouw test-tekst.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_render_admin_page() {
    echo '<div class="wrap">Hello world!</div>';
}

function jio_register_menu_page() {
  add_menu_page( 'JIO Birthdays', 'JIO Verjaardagen', 'manage_options', 'jio-birthdays', 'jio_render_admin_page');
}
add_action('admin_menu', 'jio_register_menu_page');
```

</details>
    
## B. Maak een formulier aan.

Er zijn verschillende manieren om een formulier in het admin-panel door je plugin te laten verwerken. Zo kun je gebruik maken van de WP Options API, kun je API-endpoints registreren voor asynchrone verwerking, of kun je gebruikmaken van `admin-post.php`. Die laatste is het meest eenvoudig en gaan we hier toepassen.

> Binnen Wordpress admin-menupagina's is het ongebruikelijk om data te laten verwerken door dezelfde functie die het formulier toont. De reden hiervoor is dat je vaak een gebruiker wilt redirecten nadat deze een wijziging heeft uitgevoerd, om te voorkomen dat een wijziging bij een refresh wordt herhaald. Echter, ten tijde van het aanroepen van deze functie heeft Wordpress al een `200 OK` naar de gebruiker gestuurd; je kunt op dat punt niet meer redirecten.

1. In de functie `jio_render_admin_page`, voeg een simpel formulier toe met twee tekstvelden: `name` en `birthday`. Voeg ten tweede een `hidden` input-veld toe, met als name `action` en waarde `jio_add_birthday`. Voeg tot slot een submit-knop toe.
2. Wanneer het formulier gesubmit wordt, zorg dat er een POST wordt gedaan naar `admin-post.php`.

Op dit punt zul je een witte pagina krijgen als je het formulier submit. Dit is okee.

**Checkpoint**: Controleer dat je het formulier kunt submitten, en dat je dan terecht komt op een andere, lege pagina.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_render_admin_page() {
    ?>
    <div class="wrap">
    <form action="?page=jio-birthdays" method="POST">
        <input type="hidden" name="action" value="jio_add_birthday" />
        <p>
            <label id="jio-name-label" for="jio-name">Name *</label>
            <input id="jio-name" name="name" aria-labelledby="jio-name-label" />
        </p>

        <p>
            <label id="jio-birthday-label" for="jio-birthday">Birthday *</label>
            <input id="jio-birthday" name="birthday" aria-labelledby="jio-birthday-label" placeholder="yyyy-mm-dd" />
        </p>

        <input name="submit" type="submit" value="Opslaan" />
    </form>
    </div>
    <?php
}

```

</details>

## C. Het formulier verwerken.

1. Maak een nieuwe functie aan, genaamd `jio_handle_add_birthday`.
2. In deze functie, verwerk de invoerdata. Gebruik [`$wpdb->insert`](https://developer.wordpress.org/reference/classes/wpdb/insert/) om een nieuwe entry toe te voegen aan de database tabel.
3. Aan het eind van de functie, redirect de gebruiker naar `admin.php?page=jio-birthdays&status=success`.
4. Hook de functie in de `admin_post_jio_add_birthday` action.

**Checkpoint**: Controleer dat je een verjaardag kunt toevoegen via deze admin-pagina.

<details>
<summary>Oplossing (klik om te openen)</summary>

```php
function jio_handle_add_birthday() {
    global $wpdb;

    $name = $_POST["name"] ?? null;
    $birthday = $_POST["birthday"] ?? null;
    if (!$name || !$birthday) {
        wp_redirect("admin.php?page=jio-birthdays&status=error");
        exit;
    }

    $result = $wpdb->insert("{$wpdb->prefix}jio_birthdays", ["name" => $name, "birthday" => $birthday], ["%s", "%s"]);

    $redirect_page = "admin.php?page=jio-birthdays&status=" . ($result ? "success" : "error");
    wp_redirect($redirect_page);
    exit;
}

add_action('admin_post_jio_add_birthday', 'jio_handle_add_birthday');
```

</details>

Gefeliciteerd! Op dit punt zou je de data terug moeten kunnen zien op de pagina op de website.
