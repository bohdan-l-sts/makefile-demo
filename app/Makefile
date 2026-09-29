# Step 2 — foundations: one entry point, files owned by you, onboarding in one command.

.DEFAULT_GOAL := help

# Host user/group, passed to the image so files created in containers belong to you.
UID ?= $(shell id -u)
GID ?= $(shell id -g)
export UID GID

COMPOSE := docker compose
# Add -T when stdin is not a terminal (CI, pipes), otherwise `exec` fails with "the input device is not a TTY".
TTY  := $(shell [ -t 0 ] || echo -T)
EXEC := $(COMPOSE) exec $(TTY) app

.PHONY: help init up down restart shell artisan composer test migrate logs

help: ## Show this help (default goal)
	@grep -hE '^[a-zA-Z_.-]+:.*?## ' $(MAKEFILE_LIST) \
		| awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-12s\033[0m %s\n", $$1, $$2}'

# A *file* target: runs only when .env does not exist yet.
# Deliberately no `.env.example` prerequisite: that would overwrite your .env whenever the example changes.
.env:
	cp .env.example .env
	@echo "Created .env from .env.example"

init: .env up ## First run (safe to repeat): .env, containers, composer, app key, migrations, seed
	$(EXEC) composer install
	@grep -q '^APP_KEY=base64' .env || $(EXEC) php artisan key:generate
	$(EXEC) php artisan migrate --seed

up: ## Start containers and wait until the database is healthy
	$(COMPOSE) up -d --build --wait

down: ## Stop containers
	$(COMPOSE) down

restart: down up ## Restart containers

shell: ## Open a shell in the app container
	$(EXEC) sh

artisan: ## Any artisan command: make artisan C="make:model Post -m"
	$(EXEC) php artisan $(C)

composer: ## Any composer command: make composer C="require spatie/laravel-data"
	$(EXEC) composer $(C)

migrate: ## Run migrations
	$(EXEC) php artisan migrate

test: ## Run tests: make test C="--filter=UserTest"
	$(EXEC) php artisan test $(C)

logs: ## Last 20 lines of the Laravel log
	tail -n 20 storage/logs/laravel.log
