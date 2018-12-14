---
description: Use docker to run a distributed airflow on my own machine
---

# Distributed airflow - local

**Prerequisite:**

* python 2.7-slim \(`docker pull python:2.7-slim`\)
* redis \(`docker pull redis`\)
* postgres \(`docker pull postgres`\)

### Start up the docker containers

* Redis

```bash
$ docker run -p 6379:6379 --name some-redis -d redis redis-server --appendonly yes
```

* Postgres

```bash
$ docker run -p 5432:5432 --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

* Master node \( webserver and scheduler\)

```bash
$ docker run -it -p 8080:8080 --name webserver --link some-redis:redis --link some-postgres:postgres -d python:2.7-slim
```

* Woker

```bash
$ docker run -it -p 5555:5555 -p 8793:8793 --name worker --link some-redis:redis --link some-postgres:postgres -d python:2.7-slim
```

### Airflow on master and worker

Repeat the steps for both 2 containers

Access the container `docker exec -it webserver bash`

* Install airflow and necessary configurations

```bash
$ apt-get update
$ apt-get install -y --no-install-recommends git make g++ libssl-dev libffi-dev
$ export AIRFLOW_GPL_UNIDECODE=yes
$ pip install apache-airflow==1.9.0
$ pip install celery[redis]
$ pip install psycopg2
$ apt-get install vim lsof -y

# Add AIRFLOW_HOME environment to .bash_rc
$ vim ~/.bashrc
export AIRFLOW_HOME=/home/airflow
$ source ~/.bashrc
```

* Config `airflow.cfg`  inside the `webserver` and `worker` node

```bash
$ airflow initdb
$ cd /home/airflow
$ vim airflow.cfg

# Edit airflow.cfg to our requirements
executor = CeleryExecutor
sql_alchemy_conn = postgresql+psycopg2://postgres:mysecretpassword@$POSTGRES_PORT_5432_TCP_ADDR/postgres
broker_url = redis://172.17.0.2/0
result_backend = redis://172.17.0.2/1
```

* Run `airflow worker` command to make sure that the `scheduler` and `worker` both point to same redis url. The result for both should look like:

![](../../.gitbook/assets/image%20%282%29.png)

* Simple dags for testing. Put the dag in both master and worker machine

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG('test_airflow', default_args=default_args, schedule_interval=timedelta(days=1))

t1 = BashOperator(
    task_id='print_hello',
    bash_command='echo "Hello from Task 1"',
    dag=dag)

t2 = BashOperator(
    task_id='print_world',
    bash_command='echo "World from Task 2"',
    retries=3,
    dag=dag)

t2.set_upstream(t1)
```

* Start up the airflow and check the result

```text
# On master machine
$ airflow start webserver
$ airflow scheduler

# On worker machine
$ airflow worker
```



