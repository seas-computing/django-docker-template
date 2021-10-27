# BASE
# This layer installs all of the system packages necessary for compiling/installing dependencies

# pull official base image
FROM python:3.8.0-alpine as base

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install psycopg2 dependencies
RUN apk update \
    && apk add postgresql-dev gcc python3-dev \
    musl-dev bash libffi-dev zlib zlib-dev make 

# Update pip and copy over reuirements 
RUN pip install --upgrade pip
COPY ./requirements.txt .


# DEVELOPMENT
# This creates a development image from our base image
# This keeps the system dependencies so we can add additional packages without
# needing to rebuild the whole container
FROM base as development

# Install requirements.txt, so subsequent code changes don't require a full reinstall
RUN pip install -r requirements.txt

# Set a standard workdir
WORKDIR /usr/src/app

# Copy in the code and run the app
COPY ./app .
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
CMD ["--development"]

# BUILD
# An intermediate stage that installs deps and builds the psychopg2 dependency
FROM base as builder

# Install the depenencies in an /install dir
# to be copied over to the final image
RUN mkdir /install
RUN pip install --prefix=/install -r requirements.txt


# PRODUCTION 
# Our final image, based off a fresh alpine image
FROM python:3.8.0-alpine as production

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

ENV APP_HOME=/home/app/web

# create the app user, with group app and homedir of /home/app
RUN adduser -D app && mkdir -p $APP_HOME/app/static && mkdir $APP_HOME/app/media && chown -R app:app $APP_HOME
WORKDIR $APP_HOME

# We only need the C postgres library, since psychopg2 wraps around that
RUN apk update && apk add libpq

# Copy the built requrements from the builder
# See: https://github.com/psycopg/psycopg2/issues/684#issuecomment-453803835
COPY --from=builder /install /usr/local

# copy project
COPY --chown=app:app . .

# change to the app user
USER app

# run entrypoint.prod.sh
ENTRYPOINT ["/home/app/web/entrypoint.sh"]
CMD ["--production"]
