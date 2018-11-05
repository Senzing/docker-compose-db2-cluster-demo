# docker-compose-db2-cluster-demo

## Overview

This docker formation brings up the following docker containers:

1. *senzing/db2express-c*
1. *senzing/python-db2-demo*

Also shown in the demonstration are commands to run the following Docker images:

1. *senzing/db2* in [Add Senzing schemas](#add-senzing-schemas)
1. *senzing/g2loader-db2* in [Add content](#add-content)
1. *senzing/g2command-db2* in [Run G2Command.py](#run-g2commandpy)

### Contents

1. [Preparation](#preparation)
    1. [Set environment variables](#set-environment-variables)
    1. [Clone repository](#clone-repository)
    1. [Software](#software)
    1. [Docker images](#docker-images)
1. [Run Docker formation](#run-docker-formation)
    1. [Create SENZING_DIR](#create-senzing_dir)
    1. [Set environment variables](#set-environment-variables)
    1. [Launch docker formation](#launch-docker-formation)
    1. [Add Senzing schemas](#add-senzing-schemas)
    1. [Add content](#add-content)
    1. [Run G2Command.py](#run-g2commandpy)
1. [Cleanup](#cleanup)

## Preparation

### Set environment variables

These variables may be modified, but do not need to be modified.
The variables are used throughout the installation procedure.

```console
export GIT_ACCOUNT=senzing
export GIT_REPOSITORY=docker-compose-db2-cluster-demo
```

Synthesize environment variables.

```console
export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
export GIT_REPOSITORY_URL="https://github.com/${GIT_ACCOUNT}/${GIT_REPOSITORY}.git"
```

### Clone repository

Get repository.

```console
mkdir --parents ${GIT_ACCOUNT_DIR}
cd  ${GIT_ACCOUNT_DIR}
git clone ${GIT_REPOSITORY_URL}
```

### Software

The following software programs need to be installed.

#### docker

```console
docker --version
docker run hello-world
```

#### docker-compose

```console
docker-compose --version
```

### Docker images

1. Because an independent download is needed for the DB2 ODBC client, the
   [senzing/python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base)
   docker image must be manually built.
   Follow the build instructions at
   [github.com/Senzing/docker-python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base#build)

1. Verify `senzing/python-db2-cluster-base` is a local image.

    ```console
    docker images
    ```

1. Build the following docker images.

    ```console
    docker build --tag senzing/db2express-c            https://github.com/senzing/docker-db2express-c.git
    docker build --tag senzing/db2                     https://github.com/senzing/docker-db2.git
    docker build --tag senzing/python-db2-cluster-demo https://github.com/senzing/docker-python-db2-cluster-demo.git
    docker build --tag senzing/g2loader-db2-cluster    https://github.com/senzing/docker-g2loader-db2-cluster.git
    docker build --tag senzing/g2command-db2-cluster   https://github.com/senzing/docker-g2command-db2-cluster.git
    ```

## Run Docker formation

### Create SENZING_DIR

If you do not already have an `/opt/senzing` directory on your local system, visit
[HOWTO - Create SENZING_DIR](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/create-senzing-dir.md).

### Set environment variables for docker

1. For explanation of environment variables, see:
    1. [senzing/docker-python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base#set-environment-variables-for-demonstration)
    1. [senzing/docker-db2express-c](https://github.com/Senzing/docker-db2express-c#run-docker-container)
1. **DB2_NETWORK** -
   The network created by `docker-compose`.  To view, run `docker network ls`.
1. The values in the example below are specific to
   [docker-compose.yaml](docker-compose.yaml).
1. Example:

    ```console
    export SENZING_DIR=/opt/senzing

    export DB2_HOST_CORE=senzing-db2-core
    export DB2_HOST_RES=senzing-db2-res
    export DB2_HOST_LIBFE=senzing-db2-libfe

    export DB2_PORT_CORE=50000
    export DB2_PORT_RES=50000
    export DB2_PORT_LIBFE=50000

    export DB2_STORAGE_CORE=/storage/docker/senzing/docker-compose-db2-cluster-demo-core
    export DB2_STORAGE_RES=/storage/docker/senzing/docker-compose-db2-cluster-demo-res
    export DB2_STORAGE_LIBFE=/storage/docker/senzing/docker-compose-db2-cluster-demo-libfe

    export DB2_DATABASE_ALIAS_CORE=G2_CORE
    export DB2_DATABASE_ALIAS_RES=G2_RES
    export DB2_DATABASE_ALIAS_LIBFE=G2_LIBFE

    export DB2_USERNAME_CORE=db2inst1
    export DB2_USERNAME_RES=db2inst1
    export DB2_USERNAME_LIBFE=db2inst1

    export DB2_PASSWORD_CORE=db2inst1
    export DB2_PASSWORD_RES=db2inst1
    export DB2_PASSWORD_LIBFE=db2inst1

    export DB2_NETWORK=dockercomposedb2clusterdemo_backend
    ```

### Launch docker formation

```console
cd ${GIT_REPOSITORY_DIR}
docker-compose up
```

The database storage will be on the local system at ${db2_STORAGE}.
The default database storage path is `/storage/docker/senzing/docker-compose-db2-cluster-demo`.

### Add Senzing schemas

In a separate terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command.

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${DB2_NETWORK} \
      --env DB2INST1_PASSWORD=db2inst1 \
      --env LICENSE="accept" \
      senzing/db2
    ```

1. Become "db2inst1" user. In docker container, run

    ```console
    su - db2inst1
    ```

1. Create database on "CORE" DB2 server. In docker container, run

    ```console
    db2 catalog tcpip node G2_CORE remote senzing-db2-core server 50000
    db2 attach to G2_CORE user db2inst1 using db2inst1
    db2 create database g2 using codeset utf-8 territory us
    db2 connect to g2 user db2inst1 using db2inst1
    db2 -tf /opt/senzing/g2/data/g2core-schema-db2-create.sql | tee /tmp/g2schema.out
    db2 connect reset
    ```

1. Create database on "RES" DB2 server. In docker container, run

    ```console
    db2 catalog tcpip node G2_RES remote senzing-db2-res server 50000
    db2 attach to G2_RES user db2inst1 using db2inst1
    db2 create database g2 using codeset utf-8 territory us
    db2 connect to g2 user db2inst1 using db2inst1
    db2 -tf /opt/senzing/g2/data/g2core-schema-db2-create.sql | tee /tmp/g2schema.out
    db2 connect reset
    ```

1. Create database on "LIBFE" DB2 server. In docker container, run

    ```console
    db2 catalog tcpip node G2_LIBFE remote senzing-db2-libfe server 50000
    db2 attach to G2_LIBFE user db2inst1 using db2inst1
    db2 create database g2 using codeset utf-8 territory us
    db2 connect to g2 user db2inst1 using db2inst1
    db2 -tf /opt/senzing/g2/data/g2core-schema-db2-create.sql | tee /tmp/g2schema.out
    db2 connect reset
    ```

1. Exit docker container.

    ```console
    exit
    exit
    ```

After the schema is loaded, the demonstration python/Flask app will be available at
[localhost:5000](http://localhost:5000).

### Add content

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${DB2_NETWORK} \
      --env SENZING_CORE_DATABASE_URL="db2://${DB2_USERNAME_CORE}:${DB2_PASSWORD_CORE}@${DB2_HOST_CORE}:${DB2_PORT_CORE}/${DB2_DATABASE_ALIAS_CORE}" \
      --env SENZING_RES_DATABASE_URL="db2://${DB2_USERNAME_RES}:${DB2_PASSWORD_RES}@${DB2_HOST_RES}:${DB2_PORT_RES}/${DB2_DATABASE_ALIAS_RES}" \
      --env SENZING_LIBFE_DATABASE_URL="db2://${DB2_USERNAME_LIBFE}:${DB2_PASSWORD_LIBFE}@${DB2_HOST_LIBFE}:${DB2_PORT_LIBFE}/${DB2_DATABASE_ALIAS_LIBFE}" \
      senzing/g2loader-db2-cluster \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

### Run G2Command.py

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${DB2_NETWORK} \
      --env SENZING_CORE_DATABASE_URL="db2://${DB2_USERNAME_CORE}:${DB2_PASSWORD_CORE}@${DB2_HOST_CORE}:${DB2_PORT_CORE}/${DB2_DATABASE_ALIAS_CORE}" \
      --env SENZING_RES_DATABASE_URL="db2://${DB2_USERNAME_RES}:${DB2_PASSWORD_RES}@${DB2_HOST_RES}:${DB2_PORT_RES}/${DB2_DATABASE_ALIAS_RES}" \
      --env SENZING_LIBFE_DATABASE_URL="db2://${DB2_USERNAME_LIBFE}:${DB2_PASSWORD_LIBFE}@${DB2_HOST_LIBFE}:${DB2_PORT_LIBFE}/${DB2_DATABASE_ALIAS_LIBFE}" \
      senzing/g2command-db2-cluster
    ```

## Cleanup

```console
cd ${GIT_REPOSITORY_DIR}
docker-compose down

sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-core
sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-libfe
sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-res
```
