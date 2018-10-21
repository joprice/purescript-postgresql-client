# purescript-postgresql-client

purescript-postgresql-client is a PostgreSQL client library for PureScript.

## Install

To use this library, you need to add `pg` and `decimal.js` as an npm dependency. You can also
find first of them on [https://github.com/brianc/node-postgres][pg].

## Usage

This guide is a literate Purescript file which is compiled into testing module (using [`literate-purescript`](https://github.com/Thimoteus/literate-purescript) - check `bin/docs.sh`) so it is a little verbose.

Let's start with imports and some testing boilerplate.

``` purescript
module Test.Example where

import Prelude

import Database.PostgreSQL (defaultPoolConfiguration, execute, newPool, Query(Query), withConnection)
import Database.PostgreSQL.Row (Row0(Row0), Row3(Row3))
import Data.Decimal as Decimal
import Data.Maybe (Maybe(..))
import Data.Tuple.Nested ((/\))
import Effect.Aff (Aff)

-- Our interaction with db is performed asynchronously in `Aff`
run ∷ Aff Unit
run = do

  -- Now we are able to setup connection. We are assuming here
  -- that postgres is running on a standard local port.
  -- We use `ident` authentication so configuration can be nearly empty.
  -- It requires only database name which we pass to `newPool` function.
  -- We want to close connection after a second (`idleTimeoutMillis` setting) because this code
  -- would be run by our test suite ;-)
  -- Of course you can provide additional configuration settings if you need to.

  pool <- newPool ((defaultPoolConfiguration "purspg") { idleTimeoutMillis = Just 1000 })
  withConnection pool \conn -> do

    -- We can now create our temporary table which we are going to query in this example.
    -- `execute` performs this query. It ignores result value by default.
 
    execute conn (Query """
      CREATE TEMPORARY TABLE foods (
        name text NOT NULL,
        delicious boolean NOT NULL,
        price NUMERIC(4,2) NOT NULL,
        added TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (name)
      );
    """) Row0

    -- We can insert some data calling `execute` function with `INSERT` statement.
    -- Please notice that we are passing a tuple of arguments to this query
    -- using dedicated constructor (in this case `Row3`).
    -- This library provides types from `Row0` to `Row19` and they are wrappers which
    -- provides instances for automatic conversions from and to SQL values.

    execute conn (Query """
      INSERT INTO foods (name, delicious, price)
      VALUES ($1, $2, $3)
    """) (Row3 "pork" true (Decimal.fromString "8.30"))


    -- You can also use nested tuples instead of `Row*` types but this can be a bit more
    -- verbose. `/\` is just an alias for `Tuple` constructor.

    execute conn (Query """
      INSERT INTO foods (name, delicious, price)
      VALUES ($1, $2, $3)
    """) ("sauerkraut" /\ false /\ Decimal.fromString "3.30")

```



## Generating SQL Queries

The purspgpp preprocessor has been replaced by [sqltopurs], which is a code
generator instead of a preprocessor, and easier to use.

[sqltopurs]: https://github.com/rightfold/sqltopurs


## Testing

To run tests you have to prepare "purspg" database and use standard command: `pulp test`.
