# docker-gs-ping-roach


3. Run with database (cockroachdb)
   docker volume create roach
   docker network create -d bridge mynet

3.1 start db
docker run -d \
--name roach \
--hostname db \
--network mynet \
-p 26257:26257 \
-p 8080:8080 \
-v roach:/cockroach/cockroach-data \
cockroachdb/cockroach:latest-v20.1 start-single-node \
--insecure

3.2 Init db and user
docker exec -it roach ./cockroach sql --insecure
CREATE DATABASE mydb;
CREATE USER totoro;
GRANT ALL ON DATABASE mydb TO totoro;
quit;

3.3 Start example app
gclo git@github.com:wprzechrzta/docker-gs-ping-roach.git

3.4 Build
docker build --tag docker-gs-ping-roach .

3.5 Run  and connect to cockroach
docker run -it --rm -d \
--network mynet \
--name rest-server \
-p 80:8080 \
-e PGUSER=totoro \
-e PGPASSWORD=myfriend \
-e PGHOST=db \
-e PGPORT=26257 \
-e PGDATABASE=mydb \
docker-gs-ping-roach

4. Test

curl localhost
//docker container rm --force rest-server

curl --request POST \
--url http://localhost/send \
--header 'content-type: application/json' \
--data '{"value": "Hello, Docker!"}'

or

curl -X POST \
--header 'content-type: application/json' \
--data '{"value": "Hello, Second message!"}' \
http://localhost/send

5. Cleanup
docker container stop rest-server roach
docker container rm rest-server roach

6. Restart and check that data is retained
   docker run -d \
   --name roach \
   --hostname db \
   --network mynet \
   -p 26257:26257 \
   -p 8080:8080 \
   -v roach:/cockroach/cockroach-data \
   cockroachdb/cockroach:latest-v20.1 start-single-node \
   --insecure

docker run -it --rm -d \
--network mynet \
--name rest-server \
-p 80:8080 \
-e PGUSER=totoro \
-e PGPASSWORD=myfriend \
-e PGHOST=db \
-e PGPORT=26257 \
-e PGDATABASE=mydb \
docker-gs-ping-roach

A slightly more advanced Go server/microservice example for [Docker's Go Language Guide](https://docs.docker.com/language/golang/). 

## License

[Apache-2.0 License](LICENSE)
