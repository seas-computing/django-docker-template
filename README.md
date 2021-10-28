# Django Docker Template

This repo contains a base template for running Django on Docker, in both development and production modes.

## Using this template

In order to use this template, click the "Use this template" button in the GitHub UI. 

Alternatively, you can clone the repo, remove the existing remote origin, and add a new remote for the new project repo.

```
$ git remote rm origin
$ git remote add origin <remote-url>
```

## App architecture

[When developing a new app, add high level details of app architecture in this section]

## Local Development

For development, there is a [`docker-compose.yml`](docker-compose.yml) file in the root of the project. You'll need to copy the [`template.env`](template.env) file to `.env` and fill in the appropriate values for the database connection and the Algolia application and index.

With the `.env` file created, run:

```sh
$ docker-compose up
```

Which should bring up the app and database containers. From there, you can access the Django admin interface in the browser at http://localhost:8000/admin and log in with the credentials defined in the `DJANGO_SUPERUSER_USERNAME` and `DJANGO_SUPERUSER_PASSWORD` variables.

Our [`Dockerfile`](app/Dockerfile) uses a multi-stage build, and the `docker-compose` file targets the `development` stage, which has some additional system dependencies installed and runs the built-in Django web server. To install additional python dependencies, you should put them in [`app/requirements.txt`](app/requirements.txt) and rebuild the image with:

```sh
$ docker-compose build
```

To access Django's CLI tool, you can run:

```sh
$ docker-compose exec web python manage.py
```

That will list all of the available commands. For development the most important ones will be:

```sh
# Run tests
$ docker-compose exec web python manage.py test

# Run database migrations
$ docker-compose exec web python manage.py migrate
```

## Running in Production

The docker image is built by [GitHub Action][actions] and published through GitHub container registry. To run the latest version of the app:

```sh
# Pull the latest copy of the image
$ docker pull ghcr.io/seas-computing/[REPO_NAME_HERE]:stable

# Run the image, passing through the necessary environment variables from our .env file
$ docker run -it --rm --env-file .env ghcr.io/seas-computing/[REPO_NAME_HERE]:stable
```

When running in production, the `DJANGO_SETTINGS_MODULE` environment variable should be set to `app.settings.production`. By default, the production image will run a `gunicorn` process that listens on port 8000.

There is also a [`docker-compose.prod.yml`](docker-compose.prod.yml) file that runs the container in production mode behind an nginx proxy. This is primarily useful for testing the production settings; generally our real production deployments will be using AWS Elastic Container Service, Relational Database Service, and Elastic Load Balancer.

To run in production mode, run:

```sh
$ docker-compose --file docker-compose.prod.yml up --build
```

Then visit http://localhost:1337/admin in the browser.

## Additional Commands

The `Dockerfile` is also set up to allow containers to run individual shell commands, rather than launching either the development or production servers. The primary use case for this is to enable individual ECS tasks, like the Directory Screens reindexing task. As an example, the launch command for this is below:

```sh
$ docker run -it --rm --env-file .env ghcr.io/seas-computing/sec-directory-server:stable python manage.py shell --command "from feedperson.utils import load_feed_people; load_feed_people()"
```

When running the container with an additional shell command like this, the [`app/entrypoint.sh` script](app/entrypoint.sh) will not run the `gunicorn` or development server processes; it will run the command specified, within the `/app` directory in the container. If the `DATABASE` environment variable is set to `postgres`, it will wait for the database defined by `SQL_HOST` and `SQL_PORT` to become available before proceeding.

You can also force a container to run in production or development mode by passing `--production` or `--development` as the **only** arguments. Note that when running the tasks like this, Postgres and/or Nginx containers will already need to exist, and you will need to pass the existing docker network and a port mapping in as a parameters. For example:

```sh
# For Production mode
$ docker run -it --rm --network django-docker-template_default -p 127.0.0.1:8001:8000 --env-file .env django-docker-template_web --production

# For Development mode
$ docker run -it --rm --network django-docker-template_default -p 127.0.0.1:8001:8000 --env-file .env django-docker-template_web --development
```

You can also create new `EXEC_MODE` flags, configured in the `entrypoint.sh` file, again to enable specific tasks. For example, for reindexing the directory screens. See the [`entrypoint.sh`][directory_entrypoint.sh] file in the Directory Screens project for an example of this implementation.


With no arguments, the image will default to running in production mode.


[actions]: https://github.com/seas-computing/django-docker-template/actions
[directory_entrypoint.sh]: https://github.com/seas-computing/sec-directory-server/blob/main/app/entrypoint.sh
