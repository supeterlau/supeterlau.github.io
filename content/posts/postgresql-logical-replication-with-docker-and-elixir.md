+++
author = "PeterLau"
title = "PostgreSQL Logical Replication with Docker and Elixir"
date = "2022-09-29"
description = "postgresql, logical replication, docker, elixir, wal2json"
tags = [
  "pg",
  "elixir",
  "docker"
]
categories = [
  "dev"
]
draft = false
+++


Start PostgreSQL Logical Replication and wal2json plug-in in Docker, and read the changes with Elixir Postgrex.

<!--more-->

## Start two PostgreSQL nodes (using Docker Compose)

(lr aka Logical Replication)

``` bash
mkdir lr
mkdir -p pg1/{data,share} pg2/{data,share}
touch docker-compose.yml 
```

docker-compose.yml: 

``` yaml
services:
  pg1:
    image: debezium/postgres:14-alpine
    restart: always
    volumes:
      - type: bind
        source: ./pg1/share/pg_hba.conf
        target: /usr/local/share/postgresql/pg_hba.conf
      - ./pg1/data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: dev
      POSTGRES_USER: dev

  pg2:
    image: debezium/postgres:14-alpine
    restart: always
    volumes:
      - type: bind
        source: ./pg2/share/pg_hba.conf
        target: /usr/local/share/postgresql/pg_hba.conf
      - ./pg2/data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: dev
      POSTGRES_USER: dev
```

Start Docker-compose:

`docker-compose up`

Thanks to Debezium's [postgresql-alpine-with-wal2json image](https://github.com/debezium/container-images/tree/main/postgres/14-alpine), installing Wal2JSON is extremely easy.

Last thing, copy `pg_hba.conf.example` from the container to `./pg1/share/pg_hda.conf` and `./pg2/share/pg_hda.conf`

open a new terminal:

``` bash
cd lr 
docker cp lr-pg1-1:/usr/local/share/postgresql/pg_hba.conf.sample lr/pg1/share/pg_hba.conf
docker cp lr-pg1-1:/usr/local/share/postgresql/pg_hba.conf.sample lr/pg2/share/pg_hba.conf

```

At the end of files, add

`host     all     repuser     0.0.0.0/0     md5`

restart Docker-Compose:

press Ctrl-c 

`docker-compose up`

Viewing Service pg2 Logs:

`docker-compose logs -f pg2`

## Logical Decoding

### Create tables for pg1/pg2 and add data

In pg1 as publication

``` bash
create database source_rep;
\c source_rep;
create table test_rep(id int primary key, name varchar);
create table test_rep_other(id int primary key, name varchar);
insert into test_rep values(generate_series(1,100),'data'||generate_series(1,100));
insert into test_rep_other  values(generate_series(1,100),'data'||generate_series(1,100));
select count(1) from test_rep;
select count(1) from test_rep_other;
```

create PUBLICATION

`CREATE PUBLICATION pub FOR TABLE test_rep, test_rep_other;`

In pg2 as subscription

``` bash
create database target_rep;
create table test_rep(id int primary key, name varchar);
create table test_rep_other(id int primary key, name varchar);
```

create SUBSCRIPTION

`CREATE SUBSCRIPTION sub CONNECTION 'dbname=source_rep host=pg1 user=dev password=dev port=5432' PUBLICATION pub;`

view Logical Decoding slots:

`SELECT slot_name, plugin, slot_type, database, active, restart_lsn, confirmed_flush_lsn FROM pg_replication_slots;`

### via SQL:

https://www.postgresql.org/docs/current/logical-replication.html

### via pg_recvlogical:

Make sure the slot named pub not exist

``` bash
SELECT slot_name, plugin, slot_type, database, active, restart_lsn, confirmed_flush_lsn FROM pg_replication_slots;
```

create slot 

``` bash
docker run -it --rm --network lr_default --link lr-pg1-1:postgres postgres:alpine pg_recvlogical -h pg1 -U dev -d source_rep --slot=pub --create-slot -P wal2json;
```

Read the changes

``` bash
docker run -it --rm --network lr_default --link lr-pg1-1:postgres postgres:alpine pg_recvlogical -h pg1 -U dev -d source_rep --slot=pub --start -o pretty-print=1 -f -
```

### via Elixir Postgrex: (version 0.16.5)

``` bash
mix new app
cd app
```

in mix.exs: {:postgrex, ">=0.0.0"}

Refer to the sample https://hexdocs.pm/postgrex/Postgrex.ReplicationConnection.html

set :replication to "database" when start_link. [READ MORE](https://www.postgresql.org/docs/current/protocol-replication.html)

If you start nodes as described above, be careful to modify the start_link parameter.

try_pg.exs

``` elixir
defmodule Repl do
  use Postgrex.ReplicationConnection

  def start_link(opts) do
    # Automatically reconnect if we lose connection.
    extra_opts = [
      auto_reconnect: true
    ]

    Postgrex.ReplicationConnection.start_link(__MODULE__, :ok, extra_opts ++ opts)
    |> IO.inspect()
  end

  @impl true
  def init(:ok) do
    {:ok, %{step: :disconnected}}
  end

  @impl true
  def handle_connect(state) do
    query = "CREATE_REPLICATION_SLOT postgrex TEMPORARY LOGICAL pgoutput NOEXPORT_SNAPSHOT"
    {:query, query, %{state | step: :create_slot}}
  end

  @impl true
  def handle_result(results, %{step: :create_slot} = state) when is_list(results) do
    query = "START_REPLICATION SLOT postgrex LOGICAL 0/0 (proto_version '1', publication_names 'postgrex_example')"
    {:stream, query, [], %{state | step: :streaming}}
  end

  @impl true
  # https://www.postgresql.org/docs/14/protocol-replication.html
  def handle_data(<<?w, _wal_start::64, _wal_end::64, _clock::64, rest::binary>>, state) do
    IO.inspect(rest)
    {:noreply, state}
  end

  def handle_data(<<?k, wal_end::64, _clock::64, reply>>, state) do
    messages =
      case reply do
        1 -> [<<?r, wal_end + 1::64, wal_end + 1::64, wal_end + 1::64, current_time()::64, 0>>]
        0 -> []
      end

    {:noreply, messages, state}
  end

  @epoch DateTime.to_unix(~U[2000-01-01 00:00:00Z], :microsecond)
  defp current_time(), do: System.os_time(:microsecond) - @epoch
end

{:ok, _pid} =
  Repl.start_link(
    hostname: "postgres",
    database: "source_rep",
    username: "dev",
    password: "dev"
  )

Process.sleep(:infinity)
```

To access the lr_default network in docker, start an Elixir Docker Container in path /app and link to the lr_default network:

``` bash
docker run -it --rm --network lr_default --link lr-pg1-1:postgres -v $PWD:/app elixir:alpine sh
sh#: cd /app
sh#: mix run try_pg.ex
```

using plugin pgoutput by default.

## Wait, Bonus! using wal2json!

You cannot simply replace pgoutput with wal2json in your code.

At least the parameters of pgoutput need to be removed.

Change


``` elixir
START_REPLICATION SLOT postgrex LOGICAL 0/0 (proto_version '1', publication_names 'postgrex_example')
```

to

``` elixir
START_REPLICATION SLOT postgrex LOGICAL 0/0 (proto_version '1', publication_names 'postgrex_example')
```

If you want to use [wal2json parameters](https://github.com/eulerto/wal2json#parameters), you need to modify the query at the same time.

Example of simple parameters:

``` elixir
query = ~s{START_REPLICATION SLOT postgrex LOGICAL 0/0 ("pretty-print" 'true', "add-msg-prefixes" 'wal2json')}
```

Consider local debugging if there are any errors

modify mix.exs

`{:postgrex, ">= 0.0.0"},`

to 

`{:postgrex, path: "deps/postgrex"},`

find `deps/postgrex/lib/postgrex/replication_connection.ex` function handle/5 (v0.16.5 L540)

`{:noreply, %{s | protocol: protocol, streaming: max_messages}}`

below that, you can add

`dbg()`

By executing

`mix run try_pg.exs`

to View wal2json error information.

Error example:

``` elixir
  reason: %Postgrex.Error{
    message: nil,
    postgres: %{
      code: :invalid_parameter_value,
      file: "wal2json.c",
      line: "730",
      message: "option \"pretty_print\" = \"1\" is unknown",
      pg_code: "22023",
      routine: "pg_decode_startup",
      severity: "ERROR",
      unknown: "ERROR",
      where: "slot \"postgrex\", output plugin \"wal2json\", in the startup callback"
    },
    connection_id: 3405,
    query: nil
  },
```

That's it.

Have fun with Logical Decoding with Docker and Elixir !

## WARNING:

https://www.postgresql.org/docs/current/protocol-replication.html

Postgrex.ReplicationConnection

  *this module is experimental and may be subject to changes in the future.*

## Refs:

https://www.postgresql.org/docs/current/logical-replication.html
https://www.postgresql.org/docs/current/wal-internals.html
https://www.postgresql.org/docs/current/protocol-replication.html
