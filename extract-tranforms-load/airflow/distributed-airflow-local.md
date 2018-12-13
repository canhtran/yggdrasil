---
description: Use docker to run a distributed airflow on my own machine
---

# Distributed airflow - local

**Prerequisite:**

* python 2.7-slim \(`docker pull python:2.7-slim`\)
* redis \(`docker pull redis`\)
* postgres \(`docker pull postgres`\)

#### Start up the docker containers

* Redis

```text
$ docker run -p 6379:6379 --name some-redis -d redis redis-server --appendonly yes
```

* Postgres

```text
$ docker run -p 5432:5432 --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

* Master node \( webserver and scheduler\)

```text
$ docker run -it -p 8080:8080 --name webserver --link some-redis:redis --link some-postgres:postgres -d python:2.7-slim
```

* Woker

```text
$ docker run -it -p 5555:5555 -p 8793:8793 --name worker --link some-redis:redis --link some-postgres:postgres -d python:2.7-slim
```

#### Install airflow on master and worker

Repeat the steps for both 2 containers

Access the container `docker exec -it webserver bash`

```text
$ apt-get update
$ apt-get install -y --no-install-recommends git make g++ libssl-dev libffi-dev
$ pip install apache-airflow
$ pip install celery==3.1.7
$ pip install psycopg2
```

