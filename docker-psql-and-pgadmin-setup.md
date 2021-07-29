# How to set up postgres and pgadmin containers for db dev
*note: all commands/docker image names, etc are valid as of 7/29/21*

## postgres
1. Build container/run
```
docker run \
    --name postgresql-container \
    -p 5432:5432 \
    -e POSTGRES_PASSWORD=root \
    -d postgres
```
- the `run` command will pull the image and build, so don't have to run `docker pull postgres`  

3. Check connection
- check can connect to db from command line
```
docker exec -ti postgresql-container /bin/bash
```
```
psql -U postgres -d postgres
```
- check can connect from local network
```
http://localhost:5432 will return `connection was reset` rather than `unable to connect`
```

4. Populate database
- `scp` the all_databases snapshot from server or backup server
- Copy into postgres container
```
docker cp /path/to/file.tar.gz postrgresql-container:/home
```
- exec into container
```
docker exec -ti pgadmin /bin/bash
```
- expand .tar file if necessary
```
tar -xvf file.tar.gz
```
- Navigate to .sql file and use `psql` to build databases
```
psql -U postgres -f all_databases.sql postgres
```

## pgadmin
1. Build container/run
```
docker run -p 80:80 \
    --rm \
    -e 'PGADMIN_DEFAULT_EMAIL=user@tada.com' \
    -e 'PGADMIN_DEFAULT_PASSWORD=tada' \
    --name pgadmin \
    -d dpage/pgadmin4
```
- `--rm` means the container and etc will be cleaned up when container is stopped, ie. need to do `run` each time to use  

2. Connect to database
- go to http://localhost
- enter user/pass given in `run` command
- create server
    - for `hostname/address`, must be local IP ( eg. 192.168.1.12 ), NOT localhost

## connect dev application
1. Configure `env`
```
...
DBIP = "localhost"
...
```