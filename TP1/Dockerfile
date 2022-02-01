FROM postgres:11.6-alpine

ENV POSTGRES_DB=db \
	POSTGRES_USER=usr \
	POSTGRES_PASSWORD=pwd

COPY ./docker-entrypoint-initdb.d/01-CreateScheme.sql ./docker-entrypoint-initdb.d
COPY ./docker-entrypoint-initdb.d/02-InsertData.sql ./docker-entrypoint-initdb.d