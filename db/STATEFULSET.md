# StatefulSet

Uma vez que estamos trabalhando com um banco de dados, temos a preocupação que caso o Pod com o banco de dados pare de funcionar, as informações cadastradas serão perdidas. Para evitar isso, realizaremos o mapeamento de volumes abstraindo os Pods com o objeto `StatefulSet`.

## Setup

```shell
$ kubectl create -f statefulset-database.yaml
statefulset.apps/statefulset-postgres created
```

```shell
$ kubectl create -f service-database.yaml
service/nogsantos created
```

```shell
$ kubectl create -f permissoes.yaml
persistentvolumeclaim/postgres-config created
```

### Verificar a criação dos containers

```shell
$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
statefulset-postgres-0   1/1       Running   0          13m
```

### Acessar o container pelo psql

```shell
$ kubectl exec -it statefulset-postgres-0 sh
```

### Acessar o banco no psql

```shell
# psql nogsantos postgres
psql (9.5.6)
Type "help" for help.
```

### Acessar o container pelo bash

```shell
$ kubectl exec -it statefulset-postgres-0 bash
```

## Postgres Meta-Commands

In addition to being able to submit raw SQL queries to the server via psql you can also take advantage of the psql meta-commands to obtain information from the server. Meta-commands are commands that are evaluated by psql and often translated into SQL that is issued against the system tables on the server, saving administrators time when performing routine tasks. They are denoted by a backslash and then followed by the command and its arguments. We will see some examples of this below.

[Documentação psql](https://www.postgresql.org/docs/9.2/static/app-psql.html#APP-PSQL-META-COMMANDS)

### Listing Databases

A single Postgres server process can manage multiple databases at the same time. Each database is stored as a separate set of files in its own directory within the server’s data directory. To view all of the defined databases on the server you can use the `\list` meta-command or its shortcut `\l`.

```shell
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 nogsantos | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

### Switching Databases

Most Postgres servers have three databases defined by default: template0, template1 and postgres. template0 and template1 are skeleton databases that are or can be used by the CREATE DATABASE command. postgres is the default database you will connect to before you have created any other databases. Once you have created another database you will want to switch to it in order to create tables and insert data. Often, when working with servers that manage multiple databases, you’ll find the need to jump between databases frequently. This can be done with the `\connect` meta-command or its shortcut `\c`.

```shell
postgres=# \c nogsantos
You are now connected to database "nogsantos" as user "postgres".
nogsantos=#
```

# Listing Tables

Once you’ve connected to a database, you will want to inspect which tables have been created there. This can be done with the `\dt` meta-command. However, if there are no tables you will get no output.

```shell
nogsantos=# \dt
No relations found.
nogsantos=#
```

After creating a table, it will be returned in a tabular list of created tables.

```shell
nogsantos=# CREATE TABLE leads (id INTEGER PRIMARY KEY, name VARCHAR);
CREATE TABLE
nogsantos=# \dt
        List of relations
 Schema | Name  | Type  | Owner
--------+-------+-------+--------
 public | leads | table | ubuntu
(1 row)
```
