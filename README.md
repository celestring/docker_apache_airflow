# Running Airflow in Docker



## Fetching docker-compose.yaml

To deploy Airflow on Docker Compose, you should fetch [docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/2.7.0/docker-compose.yaml).

```shell
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.7.0/docker-compose.yaml'
```

docker-compose.yaml 파일에는 다음과 같은 Container가 포함됨

- `airflow-scheduler` 
  
  The [scheduler](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/scheduler.html) monitors all tasks and DAGs, then triggers the task instances once their dependencies are complete.

- `airflow-webserver` 
  
  The webserver is available at `http://localhost:8080`.

- `airflow-worker` 
  
  The worker that executes the tasks given by the scheduler.

- `airflow-triggerer` 
  
  The triggerer runs an event loop for deferrable tasks.

- `airflow-init` 
  
  The initialization service.

- `postgres` 
  
  The database.

- `redis` 
  
  broker that forwards messages from scheduler to worker.

Optionally, you can enable flower by adding `--profile flower` option, e.g. `docker compose --profile flower up`, or by explicitly specifying it on the command line e.g. `docker compose up flower`.

- `flower` - [The flower app](https://flower.readthedocs.io/en/latest/) for monitoring the environment. It is available at `http://localhost:5555`.

Container의 일부 디렉터리를 마운트 하며, Container와 Docker를 수행하는 경로 간 연동

- `./dags` - you can put your DAG files here.

- `./logs` - contains logs from task execution and scheduler.

- `./config` - you can add custom log parser or add `airflow_local_settings.py` to configure cluster policy.

- `./plugins` - you can put your [custom plugins](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/plugins.html) here.

## Initializing Environment

Otherwise the files created in `dags`, `logs` and `plugins` will be created with `root` user ownership. You have to make sure to configure them for the docker-compose:

```shell
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

For other operating systems, you may get a warning that `AIRFLOW_UID` is not set, but you can safely ignore it. You can also manually create an `.env` file in the same folder as `docker-compose.yaml` with this content to get rid of the warning:

```shell
AIRFLOW_UID=50000
```

### Initialize the database

On **all operating systems**, you need to run database migrations and create the first user account. To do this, run.

```shell
docker compose up airflow-init
```

After initialization is complete, you should see a message like this:

```
airflow-init_1       | Upgrades done
airflow-init_1       | Admin user airflow created
airflow-init_1       | 2.7.0
start_airflow-init_1 exited with code 0
```

The account created has the login `airflow` and the password `airflow`.

## Running Airflow

Now you can start all services:

```shell
docker compose up
```

In a second terminal you can check the condition of the containers and make sure that no containers are in an unhealthy condition:

```shell
CONTAINER ID   IMAGE                  COMMAND                   CREATED              STATUS                        PORTS                    NAMES
3e5f8845449f   apache/airflow:2.7.0   "/usr/bin/dumb-init …"   About a minute ago   Up About a minute (healthy)   8080/tcp                 docker_apache_airflow-airflow-worker-1
317d82e62308   apache/airflow:2.7.0   "/usr/bin/dumb-init …"   About a minute ago   Up About a minute (healthy)   0.0.0.0:8080->8080/tcp   docker_apache_airflow-airflow-webserver-1
7bc5f185d497   apache/airflow:2.7.0   "/usr/bin/dumb-init …"   About a minute ago   Up About a minute (healthy)   8080/tcp                 docker_apache_airflow-airflow-triggerer-1
b802acae6c33   apache/airflow:2.7.0   "/usr/bin/dumb-init …"   About a minute ago   Up About a minute (healthy)   8080/tcp                 docker_apache_airflow-airflow-scheduler-1
234d30238fb1   postgres:13            "docker-entrypoint.s…"   4 minutes ago        Up 4 minutes (healthy)        5432/tcp                 docker_apache_airflow-postgres-1
be64727a299c   redis:latest           "docker-entrypoint.s…"   4 minutes ago        Up 4 minutes (healthy)        6379/tcp                 docker_apache_airflow-redis-1
```
