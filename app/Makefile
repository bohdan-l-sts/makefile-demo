# Step 4 — "ideal": everything from step 3, plus strict shell, positional arguments,
# personal overrides and the same targets in CI.

# Fail fast: unset variables, failed pipes and errors stop the recipe (default /bin/sh would carry on).
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
# A recipe that fails half-way deletes its half-written target file instead of leaving it "up to date".
.DELETE_ON_ERROR:
MAKEFLAGS += --warn-undefined-variables --no-builtin-rules
.SUFFIXES:

.DEFAULT_GOAL := help

# Keep each parallel job's output in one block. GNU make 4+ only (macOS ships 3.81), hence the feature check.
ifneq ($(filter output-sync,$(.FEATURES)),)
MAKEFLAGS += --output-sync=target
endif

UID ?= $(shell id -u)
GID ?= $(shell id -g)
export UID GID

COMPOSE := docker compose
TTY     := $(shell [ -t 0 ] || echo -T)
EXEC    := $(COMPOSE) exec $(TTY) app
# Checks run in parallel (-j), so they never ask for a terminal.
EXEC_T  := $(COMPOSE) exec -T app
NODE    := $(COMPOSE) run --rm node
C       ?=
FORCE   ?=
ARGS    ?=

# Positional arguments: `make artisan migrate:status`, `make test UserTest`, `make composer require foo/bar`.
# Everything after a proxy target is an argument, not a target. A catch-all rule turns those words into no-ops.
# Don't `$(eval $(ARGS):;@:)` or list them in .PHONY: a word like `migrate:status` has a colon, and make
# reads it as "target: prerequisite" -> "multiple target patterns. Stop."
PROXIES := artisan composer test npm
ifneq ($(filter $(firstword $(MAKECMDGOALS)),$(PROXIES)),)
ARGS := $(wordlist 2,$(words $(MAKECMDGOALS)),$(MAKECMDGOALS))
%:
	@:
endif

# Personal, git-ignored tweaks (extra aliases, a different APP_PORT, ...). See Makefile.local.example.
-include Makefile.local

.PHONY: help init up down restart shell artisan composer test migrate logs \
        dev assets npm check lint stan ci db-fresh db-dump doctor

help: ## Show this help (default goal)
	@grep -hE '^[a-zA-Z_.-]+:.*?## ' $(MAKEFILE_LIST) \
		| awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-12s\033[0m %s\n", $$1, $$2}'

# ---- setup -----------------------------------------------------------------------------------

.env:
	cp .env.example .env
	@echo "Created .env from .env.example"

# File targets: re-run only when the inputs are newer than the output.
# `make init` twice in a row skips composer install entirely.
vendor/autoload.php: composer.json composer.lock
	$(EXEC) composer install
	@touch $@

# npm writes node_modules/.package-lock.json on every install, a free timestamp to depend on.
node_modules/.package-lock.json: package.json $(wildcard package-lock.json)
	$(NODE) npm install

init: .env up vendor/autoload.php ## First run (safe to repeat): .env, containers, composer, key, migrations, seed
	@grep -q '^APP_KEY=base64' .env || $(EXEC) php artisan key:generate
	$(EXEC) php artisan migrate --seed

up: ## Start containers and wait until the database is healthy
	$(COMPOSE) up -d --build --wait

down: ## Stop containers
	$(COMPOSE) --profile node down

restart: down up ## Restart containers

# ---- everyday --------------------------------------------------------------------------------

shell: ## Open a shell in the app container
	$(EXEC) sh

artisan: ## Any artisan command: make artisan migrate:status  (or C="make:model Post -m")
	$(EXEC) php artisan $(ARGS) $(C)

composer: ## Any composer command: make composer require spatie/laravel-data
	$(EXEC) composer $(ARGS) $(C)

migrate: ## Run migrations
	$(EXEC) php artisan migrate

logs: ## Last 20 lines of the Laravel log (one shell line: `cd` does not survive to the next line)
	@cd storage/logs && { [ -f laravel.log ] && tail -n 20 laravel.log || echo "No laravel.log yet"; }

dev: node_modules/.package-lock.json ## Vite dev server in the node container (http://localhost:5174)
	$(COMPOSE) run --rm --service-ports node npm run dev

assets: node_modules/.package-lock.json ## Production build of the front-end assets
	$(NODE) npm run build

npm: ## Any npm command in the node container: make npm outdated
	$(NODE) npm $(ARGS) $(C)

# ---- quality ---------------------------------------------------------------------------------

test: ## Run tests: make test UserTest  (= --filter=UserTest; flags via C="--stop-on-failure")
	$(EXEC_T) php artisan test $(if $(ARGS),--filter=$(ARGS)) $(C)

lint: ## Pint in check mode
	$(EXEC_T) ./vendor/bin/pint --test

stan: ## PHPStan (Larastan)
	$(EXEC_T) ./vendor/bin/phpstan analyse --no-progress --memory-limit=1G

# `make -j3 check` runs the three prerequisites in parallel; --output-sync keeps each one's output together.
check: lint stan test ## Pint + PHPStan + tests (try: make -j3 check)
	@printf '\033[32mAll checks passed\033[0m\n'

ci: ## What CI runs: the same targets developers use locally
	$(MAKE) init
	$(MAKE) -j3 check

# ---- database --------------------------------------------------------------------------------

db-fresh: ## Drop all tables and re-seed (asks first; FORCE=1 skips the question)
	@printf '\033[31mThis wipes every table in the local database.\033[0m\n'
	@[ "$(FORCE)" = 1 ] || { printf 'Continue? [y/N] '; read ans; [ "$${ans:-N}" = y ]; }
	$(EXEC) php artisan migrate:fresh --seed

# Credentials come from the db container's own environment: nothing parsed from .env on the host.
# With pipefail a failed mysqldump fails the target, instead of gzip "succeeding" on empty input.
db-dump: ## Dump the database to storage/dump.sql.gz
	$(COMPOSE) exec -T db sh -c 'MYSQL_PWD="$$MYSQL_PASSWORD" exec mysqldump --no-tablespaces -u"$$MYSQL_USER" "$$MYSQL_DATABASE"' \
		| gzip > storage/dump.sql.gz
	@echo "Saved storage/dump.sql.gz ($$(wc -c < storage/dump.sql.gz) bytes)"

# ---- diagnostics -----------------------------------------------------------------------------

doctor: ## Check the host has what the project needs (docker, compose v2, free ports)
	@command -v docker >/dev/null || { echo "✗ docker is not installed"; exit 1; }
	@docker compose version >/dev/null 2>&1 || { echo "✗ docker compose v2 is missing"; exit 1; }
	@docker info >/dev/null 2>&1 || { echo "✗ docker daemon is not running (or you are not in the docker group)"; exit 1; }
	@# Ports only matter before our own containers are up (then they are ours).
	@[ -n "$$($(COMPOSE) ps -q)" ] || for port in $${APP_PORT:-8088} $${VITE_PORT:-5174}; do \
		if ss -ltn 2>/dev/null | grep -q ":$$port "; then echo "✗ port $$port is already taken"; exit 1; fi; done
	@echo "✓ docker, compose v2 and ports look good ($(MAKE_VERSION) make)"
