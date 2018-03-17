---
layout: post
title:      "Active Model Serializers & JSON Sideloading"
date:       2018-03-16 18:54:53 -0400
permalink:  active_model_serializers_and_json_sideloading
---

### *Introduction*
Depending on the application, JSON data transfers can be costly in terms of response time and data size.
When your data model has nested associations, each association gets fully converted into JSON.
This can lead to a lot of duplicated data being serialized and transmitted.
[Sideloading](https://github.com/rails-api/active_model_serializers/blob/0-10-stable/docs/general/adapters.md#include-option) repetitive association data is a way to reduce JSON payloads and improve response time.

(Note: Sideloading with active_model_serializers requires the use of the jason_api serializer adapter (see [Adapters](https://github.com/rails-api/active_model_serializers/blob/0-10-stable/docs/general/adapters.md)).
The default serializer adapter does not support sideloading.)

### *The Problem*
In this simplified example, we have monthly stock market price data for a particular financial instrument.
Each monthly price record has an `adjusted-close-price` and a `series-date`.
Note that the associated `instrument` data is nested in the price record.
```
data
  0	
    id	"0"
    type	"series"
    attributes	
      instrument	
        id	7206
        symbol	"SPY"
        name	"SPDR S&P 500"
        created_at	"2018-03-13T21:53:33.590Z"
        updated_at	"2018-03-13T21:53:33.590Z"
      time-interval	"MA"
      series-date	"2018-03-15T00:00:00.000Z"
      adjusted-close-price	"275.03"
      dividend-amount	"0.0"
      created-at	"2018-03-15T22:37:29.246Z"
      error	null
```
Here is the controller and serializer code that produced the above data -
```
def monthly_series
  series = DataCache.monthly_series(params[:symbols].split(','))
  render json: series
end

class SeriesSerializer < ActiveModel::Serializer
  attributes :id, :instrument, :time_interval, :series_date, :adjusted_close_price, :dividend_amount, :created_at, :error
end
```
In this test case, however, we're not retrieving one record, but 218. As you can see below, identical instrument data is being repeated in every month's price record -
```
data
  0	
    id	"0"
    type	"series"
    attributes	
      instrument	
        id	7206
        symbol	"SPY"
        name	"SPDR S&P 500"
        created_at	"2018-03-13T21:53:33.590Z"
        updated_at	"2018-03-13T21:53:33.590Z"
      time-interval	"MA"
      series-date	"2018-03-15T00:00:00.000Z"
      adjusted-close-price	"275.03"
      dividend-amount	"0.0"
      created-at	"2018-03-15T22:37:29.246Z"
      error	null
  1	
    id	"1"
    type	"series"
    attributes	
      instrument	
        id	7206
        symbol	"SPY"
        name	"SPDR S&P 500"
        created_at	"2018-03-13T21:53:33.590Z"
        updated_at	"2018-03-13T21:53:33.590Z"
      time-interval	"MA"
      series-date	"2018-02-28T00:00:00.000Z"
      adjusted-close-price	"271.65"
      dividend-amount	"0.0"
      created-at	"2018-03-15T22:37:29.246Z"
      error	null
  .
  .
  .
```
While the application needs information about the instrument pertaining to the price records, it doesn't need the same data 218 times.
This is a fairly trivial example.
A fully-populated instrument record will contain a lot more information than I'm showing here;
Perhaps several KB when the records are fully populated.
We could possibly make a custom serializer to provide the bare minimum instrument data for this API, but what data would that be? It would still be duplicated in every record. And, would the user then need to make a second request just to get a few pieces of missing information?

### *The Solution*
The answer to this problem is sideloading.
Sideloading is placing associated data alongside the main data, rather than embedding it.
This way, data for each instrument will appear only once in the JSON payload, instead of 218 times, as seen here -
```
data
  0	
    id	"0"
    type	"series"
    attributes	
      time-interval	"MA"
      series-date	"2018-03-15T00:00:00.000Z"
      adjusted-close-price	"275.03"
      dividend-amount	"0.0"
      created-at	"2018-03-15T23:45:30.097Z"
      error	null
    relationships	
      instrument	
        data	
          id	"7206"
          type	"instruments"
  1	
    id	"1"
    type	"series"
    attributes	
      time-interval	"MA"
      series-date	"2018-02-28T00:00:00.000Z"
      adjusted-close-price	"271.65"
      dividend-amount	"0.0"
      created-at	"2018-03-15T23:45:30.097Z"
      error	null
    relationships	
      instrument	
        data	
          id	"7206"
          type	"instruments"
  .
  .
  .
	
included
  0	
    id	"7206"
    type	"instruments"
    attributes	
      symbol	"SPY"
      name	"SPDR S&P 500"
```
You use the instrument id in each `data` record to locate the sideloaded instrument in the `included` records.
From there, you can extract the information you need.
The modified controller and serializer code looks like this -
```
def monthly_series
  series = DataCache.monthly_series(params[:symbols].split(','))
  render json: series, each_serializer: SeriesSerializer, include: 'instrument'
end
    
class SeriesSerializer < ActiveModel::Serializer
  attributes :id, :time_interval, :series_date, :adjusted_close_price, :dividend_amount, :created_at, :error
  has_one :instrument
end
```
Some overhead is added performing lookups in the `included` records, but that hit is more than offset by not serializing and transmitting so much unnecessary data.

### *Conclusion*
The original JSON size for 218 records in this trivial example was 86 KB.
By sideloading, that size was reduced to 62 KB bytes.
We saved 24 KB for a 28% size reduction.
Imagine if the instrument records were 2 KB each larger.
That would make our original payload 522 KB and the sideloaded payload 64 KB.
That's a lot of data that doesn't need to be serialized by the server our transmitted to the client.
Without sideloading, our application would not scale well.
