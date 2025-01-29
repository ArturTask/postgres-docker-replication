1) start 2 containers `docker-compose -f ./docker-compose up`


`pg-master` and `pg-replica` containers will be up (u can check it with `docker ps`)
also notice 2 directories with data:
* ./master_data
* ./relpica_data

2) change `pg-master` settings
go to `./master_data/postgresql.conf` and change theese settings

```
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
```

go to `./master_data/pg_hba.conf` and add line

```
host    replication     replica_user    0.0.0.0/0               trust
```

3) restart `pg-master` container
`docker restart pg-master`

4) copy data from `pg-master` and begin replication on `pg-replica`

4.1) stop `pg-replica` and delete all files of data directory tome
`docker stop pg-replica`

delete all files from `./replica_data` (on Linux just `rm -r ./replica_data` on windows can be done through provider)

4.2) copy all data from `pg-master` using `pg_basebackup`

`docker exec -it pg-master bash`

run there:
```
pg_basebackup -h pg-master -D /tmp/pgdump -U replica_user -Fp -Xs -P -R
exit
```

on local mashine:
`docker cp pg-master:/tmp/pgdump ./replica_data`

4.3) start `pg-replica`
`docker start pg-replica`

5) check replication

`docker exec -it pg-master bash`

```
psql -U master_user -d master_db
SELECT * FROM pg_stat_replication;
```

should see some records

then create table on `pg-master` and check it on `pg-replica`

```
CREATE TABLE test (id serial);
exit
exit
```

`docker exec -it pg-replica bash`

```
psql -U replica_user -d master_db
\dt
```

should see table named "test"!

Wuala! Congratulations if succeeded!
