# Orion High-Availability Example

This project is an example of a highly availble deployment of Orion. We have two Orion instances running behind a HAProxy load balancer and using Postgres to store the encrypted payloads.

This example **does NOT provide database replication out of the box**. This means that your database will be a single point of failure for high availability. When using this in production, make use database replication to avoid downtime due to database availability.

![Diagram](https://github.com/lucassaldanha/orion-ha-example/blob/master/diagram.png)

## First time configuration

### Configuring Orion

In `./config/orion` you'll find `orion.conf` configuration file and a pre-genereated pub/priv keypair. Please don't use these keys in a real production deployment.

You need to update the `nodeurl` and `clienturl` properties to ensure the host in the url is reachable from the outside world. If you don't change it, only other Orion nodes running on computer that is running the containers will be able to reach the nodes inside the container.

Don't change the ports set in `orion.conf`. If you want to use different ports with your Orion nodes, you need to change the load balancer configuration.

### Configuring Orion database

1. `docker-compose up -d db` create and start a postgres server running on docker
1. `docker exec -it postgres psql -U postgres` to connect to the Postgres db console (if it fails with a message `could not connect to server`, wait a few seconds until the container is up and running)
1. `CREATE DATABASE orion;` to create the database that Orion will use to store its data
1. `\connect orion` to connect to the created database
1. `CREATE USER orion WITH PASSWORD 'orion';` to create a user and password that will be used to connect to the database
1. `CREATE TABLE store (key char(60), value bytea, primary key(key));` create the table used to store Orion data. This script was copyied from [Orion repo](https://github.com/PegaSysEng/orion/blob/master/database/postgres_ddl.sql)
1. `GRANT ALL PRIVILEGES ON store TO orion;` Give total access on the store table to orion user
1. `docker-compose down`

## Usage

Running `docker-compose up` will start the database, two Orion nodes and a haproxy load balancer distributing traffic between the two nodes.

You can check that the load balance is doing its job by running `curl -X GET localhost:8888/upcheck` twice. If you check the Orion nodes logs, you should see a log message for the incoming request in each one of them.

You can visit http://127.0.0.1:9000 to check HAProxy stats page.

## Toubleshooting

I'm getting `psql: could not connect to server` when trying to connect to the database even after a few minutes

> Try running `docker-compose up db` (without the `-d` flag) and check the logs to ensure your container is starting up successfully.