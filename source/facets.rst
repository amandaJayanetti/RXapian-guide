Facets
======

Xapian provides functionality which allows you to dynamically generate complete lists of values which feature in matching documents. For example, colour, manufacturer, size values are good candidates for faceting. 

Note: For detailed explanations, refer to the section `Facets <http://getting-started-with-xapian.readthedocs.io/en/latest/howtos/facets.html>`_ in `Getting started guide <http://getting-started-with-xapian.readthedocs.io/en/latest/index.html>`_ of Xapian.


Indexing
--------

No additional work is needed to implement faceted searching, except to ensure that the values you wish to use in facets are provided as arguments to valueSlots parameter.

::

  list(list(name="COLLECTION",slot=c(0)),list(name="MAKER",slot=c(1)))

Here we’re using two value slots: 0 contains the collection, and 1 contains the name of whoever made the object. We know from the documentation of the dataset that both from fixed and curated lists, so we don’t have to worry about normalising the values before using them as facets.

A complete example is shown below;

::

  db <- c("path/to/database")
  df <-read.csv("path/to/csv")
  id <-c(0)
  indexFields<-list(list(index=2,prefix="S"),
		    list(index=8,prefix="XD"))
  stem <- c("en")
  valueSlots <- list(list(name="COLLECTION",slot=c(0)),list(name="MAKER",slot=c(1)))

  xapian_index(dbpath = db,
               dataFrame = df,
               idColumn = id,
               indexFields = indexFields,
               valueSlots = valueSlots,
               stemmer = stem)

Searching
---------

To query, Xapian uses the concept of spies to observe slots of matched documents during a search. With RXapian, you only need to pass the slot(s) you want the facets to a numeric vector called return.spy;

::

  list(return.spy=c(0,1),checkatleast=100)

Here, the argument checkatleast allows the user to specify the minimum number of items to check. 

A complete example is shown below;

::
 
 db <- c("path/to/database")
 
 enq<-list(return.spy=c(0,1),checkatleast=100)

 preF<-list(list(name="TITLE", prefix="S"),
            list(name="DESCRIPTION", prefix="XD"))

 query<-list(queryString="clock", stemmer="en", prefix.fields=preF)

 result<-xapian_search(db, enq, query)

result is a list of lists where result$output is data.frame with results for the searched query,
and result$spies is a list of spies for the required slot.

