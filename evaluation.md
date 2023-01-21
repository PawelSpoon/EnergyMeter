# before we start
## influxdb
influxdb seems to be more like mongo: a bunch of documents with timestamp
 means i can have columns in one measurement, that never meet in table view
  time-stamp  |  col1  | col2
  1           |   1.5  |
  2           |        |  2.3
  3           |   2.3  |  4.5

fill(0) for example will not work on such data
one needs to create propper data at first
that is what i do on the transition from tenminwise to hourly

query on all columns from tenminwise pretends that all values have same number of records
a null in select * from basically indicates that there is no value / record for that column
consequently i need to do fill(0) but not in tenminwise query ! i have to do it in hourly
in tenminwise it would not work due to the 'join' of two tables 

# the idea
- collect data, keep it for few months
- create hourly, daywise, month wise analysis, keep that data forever

# the flow
- shelly 3em sends data channelwise every few 100ms
- ahoy dtu sends data as whole every 30 seconds, or less based on connection
- node-red combines shelly 3em into one message into: meter
- mode-red store ahoy messages into: pv
- merge pv and meter measurements into tenminwise
 - tenminwise retention is 10 years
 - values are mean values of 10 min slots
 - tenminwise contains all values for later calc 
 - select * on tenminwise will still have null fields
- create hourly slots
 - values are mean values of 1 hr duration
 - it is now possible to calcuate diff between consumption and production, cause fill() will work on single measurement
- create daily slots
  - values are energy == integral of hourly (power) values
  - integrate hourly slots to get energy
- create monthly slots

measurement meter contains all the data from shelly-3em
pv contains all the data from solar panels
retention two_years is in fact 30 days
retention ten_years is ten years

tenminwise copies data from 2-years.pv & meter into 10-years.tenminwise
calculating mean value of last 10 mins 

# execution
CREATE RETENTION POLICY ten_years ON home DURATION 3660d REPLICATION 1
CREATE RETENTION POLICY two_years ON home DURATION 30d REPLICATION 1


CREATE CONTINUOUS QUERY "tenminwise" ON "home" BEGIN SELECT mean("0_power") AS A , mean("1_power") AS B, mean("2_power") AS C,  mean("0_power") + mean("1_power") + mean("2_power") AS TOTAL, mean("total_P_AC") as PV_TOTAL, max(total_YieldDay) AS PV_YIELD_DAY, max("total_YieldTotal") AS PV_YIELD_TOTAL INTO "ten_years"."tenminwise" FROM "two_years"."meter", "two_years"."pv" GROUP BY time(10m) END

CREATE CONTINUOUS QUERY "hourly" ON "home" BEGIN SELECT mean("A") as A, mean("B") AS B, mean("C") as C, mean("TOTAL") as TOTAL, mean("PV_TOTAL") AS PV_TOTAL, mean("TOTAL") - mean("PV_TOTAL") AS DIFF, max(PV_YIELD_DAY) AS PV_YIELD_DAY, max(PV_YIELD_TOTAL) AS PV_YIELD_TOTAL INTO "ten_years"."hourly" FROM "ten_years"."tenminwise" GROUP BY time(1h) fill(0) END

# daily has already to integrate ! 
CREATE CONTINUOUS QUERY "daily" ON "home" BEGIN SELECT integral("TOTAL")/3600 AS TOTAL, integral("PV_TOTAL")/3600 AS PV_TOTAL, max(PV_YIELD_DAY) AS PV_YIELD_DAY, max(PV_YIELD_TOTAL) AS PV_YIELD_TOTAL INTO "ten_years"."daily" FROM "ten_years"."tenminwise" GROUP BY time(1d) END

# 1M does not work, have to fake it using 730hrs
# 365/12=30.41days => 730hrs with schaltjahr it would be 730,5 which again does not work but..
# but working with hrs will not cut properly on an day shift
# with days it will not split properly into months, nor weeks .. hmm
CREATE CONTINUOUS QUERY "monthly" ON "home" BEGIN SELECT * INTO "ten_years"."monthly" FROM "ten_years"."daily" GROUP BY time(30d) END

