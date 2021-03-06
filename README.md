# Dexter

The automatic indexer for Postgres

[Read about how it works](https://medium.com/@ankane/introducing-dexter-the-automatic-indexer-for-postgres-5f8fa8b28f27)

## Installation

First, install [HypoPG](https://github.com/dalibo/hypopg) on your database server. This doesn’t require a restart.

```sh
wget https://github.com/dalibo/hypopg/archive/1.0.0.tar.gz
tar xf 1.0.0.tar.gz
cd hypopg-1.0.0
make
make install # may need sudo
```

> Note: If you have issues, make sure `postgresql-server-dev-*` is installed.

Enable logging for slow queries in your Postgres config file.

```ini
log_min_duration_statement = 10 # ms
```

And install the command line tool with:

```sh
gem install pgdexter
```

The command line tool is also available as a [Linux package](guides/Linux.md).

## How to Use

Dexter needs a connection to your database and a log file to process.

```sh
tail -F -n +1 <log-file> | dexter <database-url>
```

This finds slow queries and generates output like:

```
2017-06-25T17:52:19+00:00 Started
2017-06-25T17:52:22+00:00 Processing 189 new query fingerprints
2017-06-25T17:52:22+00:00 Index found: genres_movies (genre_id)
2017-06-25T17:52:22+00:00 Index found: genres_movies (movie_id)
2017-06-25T17:52:22+00:00 Index found: movies (title)
2017-06-25T17:52:22+00:00 Index found: ratings (movie_id)
2017-06-25T17:52:22+00:00 Index found: ratings (rating)
2017-06-25T17:52:22+00:00 Index found: ratings (user_id)
2017-06-25T17:53:22+00:00 Processing 12 new query fingerprints
```

To be safe, Dexter will not create indexes unless you pass the `--create` flag. In this case, you’ll see:

```
2017-06-25T17:52:22+00:00 Index found: ratings (user_id)
2017-06-25T17:52:22+00:00 Creating index: CREATE INDEX CONCURRENTLY ON "ratings" ("user_id")
2017-06-25T17:52:37+00:00 Index created: 15243 ms
```

## Single Statement Mode

You can also pass a single statement with:

```sh
dexter <database-url> -s "SELECT * FROM ..."
```

## Options

Name | Description | Default
--- | --- | ---
exclude | prevent specific tables from being indexed | None
interval | time to wait between processing queries, in seconds | 60
log-level | `debug` gives additional info for suggested indexes<br />`debug2` gives additional info for processed queries | info
log-sql | log SQL statements executed | false
min-time | only process queries consuming a min amount of DB time, in minutes | 0

## Future Work

[Here are some ideas](https://github.com/ankane/dexter/issues/1)

## Thanks

This software wouldn’t be possible without [HypoPG](https://github.com/dalibo/hypopg), which allows you to create hypothetical indexes, and [pg_query](https://github.com/lfittl/pg_query), which allows you to parse and fingerprint queries. A big thanks to Dalibo and Lukas Fittl respectively.

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/dexter/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/dexter/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features

To get started, run:

```sh
git clone https://github.com/ankane/dexter.git
cd dexter
bundle
rake install
```

To run tests, use:

```sh
rake test
```
