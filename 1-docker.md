# Wordpress lokaal opstarten

In deze sectie ga je lokaal met Docker een Wordpress-container opstarten.

## Vereisten

- Docker
- Docker-compose

## Stappenplan

1. Maak een folder aan waarin je aan de slag gaat met Wordpress. Bijvoorbeeld: wordpress-workshop. Voer de volgende stappen uit vanuit deze folder.

<details>
<summary>Oplossing</summary>

```bash
mkdir wordpress-workshop
cd wordpress-workshop
```

</details>

2. Maak een folder aan genaamd jio-birthdays. Dit zal onze plugin-folder zijn.

<details>
<summary>Oplossing</summary>

```bash
mkdir our-birthdays
```

</details>

3. Maak een docker-compose.yml bestand met drie containers: wordpress:latest, MySQL (database), en PHPmyadmin. Zorg dat de volume persistent is.

<details>
<summary>docker-compose.yml</summary>

```yml
version: "3.7"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - ./jio-birthdays:/var/www/html/wp-content/plugins/jio-birthdays
    ports:
      - "8888:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DEBUG: 1

  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    ports:
      - 8889:80
    environment:
      MYSQL_USERNAME: wordpress
      MYSQL_ROOT_PASSWORD: wordpress
      PMA_HOST: db:3306

volumes:
  db_data: {}
```

</details>

4. Start de docker-containers.

<details>
<summary>Oplossing</summary>

```bash
docker-compose up
```

</details>

5. Ga naar `http://localhost:8888` en voer de 5-minute Wordpress install uit.

Controleer dat je nu een werkende Wordpress-instantie hebt en kunt inloggen op `http://localhost:8888/wp-admin`.
