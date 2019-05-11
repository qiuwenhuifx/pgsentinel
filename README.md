`pgsentinel` – sampling active session history
=============================================================

[![Build Status](https://travis-ci.org/pgsentinel/pgsentinel.svg?branch=master)](https://travis-ci.org/pgsentinel/pgsentinel)

Introduction
------------

PostgreSQL provides session activity. However, in order to gather activity  
behavior user have to sample the pg_stat_activity view multiple times.
`pgsentinel` is an extension to record active session history and also link
 the activity with query statistics (`pg_stat_statements`).


The module must be loaded by adding `pgsentinel` to
`shared_preload_libraries` in postgresql.conf, means that a server restart
is needed to add or remove the module.

When `pgsentinel` is enabled, it collects the history of session activity:

 * It's implemented as in-memory ring buffer where
   samples are written with given (configurable)
   period.  Therefore, user can see some number of
   recent samples depending on history size (configurable).  
   
In combination with `pg_stat_statements` this extension can also link the session activity with
query statistics.

`pgsentinel` launches special background worker for gathering the
sessions activity.

Availability
------------

`pgsentinel` is implemented as an extension and not available in default
PostgreSQL installation. It is available from
[github](https://github.com/pgsentinel/pgsentinel)
under the same license as
[GNU AFFERO GENERAL PUBLIC LICENSE](https://github.com/pgsentinel/pgsentinel/blob/master/LICENSE)
and supports PostgreSQL 9.6+.

Installation
------------

`pgsentinel` is a PostgreSQL extension which requires PostgreSQL 9.6 or
higher. Before build and install you should ensure the following:

 * PostgreSQL version is 9.6 or higher.
 * You built PostgreSQL from source.
 * Your PATH variable is configured so that `pg_config` command available, or
   set PG_CONFIG variable.

Typical installation procedure may look like this:

As pgsentinel uses the `pg_stat_statements` extension (officially bundled with PostgreSQL) for tracking which queries get executed in your database, add the following entries to your postgres.conf:

    $ shared_preload_libraries = 'pg_stat_statements,pgsentinel'
    $ # Icncrease the max size of the query strings Postgres records
    $ track_activity_query_size = 2048
    $ # Track statements generated by stored procedures as well
    $ pg_stat_statements.track = all

restart the postgresql daemon and create the extension:

    $ git clone https://github.com/pgsentinel/pgsentinel.git
    $ cd pgsentinel/src
    $ make
    $ sudo make install
    $ psql DB -c "CREATE EXTENSION pgsentinel;"


Usage
-----

`pgsentinel` reports the active session history activity through the `pg_active_session_history` view:

 |     Column      |           Type           | Collation | Nullable | Default  |
 | ------------------ | -------------------------- | -----------  | ----------  | ---------  |
  | ash_time         | timestamp with time zone |           |          |  |
  | datid            | oid                      |           |          |  |
  | datname          | text                     |           |          |  |
  | pid              | integer                  |           |          |  |
  | usesysid         | oid                      |           |          |  |
  | usename          | text                     |           |          |  |
  | application_name | text                     |           |          |  |
  | client_addr      | text                     |           |          |  |
  | client_hostname  | text                     |           |          |  |
  | client_port      | integer                  |           |          |  |
  | backend_start    | timestamp with time zone |           |          |  |
  | xact_start       | timestamp with time zone |           |          |  |
  | query_start      | timestamp with time zone |           |          |  |
  | state_change     | timestamp with time zone |           |          |  |
  | wait_event_type  | text                     |           |          |  |
  | wait_event       | text                     |           |          |  |
  | state            | text                     |           |          |  |
  | backend_xid      | xid                      |           |          |  |
  | backend_xmin     | xid                      |           |          |  |
  | top_level_query  | text                     |           |          |  |
  | query            | text                     |           |          |  |
  | cmdtype          | text                     |           |          |  |
  | queryid          | bigint                   |           |          |  |
  | backend_type     | text                     |           |          |  |
  | blockers         | integer                  |           |          |  |
  | blockerpid       | integer                  |           |          |  |
  | blocker_state    | text                     |           |          |  |

You could see it as samplings of `pg_stat_activity` providing more information:

* `ash_time`: the sampling time
* `top_level_query`: the top level statement (in case PL/pgSQL is used)
* `query`: the statement being executed (not normalised, as it is in `pg_stat_statements`, means you see the values)
* `cmdtype`: the statement type (SELECT,UPDATE,INSERT,DELETE,UTILITY,UNKNOWN,NOTHING)
* `queryid`: the queryid of the statement which links to pg_stat_statements
* `blockers`: the number of blockers
* `blockerpid`: the pid of the blocker (if blockers = 1), the pid of one blocker (if blockers > 1)
* `blocker_state`: state of the blocker (state of the blockerpid) 

The worker is controlled by the following GUCs:.

|         Parameter name              | Data type |                  Description                | Default value | Min value  |
| ----------------------------------- | --------- | ------------------------------------------- | ------------  | -------- |
| pgsentinel_ash.sampling_period     | int4      | Period for history sampling in seconds |            1 | 1 |
| pgsentinel_ash.max_entries     | int4      | Size of history in-memory ring buffer |            1000 | 1000 |
| pgsentinel.db_name        | char      |  database the worker should connect to          |          postgres | |

Remarks for PostgreSQL 9.6
-------------------------

* The `backend_type` field does not exist into the `pg_stat_activity` view
* The `backend_type` field exists into the `pg_active_session_history` view and is always NULL (as a consequence of the previous remark)

See how to query the view in this short video
-------------
[![Alt text](https://raw.githubusercontent.com/pgsentinel/pg_ash_scripts/master/images/video_pg_ash_scripts.PNG)](https://www.youtube.com/watch?v=WVKzKjlK75U)


### The videos are available on [youtube](https://www.youtube.com/channel/UCGVciSS2YwnPhtHHGB3Ep3A) 


Contribution
------------

Please, notice, that `pgsentinel` is still in beta testing so may contain some bugs. Don't hesitate to raise
[issues at github](https://github.com/pgsentinel/pgsentinel/issues) with
your bug report.

If you're lacking of some functionality in `pgsentinel`  
then you're welcome to make pull requests.

Author
-------
 
 * Bertrand Drouvot <bdrouvot@gmail.com>,
   Metz, France [!['test'](https://www.pgsentinel.com/images/twitter.png)](https://twitter.com/BertrandDrouvot) [!['test'](https://www.pgsentinel.com/images/linkedin.png)](https://www.linkedin.com/in/bdrouvot/)
