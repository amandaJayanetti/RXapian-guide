How to filter search results
============================

In the previous section on Simple indexing and search, we
showed how to index text from the title and description fields with
separate prefixes, allowing searches to be performed across just one of
those fields.  This is a simple type of fielded search, but often some
fields in a document won't contain unconstrained text; for example, they
may contain only a few specific values or identifiers.  We often wish to
use such fields to restrict the results to only those matching a particular
value, rather than treating them as unstructured "free text".

In the museum catalog, the ``MATERIALS`` field is an example of a field
which contains text from a restricted vocabulary.  This text can be thought
of as an identifier, rather than text which needs to be parsed.  In fact,
for many records this field contains several identifiers for materials in
the object, separated by semicolons.

Note: For detailed explanations, refer to the section `How to filter search results <https://getting-started-with-xapian.readthedocs.io/en/latest/howtos/boolean_filters.html>`_ in `Getting started guide <http://getting-started-with-xapian.readthedocs.io/en/latest/index.html>`_ of Xapian.


Indexing
--------

In order to restrict the results returned from a search to only those matching a particular
value, we need to pass following arguments to the filterFields parameter of xapian_index fuction. 

:: 

  list(list(name="MATERIALS",prefix="XM",separator=";"))

Here the first element specifies the name of the column that is used to 'filter' search results, and it is to be indexed with the prefix “XM”.  For many records in the museaum catalog, MATERIALS column contains several identifiers separated by semicolons. So as the third argument we provide a semicolon. (Note: separator is an optional field) We can provide multiple columns as arguments to the filterFields parameter. A complete example is shown below,

::

  db <- c("path/to/database")
  df <-read.csv("path/to/csv")
  id <-c(0)
  indexFields<-list(list(index=2,prefix="S"),
		    list(index=8,prefix="XD"))
  stem <- c("en")
  filterFields<- list(list(name="MATERIALS",prefix="XM",separator=";"))

  xapian_index(dbpath = db,
               dataFrame = df,
               idColumn = id,
               indexFields = indexFields,
               filterFields = filterFields,
               stemmer = stem)

Searching
---------

To build a query which performs this task, we need to use two queries; A main query (query.left) and a filter query (query.right). And combine these two queries using the OP_FILTER operator of Xapian.

::

 query<-list(OP="OP_FILTER",
            query.left=list(queryString="clock",stemmer="en",prefix.fields=preF),
            query.right=list(OP="OP_OR",queries=list(list(tname="XMsteel (metal)"))))


query.left follows the simple query structure that was explained in Simple search section.
query.right tells xapian_search to combine multiple queries with OP_OR operator of Xapian. If we need to pass multiple items for 'filtering' search results, we could add each item as follows;

::

 queries=list(list(tname="XMsteel (metal)"),list(tname="XMwood"), list(tname="XMglass"))


Note: Every item starts with the prefix('XM') that was used at indexing time.

A complete example is shown below;

::
 
 db <- c("path/to/database")

 preF<-list(list(name="title", prefix="S"),
           list(name="description", prefix="XD"))

 query<-list(OP="OP_FILTER",
             query.left=list(queryString="clock",stemmer="en",prefix.fields=preF),
             query.right=list(OP="OP_OR",queries=list(list(tname="XMsteel (metal)"))))

 output <-xapian_search(dbpath=db, queryList = query)



output would be a dataframe with results for the searched query filtered to include only those containing 'steel (metal)' in the MATERIALS column.
