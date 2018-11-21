
# Curator

This is a dockerized version of the elasticsearch [curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html) tool to manage time-based indices.

The docker image for the curator is based on the [python:3.6-alpine](https://hub.docker.com/_/python/)

# Usage

## docker-compose.yml

```yaml
version: '3'

services:

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.2.4
    ports:
    - 9200:9200

  curator:
    image: markhendriks/curator
    depends_on:
      - elasticsearch
    volumes:
        - config/:/config
```

## action_file.yml

```yaml
actions:
  1:
    action: delete_indices
    description: >-
       Delete indices older than 1 day (based on index name), for logstash-
       prefixed indices. Ignore the error if the filter does not result in an
       actionable list of indices (ignore_empty_list) and exit cleanly.
    options:
      ignore_empty_list: True
      timeout_override:
      continue_if_exception: True
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: logstash-
      exclude:
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 1
      exclude:
```

## config_file.yml
Remember, leave a key empty if there is no value.  None will be a string, not a Python "NoneType"

```yaml
client:
  hosts:
    - elasticsearch
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth: ''
  timeout: 120
  master_only: True
logging:
  loglevel: INFO
  logfile:
  logformat: default
```

## crontab.txt
This file contains the cronjobs that will be run inside the container.

There are 2 cronjobs:

```console
*/3 * * * * curator --config /config/config_file.yml /config/action_file.yml
* * * * * /usr/bin/crontab /config/crontab.txt
```

* The first job will run the curator every 3 hours.
* The second job will load conjobs defined in [./config/crontab.txt](./config/crontab.txt) every minute. This enables you to change the cronjobs on the fly.

# Curator documentation

You can find the full curator documentation [here](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html).
