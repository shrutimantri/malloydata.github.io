# Percent of Total
Malloy provides a wayh to compute percent of total through level of detail (ungrouped aggregates) functions.  Functions `all()` and `exclude()` to escape grouping in aggregate calculations.  These functions are different than window functions as they operate inline with the query and can produce correct results even when the data hits a `limit` or is fanned out.  Use cases below.

```malloy
--! {"isModel": true, "modelPath": "/inline/e1.malloy"}
source: flights is table('duckdb:data/flights.parquet') {
  join_one: carriers is table('duckdb:data/carriers.parquet') on carrier=carriers.code
  measure: flight_count is count()
}
```
## Totals
You can easily product a cloumn total that includes all the data, not just the data in the table.  Southwest + USAir = 126,434 flights.  Notice that `all_flights` is the total, of all the flight, not just the ones in the table

```malloy
--! {"isRunnable": true, "isPaginationEnabled": true, "size": "large", "source": "/inline/e1.malloy", "pageSize":5000}
run: flights -> {
  group_by: carriers.nickname
  aggregate: 
    flight_count
    all_flights is all(flight_count)
    limit: 2
}
```

## Percent of Total
The `all()` function is really useful in percent of total calculations.  The `# percent` tags the result so it is displayed as a percentage.

```malloy
--! {"isRunnable": true, "isPaginationEnabled": true, "size": "large", "source": "/inline/e1.malloy", "pageSize":5000}
run: flights -> {
  group_by: carriers.nickname
  aggregate: 
    flight_count
    # percent
    percent_of_flights is flight_count/all(flight_count)
    limit: 2
}
```

## All of a particular grouping
The `all()` function can optionally take the names of output columns to show all of a particular value.  You can see that all of Southwests fights is still 88,751.  The output column name for `carriers.nickname` is `nickname` so we use that in the calculation.  The `exclude()` function lets you eliminate a dimension from grouping.

```malloy
--! {"isRunnable": true, "isPaginationEnabled": true, "size": "large", "source": "/inline/e1.malloy", "pageSize":5000}
run: flights -> {
  group_by:
    carriers.nickname
    destination
    origin
  aggregate: 
    flight_count
    flights_by_this_carrier is all(flight_count, nickname)
    flights_to_this_destination is all(flight_count, destination)
    flights_by_this_origin is all(flight_count, origin)
    flights_on_this_route is exclude(flight_count, nickname)
  limit: 20
}
```
##  As Percentages
Displaying results as percentages is often gives clues as to how numbers relate.  Is this number a large or small percentage of the group?  Level of detail calculations are great for this.  In Malloy, identifiers enclosed in back-ticks can have spaces.

```malloy
--! {"isRunnable": true, "isPaginationEnabled": true, "size": "large", "source": "/inline/e1.malloy", "pageSize":5000}
run: flights -> {
  group_by:
    carriers.nickname
    destination
    origin
  aggregate: 
    flight_count
    # percent
    `carrier as a percent of all flights` is all(flight_count, nickname)/all(flight_count)
    # percent
    `destination as a percent of all flights` is all(flight_count, destination)/all(flight_count)
    # percent
    `origin as a percent of all flights` is all(flight_count, origin)/all(flight_count)
    # percent
    `carriers as a percentage of route` is flight_count/exclude(flight_count, nickname)
}
```