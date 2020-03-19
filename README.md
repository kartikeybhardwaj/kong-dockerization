# Kong and Konga Dockerization

**Kong** is the world's most popular open source microservice API gateway. Use Kong to secure, manage and orchestrate microservice APIs.
**Konga** is an open source tool for managing Kong API Gateway.

#### Plan of action

1. Pull postgres:9.6, kong:latest, and pantsel/konga:next images
2. Write docker-compose file for kong and konga database
3. Write docker-compose file for kong and konga database preparation and migrations
4. Write docker-compose file for kong and konga
5. Execution and access

#### Pull images

    docker pull postgres:9.6
    docker pull kong:latest
    docker pull pantsel/konga:next

Show pulled image by executing

    docker images

#### Write docker-compose file for kong and konga database

Create a file named **docker-compose-kongdb.yml** for setup your docker-compose stack.

NOTE: This database serves both kong and konga.

##### Explanation of docker-compose-kongdb file

- `version` is a version of docker-compose file format, you can change to the latest version.
- `kong-database` is service name, you can change the name whatever you want.
- `container_name` is a name for your container.
- `environment` is a way to define environment variables. Here, we've defined **POSTGRES_DB**, **POSTGRES_USER** and **POSTGRES_PASSWORD**.
- `volumes` to define a file/folder that you want to use for the container.
- `./kong-data:/var/lib/postgresql/data` means you want to set data on container persist on you local folder named **kong-data**. **/var/lib/postgresql/data** is a folder that is already created inside the postgres container.
- `ports` is to define which ports you want to expose and define, in this case we're using default postgres port **5432** and exposing it to host machine.
- `ntw-kong` is used as a bridge to connect all the three services in a network.

#### Write docker-compose file for kong and konga database preparation and migrations

Create a file named **docker-compose-kong-konga-prepare.yml** for setup your docker-compose stack.
NOTE: This is to migrate data for kong and prepare konga database.

##### Explanation of docker-compose-kong-konga-prepare file

- `version` is a version of docker-compose file format, you can change to the latest version.
- `kong-migrations-bootstrap`, `kong-migrations-up` and `konga-prepare` are service names, you can change the names to whatever you want.
- `container_name` is a name for your container.
- `command` means the command to execute in container.
- `command: kong migrations bootstrap` you need to run kong migrations bootstrap only once in the entire lifecycle of a database for kong.
- `command: kong migrations up && kong migrations finish` the full migration is split into two steps, which are performed via commands `kong migrations up` (which does only non-destructive operations) and `kong migrations finish` (which puts the database in the final expected state for Kong 1.5.0).
- `command: "-c prepare -a postgres -u <YOUR_POSTGRES_CONNECTION_URI>"` prepares konga database.
- `environment` is a way to define environment variables. Here, we've defined **KONG_DATABASE**, **KONG_PG_HOST**, **KONG_PG_DATABASE**, **KONG_PG_USER** and **POSTGRES_PASSWORD**.
- `ntw-kong` is used as a bridge to connect all the three services in a network.

#### Write docker-compose file for kong and konga

Create a file named **docker-compose-kong-konga.yml** for setup your docker-compose stack.

NOTE: This serves kong and konga.

##### Explanation of docker-compose-kong-konga file

- `version` is a version of docker-compose file format, you can change to the latest version.
- `kong` and `konga` are service names, you can change the names to whatever you want.
- `container_name` is a name for your container.
- `environment` is a way to define environment variables. Here, we've defined **KONG_DATABASE**, **KONG_PG_HOST**, **KONG_PG_DATABASE**, **KONG_PG_USER**, **POSTGRES_PASSWORD** and **KONG_ADMIN_LISTEN** for kong and **DB_ADAPTER**, **DB_HOST**, **DB_USER**, **DB_PASSWORD**, **DB_DATABASE**, **NODE_ENV** for konga.
- `volumes` to define a file/folder that you want to use for the container.
- `./konga-data:/app/kongadata` means you want to set data on container persist on you local folder named **konga-data**. **/app/kongadata** is a folder that is already created inside the konga container.
- `ports` is to define which ports you want to expose and define, in this case we're using default kong ports **8000, 8001, 8443, 8444** and konga port **1337** and exposing it to host machine.
- `ntw-kong` is used as a bridge to connect all the three services in a network.

#### Execution and access

Now run docker-compose files step-wise.

01. Run `docker-compose -f docker-compose-kongdb.yml up` to start db.
02. Run `docker-compose -f docker-compose-kong-konga-prepare.yml up` to bootstrap db for kong and migrate it. It also prepares db for konga. Read the logs carefully, if all the three services didn't run as they cancelled out because of other services, you can run them again. No harm in doing that.
03. Run `docker-compose -f docker-compose-kong-konga.yml up` to start kong and konga.

Type `docker ps` to see our running containers.

Once the services are up and running, open your browser and open the url http://localhost:1337/. That's konga, waiting for setup. Once you set it up by providing admin credentials, you'll need to create a connection to the runnning kong service.

`KONG ADMIN URL = http://kong:8001`

You should target `http://localhost:8000` for kong which in turn will upstream to your services.

You are now good to create services and routes associated with them.

#### References

- [Install Elasticsearch with Docker](https://konghq.com/)
- [docker-kong/compose/docker-compose.yml](https://github.com/Kong/docker-kong/blob/master/compose/docker-compose.yml)
- [More than just another GUI to KONG Admin API](https://github.com/pantsel/konga)
- [pantsel/docker-compose.yml](https://gist.github.com/pantsel/73d949774bd8e917bfd3d9745d71febf)
