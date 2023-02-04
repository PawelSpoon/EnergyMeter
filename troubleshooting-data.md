# How to fix missing or wrong dataset
honestly influx kinda sucks, mainly due to missing online docu and tools and imho bugs in influxqml

# data provider crashes, data is missing, daily values are wrong
i do not bother that much about actual values, means stuff in tenminwise and hourly data on a long term
daily data should fit, to have something to compare across the years

## ahoy did crash and stopped to send data
daily yield and total yields are wrong

### final working way
connect to influx via influx cli
use db.retentionpolicy e.g. home.ten_years

this is important cause the stupid insert statement does not allow you to define retention !

select * from daily

delete from daily where time={timestamp of day you want to delete}

### that statement inserts but all comma separated values are treated as tags, only pv_yied_total as field
insert daily,TOTAL=5750,PV_TOTAL=1660PV_YIELD_DAY=1660 PV_YIELD_TOTAL=39.44 1675382400000000000

influx differs between fields and tags
comma separated are tags, fields with spaces
so one would think just remove spaces and done, well not, does not work

what does work is this: storing it one by one as field

insert daily TOTAL=5750 1675382400000000000
insert daily PV_TOTAL=... 16..

now how to calculate the timestamp
i store my data at 1 am: this is the timestamp for 3.2.2023: 1675382400000000000
to add a day add: 24*60*60  * 1000 000 000 (the billion is needed as it is stored in nano seconds or something)

