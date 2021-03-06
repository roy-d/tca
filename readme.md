#Twitter Count Application

Reads from a stream of timestamped tweets and allows querying the number
 of tweets in a given time range containing keywords from a pre-specified
  set of keywords.

config: ```src/main/resources/application.conf``` has keywords and twitter keys

run: ```sbt "run-main lab.tca.Server"```

test: ```sbt clean test```

#Design Considerations:
* List of keywords is pre-configured in application.conf
* Matching of **keyword** is case in-sensitive
* **start** and **end** in query are INCLUSIVE
* Queries have **end** with minute precision i.e. `hh:mm`. At `hh:mm:ss` the application
returns correct results for `hh:mm`. It is anyway impossible to return correct count
 for `hh:mm+1` at time `hh:mm:ss`.
* **Writer flow**: Twitter -> Spark Stream -> aggregation every minute -> cyclic buffer(optimized for reads)
* **Reader flow**: Finch endpoint <- get by key on Map, then use `from` 
and `to` on TreeMap <- Map (O(1)) of TreeMap (O(log(n)) red-black tree)
* **Future work**: kafka as buffer, orderly shutdown, logging, micro benchmarks, more test coverage

#CI
[![Build Status](https://travis-ci.org/roy-d/tca.svg?branch=master)](https://travis-ci.org/roy-d/tca)

#Performance Considerations:
* Monitor: ```http://localhost:4040/metrics/json/``` and get a sense if
 processing is keeping up with ingestion. Spark has many scaling and tuning option.
* Tweak batch interval, window length, sliding interval, checkpoints
* Writes from Storage are happening from driver and are aggregated by 
1 minute. Monitor write throughput.
* Storage can be partitioned by days and keyword too if there are too many
Readers. Monitor read throughput by keyword.
* Retrieval can be on Spark if needed further allowing distribution of the workload.
* Web server is Finch and it can be load tested using wrk. If needed 
there can be one web server per keyword.
* I would microbenchmark all the scala functions too e.g. ```https://github.com/roy-d/benchmark```
