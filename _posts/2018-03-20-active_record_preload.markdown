---
layout: post
title:      "Active Record Preload"
date:       2018-03-20 22:14:05 -0400
permalink:  active_record_preload
---


### *Introduction*
I have an app that retrieves 640 data points from a database (from the Series table) that are then rendered to JSON and sent to the client to be plotted on a chart.

Active Record assembles an object from one or more database records.
Since the Series class includes the Instrument class, each Series object requires an Instrument object.

Although I only make one Active Record select call in my app, I noticed in the Rails server log that there were 640 database calls generated from that one Active Record call!

Luckily, Active Record provides a way to avoid these excessive database calls.
It's called preloading, aka [Eager Loading Associations](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations).

In the test case below, we are looking up Series data for 10 Instruments (by using the Instrument's symbol name, e.g., IBM).
Each of the 10 Instruments has 64 Series records, for a total of 640 Series records.

By using preloading, I was able to reduce processing time in half.

### *Without Preload*
All 640 Series records are retrieved in one Load call.
However, for each Series record retrieved, there is 1 Instrument Load call, making for total of 641 database calls.

[NOTE: There is some optimization present -
Ten of the 640 Instrument Load calls result in an actual database lookup while the remaining 630 Instrument Load calls are satisfied by Active Record's cache (denoted by the keyword CACHE in the console log), so no database call is performed.]

Normally, if we were making SQL calls ourselves, instead of through Active Record, we would expect one Select statement to result in one database call.

Here is the Active Record call without preloading -

```
Series.joins(:intrument).where(instruments: {symbol: symbols}).distinct.order('instrument_id, time_interval, series_date')
```

And the resulting log file output -

```
Started GET "/api/monthly-series?symbols=AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY" for 127.0.0.1 at 2018-03-20 19:11:39 -0400
Processing by SeriesController#monthly_series as JSON
  Parameters: {"symbols"=>"AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY"}
  [1m[36mSeries Load (9.1ms)[0m  [1m[34mSELECT DISTINCT "series".* FROM "series" INNER JOIN "instruments" ON "instruments"."id" = "series"."instrument_id" WHERE "instruments"."symbol" IN ('AAPL', 'AMZN', 'COF', 'FBGX', 'HD', 'URTH', 'IWM', 'QQQ', 'DIA', 'SPY') ORDER BY instrument_id, time_interval, series_date[0m
[active_model_serializers]   [1m[36mInstrument Load (0.3ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 1], ["LIMIT", 1]]
[active_model_serializers]   [1m[36mCACHE Instrument Load (0.0ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 1], ["LIMIT", 1]]
[active_model_serializers]   [1m[36mCACHE Instrument Load (0.0ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 1], ["LIMIT", 1]]
  .
  .
  .
[active_model_serializers]   [1m[36mInstrument Load (0.3ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
[active_model_serializers]   [1m[36mCACHE Instrument Load (0.0ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
[active_model_serializers]   [1m[36mCACHE Instrument Load (0.0ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
[active_model_serializers]   [1m[36mCACHE Instrument Load (0.0ms)[0m  [1m[34mSELECT  "instruments".* FROM "instruments" WHERE "instruments"."id" = $1 LIMIT $2[0m  [["id", 2], ["LIMIT", 1]]
  .
  .
  .
[active_model_serializers] Rendered ActiveModel::Serializer::CollectionSerializer with ActiveModelSerializers::Adapter::JsonApi (1253.71ms)
Completed 200 OK in 1290ms (Views: 1261.4ms | ActiveRecord: 25.4ms)
```

As you can see from the last line above, Rails reported the following run times:
```
  Completed 200 OK in 1122ms (Views: 1094.9ms | ActiveRecord: 21.3ms)
```
### *With Preload*
The same query is made as above, except that ```.preload(:instrument)``` is appended to the Select statement.

ActiveRecord now makes just two queries; One for Series and one for Instruments.
It then uses that data to create all the necessary objects without going back to the database.

Here is the ActiveRecord call with preloading -

```
Series.joins(:intrument).where(instruments: {symbol: symbols}).distinct.order('instrument_id, time_interval, series_date').preload(:instrument)
```

And the resulting log file output -

```
Started GET "/api/monthly-series?symbols=AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY" for 127.0.0.1 at 2018-03-20 19:02:16 -0400
Processing by SeriesController#monthly_series as JSON
  Parameters: {"symbols"=>"AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY"}
  [1m[36mSeries Load (6.6ms)[0m  [1m[34mSELECT DISTINCT "series".* FROM "series" INNER JOIN "instruments" ON "instruments"."id" = "series"."instrument_id" WHERE "instruments"."symbol" IN ('AAPL', 'AMZN', 'COF', 'FBGX', 'HD', 'URTH', 'IWM', 'QQQ', 'DIA', 'SPY') ORDER BY instrument_id, time_interval, series_date[0m
  [1m[36mInstrument Load (0.4ms)[0m  [1m[34mSELECT "instruments".* FROM "instruments" WHERE "instruments"."id" IN (1, 2, 4, 5, 6, 10, 12, 14, 16, 17)[0m
[active_model_serializers] Rendered ActiveModel::Serializer::CollectionSerializer with ActiveModelSerializers::Adapter::JsonApi (548.4ms)
Completed 200 OK in 598ms (Views: 587.6ms | ActiveRecord: 7.6ms)
```
From the last line above, Rails reported the following run times:
```
Completed 200 OK in 598ms (Views: 587.6ms | ActiveRecord: 7.6ms)
```
### *With Include and Eager_Load*
There are two variation of preload - ```.include``` and ```.eager_load```.
```
Series.select('series.*').include(:instrument).where(instruments: {symbol: symbols}).distinct.order('instrument_id, time_interval, series_date')

Series.select('series.*').eager_load(:instrument).where(instruments: {symbol: symbols}).distinct.order('instrument_id, time_interval, series_date')
```
In this case, they both generated the same queries - a single join on Series and Instrument.
```
Started GET "/api/monthly-series?symbols=AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY" for 127.0.0.1 at 2018-03-20 20:30:19 -0400
Processing by SeriesController#monthly_series as JSON
  Parameters: {"symbols"=>"AAPL,AMZN,COF,FBGX,HD,URTH,IWM,QQQ,DIA,SPY"}
  [1m[35mSQL (11.6ms)[0m  [1m[34mSELECT DISTINCT series.*, "series"."id" AS t0_r0, "series"."instrument_id" AS t0_r1, "series"."time_interval" AS t0_r2, "series"."series_date" AS t0_r3, "series"."open_price" AS t0_r4, "series"."high_price" AS t0_r5, "series"."low_price" AS t0_r6, "series"."close_price" AS t0_r7, "series"."adjusted_close_price" AS t0_r8, "series"."volume" AS t0_r9, "series"."dividend_amount" AS t0_r10, "series"."created_at" AS t0_r11, "series"."updated_at" AS t0_r12, "instruments"."id" AS t1_r0, "instruments"."symbol" AS t1_r1, "instruments"."name" AS t1_r2, "instruments"."created_at" AS t1_r3, "instruments"."updated_at" AS t1_r4 FROM "series" LEFT OUTER JOIN "instruments" ON "instruments"."id" = "series"."instrument_id" WHERE "instruments"."symbol" IN ('AAPL', 'AMZN', 'COF', 'FBGX', 'HD', 'URTH', 'IWM', 'QQQ', 'DIA', 'SPY') ORDER BY instrument_id, time_interval, series_date[0m
[active_model_serializers] Rendered ActiveModel::Serializer::CollectionSerializer with ActiveModelSerializers::Adapter::JsonApi (479.51ms)
Completed 200 OK in 552ms (Views: 536.1ms | ActiveRecord: 12.7ms)
```
with the following run times:
```
Completed 200 OK in 552ms (Views: 536.1ms | ActiveRecord: 12.7ms)
```

In this case, the single join incurred slightly increased database processing time over the preload option.

In some database installations, round-trip calls to the database are very expensive and are to be avoided whenever possible. That tends to make complicated SQL statements more desirable than multiple simpler ones.
In this case, I'm using a local single-user Postgres database, so round-trip overhead wasn't a factor.
It pays to experiment.

### ***Conclusion***
By using preload, we were able to cut overall response time in half.
Database utilization was 1/3 of the non-preload time.
This made the app more responsive.

In a multiuser environment, in addition to improving your app's response time, reducing the load on the server and the database frees up resources that can be used to satisfy requests from other users faster.

