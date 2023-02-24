# Install-Airflow-using-Standard-Docker

## Running Airflow in Docker

### Before you begin
This procedure assumes familiarity with Docker and Docker Compose. If you haven’t worked with these tools before, you should take a moment to run through the Docker Quick Start (especially the section on Docker Compose) so you are familiar with how they work.

Follow these steps to install the necessary tools, if you have not already done so.

- Install Docker Community Edition (CE) on your workstation. Depending on your OS, you may need to configure Docker to use at least 4.00 GB of memory for the Airflow containers to run properly. Please refer to the Resources section in the Docker for Windows or Docker for Mac documentation for more information.

- Install Docker Compose v1.29.1 or newer on your workstation.

Older versions of docker-compose do not support all the features required by the Airflow docker-compose.yaml file, so double check that your version meets the minimum version requirements.

![image](https://user-images.githubusercontent.com/77673886/221151934-00e484db-3824-48a8-8bc6-dfb4a864872b.png)

```ruby
docker run --rm "debian:bullseye-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'
```

### Fetching docker-compose.yaml
To deploy Airflow on Docker Compose, you should fetch docker-compose.yaml.

```ruby
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.1/docker-compose.yaml'
```
This file contains several service definitions:

- airflow-scheduler - The scheduler monitors all tasks and DAGs, then triggers the task instances once their dependencies are complete.

- airflow-webserver - The webserver is available at http://localhost:8080.

- airflow-worker - The worker that executes the tasks given by the scheduler.

- airflow-init - The initialization service.

- postgres - The database.

- redis - The redis - broker that forwards messages from scheduler to worker.

Optionally, you can enable flower by adding --profile flower option, e.g. docker compose --profile flower up, or by explicitly specifying it on the command line e.g. docker compose up flower.

- flower - The flower app for monitoring the environment. It is available at http://localhost:5555.

All these services allow you to run Airflow with CeleryExecutor. 
https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/celery.html

For more information, see Architecture Overview.
https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html

Some directories in the container are mounted, which means that their contents are synchronized between your computer and the container.

- ./dags - you can put your DAG files here.

- ./logs - contains logs from task execution and scheduler.

- ./plugins - you can put your custom plugins here.

This file uses the latest Airflow image (apache/airflow). If you need to install a new Python library or system library, you can build your image.

### Initializing Environment
Before starting Airflow for the first time, you need to prepare your environment, i.e. create the necessary files, directories and initialize the database.

#### Setting the right Airflow user
On Linux, the quick-start needs to know your host user id and needs to have group id set to 0. Otherwise the files created in dags, logs and plugins will be created with root user ownership. You have to make sure to configure them for the docker-compose:

```ruby
mkdir -p ./dags ./logs ./plugins
echo -e "AIRFLOW_UID=$(id -u)" > .env
```

See Docker Compose environment variables
https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html#docker-compose-env-variables

For other operating systems, you may get a warning that AIRFLOW_UID is not set, but you can safely ignore it. You can also manually create an .env file in the same folder as docker-compose.yaml with this content to get rid of the warning:

```ruby
AIRFLOW_UID=50000
```

#### Initialize the database
On all operating systems, you need to run database migrations and create the first user account. To do this, run.

```ruby
docker compose up airflow-init
```

After initialization is complete, you should see a message like this:

```
airflow-init_1       | Upgrades done
airflow-init_1       | Admin user airflow created
airflow-init_1       | 2.5.1
start_airflow-init_1 exited with code 0
```

The account created has the login airflow and the password airflow.

### Cleaning-up the environment

The docker-compose environment we have prepared is a “quick-start” one. It was not designed to be used in production and it has a number of caveats - one of them being that the best way to recover from any problem is to clean it up and restart from scratch.

The best way to do this is to:

- Run docker compose down --volumes --remove-orphans command in the directory you downloaded the docker-compose.yaml file

- Remove the entire directory where you downloaded the docker-compose.yaml file rm -rf '<DIRECTORY>'

- Run through this guide from the very beginning, starting by re-downloading the docker-compose.yaml file

### Running Airflow
  
Now you can start all services:

```ruby
docker compose up
```
  
***Note***

docker-compose is old syntax. Please check Stackoverflow.
https://stackoverflow.com/questions/66514436/difference-between-docker-compose-and-docker-compose

In a second terminal you can check the condition of the containers and make sure that no containers are in an unhealthy condition:
  
```ruby 
$ docker ps 
  
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                    PORTS                              NAMES
247ebe6cf87a   apache/airflow:2.5.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-worker_1
ed9b09fc84b1   apache/airflow:2.5.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    8080/tcp                           compose_airflow-scheduler_1
7cb1fb603a98   apache/airflow:2.5.1   "/usr/bin/dumb-init …"   3 minutes ago    Up 3 minutes (healthy)    0.0.0.0:8080->8080/tcp             compose_airflow-webserver_1
74f3bbe506eb   postgres:13            "docker-entrypoint.s…"   18 minutes ago   Up 17 minutes (healthy)   5432/tcp                           compose_postgres_1
0bd6576d23cb   redis:latest           "docker-entrypoint.s…"   10 hours ago     Up 17 minutes (healthy)   0.0.0.0:6379->6379/tcp             compose_redis_1
```
  
### Accessing the environment
After starting Airflow, you can interact with it in 3 ways:
- by running CLI commands.
- via a browser using the web interface.
- using the REST API.

#### Running the CLI commands
  You can also run CLI commands, but you have to do it in one of the defined airflow-* services. For example, to run airflow info, run the following command:
  ```ruby
  docker compose run airflow-worker airflow info
  ```
  
  If you have Linux or Mac OS, you can make your work easier and download a optional wrapper scripts that will allow you to run commands with a simpler command.
  
  ```ruby
  curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.1/airflow.sh'
  chmod +x airflow.sh
  ```
  Now you can run commands easier.
  ```ruby
  ./airflow.sh info
  ```
  You can also use bash as parameter to enter interactive bash shell in the container or python to enter python container.
```ruby
  ./airflow.sh bash
  ./airflow.sh python
```  
#### Accessing the web interface
Once the cluster has started up, you can log in to the web interface and begin experimenting with DAGs.
  The webserver is available at: http://localhost:8080. The default account has the login airflow and the password airflow.
  
#### Sending requests to the REST API
  Basic username password authentication is currently supported for the REST API, which means you can use common tools to send requests to the API.
  The webserver is available at: http://localhost:8080. The default account has the login airflow and the password airflow.
  
  Here is a sample curl command, which sends a request to retrieve a pool list:
```ruby  
  ENDPOINT_URL="http://localhost:8080/"
  curl -X GET  \
    --user "airflow:airflow" \
    "${ENDPOINT_URL}/api/v1/pools"
```  
### Cleaning up
To stop and delete containers, delete volumes with database data and download images, run:
```ruby
docker compose down --volumes --rmi all
```  
### Using custom images
When you want to run Airflow locally, you might want to use an extended image, containing some additional dependencies - for example you might add new python packages, or upgrade airflow providers to a later version. This can be done very easily by specifying build: . in your docker-compose.yaml and placing a custom Dockerfile alongside your docker-compose.yaml. Then you can use docker compose build command to build your image (you need to do it only once). You can also add the --build flag to your docker compose commands to rebuild the images on-the-fly when you run other docker compose commands.

Examples of how you can extend the image with custom providers, python packages, apt packages and more can be found in Building the image.
  https://airflow.apache.org/docs/docker-stack/build.html
  
  Networking
In general, if you want to use Airflow locally, your DAGs may try to connect to servers which are running on the host. In order to achieve that, an extra configuration must be added in docker-compose.yaml. For example, on Linux the configuration must be in the section services: airflow-worker adding extra_hosts: - "host.docker.internal:host-gateway"; and use host.docker.internal instead of localhost. This configuration vary in different platforms. Please check the Docker documentation for Windows and Mac for further information.  
  
  https://docs.docker.com/desktop/networking/#use-cases-and-workarounds
  
## FAQ: Frequently asked questions
ModuleNotFoundError: No module named 'XYZ'
The Docker Compose file uses the latest Airflow image (apache/airflow). If you need to install a new Python library or system library, you can customize and extend it.
https://hub.docker.com/r/apache/airflow
https://airflow.apache.org/docs/docker-stack/index.html

### What’s Next?
From this point, you can head to the Tutorials section for further examples or the How-to Guides section if you’re ready to get your hands dirty.
  Tutorial: https://airflow.apache.org/docs/apache-airflow/stable/tutorial/index.html
  How-to-Guide: https://airflow.apache.org/docs/apache-airflow/stable/howto/index.html
  
  
### Environment variables supported by Docker Compose
Do not confuse the variable names here with the build arguments set when image is built. The AIRFLOW_UID build arg defaults to 50000 when the image is built, so it is “baked” into the image. On the other hand, the environment variables below can be set when the container is running, using - for example - result of id -u command, which allows to use the dynamic host runtime user id which is unknown at the time of building the image.
  
  ![image](https://user-images.githubusercontent.com/77673886/221150992-3c6851c7-b531-476f-ae06-b210ba392995.png)

  Note

Before Airflow 2.2, the Docker Compose also had AIRFLOW_GID parameter, but it did not provide any additional functionality - only added confusion - so it has been removed.

Those additional variables are useful in case you are trying out/testing Airflow installation via Docker Compose. They are not intended to be used in production, but they make the environment faster to bootstrap for first time users with the most common customizations.
  
  ![image](https://user-images.githubusercontent.com/77673886/221151185-2249ee4a-a96b-41ac-80a1-1b2967d013af.png)


  
  
  
  
  
  


