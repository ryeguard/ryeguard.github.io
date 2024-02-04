# Local Databases for Testing and Learning

Sometimes, you need a local database for testing/learning purposes. This post shows how to set up local databases of different types. The blog post will be updated as I learn more. The post assumes some familiarity with Docker and SQL but still has some in-depth explanations to make sure we understand the commands we're running.

## PostgreSQL

### Docker

The following steps are based on the [Docker blog post](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/) on how to use the PostgreSQL Docker official image.

Fist, run a PostgreSQL container:

```bash
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

where:

- `run` - run a command in a new container
- `name` - name of the container
- `e` - environment variable
  - `POSTGRES_PASSWORD` - password for the (default) user `postgres`
- `d` - run the container in the background (detached mode)

Then, connect to the container:

```bash
docker exec -it some-postgres psql -U postgres
```

where:

- `exec` - execute a command in a running container
- `it` - interactive terminal
- `some-postgres` - name of the container
- `psql` - command to run
  - `U` - username
    - `postgres` - default username

When connected to the container, we can run SQL commands:

```sql
CREATE DATABASE mydb; -- create a database
```

```sql
\l -- list databases
\c mydb -- connect to a database
```

```sql
CREATE TABLE table (name VARCHAR(20), age INT); -- create a table
```

```sql
\dt -- list tables
```

```sql
INSERT INTO mydb (name, age) VALUES ('John', 30);
```

To disconnect from the database and container, simply press `Ctrl/Cmd + D`.

To stop the postgres container:

```bash
docker stop some-postgres
```

To start it up again, run:

```bash
docker start some-postgres
```

and then, again, the `docker exec` command from above.
