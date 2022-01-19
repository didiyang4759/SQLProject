- [Overview](#overview)
- [Learning Objectives](#learning-objectives)
- [Tasks](#tasks)


### 1. Overview
In this project builded a database that can be used to analyze bird strikes on aircraft. For an existing data set from the FAA and will build a logical data model, a relational schema, realize the relational schema in MySQL/MariaDB, load data into the database, execute SQL queries, a finally perform some simple analysis of the data.

The graphic below shows some statistics regarding bird strikes and helps frame the data in the data file.
![image](https://github.com/didiyang4759/SQLProject/blob/main/Project1/visualization%20of%20data.png)


### 2. Learning Objectives

install/procure MySQL or MariaDB connect to MySQL/MariaDB from R in an R Notebook build a relational schema in 2NF (but ideally in BCNF) for an existing data set load data from CSV files into a relational database through R execute SQL queries against a MySQL/MariaDB database through R perform simple analytics in R identify and resolve programming errors look up details for R, SQL, and MySQL/MariaDB time-box work

### 3. Tasks
#### This project seperate 8 parts:
①Inspecting the data file; assume that this database will be used for an app that can be used by pilots (of any kind of aircraft) to report wildlife incidents. Create a new database and connect to it from R. Then create the following tables: incidents(iid, date, depPort, arrPort, airline, aircraft, flightPhase, impact). Only store the date, not the time of the incident. Use appropriate data types. Create a lookup table for airline called airlines that has attribute/columns: aid, code, airline. The file only contains the airline name but not the code, so leave that empty. Create a lookup table for airport called airports that has attributes: pid, code, name, city, state, country. Link the incidents, airlines, and airports tables via appropriate foreign keys. Leave any columns empty where you do not have data or cannot define a reasonable default. Harmonize the flight phases to be one of: takeoff, landing, inflight, unknown. For example, for row 14, the flight phase was provided as "Landing Roll" -- change that to "landing" when storing the flightPhase. Consider "Business" to be an airline name. You may either use {sql} code chunks or calls to R functions to execute the SQL statements. Use the same airport for arrival and departure.
```
```{r echo=FALSE}
library(RMySQL)
db_user <- 'cs5200'
db_password <- '12345678'
db_name <- 'SandboxDB' 
db_host <- 'cs5211-dbs.cbw6wxrvp4bq.us-east-2.rds.amazonaws.com'
db_port <- 3306
mydb <-  dbConnect(MySQL(), user = db_user, password = db_password, 
                   dbname = db_name, host = db_host, port = db_port)

```
```{r echo=FALSE}
library(RMySQL)
db_user <- 'cs5200'
db_password <- '12345678'
db_name <- 'SandboxDB' 
db_host <- 'cs5211-dbs.cbw6wxrvp4bq.us-east-2.rds.amazonaws.com'
db_port <- 3306
mydb <-  dbConnect(MySQL(), user = db_user, password = db_password, 
                   dbname = db_name, host = db_host, port = db_port)

```
```
```{sql connection=mydb}
create table airlines(
  aid int not null primary key,
  code varchar(30),
  airline varchar(40)
);
```



```{sql connection=mydb}
create table airports (
  pid varchar(80) not null primary key, 
  code  varchar(30), 
  name varchar(80),
  city  varchar(40),
  state  varchar(40),
  country  varchar(40)
);

```

```{sql connection=mydb}
create table incidents(
  iid int  not null  primary key,
  date date,
  depPort varchar(60),
  arrPort varchar(60),
  airline int, 
  aircraft  varchar(60),
  flightPhase  varchar(30)  check(flightPhase in('takeoff', 'landing', 'inflight', 'unknown')), 
  impact  varchar(30) check(impact in('Caused damage','No damage')), 
  foreign key(depPort) references airports(pid),
  foreign key(arrPort) references airports(pid),
  foreign key(airline) references airlines(aid)
);

```
②Place the Bird Strikes CSV file into the same folder as your R Notebook and the load it into R without a path name. The default path is the local folder that contains the R Notebook when you have the R Notebook in an R Project. Once loaded, populate the tables with the following subset of data. Use the following column mappings:

FlightDate ---> incidents.date 
Aircraft: Make/Model ---> incidents.
aircraft Effect: Indicated Damage ---> incidents.
impact When: Phase of flight ---> incidents.
flightPhase Airport: Name ---> airports.name 
Origin State ---> airports.state 
Aircraft: Airline/Operator ---> airlines.airline

Use default values where the data file does not contain values or leave empty. Records (rows) from the CSV that do not have flight information may be omitted. If there is no airport or airline, then link to a "sentinel" airline or airport, i.e., add an "unknown" airline and airport to the tables rather than leaving the value NULL. Assign synthetic key values to aid, iid, and pid and use them as primary keys.

##### Read Table and Insert Value
```{r warning=F, message=F}
airlines <- data.frame(
  aid=1:length(unique(BirdStrikesData$`Aircraft: Airline/Operator`)),
  code=NA,
  airline=unique(BirdStrikesData$`Aircraft: Airline/Operator`))

```
##### Read Table and Insert Value
```{r}
airlines$code <- as.character(airlines$code)
dbExecute(mydb, sqlAppendTable(mydb, "airlines", airlines, row.names = NULL))
```
##### Read Table and Insert Value
```{r warning=F, message=F}
library(tidyverse)
BirdStrikesData <- left_join(BirdStrikesData, airlines, by=c("Aircraft: Airline/Operator"="airline"))
airports <- BirdStrikesData %>% select(`Airport: Name`, `Origin State`) %>%
  mutate(code=NA, city=NA, country=NA)
colnames(airports) <- c("name", "state", "code", "city", "country")
airports <- airports[!duplicated(airports$name), ]
airports$pid <- 1:nrow(airports)
airports <- airports[, c("pid", "code", "name", "city", "state",  "country")]
dbExecute(mydb, sqlAppendTable(mydb, "airports", airports, row.names = NULL))
```
##### Read Table and Insert Value
```{r}
BirdStrikesData <- left_join(BirdStrikesData, airports, by=c("Airport: Name"="name"))
```
##### Read Table and Insert Value and fix flight phase
```{r}
BirdStrikesData$flightPhase <- rep("unknown", nrow(BirdStrikesData))
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Approach"] <- "inflight"
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Climb"] <- "takeoff"
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Descent"] <- "inflight"
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Landing Roll"] <- "landing"
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Parked"] <- "landing"
BirdStrikesData$flightPhase[BirdStrikesData$`When: Phase of flight`=="Take-off run"] <- "takeoff"
```
##### Read Table and Insert Value Assign synthetic key values to aid, iid, and pid and use them as primary keys.
```{r}
BirdStrikesData$iid <- 1:nrow(BirdStrikesData)
BirdStrikesData$aircraft <- BirdStrikesData$`Aircraft: Make/Model`
BirdStrikesData$impact <- BirdStrikesData$`Effect: Indicated Damage`
BirdStrikesData$depPort <- BirdStrikesData$pid
BirdStrikesData$arrPort <- BirdStrikesData$pid
BirdStrikesData$airline <- BirdStrikesData$aid
incidents <- BirdStrikesData %>% select(iid, date, depPort, arrPort, airline, aircraft,
                                        flightPhase, impact
                                        )
incidents$date <- as.character(incidents$date)
dbExecute(mydb, sqlAppendTable(mydb, "incidents", incidents, row.names = NULL))
```

③ Show that the loading of the data worked by displaying parts of each table (do not show the entire tables).  
Document and explain your decisions. See the Hints below for information on db4free. All data manipulation and importing work must occur in R. You may not modify the original data outside of R -- that would not be reproducible work. It may be helpful to create a subset of the data for development and testing as the full file is quite large and takes time to load.

##### Show that the loading of the data
```{sql connection=mydb}
select date,aircraft,flightPhase,impact from incidents
```
##### Show that the loading of the data
```{sql connection=mydb}
select pid, name,state from airports
```
##### Show that the loading of the data
```{sql connection=mydb}
select aid, airline  from airlines
```
④Create a SQL query against your database to find the number of bird strike incidents for each airline arriving at LaGuardia airport during any phase of landing. You may either use a {sql} code chunk or an R function to execute the query. It must be a single query.

```{sql connection=mydb}
select airlines.airline,count(*)
from airlines inner join incidents on airlines.aid=incidents.airline
inner join airports on incidents.arrPort=airports.pid
where airports.name='LaGuardia NY' and flightPhase='landing'
group by airlines.airline;
```

⑤Create a SQL query against your database to find the airport that had the most bird strike incidents (during any flight phase). Include all commercial airlines, i.e., no business, private, or military flights. You may either use a {sql} code chunk or an R function to execute the query. It must be a single query. Use reasonable rules to recognize business, private, or military flights. If you have some mixed in it is not a problem.
```{sql connection=mydb}
select  tt.name
from
(
 select t.name, count(*) as nums
 from 
 (
  select airports.name,incidents.iid
  from airlines inner join incidents on airlines.aid=incidents.airline
  inner join airports on incidents.depPort=airports.pid
  where airlines.airline not in('Business','PRIVATELY OWNED','MILITARY')  
  union all
  select airports.name,incidents.iid
  from airlines inner join incidents on airlines.aid=incidents.airline
  inner join airports on incidents.arrPort=airports.pid
  where airlines.airline not in('Business','PRIVATELY OWNED','MILITARY')  
 )  t
 group by t.name
    order by  count(*)  desc
    limit 1
) tt;
```
⑥Create a SQL query against your database to find the number of bird strike incidents by year. Include all airlines and all flights. You may either use a {sql} code chunk or an R function to execute the query. It must be a single query.
```{r}
library(sqldf)
library(RSQLite)
fpath = "/Users/diyang/Desktop/CS5200/00/"
dbfile = "BirdStrikesData.csv"
```
```{r}
dbDisconnect(dbcon)
```
#### find the number of bird strike incidents by year
```{r}
sqlCmd = "select year(date) as year,count(*) as num
from incidents
group by year(date);"
rs = dbGetQuery(mydb,sqlCmd)
rs
```

⑦Using the above data, build a line chart that visualizes the number of bird strikes incidents per year from 2005 to 2011. Adorn the graph with appropriate axis labels, titles, legend, data labels, etc.

##### First find the bird strikes incidents per year from 2005 to 2011
```{r}
sqlCmd = "select year(date) as year,count(*) as sumNumberOfBirdStrike
from incidents where year(date) between '2005' and '2011'
group by year(date);"
df = dbGetQuery(mydb,sqlCmd)
df
```

##### Creat line plot for bird strikes incidents per year from 2005 to 2011
```{r}
library(ggplot2)
library(lubridate)
p<-ggplot(df,aes(x=year,y=sumNumberOfBirdStrike)) + geom_line()+geom_point() + ggtitle("SumNumberOfBirdStrike")
p
```
⑧Create a stored procedure in MySQL (note that if you used SQLite, then you cannot complete this step) that removes a bird strike incident from the database. You may decide what you need to pass to the stored procedure to remove a bird strike incident, e.g., departure airport, airlines, or some ID. Show that the deletion worked as expected.
##### Removes a bird strike incident from the database
```{sql connection=mydb}

create procedure proc_1(in airline_name varchar(100))
begin
  delete from incidents
     where airline in (select aid
      from airlines
                        where airline=airline_name);
end ;
call proc_1('AMERICAN AIRLINES'); 


```

