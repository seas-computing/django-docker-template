# Django on Docker template

## Introduction

This repo contains a base template for running Django on Docker, in both development and production modes.

### Setup

* Clone the repo
* Follow the relevant instructions below.

#### Development mode

* Copy `template.env.dev` and rename it `.env.dev`.
* Fill in the relevant environment variables.
  * If you fill in the POSTRES-*, SQL-* and DATABASE environment variables you will get a Postgres database.
  * Otherwise you will end up with a sqlite3 database (see `app/app/settings/base.py` for database config)
* From the parent directory, run `docker-compose -f docker-compose.dev.yml up -d --build`
* Once the containers have been built, go to `localhost:8000`

