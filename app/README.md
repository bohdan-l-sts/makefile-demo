# MakeDemo

The only things you need on the host are **Docker** (with Compose v2) and **make**. No PHP, Composer, Node or nvm.

```bash
make doctor   # does this machine have what the project needs?
make init     # first run: .env, containers, dependencies, key, migrations, seed (safe to repeat)
make          # list every command
```

App: http://localhost:8088 · Vite dev server: `make dev` (http://localhost:5174)

Everyday commands:

```bash
make artisan migrate:status          # any artisan command
make composer require spatie/laravel-data
make test UserTest                   # = php artisan test --filter=UserTest
make -j3 check                       # Pint + PHPStan + tests in parallel (what CI runs)
make db-fresh                        # asks before wiping the database
```

Personal tweaks go in `Makefile.local` (git-ignored); see `Makefile.local.example`.
