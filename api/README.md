# Heinz-95729 Project API

This is a sample API to support an ecommerce experience. The API includes the following domains:

-   [users](src/lib/users)
-   [products](src/lib/products)
-   [books](src/lib/books)

These domains are composed into a koa app (API only) in the [src/api](src/api) directory.

## Getting Started

This project uses Architecture Decision Records (ADRs). In addition to the documentation in this file, you should [read the ADRs](adr). They include information on project organization (file and folder naming, design conventions, app composition), library decision context, etc. As you contribute to this project, please make sure to include ADRs for any significant decisions that you make.

These instructions assume you are using a bash compatible shell (e.g. https://ohmyz.sh/) and that you navigated to the directory this README is in in your terminal (e.g. `cd ~/.../heinz-95729-project/api`).

### ENVVARS

ENVVARS is an abbreviation for Environment Variables. In order for the API to run, you need to set the ENVVARS. There are examples for what you need to set in _.env-example_ (you may need to show hidden files on your OS to be able to see files that start with a `.`). On a linux based system, you can `export ENVVAR_NAME=value` to set an ENVVAR. The drawback of this approach is that you have to do it every time you open a new terminal tab. This API uses a tool to persist your ENVVARS. To take advantage of that:

1. Copy the `.env-example` file
1. Paste the file in this same directory, giving it the name, `.env`

### Install the dependencies

#### Docker

Most of the dependencies can be installed via the command line. _Docker_ is not necessarily one of them, although there is a brew cask for macos and linux. Before installing docker, make sure you don't have a conflict:

```Shell
which docker # note this will not return a result if docker isn't running
```

1. If that printed "/usr/local/bin/docker" you're all set :+1:... you can move on to provisioning the database
1. If that printed something else, you might need to get rid of what you have. For instance, if it points to a global npm package (e.g. in your nvm directory), remove it.
1. When you're confident there is no conflict, install [Docker for Mac](https://www.docker.com/docker-mac), or [Docker for Windows](https://www.docker.com/docker-windows)
1. Start Docker

#### Provisioning a Postgres Database

This app uses [PostgreSQL](https://www.postgresql.org/) for the database. You can install it directly on your computer if you like. I prefer to install it in a docker container because it's easier to destroy and rebuild it. You'll need a database IDE as well. There are lots of good options out there. I often use [TablePlus](https://www.tableplus.io/download).

To initiate a PostgreSQL container, follow the instructions at: [How to Use the Postgres Docker Official Image](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/)

##### Initiating a Postgres container with bash

Alternatively, you can:

1. Create a bash script in this folder (e.g. `touch pgup.sh`)
1. Make the bash script executable (e.g. `chmod 766 pgup.sh`)
1. Paste into that file, the code from the block below
1. Change the path for `DIR=` so that it matches the path to this directory
1. Execute the script (e.g. `./pgup.sh`)

> NOTE that if you have a readonly filesystem, you will need to comment out the `mkdir -p $HOST_VOLUME_PATH` and make those directories by hand. The error message you receive should include the `$HOST_VOLUME_PATH`, so try running the script without commenting that first.

```Shell
#!/bin/bash
#
# Creates a docker image for Postgres, and runs it
# https://hub.docker.com/_/postgres

printf "\nrunning up...\n\n"
# !!!!!!!!!!!!!!!! IMPORTANT: you need to set the DIR path below to the location of this folder
DIR=/Users/[the path to where you cloned this repo...]/heinz-95729-project/api
HOST_VOLUME_PATH=${DB_HOST_VOLUME_PATH:=$DIR/docker_volumes/postgresql/data}
CTNR_VOLUME_PATH=${DB_CTNR_VOLUME_PATH:=/var/lib/postgresql/data}

# stop on error (e) and print each command (x)
set -ex

mkdir -p $HOST_VOLUME_PATH

# NOTE: the variables in this script can be overridden
# by exporting the values in your terminal
# e.g. export DB_IMAGE_NAME=my_app_db

# Postgres variables
DB_USERNAME=${DB_USERNAME:=app_admin}
DB_PASSWORD=${DB_PASSWORD:=parsley-lumber-informal-Spectra-8}
DB_NAME=${DB_NAME:=heinz_95729_app}

# Docker image variables
DB_PORT=${DB_PORT:=5432}
DB_IMAGE_VERSION=${DB_IMAGE_VERSION:=11.8-alpine}
DB_IMAGE=${DB_IMAGE:=postgres:$DB_IMAGE_VERSION}
DB_IMAGE_NAME=${DB_IMAGE_NAME:=pgdb}
DB_IMAGE_PORT=${DB_IMAGE_PORT:=$DB_PORT:$DB_PORT}

# Run postgres in daemon mode, expose port 5432
# and mount this directory to /repo
printf "\nrunning $DB_IMAGE_NAME...\n\n"
docker run \
  --name $DB_IMAGE_NAME \
  --mount "type=bind,src=$DIR,dst=/repo" \
  -p $DB_IMAGE_PORT \
  -d \
  -e POSTGRES_USER=$DB_USERNAME \
  -e POSTGRES_PASSWORD=$DB_PASSWORD \
  -e POSTGRES_DB=$DB_NAME \
  -e TZ=GMT \
  -v $HOST_VOLUME_PATH:$CTNR_VOLUME_PATH \
  $DB_IMAGE
```

Here are some other scripts you can use to interact with the docker image:

```Shell
# list all running instances
docker ps

# list all instances (running or not)
docker ps -a

# destroy the image:
docker rm -fv ${DB_IMAGE_NAME:=pgdb}
rm -rf $HOST_VOLUME_PATH

# start the image
docker container start ${DB_IMAGE_NAME:=pgdb}

# stop the image
docker container stop ${DB_IMAGE_NAME:=pgdb}

# connect to the shell of the docker image
# | switch | description                                      |
# |--------|--------------------------------------------------|
# | -i     | Keep STDIN open even if not attached             |
# | -t     | Allocate a pseudo-TTY                            |
docker exec -it ${DB_IMAGE_NAME:=pgdb} /bin/bash

# connect to the postgres repl on the docker image
# | switch | description                                      |
# |--------|--------------------------------------------------|
# | -i     | Keep STDIN open even if not attached             |
# | -t     | Allocate a pseudo-TTY                            |
docker exec -it $DB_IMAGE_NAME \
  sh -c 'exec psql --username=$POSTGRES_USER --dbname=$POSTGRES_DB'
```

#### Install NVM

Before you install NVM, you have to have a sh profile (~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc). Macos uses zsh by default now, and it does not include a profile by default. If you're using zsh, you can create a profile with `touch ~/.zshrc`.

If you forget to do this, or for some reason you see an error like "nvm: command not found", checkout the [troubleshooting guides](https://github.com/nvm-sh/nvm#troubleshooting-on-macos).


##### With HomeBrew

```Shell
# these instructions use HomeBrew: https://brew.sh/
brew update

# install NodeJS Version Manager if you don't already have it
brew install nvm
source $(brew --prefix nvm)/nvm.sh
# install node LTE 16.3.0
nvm install 16.3.0
# Use it
nvm use 16.3.0
# And set it as your default (optional)
nvm alias default 16.3.0

# install HTTPIE
brew install httpie

# cleanup cached brew files
brew cleanup -s
rm -rf "$(brew --cache)"
```

##### Without HomeBrew

[Install NVM](https://github.com/nvm-sh/nvm#installing-and-updating). Then in a bash derived shell, use NVM to install a version of NodeJS, and then install pnpm. If you're running Windows, install [Chocolatey](https://chocolatey.org/)

```Shell
# install node LTE 16.3.0
nvm install 16.3.0
# Use it
nvm use 16.3.0
# And set it as your default (optional)
nvm alias default 16.3.0

# IFF you're running linux
install httpie
# or if you're using snaps
snap install httpie

# IFF you're running windows
choco install httpie
```

#### The rest

```Shell
# Install pnpm (pnpm is more reliable, efficient, and secure than npm)
npm install -g pnpm

# Install the app's dependencies
pnpm install --recursive

# migrate the tables
pnpm run migrate:up

# Start the app in _watch_ mode
pnpm run watch

# Add a product
http POST http://localhost:3000/books <<< '{ "title": "This Is Where I Leave You: A Novel", "uid": "where_i_leave_you", "description": "The death of Judd Foxman'"'"'s father marks the first time that the entire Foxman clan has congregated in years. There is, however, one conspicuous absence: Judd'"'"'s wife, Jen, whose affair with his radio- shock-jock boss has recently become painfully public. Simultaneously mourning the demise of his father and his marriage, Judd joins his dysfunctional family as they reluctantly sit shiva-and spend seven days and nights under the same roof. The week quickly spins out of control as longstanding grudges resurface, secrets are revealed and old passions are reawakened. Then Jen delivers the clincher: she'"'"'s pregnant.", "metadata": { "authors": [{ "name": "Jonathan Tropper" }], "keywords": ["funeral", "death", "comedy"] }, "price": 7.99, "thumbnailHref": "https://m.media-amazon.com/images/I/81hvdUSsatL._AC_UY436_QL65_.jpg", "type": "book" }'

# Get a book
# Replace the `:uid` with the uid of the product you created
# http://localhost:3000/books/:uid
http http://localhost:3000/books/where_i_leave_you

# Find a book
# Replace the `:uid` with the uid of the product you created
http http://localhost:3000/books?q=tropper
```

> NOTICE all the commands are using `pnpm`, not `npm`. See [adr/20210207-choose-package-manager.md](adr/20210207-choose-package-manager.md) for more information.

_If you experience an EACCESS error when trying to use pnpm, follow [npm's guide to troubleshooting globally installed packages](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)_

## Running Tests

```Shell
# set the database connection ENVVAR
# note that your connection string might be different than the example
export DB_CONNECTION_STRING=postgresql://app_admin:parsley-lumber-informal-Spectra-8@0.0.0.0:5432/heinz_95729_app?sslmode=disable

# run the tests locally
pnpm test -- -r nyan

# alt example (skip the pnpm run syntax)
node test -r nyan

# alt example (Saving Markdown test output is great for sharing with analysts, QA, client, etc.)
node test -r md > markdown-to-share-with-client.md

# alt example (TAP, and summary are great for CI)
node test -m ONLY -r tap,summary

# alt example (Want deterministic ordering? Want to pipe into other commands / 3rd party reporters?)
node test -m ONLY -r tap,summary -o deterministic | npx tap-parser -j | jq

# alt example (JSON)
node test -r json | jq
```

> See [supposed docs](https://github.com/losandes/supposed#arguments-and-envvars) for more information on the args, and reporters that are supported.
>
> NOTE that if/as you change dependencies, you need to `pnpm install --recursive`, or install the dependencies in the package that you changed dependencies in.

## Adding Domains, Routes, & Migrations

In this API, domains are created as their own packages to promote decoupling, and re-use. The domains can be found in _src/lib_. Each domain should have it's own package.json, migrations, and test files. When adding a new domain:

1. Create your package in _src/lib_, and make sure to give it a unique name in the package.json
1. Create, or copy/paste-and-edit the base files and folders from another domain: index.js, knexfile and migrate.js (if there are migrations), test.js, test-plan.js, src/, migrations/.
1. In your terminal, navigate to _src/api_
1. Install the domain (e.g. `pnpm add ../lib/new-domain`)
1. Require the package and initialize it in _src/api/compose-domains.js_ (Poor Man's DI). Note this is where you will register routes and database schema migrations, as well.
1. If the domain includes features that should be added to the health check, add a test to _src/api/compose-test.js_

> See the _src/lib/products_, and _src/lib/books_ domains for examples that include migrations, routes, and that are tested on startup, and by the healthcheck

## Local Database Development

This app uses [knex](https://knexjs.org) for database development and connections. Migrations run automatically when the app starts. You can also execute them manually without starting the app.

```Shell
# set your ENVVARS
export NODE_ENV=local
export DB_CONNECTION_STRING=postgresql://app_admin:parsley-lumber-informal-Spectra-8@0.0.0.0:5432/heinz_95729_app?sslmode=disable

# run any migrations that haven't been commited yet
pnpm run migrate:up

# destroy all migrations (teardown)
pnpm run migrate:down
```

You have more control when not running the migrations in aggregate. Following is an example for using the [knex Migrations CLI](https://knexjs.org/#Migrations-CLI) with a specific domain.

```Shell
# set your ENVVARS
export NODE_ENV=local
export DB_CONNECTION_STRING=postgresql://app_admin:parsley-lumber-informal-Spectra-8@0.0.0.0:5432/heinz_95729_app?sslmode=disable

# navigate to the domain
cd src/lib/users

# make sure the dependencies are installed
pnpm install

# run the next migration
npx knex migrate:up

# rollback the last migration
npx knex migrate:down

# create a new migration file
npx knex migrate:make add_table
```

### Migration file naming conventions

As with ADRs and commit messages, and to help with readability, the name should be a _present tense imperative verb phrase_. To balance readability, and system usability, names should be lower-snake-case (lowercase with underscores). e.g.:

* add_users_table
* alter_users_table_add_foo_column
* alter_users_table_drop_foo_column
* add_indexes_to_users_table
