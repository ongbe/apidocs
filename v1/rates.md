---
title: Rates | OANDA API
---

# Rate Endpoints

* TOC
{:toc}

----------------------------

## Get an instrument list

Get a list of tradeable instruments (currency pairs, CFDs, and commodities) that are available for trading with the account specified.

    GET /v1/instruments

#### Input Query Parameters

accountId
: _Required_ The account id to fetch the list of tradeable instruments for. 

fields
: _Optional_ An URL encoded (*%2C*) comma separated list of instrument fields that are to be returned in the response.
             The __instrument__ field will be returned regardless of the input to this query parameter.
             Please see the Response Parameters section below for a list of valid values.

instruments
: _Optional_ An URL encoded (*%2C*) comma separated list of instruments that are to be returned in the response.
             If the instruments option is not specified, all instruments will be returned.

#### Example
    curl -X GET "http://api-sandbox.oanda.com/v1/instruments?accountId=12345&instruments=AUD_CAD%2CAUD_CHF"

#### Response

###### Header

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 264
~~~

###### Body

~~~json
{
  "instruments" : [
    {
      "instrument" : "AUD_CAD",
      "displayName" : "AUD\/CAD",
      "pip" : "0.0001",
      "maxTradeUnits" : 10000000
    },
    {
      "instrument" : "AUD_CHF",
      "displayName" : "AUD\/CHF",
      "pip" : "0.0001",
      "maxTradeUnits" : 10000000
    }
  ]

}
~~~


#### Response Parameters


instrument
: Name of the instrument.  This value should be used to fetch prices and create orders and trades.

displayName
: Display name for the end user.

pip
: Value of 1 pip for the instrument. [More on pip](http://www.babypips.com/school/pips-and-pipettes.html)

maxTradeUnits
: The maximum number of units that can be traded for the instrument.

precision
: The smallest unit of measurement to express the change in value between the instrument pair. 

maxTrailingStop
: The maximum trailing stop value (in pips) that can be set when trading the instrument.

minTrailingStop
: The minimum trailing stop value (in pips) that can be set when trading the instrument.

marginRate
: The margin requirement for the instrument. A 3% margin rate will be represented as 0.03.
 
If the __fields__ parameter was not specified in the request, the default instrument fields returned are __instrument__, __displayName__, __pip__, __maxTradeUnits__.

----

## Get current prices

    GET /v1/prices

Fetch live prices for specified instruments that are available on the OANDA platform.

#### Input Query Parameters

instruments
: _Required_  An URL encoded (*%2C*) comma separated list of instruments to fetch prices for.  Values should be one of the available instrument from the /v1/instruments response.

since
: _Optional_  When specified, only prices that occurred after the specified timestamp are returned.  The value specified must be in a valid [datetime format](/docs/v1/guide/#datetime-format).


#### Example
    curl -X GET "http://api-sandbox.oanda.com/v1/prices?instruments=EUR_USD%2CUSD_JPY%2CEUR_CAD"

#### Response

###### Header

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 379
~~~

###### Body

~~~json
{
  "prices": [
    {
      "instrument":"EUR_USD",
      "time":"2013-06-21T17:41:04.648747Z",  // time in RFC3339 format
      "bid":1.31513,
      "ask":1.31528
    },
    {
      "instrument":"USD_JPY",
      "time":"2013-06-21T17:49:02.475381Z",
      "bid":97.618,
      "ask":97.633
    },
    {
      "instrument":"EUR_CAD",
      "time":"2013-06-21T17:51:38.063560Z",
      "bid":1.37489,
      "ask":1.37517,
      "status": "halted"                    // this response parameter will only appear if the instrument is currently halted on the OANDA platform.
    }
  ]
}
~~~

----

## Retrieve instrument history

Get historical information on an instrument

    GET /v1/candles

#### Input Query Parameters

instrument
: _Required_  Name of the instrument to retrieve history for.  The instrument should be one of the available instrument from the /v1/instruments response.

granularity<sup>1</sup>
: _Optional_  The time range represented by each candlestick.  The value specified will determine the alignment of the first candlestick.
    
	Valid values are:

	* __Top of the minute alignment__
		* "S5"  - 5 seconds
		* "S10" - 10 seconds
		* "S15" - 15 seconds
		* "S30" - 30 seconds
		* "M1"  - 1 minute
	* __Top of the hour alignment__
		* "M2"  - 2 minutes
		* "M3"  - 3 minutes
		* "M5"  - 5 minutes
		* "M10" - 10 minutes
		* "M15" - 15 minutes
		* "M30" - 30 minutes
		* "H1"  - 1 hour
	* __Start of day alignment (default 17:00, Timezone/New York)__
		* "H2"  - 2 hours
		* "H3"  - 3 hours
		* "H4"  - 4 hours
		* "H6"  - 6 hours
		* "H8"  - 8 hours
		* "H12" - 12 hours
		* "D"   - 1 Day
	* __Start of week alignment (default Friday)__
		* "W"   - 1 Week
	* __Start of month alignment (First day of the month)__
		* "M"   - 1 Month
	

    The default for __granularity__ is "S5" if the granularity parameter is not specified.

count
: _Optional_  The number of candles to return in the response. This parameter may be ignored by the server depending on the time range provided. See "Time and Count Semantics" below for a full description.  * 
If not specified, __count__ will default to 500. The maximum acceptable value for __count__ is 5000.
             
	__count__ should not be specified if both the __start__ and __end__ parameters are also specified.

start<sup>2</sup>
: _Optional_  The start timestamp for the range of candles requested.  The value specified must be in a valid [datetime format](/docs/v1/guide/#datetime-format).

end<sup>2</sup>
: _Optional_  The end timestamp for the range of candles requested.  The value specified must be in a valid [datetime format](/docs/v1/guide/#datetime-format).

candleFormat
: _Optional_ Candlesticks representation ([about candestick representation](#about-candlestick-representation)). This can be one of the following:

	* "midpoint" - Midpoint based candlesticks.
	* "bidask" - Bid/Ask based candlesticks

	The default for __candleFormat__ is "bidask" if the candleFormat parameter is not specified.

includeFirst
: _Optional_  A boolean field which may be set to "true" or "false". If it is set to "true", the candlestick covered by the <i>start</i> timestamp will be returned. If it is set to "false", this candlestick will not be returned.
This field exists to provide clients a mechanism to not repeatedly fetch the most recent candlestick which it is not a "Dancing Bear".

    The default for __includeFirst__ is "true" if the includeFirst parameter is not specified.

dailyAlignment
: _Optional_  The hour of day used to align candles with hourly, daily, weekly, or monthly granularity. The value specified is interpretted as an hour in the timezone set through the alignmentTimezone parameter and must be an integer between 0 and 23.

    The default for __dailyAlignment__ is 17 if the dailyAlignment parameter is not specified.

alignmentTimezone
: _Optional_  The timezone to be used for the dailyAlignment parameter. This parameter does NOT affect the returned timestamp, the start or end parameters, these will always be in UTC. The timezone format used is defined by the [IANA Time Zone Database](http://en.wikipedia.org/wiki/Tz_database), a full list of the timezones supported by the REST API can be found [here](/docs/timezones.txt).

    The default for __alignmentTimezone__ is "America/New_York" if the alignmentTimezone parameter is not specified.

weeklyAlignment
: _Optional_ The day of the week used to align candles with weekly granularity. The value specified will be used as the start/end day when calculating the weekly candles. Valid values are: "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday".

    The default for __weeklyAlignment__ is "Friday" if the weeklyAlignment parameter is not specified.

<sup>1</sup> No candles are published for intervals where there are no ticks.  This will result in gaps in between time periods.<br>
<sup>2</sup> If neither __start__ nor __end__ time are specified by the requester, __end__ will default to the current time and __count__ candles will be returned.<br>

#### Example
    curl -X GET "http://api-sandbox.oanda.com/v1/candles?instrument=EUR_USD&count=2&candleFormat=midpoint&granularity=D&dailyAlignment=0&alignmentTimezone=America%2FNew_York"

#### Response

###### Header

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 429
~~~

###### Body

~~~json
{
    "instrument" : "EUR_USD",
    "granularity" : "D",
    "candles" : [
        {
            "time" : "2014-07-02T04:00:00.000000Z", // time in RFC3339 format
            "openMid" : 1.36803,
            "highMid" : 1.368125,
            "lowMid" : 1.364275,
            "closeMid" : 1.365315,
            "volume" : 28242,
            "complete" : true
        },
        {
            "time" : "2014-07-03T04:00:00.000000Z", // time in RFC3339 format
            "openMid" : 1.36532,
            "highMid" : 1.366445,
            "lowMid" : 1.35963,
            "closeMid" : 1.3613,
            "volume" : 30487,
            "complete" : false
        }
    ]
}
~~~

#### Example
    curl -X GET "http://api-sandbox.oanda.com/v1/candles?instrument=EUR_USD&start=2014-06-19T15%3A47%3A40Z&end=2014-06-19T15%3A47%3A50Z"

#### Response

###### Header

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 634
~~~

###### Body

~~~json
{
    "instrument" : "EUR_USD",
    "granularity" : "S5",
    "candles" : [
        {
            "time" : "2014-06-19T15:47:40.000000Z",
            "openBid" : 1.25682,
            "openAsk" : 1.25691,
            "highBid" : 1.25682,
            "highAsk" : 1.25691,
            "lowBid" : 1.25642,
            "lowAsk" : 1.25651,
            "closeBid" : 1.25642,
            "closeAsk" : 1.25651,
            "volume" : 9,
            "complete" : true
        },
        {
            "time" : "2014-06-19T15:47:45.000000Z",
            "openBid" : 1.25644,
            "openAsk" : 1.25653,
            "highBid" : 1.25644,
            "highAsk" : 1.25653,
            "lowBid" : 1.25634,
            "lowAsk" : 1.25643,
            "closeBid" : 1.25634,
            "closeAsk" : 1.25643,
            "volume" : 4,
            "complete" : true
        }
    ]
}
~~~

----

## About Candlestick Representation

__midpoint__ midpoint-based candlesticks with tick volume

~~~json
{
  "time":<TS>,
  "openMid":<O_m>,
  "highMid":<H_m>,
  "lowMid":<L_m>,
  "closeMid":<C_m>,
  "volume":<V>,
  "complete":<DB>
}
~~~

__bidask__ - BID/ASK-based candlesticks with tick volume

~~~json
{
  "time":<TS>,
  "openBid":<O_b>,
  "openAsk":<O_a>,
  "highBid":<H_b>,
  "highAsk":<H_a>,
  "lowBid":<L_b>,
  "lowAsk":<L_a>,
  "closeBid":<C_b>,
  "closeAsk":<C_a>,
  "volume":<V>,
  "complete":<DB>
}
~~~

The fields in the above candlesticks have the following meanings:

* TS  - Start timestamp of the candlestick
* O_b - Open Tick Bid
* O_a - Open Tick Ask
* O_m - Open Tick Mid
* H_b - High Tick Bid
* H_a - High Tick Ask
* H_m - High Tick Mid
* L_b - Low Tick Bid
* L_a - Low Tick Ask
* L_m - Low Tick Mid
* C_b - Close Tick Bid
* C_a - Close Tick Ask
* C_m - Close Tick Mid
* V   - Candlestick tick volume
* DB  - "Dancing Bear". This field indicates that the candlestick
        may still be modified by the creation of ticks in the future because
        the current server time is contained by the time range which the
        candlestick represents.

