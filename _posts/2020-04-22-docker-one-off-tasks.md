---
layout: post
title: "Should we run database seed and migrations in Dockerfile?"
categories: [All]
tags: [docker, docker-compose, docker-compose up]
fullview: false
excerpt: This answers the question about one-off tasks in Dockerfile
comments: true
---

## A common question

docker-compose is a popular approach to manage development environments. A common question from docker newbies is: What steps should we put in Dockerfile?

More specifically, when working with full-stack frameworks that manage database migrations, e.g., Rails or Elixir/Phoenix, it is tempting to add `rails db:migrate` or `mix ecto.migrate` step in the dockerfile. The goal is to make `docker-compose up` the ONLY thing we need for setting up environment.

However, this approach is problematic. Let me explain why.

## One-off tasks are special
Imagine we don't have docker, and we are setting up our local environment. To setup database, we usually need the following steps:

- Create database
- Run migrations
- Seed database

If you look carefully, these are usually one-off tasks. Database create can fail if you run it the second time. Same as seeding, usually you intentionally design your seeding script to run idempotently.

The exception is running database migrations, because frameworks like Rails or Ecto assign timestamps to migrations, and running the migration command will check the existing state, and avoid repeating a migration the second time.

However, even for migrations, I argue that we should avoid running it in Dockerfile. This is particularly for dockerized production environments. Running migrations while building docker image means we can lose control when the migrations will be run. Some database migration could lead to downtime or requires coordination. So running it out of band may be the best option.

For these one-off tasks, they are better documented in README and run them outside of Dockerfile or docker-compose.yml

The other idea I found is to [treat them services, add them in `docker-compose.yml`](https://phauer.com/2018/local-development-docker-compose-seeding-stubs/), so developers run it ONCE, with commands like `docker-compose up -d db_setup`. But the trade-off here is we can't do `docker-compose up` all the time because it runs one-off tasks, too. And this has to be documented somewhere, too.

## Use a platform repo to manage shared services
It is common to see teams having one monolithic app and multiple other repos for microservices. We often use a separate a `platform` repo where there is a long `docker-compose.yml` that maps to each small repo. The purpose is to avoid having a different `docker-compose.yml` in each separate repos.

I've also seen a `docker-compose.yml` in a separate repo where we [mock the dependent service](https://phauer.com/2018/local-development-docker-compose-seeding-stubs/). This can be helpful if the dependent service is simple to mock, and has no or little state to keep.

## docker-compose `host` network mode?
Once I saw a fellow co-worker uses [`host` network mode](https://docs.docker.com/network/host/), which lets the docker container access the host services. The intension was to share services with the host and spin less docker serives. 

However, this is against the ideal of docker. The whole point is to run isolated environment. With `host` network mode, it introduces the dependencies on the host environment. 

Even in the case you have to host network, we can consider using the [special DNS name](https://docs.docker.com/docker-for-mac/networking/#use-cases-and-workarounds): `host.docker.internal`.

From my experience, using docker-compose [default network mode](https://docs.docker.com/compose/networking/), which spins a single network, is often the way we prefer.

## Concluding thoughts

When using docker-compose to manage development environments, it is helpful to distinguish the tasks which are safe to run on every docker build, and those one-off tasks.

Also, I discuss other aspects of docker-compose, e.g., running a platform repo to spin up shared services and why we should stick to docker-compose default network mode.

Thank you for read!