Range queries
=============

I'm only interested in the 1980s
--------------------------------

In the museums dataset we used in our earlier examples, there is a
column `DATE_MADE` that tells us when the object in question was made,
so one of the natural things people might want to do is to only search
for objects made in a particular time period. 

If we look through the other fields in the data set, there are more
that could be useful for range queries: we could extract the longest
dimension from `MEASUREMENTS` to enable people to restrict to only
very large or very small objects, for instance.

Note: For detailed explanations, refer to the section `Range queries <https://getting-started-with-xapian.readthedocs.io/en/latest/howtos/range_queries.html>`_ in `Getting started guide <http://getting-started-with-xapian.readthedocs.io/en/latest/index.html>`_ of Xapian.

Indexing
---------

DATE_MADE column of museums dataset contains records with years, year ranges ("1671-1700"), approximate
years ("c. 1936") and commentary ("patented 1885", or "1642-1649 (original); 1883 (model)"). Additionally, some records have no
information about when the object was made. So these values should be turned into a consistent form before storing that information in the Xapian database.

We can extract and modify DATE_MADE column of data frame as follows;

::

  df <-read.csv("path/to/csv")
  dateCol<-df$DATE_MADE
  dateVec<-as.double(gsub("([0-9]+).*$", "\\1",dateCol))


We can extract and modify MEASUREMENTS column of data frame as follows;

::

  library(stringr)
  m <- df$MEASUREMENTS
  y <- str_extract_all(m ,"\\(?[0-9.]+\\)?")
  mVec<-c()
  for(element in y){
    if(length(element)==0)
      mVec<-append(mVec,NA)
    else{
      ele<-as.double(element)
      mVec<-append(mVec,round(max(ele)))
    }
  }




In order to support range queries, we need to pass the following arguments to the valueSlots parameter.

::

  valueSlots <-list(list(slot=c(0),serialise=TRUE,values=mVec,type=c("double")),
	            list(slot=c(1),serialise=TRUE,values=dateVec,type=c("double"))

Here mVec and dateVec are the modified MEASUREMENTS and DATE_MADE columns of the data frame respectively.
A complete example is shown below,

::

  db <- c("path/to/database")
  df <-read.csv("path/to/csv")
  id <-c(0)
  indexFields<-list(list(index=2,prefix="S"),
		    list(index=8,prefix="XD"))
  stem <- c("en")
  valueSlots <-list(list(slot=c(0),serialise=TRUE,values=mVec,type=c("double")),
	            list(slot=c(1),serialise=TRUE,values=dateVec,type=c("double"))

  xapian_index(dbpath = db,
               dataFrame = df,
               idColumn = id,
               indexFields = indexFields,
               valueSlots = valueSlots,
               stemmer = stem)


Searching
----------

For range queries we need to pass a new argument (VRP) to queryList parameter of xapian_search function. 

::

  list(type="proc.num", value=c(0), check.str="mm", flags=c("RP_SUFFIX"))
  list(type="proc.num", value=c(1), check.str="")


A complete example is shown below;

::

  db <- c("path/to/database")

  preF<-list(list(name="title", prefix="S"),
             list(name="description", prefix="XD"))

  vrp <- list(list(type="proc.num", value=c(0), check.str="mm", flags=c("RP_SUFFIX")),
	      list(type="proc.num", value=c(1), check.str=""))

  query<-list(queryString="..50mm", stemmer="en", prefix.fields=preF, VRP=vrp)

  output <-xapian_search(dbpath=db, queryList = query)

In the above example search results is restricted to everything at most 50mm in its longest dimension. We can also search across years by providing a range like "1980..1989":

::

  query<-list(queryString="1980..1989", stemmer="en", prefix.fields=preF, VRP=vrp) 
  
We can of course combine this with ‘normal’ search terms, such as all clocks made from 1960 onwards:

::

  query<-list(queryString="clock 1960..", stemmer="en", prefix.fields=preF, VRP=vrp) 

and even combining both ranges at once, such as all large objects from the 19th century:

::

  query<-list(queryString="1000..mm 1800..1899", stemmer="en", prefix.fields=preF, VRP=vrp) 


Handling dates
--------------

To show how to restrict search to a date range, we’re going to need to use a different dataset, because the museums data only gives years the objects were made in.

We need a new indexer for this as well. But first of all we need to extract and modify the admitted and population columns of the data set, in order to bring them to the required format.

::
  
  adm<-data$admitted
  
  ch<-as.character(adm)
  vec<-c()
  for(element in ch){
    w<-as.list(strsplit(element, ",")[[1]])
    f<-as.character(w[2])
    l<-as.list(strsplit(f," ")[[1]])
    qq<-as.numeric(l[2])
    vec<-append(vec,qq)
  }


::

  pop<-data$population
  pVec<-c()
  for(element in pop){
    ele<-as.list(strsplit(element, " ")[[1]])
    first<-as.character(ele[1])
    pVec<-append(pVec,first)
  }
  popVec<-as.numeric(gsub(",","",pVec))
  popVec[33]<-c(6346105) # above modifications cause popVec[33] to be NA
  popVec[41]<-c(12702379) #above modifications cause popVec[41] to be NA



To do: Add code to modify the date of admission to bring all records to the format ‘YYYYMMDD’

A complete example is shown below,

::

  db <- c("path/to/database")
  df <-read.csv("path/to/csv")
  id <-c(0)
  indexFields<-list(list(index=2,prefix="S"),
		    list(index=8,prefix="XD"))
  stem <- c("en")
  valueSlots <-list(list(slot=c(1),serialise=TRUE,values=vec,type=c("double")),
                    list(slot=c(2),serialise=FALSE,values=admVec,type=c("character")
	            list(slot=c(3),serialise=TRUE,values=popVec,type=c("double")))

  xapian_index(dbpath = db,
               dataFrame = df,
               idColumn = id,
               indexFields = indexFields,
               valueSlots = valueSlots,
               stemmer = stem)

Here two numbers are stored: year of admission in value slot 1 and population in slot 3. It also stores the date of admission as ‘YYYYMMDD’ in slot 2.

To do:: Add code to search a date range.

