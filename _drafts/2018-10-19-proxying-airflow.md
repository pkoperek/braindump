---
layout: post
title: Proxying Apache Airflow
comments: true
---

[Apache Airflow][1] is a great ETL tool and a very lively project. Using it has
many advantages (see comparisons [here][2] and [here][3]). You can control it
either through CLI utility called `airflow` or through a rich Web UI. The
latter provides many tools which make maintenance and debugging easier.

Deploying a webapplication such as Airflow's web UI raises some questions and
challenges (e.g. security considerations, how to load balance the traffic, how
to cover traffic with SSL etc). In many scenarios, accessing the UI through a
proxy would be a desired solution. 

Apache Airflow's web UI is a [WSGI][4] app which is deployed to [gunicorn][5]
server. [gunicorn authors strongly recommend to use it behind a proxy][6] but
they focus on [nginx][7] in their documentation. `nginx` is great, but in many
cases it is actually easier to configure Apache Httpd to do the job. From
project's perspective, performance gains we would get from nginx wouldn't
really matter much ([good read about it here][8]). In this
post I'll show you how to configure Apache to proxy requests to Apache Airflow.





[1]: apache airflow link
[2]: write down my evaluation of apache airflow vs luigi/azkaban etc
[3]: link to marton's evaluation
[4]: link to wsgi
[5]: https://github.com/benoitc/gunicorn
[6]: https://docs.gunicorn.org/en/stable/deploy.html?highlight=proxy
[7]: nginx server
[8]: https://djangodeployment.com/2016/11/15/why-nginx-is-faster-than-apache-and-why-you-neednt-necessarily-care/
