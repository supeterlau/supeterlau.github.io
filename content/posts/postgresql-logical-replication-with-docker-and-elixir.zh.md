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


# 启动两个 pg 节点 (using Docker Compose)

(lr aka Logical Replication)
mkdir lr

mkdir -p pg1/{data,share} pg2/{data,share}

touch docker-compose.yml 

content: 

start compose:

docker-compose up

感谢 Debezium 的 [postgresql-alpine-with-wal2json image](https://github.com/debezium/container-images/tree/main/postgres/14-alpine) 让安装 wal2json 很容易.

从容器中复制一份 pg_hba.conf.example 到 ./pg1/share/pg_hda.conf 和 ./pg2/share/pg_hda.conf 

start a new terminal: 

cd lr 

docker cp lr-pg1-1:/usr/local/share/postgresql/pg_hba.conf.sample lr/pg1/share/pg_hba.conf

docker cp lr-pg1-1:/usr/local/share/postgresql/pg_hba.conf.sample lr/pg2/share/pg_hba.conf

末尾添加

host     all     repuser     0.0.0.0/0     md5

restart docker-compose 

Ctrl-c 

docker-compose up

查看 pg2 Log: docker-compose logs -f pg2

# Logical Decoding

## Create tables and data

In pg1 as publication

create database source_rep;
\c source_rep;
create table test_rep(id int primary key, name varchar);
create table test_rep_other(id int primary key, name varchar);
insert into test_rep values(generate_series(1,100),'data'||generate_series(1,100));
insert into test_rep_other  values(generate_series(1,100),'data'||generate_series(1,100));
select count(1) from test_rep;
select count(1) from test_rep_other;

创建 PUBLICATION
CREATE PUBLICATION pub FOR TABLE test_rep, test_rep_other;

In pg2 as subscription

create database target_rep;
create table test_rep(id int primary key, name varchar);
create table test_rep_other(id int primary key, name varchar);

创建 SUBSCRIPTION

CREATE SUBSCRIPTION sub CONNECTION 'dbname=source_rep host=pg1 user=dev password=dev port=5432' PUBLICATION pub;

一个常用语句 check solts

SELECT slot_name, plugin, slot_type, database, active, restart_lsn, confirmed_flush_lsn FROM pg_replication_slots; 

## via SQL:

## via pg_recvlogical:

确保名为 pub 的 slot 不存在

SELECT slot_name, plugin, slot_type, database, active, restart_lsn, confirmed_flush_lsn FROM pg_replication_slots; 

创建 slot 

docker run -it --rm --network lr_default --link lr-pg1-1:postgres postgres:alpine pg_recvlogical -h pg1 -U dev -d source_rep --slot=pub --create-slot -P wal2json

读取变动 

docker run -it --rm --network lr_default --link lr-pg1-1:postgres postgres:alpine pg_recvlogical -h pg1 -U dev -d source_rep --slot=pub --start -o pretty-print=1 -f -

## via Elixir Postgrex: (version 0.16.5)

mix new app 

cd app 

in mix.exs: {:postgrex, ">=0.0.0"}

参考样例 https://hexdocs.pm/postgrex/Postgrex.ReplicationConnection.html

原理，启动链接时设置 :replication 为 "database". [READ MORE](https://www.postgresql.org/docs/current/protocol-replication.html)

如果你按照上边的步骤启动的节点 注意修改 start_link 的参数

try_pg.exs

为了访问 lr_default 网络，需要在 app 内 和 lr_default 网络中启动 elixir docker container

docker run -it --rm --network lr_default --link lr-pg1-1:postgres -v $PWD:/app elixir:alpine sh

sh#: cd /app

sh#: mix run try_pg.ex

默认使用 plugin pgoutput

# Wait, Bonus! using wal2json!

不能简单地将代码中的 pgoutput 替换为 wal2json 
还需要将 pgoutput 的参数去掉

即 
START_REPLICATION SLOT postgrex LOGICAL 0/0 (proto_version '1', publication_names 'postgrex_example')
改为
START_REPLICATION SLOT postgrex LOGICAL 0/0 (proto_version '1', publication_names 'postgrex_example')

如果你希望使用 [wal2json 参数](https://github.com/eulerto/wal2json#parameters) 需要修改 query.

简单示例:

query = ~s{START_REPLICATION SLOT postgrex LOGICAL 0/0 ("pretty-print" 'true', "add-msg-prefixes" 'wal2json')}

如果有任何错误考虑进行本地debug

修改 mix.exs 中 为

{:postgrex, ">= 0.0.0"},
{:postgrex, path: "deps/postgrex"},

找到 handle/5 中 

{:noreply, %{s | protocol: protocol, streaming: max_messages}} 下方添加 dbg()

即可通过执行

mix run try_pg.exs 查看 wal2json 参数错误信息

例如

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

That's it.

Have fun with Logical Decoding in Elixir!

WARNING:

https://www.postgresql.org/docs/current/protocol-replication.html

Postgrex.ReplicationConnection
  this module is experimental and may be subject to changes in the future.


Refs:

https://www.postgresql.org/docs/current/logical-replication.html
https://www.postgresql.org/docs/current/wal-internals.html
https://www.postgresql.org/docs/current/protocol-replication.html


https://github.com/eulerto/wal2json/issues/165

