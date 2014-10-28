#Makara Repro 

* Set up Postgres containers

```
docker run -e DB_USER=yammer -e DB_PASSWORD=postgres -e DB_NAME=yam_development -e REP_USER=rtest -e REP_PASSWORD=test -e ROLE=master --name pg-primary -p 5432:5432 -d hmarr/postgresql
```

```
docker run -e DB_USER=yammer -e DB_PASSWORD=postgres -e DB_NAME=yam_development -e REP_USER=rtest -e REP_PASSWORD=test -e ROLE=slave -e MASTER_HOST=$(docker inspect pg-primary | jq -r ".[].NetworkSettings.IPAddress") -e MASTER_PORT=5432 --name pg-secondary -p 6432:5432 -d hmarr/postgresql
```

```
docker run -e DB_USER=yammer -e DB_PASSWORD=postgres -e DB_NAME=yam_development -e REP_USER=rtest -e REP_PASSWORD=test -e ROLE=slave -e MASTER_HOST=$(docker inspect pg-primary | jq -r ".[].NetworkSettings.IPAddress") -e MASTER_PORT=5432 --name pg-tertiary -p 7432:5432 -d hmarr/postgresql
```

* Confirm that `host` IP address in `config/database.yml` matches the IP of your docker VM (if boot2docker, `boot2docker ip`). 

* Set up database
```bundle exec rake db:migrate db:seed```

* Start rails app `bundle exec rails s` and confirm that seeded records display at `localhost:3000/groups`. Watch `stdout` and confirm that requests to `/groups` are first served by the primary, then alternating between the replicas. 

* Stop any one of the docker containers (`docker stop pg-secondary`) and observe exception on subsequent requests for whichever container was stopped

```
Started GET "/groups" for 127.0.0.1 at 2014-10-24 10:25:23 -0700

PG::ConnectionBad (could not connect to server: Connection refused
  Is the server running on host "192.168.59.103" and accepting
  TCP/IP connections on port 6432?
):
  activerecord (4.1.6) lib/active_record/connection_adapters/postgresql_adapter.rb:888:in 'initialize'
  activerecord (4.1.6) lib/active_record/connection_adapters/postgresql_adapter.rb:888:in 'new'
  activerecord (4.1.6) lib/active_record/connection_adapters/postgresql_adapter.rb:888:in 'connect'
  activerecord (4.1.6) lib/active_record/connection_adapters/postgresql_adapter.rb:568:in 'initialize'
  activerecord (4.1.6) lib/active_record/connection_adapters/postgresql_adapter.rb:41:in 'new'
  ...
  
```
