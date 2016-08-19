RXapian functions
=====================

Rxapian provides an R interface to Xapian search engine library via two main functions; xapian_index and xapian_search

xapian_index
############

xapian_index function takes the following format:

**xapian_index(database, dataFrame, idColumn, indexFields, filterFields, valueSlots, stemmer)**

**database:** This parameter requires the path to a Xapian::Database. If a database is not already present in the given path, a new database will be created.

**dataFrame:** This parameter requires a data frame whose content will be indexed. Each row in the data frame will be used to create a Xapian::Document during indexing.

**idColumn:** This parameter requires the index of a column in the database whose row value will be used as a unique identifier to each document(row). [Prefix "Q" will be used as it is the convention used to prefix a unique id as stated here: https://xapian.org/docs/omega/termprefixes.html)

**indexFields:** This parameter requires a list of 'list of fields' that will be indexed using a Xapian::TermGenerator.
Rxapian supports name indexing as well as numeric indexing.

Numeric Indexing:

:: 

  list( index=2 , prefix= "S")
  list( index=8 , prefix= "XD")

Here the first element of the lists specifies the index of the column that requires to be indexed with the prefix “S”.

Name indexing:

::

  list( name= "title" , prefix = "S")
  list( name= "description" , prefix = "XD")

Here the first element of the lists specifies the name of the column that requires to be indexed with the prefix “S”.

**filterFields:** This parameter requires a list of arguments that will be used to facilitate 'filtering documents’ (See section ). Each argument of filterFields list should take one of the following formats.

Case I: If row value of the column ("Material") used for filtering search results contain multiple items separated by a semicolon,colon etc. the 3rd element of the list should specify the character used for separation. 

::
  
  list( name = "Material" , prefix = "XM" , separator = ";")


Case II: Entire row value of the Material column contains a single item

::

  list( name = "Material" , prefix = "XM")

Case III: Identical to case II, but numeric index is used to specify the column instead of the name.

::

  list( index = 6 , prefix = "XM")

**valueSlots:** This parameter requires a list of arguments depending on the type of search requirement. Each argument of valueSlots list should take one of the following formats.

Case I: Here the first element of the list specifies the slot no. Second element specifies the column index. Column name can be specified instead of the index ( list( name= “Material” , slot = 0) )

::

  list( index = 6, slot = 0) 

Case II: In this case, row value would be serialised with Xapian::sortable_serialise()

::
  
  list(index = 6, slot = 0, serialise = TRUE)

Case III: If row values should be modified instead of using the original row values in the data frame, a column vector of modified row values and also its type("double" or "character")should be given as shown below.

::

  list(index = 6, slot = 0, serialise = TRUE, values=colVec, type="double")


**stemmer:** This parameter requires the stemmer that should be applied to the Xapian::TermGenerator. It could be either the name or the two letter ISO639 code.


xapian_search
#############

xapian_search function takes the following format:

**xapian_search(dbpath , queryList , enquireList )**

**database:** This parameter requires the path to a Xapian::Database that should be queried.

**queryList:** This parameter requires a list of arguments depending on the type of search requirement. 

Case I: Build a Query from a user specified query string. 

* queryString: A free-text query.

* prefix.fields: Each element of prefix.fields should take the following format:

::

  list(name="title", prefix="S")

ALternately, index could've been used:

::

  list(index=2, prefix="S")

Here, the first element specifies the index (or name) of the column of the data frame to be prefixed with the prefix "S".

* stemmer: The stemmer that should be applied to the Xapian::TermGenerator. It could be either name the two letter ISO639 code. (example: "english" or "en")

* VRP: An optional argument passed to queryList for range queries. Following elements should be passed to VRP.

	* type: Simplest type of VRP is "proc.str". For queries involving number ranges and date ranges "proc.num" and "proc.date" should be used respectively. 

	* value: The value slot number to query.

* For the VRP of type "proc.num" following optional arguments could be provided,

	* check.str: A string to look for to recognise values as belonging to this numeric range.

	* flags: A list of zero or more of the following flags could be provided:

		* RP_SUFFIX - require str_ as a suffix instead of a prefix.
		* RP_REPEATED - optionally allow str_ on both ends of the range - e.g. $1..$10 or 5m..50m. By default a prefix is only checked for on the start (e.g. date:1/1/1980..31/12/1989), and a suffix only on the end (e.g. 2..12kg).

* For the VRP of type "proc.str" following optional arguments could be provided,

	* check.str: A string to look for to recognise values as belonging to this range (as a prefix by default, or as a suffix if flags Xapian::RP_SUFFIX is specified).

	* flags: A list of zero or more of the following flags could be provided:

		* RP_SUFFIX - require str_ as a suffix instead of a prefix.
		* RP_REPEATED - optionally allow str_ on both ends of the range - e.g. $1..$10 or 5m..50m. By default a prefix is only checked for on the start (e.g. date:1/1/1980..31/12/1989), and a suffix only on the end (e.g. 2..12kg).

* For the VRP of type "proc.date" following optional arguments could be provided,

	* flags: A list of zero or more of the following flags could be provided:

		* RP_SUFFIX - require str_ as a suffix instead of a prefix.
		* RP_REPEATED - optionally allow str_ on both ends of the range - e.g. $1..$10 or 5m..50m. By default a prefix is only checked for on the start (e.g. date:1/1/1980..31/12/1989), and a suffix only on the end (e.g. 2..12kg).
		* RP_DATE_PREFER_MDY - interpret ambiguous dates as month/day/year rather than day/month/year.

	* epoch_year: Year to use as the epoch for dates with 2 digit years (default: 1970, so 1/1/69 is 2069 while 1/1/70 is 1970).


Use of different forms of queryList parameter is elaborated in each of the subsequent sections of this guide.

Case II: Build a Query for a term. 

* tname: A term
* wqf: A counts of terms. (default: 1)
* pos: The term position within the query (default: 0) 

Case III: 

**enquireList:** This is an optional parameter and it takes the following format.

Different forms of arguments that should be provided as inputs to this parameter is elaborated in the Facets and Sorting sections of this guide.

xapian_search returns a data frame containing results for the searched query.
