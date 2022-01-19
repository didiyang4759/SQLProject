### 1. In this project builded a database that can be used to analyze bird strikes on aircraft. For an existing data set from the FAA [1], 
and will build a logical data model, a relational schema, realize the relational schema in MySQL/MariaDB, load data into the database,
execute SQL queries, a finally perform some simple analysis of the data.

### 2. In this Project will learn to:
install/procure MySQL or MariaDB
connect to MySQL/MariaDB from R in an R Notebook
build a relational schema in 2NF (but ideally in BCNF) for an existing data set
load data from CSV files into a relational database through R
execute SQL queries against a MySQL/MariaDB database through R
perform simple analytics in R
identify and resolve programming errors
look up details for R, SQL, and MySQL/MariaDB
time-box work

### 3.This project seperate 8 parts:

①Inspecting the data file; assume that this database will be used for an app that can be used by pilots (of any kind of aircraft) to report wildlife incidents. 
Create a new database and connect to it from R. Then create the following tables: incidents(iid, date, depPort, arrPort, airline, aircraft, flightPhase, impact). 
Only store the date, not the time of the incident. Use appropriate data types. Create a lookup table for airline called airlines that has attribute/columns: aid, code, airline. 
The file only contains the airline name but not the code, so leave that empty. Create a lookup table for airport called airports that has attributes: pid, code, name, city, state, country. 
Link the incidents, airlines, and airports tables via appropriate foreign keys. Leave any columns empty where you do not have data or cannot define a reasonable default. 
Harmonize the flight phases to be one of: takeoff, landing, inflight, unknown. For example, for row 14, the flight phase was provided as "Landing Roll" -- change that to "landing" when storing the flightPhase. 
Consider "Business" to be an airline name. You may either use {sql} code chunks or calls to R functions to execute the SQL statements. Use the same airport for arrival and departure.

②Place the Bird Strikes CSV file into the same folder as your R Notebook and the load it into R without a path name. 
The default path is the local folder that contains the R Notebook when you have the R Notebook in an R Project. 
Once loaded, populate the tables with the following subset of data. Use the following column mappings:

FlightDate ---> incidents.date
Aircraft: Make/Model ---> incidents.aircraft
Effect: Indicated Damage ---> incidents.impact
When: Phase of flight ---> incidents.flightPhase
Airport: Name ---> airports.name
Origin State ---> airports.state
Aircraft: Airline/Operator ---> airlines.airline

Use default values where the data file does not contain values or leave empty. 
Records (rows) from the CSV that do not have flight information may be omitted. 
If there is no airport or airline, then link to a "sentinel" airline or airport, i.e., 
add an "unknown" airline and airport to the tables rather than leaving the value NULL.
Assign synthetic key values to aid, iid, and pid and use them as primary keys.

③ Show that the loading of the data worked by displaying parts of each table (do not show the entire tables).  
Document and explain your decisions. See the Hints below for information on db4free. 
All data manipulation and importing work must occur in R. You may not modify the original data outside of R -- that would not be reproducible work. 
It may be helpful to create a subset of the data for development and testing as the full file is quite large and takes time to load.

④Create a SQL query against your database to find the number of bird strike incidents for each airline arriving at LaGuardia airport during any phase of landing.
You may either use a {sql} code chunk or an R function to execute the query. It must be a single query.

⑤Create a SQL query against your database to find the airport that had the most bird strike incidents (during any flight phase). 
Include all commercial airlines, i.e., no business, private, or military flights. You may either use a {sql} code chunk or an R function to execute the query.
It must be a single query.  Use reasonable rules to recognize business, private, or military flights. If you have some mixed in it is not a problem.

⑥Create a SQL query against your database to find the number of bird strike incidents by year. Include all airlines and all flights. 
You may either use a {sql} code chunk or an R function to execute the query. It must be a single query.

⑦Using the above data, build a line chart that visualizes the number of bird strikes incidents per year from 2005 to 2011. 
Adorn the graph with appropriate axis labels, titles, legend, data labels, etc.

⑧Create a stored procedure in MySQL (note that if you used SQLite, then you cannot complete this step) that removes a bird strike incident from the database. 
You may decide what you need to pass to the stored procedure to remove a bird strike incident, 
e.g., departure airport, airlines, or some ID. Show that the deletion worked as expected.
