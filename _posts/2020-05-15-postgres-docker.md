---
layout: post
title: "Run Postgres in Docker with One Command"
categories: [All, Technical]
tags: [sql, postgres, docker]
fullview: false
excerpt: How to run Postgres in Docker without installing local postgres
comments: true
---

## Prerequites

You have installed [Docker](https://www.docker.com/). 

For Mac OS X, you can follow [here](https://docs.docker.com/docker-for-mac/install/) to install Docker for Mac.

## THE Command

```bash
$ docker run -d --name my_postgres -v my_dbdata:/var/lib/postgresql/data -p 5432:5432 postgres:11.2-alpine
```

The command comes from [this site](https://www.saltycrane.com/blog/2019/01/how-run-postgresql-docker-mac-local-development/).

### Explanation

`-d` is running the container in detached mode, so you can continue your shell session to run other commands.

`--name my_postgres` is giving the container a name.

`-v my_dbdata:/var/lib/postgresql/data` is using a [Docker volume](https://docs.docker.com/storage/volumes/). You can view it as letting the docker container access a host directory.

You can check the volume info with command:

```bash
$ docker volume ls                                                                                                                  
DRIVER              VOLUME NAME
local               my_dbdata

$ docker volume inspect my_dbdata                                                                                                   
[
    {
        "CreatedAt": "2020-05-15T11:42:46Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my_dbdata/_data",
        "Name": "my_dbdata",
        "Options": null,
        "Scope": "local"
    }
]
```

The `-p 5432:5432` part means it is mapping the host port 5432 to the container 5432, which is the default port Postgres listens to. With this,
you can't access the running Postgres via host port 5432. 

If you have a local copy postgres running on port `5432`, you can map to a different host port, say `15432`, you can change the part like this: `-p 15432:5432`.

Finally, the command runs a Docker container based on `postgres:11.2-alpine`, a [small image running Alpine Linux](https://hub.docker.com/_/alpine).
The base image is only 5MB. 

### How to access the database
You can use any database client to access the database. e.g., with psql:

```
$ psql -U postgres --port 5432 -h localhost
```
The default username and password are both `postgres`.

## Why run PostgreSQL in Docker? Why not install directly?

Because it's hard to install directly. There could be other dependencies that fail your installation and you don't know how to fix.

Running inside Docker container doesn't have this issue, because it is an isolated environment. When you `docker pull postgres-alpine`, you pull
the entire running environment which is setup properly for you. 

Also, because Docker is isolated from your local machine. You don't have to worry about messing with your other dependencies.

Finally, you can run multiple versions of the Postgres easily with Docker. It is a matter of pulling the image with the right version tag.

## Closing thoughts

Hopefully you now know how simple it is to run Postgres in docker with one command.

If you interested in learning more about Postgres or SQL, I'm [recording a course](http://blog.junjizhi.com/all/2020/05/16/launch-backtosql.html) about SQL. You can sign up at [BackToSQL.com](https://backtosql.com/).

Thank you for your read!