---
layout: post
title:      "Active Record SQL & Explain Plan"
date:       2018-04-02 07:57:56 -0400
permalink:  active_record_sql_and_explain_plan
---


### *Introduction*
I have an app that retrieves 640 data points from a Postgres database with a two-table join.
The join was taking about 3 minutes to return results, so I decided to see if that could be improved.
I examined the query plans and added a ```distinct``` clause to the query.
This dropped the query response time from 3 minutes to less than 1 second.
Active Record's [.explain](http://guides.rubyonrails.org/active_record_querying.html#running-explain) function is an excellent resource to help tune SQL performance.
Explain will display your query's query execution plan.
From there, you can tell whether or not the query is behaving as you expect.
For instance, whether the query is taking advantage of available indexes.

Sometimes, when the query does not appear to be optimal, there is a good reason for it.
E.g., if using an index would take longer than just scanning the table.
Other times, the query execution is not recognizing factors that would improve performance.

Since Explain Plan is not a SQL standard, each database vendor has there own syntax for calling it.
It appears that Active Record is able to convert the ```.explain``` call on your query to the explain plan syntax required by your particular database's vendor.

#### *The Data*
The two tables are ```series``` and ```instruments```.
The ```instruments``` table has ```id``` as it's unique primary key.
The ```series``` table has ```instrument_id``` as a foreign key to the ```instruments``` table.
The table schemas are -

***The Instruments Table***

An instrument describes an equity trading on an exchange.
The instrument has a symbol and an associated company name.
For example, INTC & Intel Corporation.
```
> \d+ instruments
Table "public.instruments"
   Column   |            Type             |                        Modifiers                         |
------------+-----------------------------+----------------------------------------------------------+
 id         | bigint                      | not null default nextval('instruments_id_seq'::regclass) |
 symbol     | character varying           | not null                                                 |
 name       | character varying           | not null                                                 |
 created_at | timestamp without time zone | not null                                                 |
 updated_at | timestamp without time zone | not null                                                 |
Indexes:
 "instruments_pkey" PRIMARY KEY, btree (id)
 "instruments_by_symbol" UNIQUE, btree (symbol)
Referenced by:
 TABLE "positions" CONSTRAINT "fk_rails_2adf75f9a8" FOREIGN KEY (instrument_id) REFERENCES instruments(id)
 TABLE "series" CONSTRAINT "fk_rails_2bd701b643" FOREIGN KEY (instrument_id) REFERENCES instruments(id)
 TABLE "trades" CONSTRAINT "fk_rails_c19f3a5dfc" FOREIGN KEY (instrument_id) REFERENCES instruments(id)

> select count(*) from instruments;
8586
```
***The Series Table***

The ```series``` table contains monthly price data points for an instrument.
At this time, there are typically 64 data points in the ```series``` table for each instrument.
```
> \d+ series
Table "public.series"
   Column             |            Type             |                        Modifiers                    |
----------------------+-----------------------------+-----------------------------------------------------+
 id                   | bigint                      | not null default nextval('series_id_seq'::regclass) |
 instrument_id        | bigint                      | not null                                            |
 time_interval        | character varying           | not null                                            |
 series_date          | timestamp without time zone | not null                                            |
 open_price           | numeric                     | not null                                            |
 high_price           | numeric                     | not null                                            |
 low_price            | numeric                     | not null                                            |
 close_price          | numeric                     | not null                                            |
 adjusted_close_price | numeric                     | not null                                            |
 volume               | numeric                     | not null                                            |
 dividend_amount      | numeric                     | not null                                            |
 created_at           | timestamp without time zone | not null                                            |
 updated_at           | timestamp without time zone | not null                                            |
Indexes:
 "series_pkey" PRIMARY KEY, btree (id)
 "index_series_on_instrument_id_and_time_interval_and_series_date" UNIQUE, btree (instrument_id, time_interval, series_date)
 "index_series_on_instrument_id" btree (instrument_id)
Foreign-key constraints:
 "fk_rails_2bd701b643" FOREIGN KEY (instrument_id) REFERENCES instruments(id)

> select count(*) from series;
406002

> select count(distinct instrument_id) from series;
7984
```
#### *The Query*
The query is looking for all instruments that do **not** have any data in the series table.

***The Original Query***

This Active Record query -
```
 Instrument.select(:id, :symbol).where.not(id: Series.select('instrument_id'))
 ```
produced this SQL -
```
 SELECT "instruments"."id", "instruments"."symbol" FROM "instruments" WHERE ("instruments"."id" NOT IN (SELECT "series"."instrument_id" FROM "series"))
```
and this query plan -
```
QUERY PLAN
-----------------------------------------------------------------------------
 Seq Scan on instruments  (cost=0.00..62760584.18 rows=4293 width=12)
   Filter: (NOT (SubPlan 1))
   SubPlan 1
     ->  Materialize  (cost=0.00..13624.63 rows=397842 width=8)
           ->  Seq Scan on series  (cost=0.00..10080.42 rows=397842 width=8)
```
As you can see from the query plan, we are doing table scans on instruments and series.
No use of indices is evident.
This query was taking approximately 3 minutes to execute.

***The New Query***

We have a unique instrument id index on the ```instruments``` table and a foreign key index on instrument_id in the ```series``` table, but they are not being used.
I'm pretty sure that a smarter query analyzer would know that ```distinct``` is implied when you have a subselect with an IN clause.
In this case, Postgres' query analyzer didn't seem to pick up on that.
So I added the distinct keyword to the query.

This Active Record query -
```
 Instrument.select(:id, :symbol).where.not(id: Series.select('instrument_id').distinct)
```
produced this SQL -
```
 SELECT "instruments"."id", "instruments"."symbol" FROM "instruments" WHERE ("instruments"."id" NOT IN (SELECT DISTINCT "series"."instrument_id" FROM "series"))
```
and this query plan -
```
QUERY PLAN
-----------------------------------------------------------------------------
 Seq Scan on instruments  (cost=11160.96..11369.29 rows=4293 width=12)
   Filter: (NOT (hashed SubPlan 1))
   SubPlan 1
     ->  HashAggregate  (cost=11076.84..11144.14 rows=6730 width=8)
           Group Key: series.instrument_id
           ->  Seq Scan on series  (cost=0.00..10082.07 rows=397907 width=8)
```
As you can see from the query plan, we are now taking advantage of the ```series``` table instrument id index.
This query is taking less than 1 second to execute.

### ***Conclusion***
By using the distinct keyword, we were able to cut database response time from 3 minutes to 1 second.
This was a substantial reduction in the load on the database.

In a multiuser environment, in addition to improving your app's response time, reducing the load on the database frees up resources that can be used to satisfy requests from other users faster.

