---
layout: post
title: "Data Warehousing Notes"
subtitle: "By Kaus"
author: "Kaus"
header-img: "img/in-post/2022-08-15-spark.jpg"
header-mask: 0.4
tags: 
    - data
    - warehouse
    - notes
---


# Notes from Data Warehousing Course

## Data warehousing vs Big Data
Data warehousing is concept vs Big Data is Technology.  
Data warehousing is concept to gather Analytical Opertaions, avoid slowdowns in transaction response time and minimize the increased CPU Costs.


## Why Do we need Data Warehousing

* Consolidation of information resources. (Collecting the multiple sources to single place)
* Improve Query Performance. (When centralized the data Query Operation improved)
* Seperate research and decision suppport functions from the operational systems.
* Foundation for data mining, data visualization, advanced reporting and OLAP tools.

## What is Data Warehouse.
It is Relational Database that is designed for query and analysis rather than for transaction processing.  
Concept of maintaining the historical records and analyzing the data to gain a better understanding of the business and to improve the business.
  
Other Components used in Data Warehousing  

* ETL Solutions : To regularly import the data from transtional sources from different databases.  
* Statistical Analysis
* Reporting
* Data Mining Capabilities
* Client Analysis tools
* Other In-house applications


## Characteristics of Data Warehousing
Bill Inmon : First person to implment concepts of Data Warehousing

* Subject Oriented : Each Database is unique and oriented to single subject. Each have related Data.
* Integrated : Should integrate the Data from different sources.
* Non - Volatile : Shouldnt delete. Should maintain for long time for long time analysis.
* Time Variant : Information should always available all the time and fast with historical data.

## Key Features of Data Warehousing
* Sturcted Data
* Better Response Time
* Available Historical Data
* Options for Ad-Hoc Queries : Easier to write the Queries 

## Business Intelligence
Technology driven process for analyzing data and presenting actionable information to help corperate executives, business managers and executives to make more informed decisions.
Once in desired Format we can make visualizations to make decisions.

* To optimize the decision making process
* Increase the Operational Efficiencies.
* To get the results without Queries as BI Users are not techies.
* They combine broad data analysis tools : AdHoc Analysis, Online analytical support, enterprise support and cloud based software in real time
* Data Warehouse or Data Mart is considered as backwend for business intelligence to run upon.
Ad-Hoc : On the Fly report that can include any form of data from any Row and Column.

## Uses 
* Business Operations reporting (Using softwares like Tabaleu,  
* Forecasting
* Dashboard
* Multidimensional Analysis
* Finding correlation among different factors


## Tools used for BI
* Excel
* Reporting tool
* OLAP tool : Used by advanced user for Multi Dimensional analysis
* Data mining tool

## Types of data Warehouses

### Central/Enterprise Data warehouse
* Only one central database. 
* All the Operational systems will Select/Extract/Transform/Integrate/Maintain from central Data warehouse and it will provide to all end users like Query tools, OLAP, Web browsers, Data  Mining.

### Single Department DataMart / Federated Artitecture
* Multiple Databases on different servers that can be on different locations.
* They are seperated logically to store data of single domain.

### Multi Tiered Data Warehouse
* Distributed approach where ETL operations bring data to single dataware house and then copied in multiple daata
* Can have multiple layers of Divison in warehouse
* You load you data from ETL to central warehouse and then distribute to other data marts

## Components of Data Warehouse Architecture
* Sources : Source the data is coming from
* ETL Tools
* Data Warehouse maintenance and administration tools
* Data modeling tool and interface to external data models
* Warehouse databases
* End user who access and analysis tools
* Local meta Data : Base used for rest of data modeling designing
* Data modeling Tools : Power Desiner when convert Logical data model to physical data model, you define data model as well.


## Staging Area
* Phase in artitecture where you store temp data.
* Can be used for filtering, transform, loading stage before you use other tool to load data to stage.
* Advantage is that all different types of data to single form and simply verify.
* Disadvantage is data gets redundant.

## Advantages of Data Warehouse
* Single point of Data Storage. (All Applications can access from single source)
* Quick and easy access to data. 
* Data Quality and consistency. (Validation is applied, so clean data is available in org)
* Better Decision making. (Clear data helps to make better decision)
* Better data Visualization and analytics.

## Disadvantages of Data Warehouse
* Since data is coming from different sources, it is difficult to place data in single place or RDBMS which cause schema problems.
* Source User looses ownership of data and everyone have access to data.
* Long initial implementation time and high cost.
* Adding new data souces take time and associated high cost.
* Limited Flexibility of use and types of user
* Data is static and dated one placed in Data Warehouse
* Difficult to visualize and can only be done with some external tools.
* Difficult to accomodate changes in data types and ranges, data source schema, indexes and queries.
* Difficult to actively monitor changes in data.

