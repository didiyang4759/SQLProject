    
- [Overview](#overview)
- [Part1:Load XML](#part1-load-xml)
- [Create Star/Snowflake Schema](#create-star-snowflake-schema)
- [Explore and Mine Data](#explore-and-mine-data)

### Overview
In Part 1 created a normalized relational OLTP database and populate it with data from an XML document. In Part 2 added to the normalized schema fact tables and turn the normalized schema into a denormalized schema suitable for OLAP. In Part 3 used the OLAP star/snowflake schema to do some (simple) data mining.

### Part 1 . Load XML
1.  Create a normalized relational schema that contains the following entities/tables:  _Articles_,  _Journals_,  _Authors_. Use the XML document to determine the appropriate attributes (fields/columns) for the entities (tables). While there may be other types of publications in the XML, you only need to deal with articles in journals. Create appropriate primary and foreign keys. Where necessary, add surrogate keys. Include an image of an ERD showing your model in your R Notebook. For articles you should minimally store the article title (<ArticleTitle>) and date created (<DateCreated>); for journals store the journal name/title, volume, issue, and publication date. For authors you should store last name, first name, initial, and affiliation.

![image](https://github.com/didiyang4759/SQLProject/blob/main/Project2/DBMS%20ER%20diagram.png)

2.  Realize the relational schema in SQLite (place the CREATE TABLE statements into SQL chunks in your R Notebook). Use the appropriate tag for publication date.
```{sql connection=dbcon}
CREATE TABLE Article (
ArticleId TEXT NOT NULL, 
ArticleTitle TEXT, 
PublicationModel TEXT, 
Language TEXT, 
ElocationID TEXT,
PRIMARY KEY (ArticleId)
)
```


```{sql connection=dbcon}
CREATE TABLE Authorship ( 
ArticleId TEXT NOT NULL, 
AuthorId INTEGER NOT NULL, 
PRIMARY KEY (ArticleId,AuthorId), 
FOREIGN KEY (ArticleId) REFERENCES Article (ArticleId), 
FOREIGN KEY (AuthorId) REFERENCES Author (AuthorId)
)
```


```{sql connection=dbcon}
CREATE TABLE Author ( 
AuthorId INTEGER NOT NULL, 
LastName TEXT, 
ForeName TEXT, 
Initials TEXT, 
ValidYN TEXT, 
Affiliation TEXT, 
PRIMARY KEY (AuthorId) 
)
```


```{sql connection=dbcon}
CREATE TABLE Journal( 
Issue_Id INTEGER NOT NULL, 
ISSN TEXT, 
CitedMedium INTEGER, 
Volume INTEGER, 
Issue INTEGER, 
PubDate date, 
Title TEXT, 
ISOAbbreviation TEXT, 
CONSTRAINT PK_Journal,
PRIMARY KEY (Issue_Id) 
)
```



```{sql connection=dbcon}
CREATE TABLE Journal_Ownership ( 
ArticleId TEXT NOT NULL, 
Issue_Id INTEGER NOT NULL, 
PRIMARY KEY (ArticleId,Issue_Id), 
FOREIGN KEY (ArticleId) REFERENCES Article (ArticleId), 
FOREIGN KEY (Issue_Id) REFERENCES Journal (Issue_Id) 
)
```

```{sql connection=dbcon}
CREATE TABLE Grant ( 
GrantIdNo INTEGER NOT NULL, 
GrantId TEXT NOT NULL, 
Acronym TEXT, 
Agency TEXT, 
Country TEXT, 
PRIMARY KEY (GrantIdNo) 
)
```


```{sql connection=dbcon}
CREATE TABLE Grant_Ownership ( 
ArticleId TEXT NOT NULL, 
GrantIdNo NONE NOT NULL, 
PRIMARY KEY (ArticleId,GrantIdNo), 
FOREIGN KEY (ArticleId) REFERENCES Article (ArticleId), 
FOREIGN KEY (GrantIdNo) REFERENCES Grant (GrantIdNo)
)
```


```{sql connection=dbcon}
CREATE TABLE PubMedHistory ( 
PubMedId INTEGER NOT NULL, 
PubStatus TEXT, 
PubMedDate date, 
PRIMARY KEY (PubMedId) 
)
```

```{sql connection=dbcon}
CREATE TABLE PubMed ( 
ArticleId TEXT NOT NULL, 
PubMedId INTEGER NOT NULL, 
PRIMARY KEY (ArticleId,PubMedId),
FOREIGN KEY (ArticleId) REFERENCES Article (ArticleId),
FOREIGN KEY (PubMedId) REFERENCES PubMedHistory (PubMedId)
)
```


```{r}
library(XML)
library(DBI)
library(knitr)
fn <- "pubmed_sample.xml"
fpn = paste0(path, fn)
xmlDOM <- xmlParse(file = fpn)
root <- xmlRoot(xmlDOM)
```


3. Extract and transform the data from the XML and then load into the appropriate tables in the database. You cannot (directly and solely) use  `xmlToDataFrame`  but instead must parse the XML node by node using a combination of node-by-node tree traversal and  _XPath_. It is not feasible to use  _XPath_  to extract all journals, then all authors, etc. as some are missing and won't match up. You will need to iterate through the top-level nodes. While outside the scope of the course, this task could also be done through XSLT. Do not store duplicate authors or journals. For dates, you need to devise a conversion scheme, document your decision, and convert all dates to your encoding scheme.
```{r}
library(XML)
library(DBI)
library(knitr)
fn <- "pubmed_sample.xml"
fpn = paste0(path, fn)
xmlDOM <- xmlParse(file = fpn)
root <- xmlRoot(xmlDOM)
```


```{r}

journalTitle<-xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['Article']]
                                           [['Journal']][["Title"]]))

volume<-xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['Article']]
                                           [['Journal']][['JournalIssue']][['Volume']]))

issue<-xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['Article']][['Journal']]
                                          [['JournalIssue']][['Issue']]))

pubyear<-xmlSApply(root,function(x)xmlValue(x[["PubmedData"]][["History"]][["PubMedPubDate"]][['Year']]))

pubmonth<-xmlSApply(root,function(x)xmlValue(x[["PubmedData"]][["History"]][["PubMedPubDate"]][['Month']]))


pubday<-xmlSApply(root,function(x)xmlValue(x[["PubmedData"]][["History"]][["PubMedPubDate"]][['Day']]))

journalTitle<-xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['Article']]
                                                    [['Journal']][['Title']]))


df.Journals<-data.frame('journalTitle'= journalTitle,'volume'=volume,'issue'=issue,
                       'pubyear'=pubyear,'pubmonth'=pubmonth,'pubday'=pubday)


df.Journals$articleId<-seq(1:19)

df.Journals<-tibble::rowid_to_column(data.frame(df.Journals), 'journalId')

dbWriteTable(dbcon,'Journals',df.Journals, append=TRUE) 

```


```{sql connection=dbcon}
select * from Journals
```


```{r}
articleTitle<-xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['Article']][['ArticleTitle']]))

articleId <-xmlSApply(root,function(x)xmlValue(x[['articleId']]))

year <- xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['DateCreated']][['Year']]))

month <- xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['DateCreated']][['Month']]))

day<- xmlSApply(root,function(x)xmlValue(x[['MedlineCitation']][['DateCreated']][['Day']]))

df.Article<-data.frame('articleTitle'=articleTitle,'year'=year, 'month'=month,'day'=day)

df.Article<-tibble::rowid_to_column(data.frame(df.Article), 'articleId')

dbWriteTable(dbcon,'Article',df.Article, append=TRUE)
```

```{sql connection=dbcon}
select * from Article
```


```{r}
lastName<-c()
firstName<-c()
initial<-c() 
affiliation<-c()
articleId <-c()

for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][['MedlineCitation']][['Article']][['AuthorList']])))) { lastName<-c(lastName,xmlValue(root[[i]][['MedlineCitation']][['Article']][['AuthorList']][[j]][['LastName']]) ) } }
for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][['MedlineCitation']][['Article']][['AuthorList']])))) { firstName<-c(firstName,xmlValue(root[[i]][['MedlineCitation']][['Article']][['AuthorList']][[j]][["ForeName"]])) } }
for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][['MedlineCitation']][['Article']][['AuthorList']])))) { initial<-c(initials,xmlValue(root[[i]][['MedlineCitation']][['Article']][['AuthorList']][[j]][['Initials']]) ) } }
for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][['MedlineCitation']][['Article']][['AuthorList']])))) { affiliation<-c(affiliation,xmlValue(root[[i]][['MedlineCitation']][['Article']][['AuthorList']][[j]][["Affiliation"]]) ) } }
for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][['MedlineCitation']][['Article']][['AuthorList']])))) { articleId<-c(articleId,i) } }
length(initial) <- length(affiliation)
df.Author<- data.frame('lastName'=lastName,'firstName'=firstName,
                       'initial'=initial,'affiliation'=affiliation)
df.Author<-tibble::rowid_to_column(data.frame(df.Author), "authorId")
colnames(df.Author)
df.Author$articleId<-articleId
dbWriteTable(dbcon,"Author",df.Author, append=TRUE)
```


### Part 2 . Create Star/Snowflake Schema

1.  Create and populate a star schema with dimension and summary fact tables in either SQLite or MySQL. Each row in the fact table will represent one journal fact. It must include (_minimally_) the journal id, number of articles, and the average number of days elapsed between submission (date created in the XML) and date of publication in the journal by by year and by quarter. Add a few additional facts that are useful for future analytics. Populate the star schema via R. When building the schema, look a head to Part 3 as the schema is dependent on the eventual OLAP queries. Note that there is not a single way to create the fact table -- you may use dimension tables or you may collapse the dimensions into the fact table. Remember that the goal of fact tables is to make interactive analytical queries fast through pre-computation and storage -- more storage but better performance. This requires thinking and creativity -- there is not a single best solution.
```{sql connection=dbcon}
CREATE TABLE Journaldim(
journaldimId integer NOT NULL,
journalTitle Text,
year Text,
month integer,
quarter integer,
day integer,
sumArti integer,
PRIMARY KEY (journaldimId)
)
```

```{sql connection=dbcon}
CREATE TABLE Datedim(
DatedimId integer NOT NULL,
articleId int,
minusdate date,
PRIMARY KEY (DatedimId)
)
```

```{sql connection=dbcon}
CREATE TABLE articledim(
articledimId integer NOT NULL,
articleId int,
year int,
month int,
quarter int,
day int,
PRIMARY KEY (articledimId)
)
```

```{sql connection=dbcon}
CREATE TABLE FactTable(
journaldimId int,
year int,
quarter int,
sumArti integer,
avgyear integer,
avgquarter integer,
PRIMARY KEY (journaldimId)
)
```

```{r}
pubjournaldim <- dbGetQuery(dbcon,"select j.journalTitle as journalTitle,
ABS(a.year-j.pubyear)as year, ABS(a.day-j.pubmonth)as month, ABS(a.day-j.pubday)as day,
h.quarter as quarter,h.day as day,count(a.articleId) as sumArti from journals as J, hisdim as h, Article as a where a.articleId = j.articleId and j.articleId = h.articleId
group by j.journalTitle,h.year,h.quarter,h.Month,h.day")
dbWriteTable(dbcon,"Journaldim",pubjournaldim, append=TRUE)
```

```{sql connection=dbcon}
select * from Journaldim
```


```{r}
minusdate<-c()

ExtractDate<-function(x) {

as.Date(paste(x['Year'],x['Month'], x['Day'], sep="/"), format="%Y/%m/%d")
}

for (i in seq(1:length(names(root)))) { for (j in seq(1:length(names(root[[i]][["PubmedData"]][["History"]])))){
minusdate<-c(minusdate,ExtractDate(xmlApply(root[[i]][["PubmedData"]][["History"]][[j]],function(x)xmlValue(x)) ) )
  }
}

minusdate<-as.character(as.Date(minusdate, origin ="1970-01-01"))

length(minusdate) <- length(articleId)
df.Datedim<- data.frame('minusdate'= minusdate,'articleId'=articleId)

dbWriteTable(dbcon,"Datedim",df.Datedim, append=TRUE)
```

```{r}
hisdim<- dbGetQuery(dbcon,"select articleId, strftime('%Y',minusdate) as 'year', strftime('%m',minusdate) as 'month',CASE WHEN cast(strftime('%m', minusdate) as integer) BETWEEN 1 AND 3 THEN 1 WHEN cast(strftime('%m', minusdate) as integer) BETWEEN 4 and 6 THEN 2 WHEN cast(strftime('%m', minusdate) as integer) BETWEEN 7 and 9 THEN 3 ELSE 4 END as quarter,strftime('%d',minusdate) as 'day' from Datedim")

dbWriteTable(dbcon,"hisdim",hisdim, append=TRUE)
```

```{sql connection=dbcon}
select year, quarter from hisdim
```  

```{r}
FactTable <- dbGetQuery(dbcon,"select journaldimId, year, quarter,sumArti,
(quarter/sumArti) as avgquarter, (year/sumArti) as avgyear
from Journaldim
group by journalTitle, quarter")
dbWriteTable(dbcon,"FactTable",FactTable, append=TRUE)
```

### Part 3. Explore and Mine Data

1. Write queries using your data warehouse to populate a fictitious dashboard that would allow an analyst to explore whether the number of publications show a seasonal pattern. Create a line graph that shows the average days elapsed between submission and publication for all journals per quarter. If necessary, adjust your fact table(s) as needed to support your new queries. If you need to update the fact table, document your changes and your reasons why the changes are needed. This requires thinking and creativity -- there is not a single best solution.

```{sql connection=dbcon}
select * from FactTable

```
```{r}
library(ggplot2)
library(lubridate)
p<-ggplot(FactTable,aes(x=quarter,y=avgquarter)) + geom_line()+geom_point() + ggtitle("elapsed per quarter")
p
```
