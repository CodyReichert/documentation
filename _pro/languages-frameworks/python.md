---
title: Using Python In CI/CD with Docker and Codeship Pro
shortTitle: Python
menus:
  pro/languages:
    title: Python
    weight: 2
tags:
  - python
  - languages
  - docker
  - flask
  - django

redirect_from:
  - /docker-integration/python/
---

<div class="info-block">
You may want to read the [Codeship Pro Getting Started Guide]({% link _pro/quickstart/getting-started.md %}) to learn more about how Codeship Pro works. You can also [watch a short demo video here](https://codeship.com/features/pro).
</div>

* include a table of contents
{:toc}

## Python on Codeship Pro

Any Python framework or tool that can run inside a Docker container will run on Codeship Pro. This documentation article will highlight simple configuration files for a Node-based Dockerfile with nosetest and py.test.

## Services File

The following is an example of a [Codeship Services file]({% link _pro/builds-and-configuration/services.md %}). Note that it is using a [PostgreSQL image](https://hub.docker.com/_/postgres/) and a [Redis image](https://hub.docker.com/_/redis/) via the Docker Hub as linked services.

When accessing other containers please be aware that those services do not run on `localhost`, but on a different host, e.g. `postgres` or `mysql`. If you reference `localhost` in any of your configuration files you will have to change that to point to the service name of the service you want to access. Setting them through environment variables and using those inside of your configuration files is the cleanest approach to setting up your build environment.

```yaml
project_name:
  build:
    image: organisation_name/project_name
    dockerfile: Dockerfile
  links:
    - redis
    - postgres
  environment:
    - DATABASE_URL=postgres://postgres@postgres/YOUR_DATABASE_NAME
    - REDIS_URL=redis://redis
redis:
  image: redis:3
postgres:
  image: postgres:9.6
```

## Steps File

The following is an example of a [Codeship Steps file]({% link _pro/builds-and-configuration/steps.md %}).

Note that every step runs in isolated containers, so changes made on one step do not persist to the next step.  Because of this, any required setup commands, such as migrating a database, should be done via a custom Dockerfile, via a `command` or `entrypoint` on a service or repeated on every step.

```yaml
- name: ci
  type: parallel
  steps:
  - service: project_name
    command: nosetests tests/unit
  - service: project_name
    command: nosetests tests/acceptance
  - service: project_name
    command: py.test tests/unit
  - service: project_name
    command: py.test tests/acceptance
```

## Dockerfile

Following is an example Dockerfile with inline comments describing each step in the file. The Dockerfile shows the different ways you can install extensions or dependencies so you can extend it to fit exactly what you need. Also take a look at the Python image documentation on [the Docker Hub](https://hub.docker.com/_/python/).

```
# Starting from Python 3 base image
FROM python:3

# Set the WORKDIR to /app so all following commands run in /app
WORKDIR /app

# Adding requirements files before installing requirements
COPY requirements.txt dev-requirements.txt ./

# Install requirements and dev requirements through pip. Those should include
# nostest, pytest or any other test framework you use
RUN pip install -r requirements.txt -r dev-requirements.txt

# Adding the whole repository to the image
COPY . ./
```

## Notes And Known Issues

Because of version and test dependency issues, it is advised to try using [the Jet CLI]({% link _pro/builds-and-configuration/cli.md %}) to debug issues locally via `jet steps`.

## Caching

You can enable caching per service in your Services file.

You can [read more about how caching works on Codeship Pro here]({{ site.baseurl }}{% link _pro/builds-and-configuration/caching.md %}).
