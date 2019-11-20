# docker-compose-db2-cluster-demo

## :warning: Obsolete

This repository has not been updated to use the RPM/DEB installation of Senzing.

## Overview

This demonstration illustrates how to
[scale out your database with clustering](https://senzing.zendesk.com/hc/en-us/articles/360010599254-Scaling-out-your-database-with-Clustering)
by creating three DB2 database instances for use by Senzing.

This docker formation brings up the following docker containers:

1. *senzing/db2express-c*  (3 instances)
1. *senzing/python-db2-cluster-demo*

Also shown in the demonstration are commands to run the following Docker images:

1. *senzing/db2* in [Add Senzing schemas](#add-senzing-schemas)
1. *senzing/g2loader-db2-cluster* in [Add content](#add-content)
1. *senzing/g2command-db2-cluster* in [Run G2Command.py](#run-g2commandpy)

### Contents

1. [Expectations](#expectations)
    1. [Space](#space)
    1. [Time](#time)
    1. [Background knowledge](#background-knowledge)
1. [Preparation](#preparation)
    1. [Prerequisite software](#prerequisite-software)
    1. [Clone repository](#clone-repository)
    1. [Docker images](#docker-images)
    1. [Create SENZING_DIR](#create-senzing_dir)
1. [Demonstration](#demonstration)
    1. [Set environment variables for docker](#set-environment-variables-for-docker)
    1. [Launch docker formation](#launch-docker-formation)
    1. [Add Senzing schemas](#add-senzing-schemas)
    1. [Add content](#add-content)
    1. [Run G2Command.py](#run-g2commandpy)
1. [Cleanup](#cleanup)

### Legend

1. :thinking: - A "thinker" icon means that a little extra thinking may be required.
   Perhaps you'll need to make some choices.
   Perhaps it's an optional step.
1. :pencil2: - A "pencil" icon means that the instructions may need modification before performing.
1. :warning: - A "warning" icon means that something tricky is happening, so pay attention.

## Expectations

### Space

This repository and demonstration require 6 GB free disk space.

### Time

Budget 2 hours to get the demonstration up-and-running, depending on CPU and network speeds.

### Background knowledge

This repository assumes a working knowledge of:

1. [Docker](https://github.com/Senzing/knowledge-base/blob/master/WHATIS/docker.md)
1. [Docker-compose](https://github.com/Senzing/knowledge-base/blob/master/WHATIS/docker-compose.md)

## Preparation

### Prerequisite software

The following software programs need to be installed:

1. [docker](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker.md)
1. [docker-compose](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker-compose.md)

### Clone repository

For more information on environment variables,
see [Environment Variables](https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md).

1. Set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-db2-cluster-demo
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

1. Follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md) to install the Git repository.

### Docker images

1. Because an independent download is needed for the DB2 ODBC client, the
   [senzing/python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base)
   docker image must be manually built.
   Follow the build instructions at
   [github.com/Senzing/docker-python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base#build)

1. Verify `senzing/python-db2-cluster-base` is a local image.

    ```console
    sudo docker images
    ```

1. Build the following docker images.

    ```console
    sudo docker build --tag senzing/db2express-c            https://github.com/senzing/docker-db2express-c.git
    sudo docker build --tag senzing/db2                     https://github.com/senzing/docker-db2.git
    sudo docker build --tag senzing/python-db2-cluster-demo https://github.com/senzing/docker-python-db2-cluster-demo.git
    sudo docker build --tag senzing/g2loader-db2-cluster    https://github.com/senzing/docker-g2loader-db2-cluster.git
    sudo docker build --tag senzing/g2command-db2-cluster   https://github.com/senzing/docker-g2command-db2-cluster.git
    ```

### Create SENZING_DIR

Note: this is an obsolete method.

If you do not already have an `/opt/senzing` directory on your local system, visit
[HOWTO - Create SENZING_DIR](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/create-senzing-dir.md).

## Demonstration

### Set environment variables for docker

1. **DB2_NETWORK** -
   The network created by `docker-compose`.  To view, run `docker network ls`.
1. For explanation of other environment variables, see:
    1. [senzing/docker-python-db2-cluster-base](https://github.com/Senzing/docker-python-db2-cluster-base#set-environment-variables-for-demonstration)
    1. [senzing/docker-db2express-c](https://github.com/Senzing/docker-db2express-c#run-docker-container)
1. The values in the following example are specific to
   [docker-compose.yaml](docker-compose.yaml):

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-db2-cluster-demo
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"

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
sudo docker-compose up
```

The DB2 database storage will be on the local system at ${DB2_STORAGE_*} paths.
Example: `/storage/docker/senzing/docker-compose-db2-cluster-demo-core`.
The default database storage path is `/storage/docker/senzing/docker-compose-db2-cluster-demo-XXXX`.

### Add Senzing schemas

In a separate terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command.

    ```console
    sudo docker run -it  \
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
    sudo docker run -it  \
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
    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${DB2_NETWORK} \
      --env SENZING_CORE_DATABASE_URL="db2://${DB2_USERNAME_CORE}:${DB2_PASSWORD_CORE}@${DB2_HOST_CORE}:${DB2_PORT_CORE}/${DB2_DATABASE_ALIAS_CORE}" \
      --env SENZING_RES_DATABASE_URL="db2://${DB2_USERNAME_RES}:${DB2_PASSWORD_RES}@${DB2_HOST_RES}:${DB2_PORT_RES}/${DB2_DATABASE_ALIAS_RES}" \
      --env SENZING_LIBFE_DATABASE_URL="db2://${DB2_USERNAME_LIBFE}:${DB2_PASSWORD_LIBFE}@${DB2_HOST_LIBFE}:${DB2_PORT_LIBFE}/${DB2_DATABASE_ALIAS_LIBFE}" \
      senzing/g2command-db2-cluster
    ```

## Cleanup

In a separate (or reusable) terminal window:

1. Use environment variable describe in "[Clone repository](#clone-repository)" and "[Configuration](#configuration)".
1. Run `docker-compose` command.

    ```console
    cd ${GIT_REPOSITORY_DIR}
    sudo docker-compose down
    ```

1. Delete storage.

    ```console
    sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-core
    sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-libfe
    sudo rm -rf /storage/docker/senzing/docker-compose-db2-cluster-demo-res
    ```

1. Delete git repository.

    ```console
    sudo rm -rf ${GIT_REPOSITORY_DIR}
    ```
