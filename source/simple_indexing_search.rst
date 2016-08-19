Simple indexing and search
==========================

Rxapian enables content of  a data frame to be indexed with xapian_index function. Following example explains how to build a simple search system based on museum catalogue data released under the `Creative Commons Attribution-NonCommercial-ShareAlike <https://creativecommons.org/licenses/by-nc-sa/3.0/>`_ license by the `Science Museum in London, UK <http://www.sciencemuseum.org.uk/>`_.

Note: For detailed explanations, refer to the section `A practical example <http://getting-started-with-xapian.readthedocs.io/en/latest/practical_example/index.html>`_ in `Getting started guide <http://getting-started-with-xapian.readthedocs.io/en/latest/index.html>`_ of Xapian.

Simple Indexing
###############

::

  db <- c("path/to/database")
  df <-read.csv("path/to/csv")
  id <-c(0)
  indexFields<-list(list(index=2,prefix="S"),
		    list(index=8,prefix="XD"))
  stem <- c("en")

  xapian_index(dbpath = db, dataFrame = df, idColumn = id, indexFields = indexFields, stemmer = stem)


Simple search
#############

::

  db <- c("path/to/database")
  query<-list(queryString="watch",
              prefix.fields=list(list(prefix="S",name="title"),
                                 list(prefix="XD",name="description")),
              stemmer="en")

  output <- xapian_search(dbpath=db, queryList=query)


output would be a data frame containing results for the searched query.
