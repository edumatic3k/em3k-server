# em3k Server

This repository is the Laravel server side of em3k. It currently contains a mostly stock Laravel 12 application with a Filament 3 admin panel; the `em3k-viewer` is the more developed part of the project. The server does not currently define custom em3k API routes or Filament resources, and it is not required to serve the viewer's local JSON content.

The admin panel is available at `/admin`. Current migrations provide Laravel users, sessions, cache and queue tables, plus media-library and Sanctum tables. They create the database schema, not content from another computer.

## Requirements

- PHP 8.2 or newer, with the PDO MySQL extension enabled (`pdo_mysql`)
- Composer
- MariaDB running locally, with a database and a user that can create and alter tables
- Node.js and npm if you want to run or build the Laravel Vite assets

## First-Time Setup

Run these commands from this directory (`em3k-server`).

1. Install the PHP dependencies and create your local environment file:

   ```sh
   composer install
   cp .env.example .env
   ```

2. Create an empty MariaDB database. For example, run the following as a MariaDB administrator. Use a local-only password of your choice:

   ```sql
   CREATE DATABASE em3k CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'em3k'@'127.0.0.1' IDENTIFIED BY 'replace-with-a-local-password';
   GRANT ALL PRIVILEGES ON em3k.* TO 'em3k'@'127.0.0.1';
   ```

   You can instead create the database and grant a user access through phpMyAdmin. The account must match the host and credentials configured below.

3. Edit `.env`. Set the app values and MariaDB connection, replacing the example password:

   ```dotenv
   APP_NAME=em3k
   APP_ENV=local
   APP_DEBUG=true
   APP_URL=http://127.0.0.1:8000

   DB_CONNECTION=mariadb
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=em3k
   DB_USERNAME=em3k
   DB_PASSWORD=replace-with-a-local-password
   ```

   Keep `.env` on this machine; it is ignored by Git and should not be committed.

4. Generate the app key and create the tables:

   ```sh
   php artisan key:generate
   php artisan migrate
   ```

   Migrations are tracked in the database. Running `php artisan migrate` again later applies only pending migrations. Do not use `php artisan migrate:fresh` on a database you want to keep; it drops all tables.

5. Create your own Filament login:

   ```sh
   php artisan make:filament-user
   ```

   Follow the prompts to set the name, email, and password. Do not run `php artisan db:seed` for routine setup: the current development seeder creates a test admin account with a predictable password.

## Run Locally

Start Laravel from this directory:

```sh
php artisan serve
```

Open [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) and sign in with the Filament user you created.

For Vite asset development, install the npm dependencies and start Vite in a second terminal:

```sh
npm install
npm run dev
```

To build those assets instead, run `npm run build`.

## Useful Commands

```sh
php artisan about             # Show effective app and database configuration
php artisan migrate:status    # Show which migrations have run
php artisan test              # Run the Laravel test suite
```

If you change `.env` after configuration has been cached, refresh Laravel's configuration with:

```sh
php artisan config:clear
```

## Troubleshooting

- **SQLite database file does not exist:** check that `.env` exists and has `DB_CONNECTION=mariadb`. The example environment defaults to SQLite, but this setup uses MariaDB.
- **Could not find driver:** enable/install the PHP `pdo_mysql` extension for the PHP CLI used to run Artisan, then check with `php -m`.
- **Database connection refused or access denied:** confirm MariaDB is running and that `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD` match the database and user you created.
- **Application encryption key is missing:** run `php artisan key:generate` for a new local environment. Do not regenerate the key on an existing environment without understanding the effect on encrypted data and sessions.
- **HTTP 500:** check `storage/logs/laravel.log` and run `php artisan migrate:status`. Keep `APP_DEBUG=true` only for local development.