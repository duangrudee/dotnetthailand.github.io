---
title: Docker compose for Postgres
showMetadata: true
editable: true
---

To use Postgres Docker compose, we need to create required files and add contents to theme.
- main docker-compose.yml
- initialize database file
- .env file

# docker-compose.yml
- Example content of docker-compose.yml
```yml
# docker-compose.yml

version: "3.8" # https://docs.docker.com/compose/compose-file/compose-file-v3/

# .NET connection string value:
#  Host=localhost;Port=5432;Database=my-db;Username=postgres;Password=12345Abc$
services:
  postgres:
    # https://hub.docker.com/_/postgres
    image: postgres:13
    restart: always
    container_name: postgres-db
    environment:
      POSTGRES_DB: my-db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 12345Abc$$ # escape $ with $$
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d
    networks:
      - compose_network

# Create name volumes managed by Docker to not lose data when remove a container
# https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes
volumes:
  pgdata:

networks:
  compose_network:
```

# Initialize a database
- Put SQL files in `init` folder.
- Example content of `init/1.create-user-table.sql`
```sql
-- init/1.create-user-table.sql

CREATE TABLE "user" (
  id SERIAL PRIMARY KEY,
  -- equivalent to id integer NOT NULL DEFAULT nextval('table_name_id_seq')
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  date_of_birth DATE NOT NULL -- date only and no time portion
);

INSERT INTO "user" VALUES (DEFAULT, 'Jose', 'Realman', '2018-01-01');

/*
 To test if a table created successfully, run
 SELECT  * FROM "user" u

 Expected result:

id	first_name	last_name	date_of_birth
1	Jose	Realman	2018-01-01
*/
```

# .env file
- We can control prefix of our volumes/networks by specific a value of COMPOSE_PROJECT_NAME
- Example content of .env file
```
# .env

# https://docs.docker.com/compose/reference/envvars/#compose_project_name
# Explicit prefix volume with this value or use -P with a docker run commmand
COMPOSE_PROJECT_NAME=db-compose
```

# File structure of our Postgres Docker compose
```
tree . -a
.
├── .env
├── docker-compose.yml
└── init
    └── 1.create-user-table.sql
```

#  Useful Docker compose commands
- To launch a container.
```sh
docker-compose up
```

- To remove a container with its volumes.
```sh
docker-compose down --volumes
```

# Connect a database server
- Use these value to connect to a server server
  - Host=localhost
  - Port=5432
  - Database=my-db
  - Username=postgres
  - Password=12345Abc$