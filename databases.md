psql summary
===============
#### Connect
```bash
psql -U postgres
psql -U postgres -h localhost -p 5432
```

#### List databases
```
\l
```

#### Connect to a database
```
\c database-name
```

#### List all
```
\d
```

#### List tables
```
\dt
```

#### List tables from all schemas
```
\dt *.*
```

#### Describe table
```
\d table-name
```

#### General help
```
\?
```



Complete Intro to Databases
===========================
https://frontendmasters.com/courses/databases/
https://frontendmasters.com/courses/databases/introduction/  
https://btholt.github.io/complete-intro-to-databases/

#### Search engines databases
Elasticsearch, Solr, Sphinx  
*Lucene is a Java fulltext search lib*  
*used in Solr and Elasticsearch*

#### Wide column databases
Apache Cassandra, Google Bigtable  
fairly new type of database  
like key-value but 2D

#### Mesage brokers
Apache Kafka, RabbitMQ  
pass messages between programs  
underutilized by node developers  
works very well with node

#### Multi model databases
database that knows how to act both as relational  
and non-relational at the sametime  
Azure Cosmos DB, ArangoDB

### ACID
not all databases have these qualities  
ACID is safe but slow, its a tradeoff  
a lot of it comes from banking systems

#### Atomicity
does query happen all at once  
by query we also mean something more complex  
deduce money from one account and increase in other account

#### Consistency
you have multiple database servers running  
if one goes away, are the others in-sync  
(if not, you may lose data)

#### Isolation
changes made by one query are not visible for others  
until that query finishes

#### Durability
if your db server crashes you can restore it where it was

### Transactions
execute several queries like it would be one query  
all of these queries will happen or none at all

its a solution to: I have to do this thing as atomic  
but I cannot do it in single query

### NoSQL
Marketing term (dumb) hot in like 2010  
like Serverless

*Some NoSQL can handle SQL*  
*Cassandra, Couchbase*

better name: document based

#### reasons to use
Schemaless, define schema as you go  
easy for up and running  
conducive to (works well with) dynamic languages  
like Javascript

#### risks
if you misspell field name DB will accept it  
and think you want to create new field  
you have to be more careful

the more discipline you have early  
the more "queryable" db will be later



Mongo overview
=======================
#### how mongo data is organized?
database  
collection  
document  
field

#### examples of queries in mongo
they are in JSON format

```javascript
db.collection.find({})
db.collection.find({ "age": 25 })
db.collection.find({ "age": 25, "name": "John" })
db.collection.find({
   $or: [
      { "age": 25 },
      { "name": "John" }
   ]
})
db.collection.find({ "address.city": "New York" })
db.collection.find().sort({ "age": -1 })
db.collection.find().limit(5)
db.collection.find({ "address.zip": { $exists: true } })
db.collection.find({
    $or: [
        { "address.zip": "10001" },
        { "address.zip": { $exists: false } }
    ]
})
db.collection.updateOne({ "name": "John" }, { $set: { "age": 26 } })
db.collection.updateMany(
    { "address.city": "Los Angeles" },
    { $set: { "address.zip": "90001" } }
)
db.collection.updateOne(
    { "name": "Alice" },
    [
        { $set: { "address.city": { $concat: ["$address.city", " - USA"] } } }
    ]
)
db.collection.updateOne(
    { "name": "Alice" },
    { $set: { "address.zip": "10002" } }
)
db.collection.updateMany(
    { "address.city": "Los Angeles" },
    { $set: { "address.zip": "90001" } }
)
```

#### downsides of mongodb
Complex Aggregations: While MongoDB supports aggregation pipelines,  
performing complex queries and aggregations can sometimes be more cumbersome  
compared to SQL-based databases.

Limited Joins: MongoDB supports basic joins through $lookup, but these are not  
as powerful or straightforward as SQL joins,  
making certain data relationships more challenging to handle.

#### does mongo have transactions
MongoDB 4.0 adds support for multi-document ACID transactions now

#### lookup query
always acts as left outer join

```javascript
db.orders.aggregate([
    {
        $lookup: {
            from: "customers",          // The collection to join with
            localField: "customer_id",  // Field from the input documents (orders)
            foreignField: "_id",        // Field from the documents of the "from" collection (customers)
            as: "customer_details"      // Output array field that will contain matching documents
        }
    }
])
```

#### does mongo have foreign keys, on delete actions and constraints?
no check of relations  
no on delete actions  
you can create indexes with unique constraints  
(mongo has indexes)

#### is mongo eventually consistient or stronger?
MongoDB provides both eventual consistency and strong consistency,  
depending on the configuration

DynamoDB: noSQL database



Complete Intro to SQLite
========================
https://frontendmasters.com/courses/sqlite/  
Frontend Masters, Brian Holt  
Complete intro to SQLite

#### How to pronounce
eskiulite

#### SQLite
created by Dr. Hipp for navy  
software for destroyers  
checked alternatives but decided to go with own  
open sourced and public domain  
but not open for contributions

Dr. Hipp also didn't wanted to use git  
so he wrote alternative "Fossil"

SQLite is most used database ever  
on every  
- android
- iphone
- mac
- all browsers
- skype
- itunes
- dropbox client

#### What SQLite is
It's not a server  
a way to read and write to file  
it has no network access, no users, no passwords  
designed to work on same computer as app that is using it  
zero configuration  
very fast (as fast as your disc, because there is no network)  
intentionally limited SQL features

#### can be used in production?
in webservices no replication may be a problem  
sqlite is often used to start project and then later move to posgres  
like in rails

most often production use case: embeding sqlite into app like a client in C++  
... whenever you need to move from simple arrays

also used in wasm

#### run
```bash
sqlite Chinook_Sqlite.sql
```

#### built in help
```sql
.help
.tables
.read ./Chinook_Sqlite.sql
```

```sql
.shell clear
```
clear shell, like in terminal

#### Chinook database
example of Accounting database for Authors and Records  
Genre

#### SQL
very structured  
it's in the name  
Structured query language

capitalization is not important  
sElEcT * fRoM aRtIsT;

#### inspect schema
.schema Artist

#### quotes
'' for string values  
"" is for colum names (with spaces in name)

#### approach to failing
early 2000s:  
try not not to fail, be permissive  
(but find out that there is problem way way later)  
sqlite is here, it will try to make a sense of a wrong query  
`artistid = '174'` will work (altough should be `where artistid = 174`)

recent way:  
fail quickly

#### like
in sqlite it ignores case:
```sql
select artistid from artist where name like '%Postal Service';
```

in other databases:  
ilike: ignore case in postgres

#### pagination
```sql
select * from artist limit 5 offset 5;
```

however if something can be inserted in between  
... it's better to use where in such cases
```sql
select * from artist where artistid >= 16 limit 5;
```

#### order
```sql
select * from artist order by name limit 5;
```

descending:
```sql
select * from artist order by name desc limit 5;
```

#### insert
```sql
insert into artist (name) values ('Radiohead');
```
many fields:
```sql
INSERT INTO food (name, food_group, color) VALUES ('carrot', 'vegetable', 'orange'); -- notice the order is the same
```

insert many:

```sql
INSERT INTO
    BandMember
    (name, role)
VALUES
    ('Jonny Greenwood', 'guitarist'),
    ('Colin Greenwood', 'bassist'),
    ('Ed O''Brien', 'guitarist'),
    ('Philip Selway', 'drummer')
RETURNING *;
```

#### update
```sql
update artist set name = 'Daft Punk' where name = 'Radiohead';
```

be careful  
without where
```sql
update artist set name = 'Daft Punk';
```
it will update EVERY row

#### update ... RETURNING
if instead of silence you want to see
```sql
update artist set name = 'Daft Punk' where name = 'Radiohead' returning *;
```

or return subset
```sql
update artist set name = 'Daft Punk' where name = 'Radiohead' returning name;
```

#### create table

```sql
create table BandMember (
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE NOT NULL,
  role TEXT VARCHAR
)`;
```

VARCHAR(100) will work but in the end will not limit you

in engine itself sqlite has only four types:  
`INTEGER`, `REAL`, `TEXT`, and `BLOB`

#### alter column
```sql
alter table bandmember add column image text;
alter table bandmember drop column image text;
```
you cannot edit many columns in one query in sqlite

```sql
alter table bandmember add column nationality text not null default 'uk';
```
if altering table with not null column you have to add default

strange, omitting name also works,  
but then there seems to be no way to use such unnamed column
```sql
alter table bandmember add column text not null default 'uk';
```

### relations
one of points of relational db is:  
tables are joined and you can easily edit data like category name for all music albums

you don't repeat category name in every album

one-to-one: usually it means it's the same table and is not often

### joins

```sql
select artist.name, album.title
from album
join artist on album.artistid = artist.artistid
limit 5;
```

use aliases:

```sql
select ar.name, al.title
from album al
join artist ar on ar.artistid = al.artistid
limit 5;
```

### inner/outer join
default type of join  
you can omit "inner"

artist join album

left join: give me also artists without albums  
right join: give me also orphan albums without artists  
full outer join: give me all including orphans from both sides

`right join` is same as `right outer join`

cross join: give full product, every combination  
in pracitice not often used  
sqlite may not support it

natural join  
bad idea
```sql
select b.name, a .title from album a natural join b limit 5;
```
no "ON" part  
it will try to find common columns  
... it's too magical and prone to break

### foreign keys
least favorite part of sqlite

```sql
.schema Album

CREATE TABLE [Album]
(
  ...
  FOREIGN KEY ([ArtistId]) REFERENCES [Artist] ([ArtistId])
  ON DELETE NO ACTION ON UPDATE NO ACTION
);
```

no action = don't allow for that action

at extreme scale foreign key may be issue with performance  
but generally you should probably use them always  
(apart from being Google scale)

#### sqlite by default doesn't respect foregin keys
reason is sqlite is aggresivelly backward compatible  
and apps written in very early days, when there was no foreign keys could start to break because of this behaviour change

to fix this use:
```sql
PRAGMA foreign_keys = on;
```
but it will only last for duration of connection

there is option to have this in on open handler of client  
but this may be even worse if you switch client and forget

PRAGMA: per connection policies to be set

#### foreign keys constraints
no action: prevent  
restrict: same as no action (with some minor neuance)  
on delete cascade  
on delete set null  
on delete set default

#### count
```sql
select count(*) from track;
```

#### distinct
```sql
select count(distinct artistid) from track;
select distinct genreid from track
```

#### group

```sql
select genreid, count(genreid) as count
from track
group by genreid;
```

group with join

```sql
select g.name, count(t.genreid)
from track t
join genre g on t.genreid = t.genreid
group by t.genreid;
```

#### sqlite in pipeline
have a `test.db` file, preseed it  
and then tests can obliterate that data  
and just not commit changes

#### subquery
often its alternative to join

but may have a different query plan

```sql
select invoiceid
from invoice
where customerid = (
  select customerid from customer
  where email = 'hholy@gmail.com'
);
```

but it can get quite complicated if you have many subqueries

if result of subquery is a set then by default the first row is used

#### optimization
in traditional dbs:  
running one big sql is usually faster than many  
because network latency is important

in sqlite  
running several smaller is often faster

#### npm for sqlite
node 22.20+, bun, deno: builtin support  
better-sqlite3  
promised-sqlite3: same, but promisses, thin wrapper

#### ORMs
Prisma  
Drizzle  
Sequelize

#### Flexibe typing
no timestamp type in sqlite

#### limits
sqlite is not very limited  
max row size is 1GB  
each row can be 1GB!  
(better put it into s3)

2000 columns on table  
... if you have that many you want to use mongo

query can be bilion characters  
max number of tables joined: 64  
max like globs: 50k characters  
max attached databases: 10 databases

database can be 281TB  
theoretical limit of rows: 1e19

#### views
imagine spotify  
you often want song name, artist and album

```sql
SELECT
  t.TrackId as id,
  ar.Name as artist,
  al.Title as album,
  t.Name as track
FROM Track t
JOIN Album al ON t.AlbumId = al.AlbumId
JOIN Artist ar ON ar.ArtistId = al.ArtistId
LIMIT 5;
```

to not do this query every time, create view

```sql
CREATE VIEW EasyTracks AS
SELECT
  t.TrackId as id,
  ar.Name as artist,
  al.Title as album,
  t.Name as track
FROM Track t
JOIN Album al ON t.AlbumId = al.AlbumId
JOIN Artist ar ON ar.ArtistId = al.ArtistId;
```

and then
```sql
select * from EasyTracks
```

however every time you run this query the underlining sql will be run

materialized view:
```sql
CREATE MATERIALIZED VIEW
```
however it's not present in sqlite  
also sqlite cannot insert into view  
(both are in postgres)

they can help a lot with performance  
but also they require manual refresh

#### explain
```sql
explain select * from track where name = 'Black Dog';
```
result is complicated...

way easier is
```sql
explain query plan select * from track where name = 'Black Dog';
```
and look for "SCAN"  
*which often means you are missing index*

also to turn it all for connection
```sql
.eqp on
```

#### index
little can help  
too much can hurt

list indexes on table
```sql
PRAGMA index_list('Track')
```

by defualt all primary keys have index

```sql
create index idx_track_name on Track (Name);
```
it will create:  
b-tree (balanced tree)  
(in postgres there are many types to choose from)

and then if you check explain plan  
look for SCAN changing into SEARCH  
(meaning index is used)

adding index:  
search: will be faster  
insert, update, delete: will be slower

it's ok to wait for problem and then add index  
sometimes it's obvious

#### full text search
searching songs  
can be done with LIKE  
but if there are many fields, it can be complicated fast

We could write three LIKE queries or try to hack it around it,  
but luckily there's something that already exists

FTS: full text search  
great for search on site  
but also autocompletion

```sql
create virtual table track_search using fts5(content="EasyTracks", content_rowid='id', track, album, artist);
```

double quotes here:
```sql
content="EasyTracks"
```

you will see additional tables when checking with
```sql
.tables
```
that's ok

to search use MATCH or one of alternative syntax:

```sql
SELECT * FROM track_search WHERE track_search MATCH 'black';
SELECT * FROM track_search WHERE track_search = 'white';
SELECT * FROM track_search('red');
```

but result is empty until you manually populate:
```sql
insert into track_search select album, artist, track from EasyTracks;
```

now this will work:
```sql
select * from track_search where track_search MATCH 'black';
```

in reality you have to add table triggers to keep in sync  
something like:

```sql
-- Create a table. And an external content fts5 table to index it.
CREATE TABLE tbl(a INTEGER PRIMARY KEY, b, c);
CREATE VIRTUAL TABLE fts_idx USING fts5(b, c, content='tbl', content_rowid='a');

-- Triggers to keep the FTS index up to date.
CREATE TRIGGER tbl_ai AFTER INSERT ON tbl BEGIN
 INSERT INTO fts_idx(rowid, b, c) VALUES (new.a, new.b, new.c);
END;
CREATE TRIGGER tbl_ad AFTER DELETE ON tbl BEGIN
 INSERT INTO fts_idx(fts_idx, rowid, b, c) VALUES('delete', old.a, old.b, old.c);
END;
CREATE TRIGGER tbl_au AFTER UPDATE ON tbl BEGIN
 INSERT INTO fts_idx(fts_idx, rowid, b, c) VALUES('delete', old.a, old.b, old.c);
 INSERT INTO fts_idx(rowid, b, c) VALUES (new.a, new.b, new.c);
END;
```

NEAR
```sql
MATCH 'NEAR("term1" "term2", 10)'
```
This searches for term1 and term2 appearing near each other in the same document  
10 is default distance

also possible to search with score
```sql
select bm25(track_search), * from track_search where track_search match 'black' order by bm25(track_search) limit 15;
```
the smaller the number the better the match is  
(it's also called distance)

in reality in Spotify you want to sort not by distance  
but also how popular or new song is

### JSON

#### extensions
sqlite has small core  
json is extension  
sqlpkg: package manager  
http://sqlpkg.org  
Richard Hipp

#### sqlite-hello
test are extensions working  
also usefull if you want to write own extension

```bash
sqlpkg install asg017/hello
```

there are extensions for:  
calling http from database (why?)

#### json extension

```bash
sqlpkg install sqlite/json1
sqlpkg which sqlite/json1
```

enter sqlite3, then

```sql
.load <copy path>

SELECT json('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}');

-- create an array
SELECT json_array(1, 2, 3);

-- get the length of an array
SELECT json_array_length('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}', '$.favorites');

-- get the type of a field in an object
SELECT json_type('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}', '$.username');

-- construct a new object using pairs
SELECT json_object('username', 'btholt', 'favorites', json_array('Daft Punk', 'Radiohead'));
```

manipulation

```sql
select json_insert('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}', '$.city', 'Sacramento');
```

read one field
```sql
SELECT json('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}') -> 'username';
```
"btholt"

this -> can be chained
```sql
SELECT json('{"username": "btholt", "name": { "first": "Brian" }, "favorites":["Daft Punk", "Radiohead"]}') -> 'name' -> 'first';
```

read as value, not as json
```sql
SELECT json('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}') ->> 'username';
```

#### jsonb
json is usually stored as a text  
it's good only if you need to store jsons in human readible way  
but if you don't, jsonb is better

there is a better way:  
faster for queries
```sql
SELECT jsonb('{"username": "btholt", "favorites":["Daft Punk", "Radiohead"]}');
```
cost is that store format is more messed up

there are also jsonb_insert, jsonb_replace, ...

```sql
INSERT INTO
  users
  (email, data)
VALUES
  ('brian@example.com', jsonb('{"favorites":["Daft Punk", "Radiohead"], "name": {"first": "Brian", "last": "Holt"}}')),
  ('bob@example.com', jsonb('{"favorites":["Daft Punk"], "name": {"first": "Bob", "last": "Smith"}}')),
  ('alice@example.com', jsonb('{"admin": true, "favorites":["The Beatles", "Queen"], "name": {"first": "Alice", "last": "Johnson"}}')),
  ('charlie@example.com', jsonb('{"favorites":["Nirvana", "Pearl Jam"], "name": {"first": "Charlie", "last": "Brown"}}')),
  ('dave@example.com', jsonb('{"favorites":["Pink Floyd", "Led Zeppelin"], "name": {"first": "Dave", "last": "Wilson"}}')),
  ('eve@example.com', jsonb('{"favorites":["Madonna", "Michael Jackson"], "name": {"first": "Eve", "last": "Davis"}}')),
  ('frank@example.com', jsonb('{"favorites":["Queen", "David Bowie"], "name": {"first": "Frank", "last": "Miller"}}')),
  ('grace@example.com', jsonb('{"favorites":["Radiohead", "Led Zeppelin"], "name": {"first": "Grace", "last": "Lee"}}')),
  ('hank@example.com', jsonb('{"favorites":["U2", "Radiohead"], "name": {"first": "Hank", "last": "Taylor"}}')),
  ('ivy@example.com', jsonb('{"favorites":["Adele", "Beyoncé"], "name": {"first": "Ivy", "last": "Anderson"}}')),
  ('jack@example.com', jsonb('{"favorites":["Radiohead", "Muse"], "name": {"first": "Jack", "last": "Thomas"}}')),
  ('kate@example.com', jsonb('{"favorites":["Taylor Swift", "Madonna"], "name": {"first": "Kate", "last": "Martinez"}}')),
  ('leo@example.com', jsonb('{"favorites":["Nirvana", "Daft Punk"], "name": {"first": "Leo", "last": "Garcia"}}'));
  ```

after this is not usable:
```sql
select * from users;
```
because data is stored as binary jsons
don't ever work with it directly, instead use:
```sql
select data -> 'name' ->> 'first', data -> 'name' ->> 'last' from users;
```
of to help column names
```sql
select data -> 'name' ->> 'first' as first, data -> 'name' ->> 'last' as last from users;
```
only if user has less than two favorites
```sql
SELECT data -> 'name' ->> 'first', data -> 'name' ->> 'last' FROM users WHERE json_array_length(data, '$.favorites') < 2;
```

advanced queries, table valued functions

```sql
SELECT  
  COUNT(f.value) AS count, f.value  
FROM  
  users, json_each(data ->> 'favorites') f  
GROUP BY  
  f.value  
ORDER BY  
  count DESC;  
  ```

#### manipulate
update users

```sql
set data = jsonb_insert(
  (select data from users where email = 'brian@example.com'),
  '$.favorites[#]',
  'The xx'
) where email = 'brian@example.com';
```

if you forgot to add `returning *`, check with
```sql
select * from users where email = 'brian@example.com';
```

#### convert from json to jsonb
```sql
update users set data = jsonb(data) where email = 'brian@example.com';
```

### Backups
if you have blog  
and want to deploy database together with app  
not replication, not sharding  
only a way to recover if all data was wiped

#### Litestream:
It allows you with very little effort to be able to stream point-in-time backups to somewhere else.

#### Minio
S3 but OpenSource and running on local  
using s3 protocol

start replicating

```bash
# be sure to replace data.db with something else if you called the file something else
litestream replicate data.db s3://chinook-backup.localhost:9000/data.db
```

restore to last version:
```bash
litestream restore -o data2.db s3://chinook-backup.localhost:9000/data.db
```

### Replication
probably at this point you want to move to postgres  
but there is a option to stay with sqlite and have replication

postgres: will allow to scale database and app independent  
sqlite: database and app will fight for same network resources  
(assuming there is replication of sqlite using litefs)

#### litefs
product of fly.io  
autoscalling vms  
which is successor of heroku  
which took market at store  
was expensive but so good to use  
but salesforce bought it and now heroku is shadow of itself

litefs: system for replicating sqlite across cluster  
FUSE-based: shared filesystem  
*(perhaps a builtin of linux)*

it's **eventually consistient**  
there is a lag, usually 200ms  
(if you need strong consistiency this is not it)

### libSQL
separate version of sqlite3  
open source and opening contributions  
- libSQL works as a server, more similar to Postgres
- it has user management
- libSQL works over HTTP (there is client in javascript)

after running libSQL deamon it opens a port  
and you can send REST POST to '/' with sql string

there is official npm for libsql

but if you need to replace app from sqlite3 to libSQL:  
npm @libsql/sqlite3  
almost the same as sqlite  
but Database is created with http url, like `localhost:3000`

setup of libSQL is some more work  
to self host

#### turso
you can also use  
turso: SQLite for Production  
https://turso.tech/

### Local first
you have frontend  
and you have local database  
in background local database is syncing with backend database

**turso**: has suppport for this  
**electric**: alternative, using postgres and pglite (similar to sqlite)

also possible to setup manually  
start libSQL with flag gprc, set it to port  
this means: I expect someone will be syncing with me here

in client

```javascript
const db = createClient({
  url: "file:local-data.db",
  syncUrl: "http://localhost:5001",
  syncPeriod: 60,
});
```

and then:
```javascript
const rep = await db.sync();
```

there is a way to run sqlite in web assembly wasm

when localfirst is NOT good idea:  
- if you need to be consistient on data all the time (not eventually consistient)
- it's more indirect and complicated setup
- requires more discipline how you structure data (divide db into shards, you cannot share everything)
- you can target only devices that can run wasm

#### Postgres
probably you need a reason to not use postgres

#### Vector data
store arrays of numbers,  
Vector data types allow the storage, manipulation, and querying of vectors (arrays of numbers) directly within the database.  
pgvector  
pinecone

### Other
npm google-spreadsheet: use google spreadsheet as database  
can you use drizzle to read from google-spreadsheet?  
Bruno: api client like Postman, Insomnia (and simpler, like Postman and Insomnia used to be)  
nodemon: nodemon is a tool that helps develop Node.js based applications by automatically restarting the node application when file changes in the directory are detected.  
node --watch server.js: no more need for nodemon  
Neon: Postgres service, serverless (so you don't need to setup your db service), scales to zero (so you don't pay if you don't use it)  
Vector data: store arrays of numbers, Vector data types allow the storage, manipulation, and querying of vectors (arrays of numbers) directly within the database.
