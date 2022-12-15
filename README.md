# wait-for-postgres

`wait-for-postgres.py` is a python script that will wait on the availability of a
database  It is useful for synchronizing the spin-up of
interdependent services, such as linked docker containers.

## Usage

```text
python3 wait-for-postgres.py -s | --host HOST [-p | --port PORT]  [-t | --timeout TIMEOUT] -c | --command ARGS
-s HOST | --host=HOST           Host or IP
-p PORT | --port=PORT           TCP port (Optional, default 5432)
-t TIMEOUT | --timeout=TIMEOUT  Timeout in seconds (Optional, default 2 sec)
-c COMMAND | --command=COMMAND ARGS    Execute command with args
```

## Examples

For example, use Asterisk image [andrius/asterisk]([/guides/content/editing-an-existing-page](https://hub.docker.com/r/andrius/asterisk)), and in Dockerfile add next lines:

```
FROM andrius/asterisk

RUN apk add --update --no-cache python3 py3-pip
RUN pip3 install psycopg2-binary

WORKDIR /etc/asterisk
RUN rm -rf ./*
COPY configs .
WORKDIR /var/lib/asterisk/sounds/
COPY sounds .
RUN apk add --update less psqlodbc asterisk-odbc asterisk-pgsql \
  &&  rm -rf /var/cache/apk/*
WORKDIR /
# necessary wrapping script for checking database status
COPY wait-for-postgres.py .
ENTRYPOINT ["/usr/bin/python3", "wait-for-postgres.py", "--host", "172.16.238.5", "--command", "./docker-entrypoint.sh"]
```