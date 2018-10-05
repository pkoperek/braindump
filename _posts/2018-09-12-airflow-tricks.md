---
layout: post
title: Apache Airflow tricks
comments: true
---

_I will update this post from time to time with more learnings._

## Problem: I fixed problem in my pipeline but airflow doesn't see this.

Possible things you can do:
* check if you actually did fix it :)
* [try to refresh the DAG through UI](https://stackoverflow.com/questions/43606311/refreshing-dags-without-web-server-restart-apache-airflow)
* remove `*.pyc` files from the dags directory
* check if you have installed the dependencies in the right python environment
  (e.g. in the right version of system-wide python or in the right virtualenv)

## Problem: This DAG isn't available in the web server's DagBag object. It shows up in this list because the scheduler marked it as active in the metadata database.

* Refresh the DAG code from the UI
* Restart webserver - this did the trick in my case. Some people report that
  there might be a stalled gunicorn process. If restart doesn't help, try to
  find rogue processes and kill them manually ([source](https://stackoverflow.com/questions/43684434/airflow-new-dag-is-not-found-by-webserver), [source 2](https://stackoverflow.com/questions/41560614/airflow-this-dag-isnt-available-in-the-webserver-dagbag-object?rq=1))

## Problem: I want to delete a DAG

* Airflow 1.10 has a command for this: `airflow delete ...`
* Prior to 1.10, you can use following script:

```python
import sys
from airflow.hooks.postgres_hook import PostgresHook

dag_input = sys.argv[1]
hook=PostgresHook( postgres_conn_id= "airflow_db")

for t in ["xcom", "task_instance", "sla_miss", "log", "job", "dag_run", "dag" ]:
    sql="delete from {} where dag_id='{}'".format(t, dag_input)
    hook.run(sql, True)
```

(_tested: works like a charm :)_)

[Source](https://stackoverflow.com/questions/40651783/airflow-how-to-delete-a-dag/49683543)
