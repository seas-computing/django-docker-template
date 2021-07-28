# Django on Docker template

## Introduction

This repo contains a base template for running Django on Docker, in both development and production modes.

### Setup

* Clone the repo
* Follow the relevant instructions below.

#### Development mode

* Copy `template.env.dev` and rename it `.env.dev`.
* Fill in the relevant environment variables.
  * If you fill in the `SQL-*` and `DATABASE` environment variables you will get a Postgres database.
  * Otherwise you will end up with a sqlite3 database (see `app/app/settings/base.py` for database config).
  * Note that even if you opt for sqlite3, the docker compose file will still create a Postgres container.
* From the parent directory, run `docker-compose -f docker-compose.dev.yml up -d --build`
* Once the containers have been built, go to `localhost:8000`

#### Production mode

The repo also contains docker config to run using the production settings. This utilizes an additional nginx container and runs the Django app using WSGI via Gunicorn, rather than using the django dev server.

* Copy `template.env.prod` and rename it `.env.prod`.
* Fill in the relevant environment variables.
  * If you fill in the `SQL-*` and `DATABASE` environment variables you will get a Postgres database.
  * Otherwise you will end up with a sqlite3 database (see `app/app/settings/base.py` for database config).
  * Note that even if you opt for sqlite3, the docker compose file will still create a Postgres container.
  * From the parent directory, run `docker-compose -f docker-compose.prod.yml up -d --build`
* Once the containers have been built, go to `localhost:1337/admin`
  * You will get a "Not found" error at the root address. This is expected as there are no roots or templates set up.