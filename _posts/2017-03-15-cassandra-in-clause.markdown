---
layout: post
title:  "Using IN clause in cassandra."
date:   2017-03-15 20:52:29 -0700
categories: jekyll update
---

# Cassandra IN clause
Cassandra supports IN clause in addition to =, in the where clause.
Here’s where the IN operator is supported:
- The last column in the partition key, assuming the = operator is used on the first N-1 columns of the partition key
- The last clustering column, assuming the = operator is used on the first N-1 clustering columns and all partition keys are restricted
- The last clustering column, assuming the = operator is used on the first N-1 clustering columns and ALLOW FILTERING is specified

> CREATE TABLE example.pizza_orders_bycustomer (
 	customerid text,
 	orderdate timestamp,
 	zoneid text,
 	PRIMARY KEY (customerid,orderdate,zoneid)
 )WITH CLUSTERING ORDER BY (orderdate DESC) ;


> select * from example.pizza_orders_bycustomer WHERE customerid IN ('100','101');

Or use the clustering column with ALLOW FILTERING.(Not encouraged obviously)
>select * from example.pizza_orders_bycustomer WHERE orderdate in(1490324803488) ALLOW FILTERING;

Best used with = operator on the partition key, for e.g

> select * from example.pizza_orders_bycustomer WHERE customerid ='100' and orderdate in (1490324803488,1490324826882);

This will give an error because orderdate is not restricted

>select * from example.pizza_orders_bycustomer WHERE customerid ='100' and zoneid in ('1','2') ;


