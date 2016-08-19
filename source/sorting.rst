Sorting
=======

By default, Xapian orders search results by decreasing relevance score. However, it also allows results to be ordered by other criteria, or a combination of other criteria and relevance score.

If two or more results compare equal by the sorting criteria, then their order is decided by their document ids. By default, the document ids sort in ascending order (so a lower document id is “better”), but descending order can be chosen by passing "DESCENDING" to docid_order. If you have no preference, you can tell Xapian to use whatever order is most efficient by passing "DONT_CARE" to docid_order.

::

  enquireList<-list(docid_order="DONT_CARE")

There are three methods which are used to specify how the value is used to sort, depending if/how you want relevance used in the ordering:

* To specify the relevance doesn’t affect the ordering at all:

::
  
  enquireList<-list(sortby="value")

* To specify that relevance is used for ordering any groups of documents for which the value is the same:

::

  enquireList<-list(sortby="value_then_relevance")

* To specify that documents are ordered by relevance, and the value is only used to order groups of documents with identical relevance values.

::

  enquireList<-list(sortby="relevance_then_value")

For indexing we can use the same indexing code that was used in Range queries section:

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

This has three document values: slot 1 has the year of admission to the union, slot 2 the full date (as “YYYYMMDD”), and slot 3 the latest population estimate. So if we want to sort by year of entry to the union and then within that by relevance, we want to pass the following arguments to enquireList parameter:

::

  enquireList<-list(sortby="value_then_relevance",valueNo=1,reverse_sort_order=FALSE)

Set the final parameter TRUE for ascending order or FALSE for descending order.

A complete example is shown below:

::

  db<-c("path/to/db")

  enq<-list(sortby="value_then_relevance",valueNo=1,reverse_sort_order=FALSE)

  preF<-list(list(name="name", prefix="S"),
             list(name="description", prefix="XD"))

  query<-list(queryString="spanish", stemmer="en", prefix.fields=preF)

  result<-xapian_search(db, enq, query)

Sorting by multiple values
--------------------------

For sorting on more than one document value (so the first document value specified determines the order; amongst groups of documents where that’s the same, the second document value determines the order, and so on) we need to pass another argument (keymaker) to the enquireList.

We’ll use the following line to change our sorted search above to order by year of entry to the union and then by decreasing population.

::

 keyMaker <-list(list(slot=1,reverse=FALSE),list(slot=3,reverse=TRUE))

The second parameter reverse, if TRUE, reverses the sort order.

A complete example is shown below:

::

  db<-c("path/to/db")

  keyMaker <-list(list(slot=1,reverse=FALSE),list(slot=3,reverse=TRUE))

  enq<-list(sortby="key_then_relevance",keyMaker=keyMaker,reverse_sort_order=FALSE)

  preF<-list(list(name="title", prefix="S"),
             list(name="description", prefix="XD"))

  query<-list(queryString="State", stemmer="en", prefix.fields=preF)

  result<-xapian_search(db, enq, query)


