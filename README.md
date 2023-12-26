# dezoomcamp2024
DataTalksClub Data Engineering Zoomcamp 2024

Repository for the Zoomcamp 2024
@amilcarnetto



# common scripts:
# create a volume
docker volume create --name dtc_postgres_volume_local -d local dtc_postgres_volume_local

# create a network between docker containers
docker network create pg-network


# postgres on docker

docker run -it \
-e POSTGRES_USER="root" \
-e POSTGRES_PASSWORD="root" \
-e POSTGRES_DB="ny_taxi" \
-v dtc_postgres_volume_local:/var/lib/postgresql/data \
-p 5432:5432 \
--network=pg-network \
--name=pg-database postgres:13

# pgadmin on docker
docker run -it \
-e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
-e PGADMIN_DEFAULT_PASSWORD="root" \
-p 8080:80 \
--network=pg-network \
--name=pgadmin dpage/pgadmin4