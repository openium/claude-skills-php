.PHONY: help up down build shell composer console test logs phpstan fixtures db-migrate messenger-consume worker-logs mail sequelace

.DEFAULT_GOAL := help

# Must match the `database` service credentials in compose.yaml.
SEQUELACE_HOST ?= 127.0.0.1
SEQUELACE_PORT ?= 3307
SEQUELACE_NAME ?= <project>
SEQUELACE_USER ?= user
SEQUELACE_PASSWORD ?= password
SEQUELACE_FILE := docker/sequelace/<project>-dev.spf

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

up: ## Start the Docker containers
	docker compose up -d

down: ## Stop the Docker containers
	docker compose down

build: ## Build the Docker images
	docker compose build

shell: ## Open a shell in the PHP container
	docker compose exec php sh

composer: ## Install Composer dependencies
	docker compose exec php composer install

console: ## Run a bin/console command (e.g. make console -- app:my-command)
	docker compose exec php bin/console $(filter-out $@,$(MAKECMDGOALS))

test: ## Run PHPUnit tests
	docker compose exec -e APP_ENV=test php bin/phpunit

logs: ## Show logs from all containers
	docker compose logs -f

phpstan: ## Run PHPStan static analysis
	docker compose exec php vendor/bin/phpstan --memory-limit=-1

fixtures: ## Load Doctrine fixtures
	docker compose exec php bin/console doctrine:fixtures:load

db-migrate: ## Run Doctrine migrations
	docker compose exec php bin/console doctrine:migrations:migrate

# --- Optional targets: keep only what matches the services actually enabled in compose.yaml ---

messenger-consume: ## Run the Messenger worker (async) — remove if no async transport
	docker compose exec php bin/console messenger:consume async -vv

worker-logs: ## Show the Messenger worker logs — remove if no messenger-worker service
	docker compose logs -f messenger-worker

mail: ## Open the Mailpit web UI — remove if no mailpit service
	@open http://localhost:8025

# --- Default on macOS + MariaDB (the team's common case) — drop this target on Linux/Windows
#     or if the project doesn't use MariaDB/MySQL, or if the team doesn't use Sequel Ace. ---

sequelace: ## Generate and open the Sequel Ace connection file for the local DB (dev)
	@mkdir -p $(dir $(SEQUELACE_FILE))
	@printf '%s\n' \
		'<?xml version="1.0" encoding="UTF-8"?>' \
		'<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">' \
		'<plist version="1.0">' \
		'<dict>' \
		'	<key>SPFVersion</key>' \
		'	<integer>1</integer>' \
		'	<key>format</key>' \
		'	<string>connection</string>' \
		'	<key>encrypted</key>' \
		'	<false/>' \
		'	<key>data</key>' \
		'	<dict>' \
		'		<key>connection</key>' \
		'		<dict>' \
		'			<key>type</key>' \
		'			<string>SPTCPIPConnection</string>' \
		'			<key>name</key>' \
		'			<string><project> - Docker (dev)</string>' \
		'			<key>host</key>' \
		'			<string>$(SEQUELACE_HOST)</string>' \
		'			<key>port</key>' \
		'			<integer>$(SEQUELACE_PORT)</integer>' \
		'			<key>user</key>' \
		'			<string>$(SEQUELACE_USER)</string>' \
		'			<key>password</key>' \
		'			<string>$(SEQUELACE_PASSWORD)</string>' \
		'			<key>database</key>' \
		'			<string>$(SEQUELACE_NAME)</string>' \
		'			<key>useSSL</key>' \
		'			<integer>0</integer>' \
		'		</dict>' \
		'	</dict>' \
		'</dict>' \
		'</plist>' \
		> $(SEQUELACE_FILE)
	@echo "Sequel Ace connection file generated: $(SEQUELACE_FILE)"
	@open $(SEQUELACE_FILE)
