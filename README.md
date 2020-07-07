# Debezium-Kafka-Connector on Docker

Start ZooKeeper in a container:(new terminal)
===================================

docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.1

Start Kafka in a container:(new terminal)
===============================

docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.1


Start MySQL database server in a new container with preconfigured inventory database:(new terminal)
================================================================================

docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:1.1



Start a MySQL command line client :(new terminal)
=======================================

docker run -it --rm --name mysqlterm --link mysql --rm mysql:5.7 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

use inventory;

select * from customers;



Open a new terminal, and use it to start the Kafka Connect service in a container:
===============================================================

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:1.1


New Terminal:
===========

check the status of the Kafka Connect service:
		
    curl -H "Accept:application/json" localhost:8083/
		
List of connectors registered with Kafka Connect:
		
    curl -H "Accept:application/json" localhost:8083/connectors/
		

In the same terminal and register the Debezium MySQL connector:
====================================================

curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.whitelist": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'

now you verify the list of registered connectors:
		
    curl -H "Accept:application/json" localhost:8083/connectors/
		

Review the connector’s tasks:
		
    curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector
		


Open a new terminal and watch-topic (dbserver1.inventory.customers):
========================================================

docker run -it --rm --name watcher --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:1.1 watch-topic -a -k dbserver1.inventory.customers



Go to MySQL Client window and perform some database changes on customers table:
==================================================================
UPDATE customers SET first_name='Sashvin' WHERE id=1004;

Note: Monitor the above terminal to watch the CDC on kafka topic (dbserver1.inventory.customers)


Cleaning up:

docker stop mysqlterm watcher connect mysql kafka zookeeper

docker ps -a
		

		
