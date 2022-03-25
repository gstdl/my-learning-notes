# Importing CSV Data Into Neo4j

## Overview

When handling data, data comes from multiple sources including RDBMS, APIs, BI Tools, Excel, etc. The data coming from those sources are usually stored in CSV, JSON, XML, or other formats. In this writing, we will try to explore how to import CSV data into Neo4j.

Cypher has a built-in clause to directly import CSV data, `LOAD CSV`. If you are dealing with JSON or XML files, you need to use the APOC Library (which also supports CSV files). The data stored in a CSV file may not be a 1 to 1 mapping of your graph data model, it may not even represent a graph at all. The following are steps required to import CSV data into Neo4j:
1. Understand the data in the source CSV files.
2. Inspect and clean (if necessary) the data in the source data files.
3. Create or understand the graph data model you will be implementing during the import.

## Importing CSVs

By default CSV's delimiter is a comma, but in case it's not, you need to define it by using the `FIELDTERMINATOR` clause in the cypher `LOAD CSV` query. 

When loading a CSV data into Neo4j, the data must be normalized and IDs representing a node must be unique.

![](https://graphacademy.neo4j.com/courses/importing-data/1-preparing/2-understand-data/images/unique-ids.png)

Besides of the aforementioned things, you also need to check whether the CSV file can be read by a program.

A full cypher query to load data from CSV is as follows:
```cypher
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/ratings.csv'
AS row
RETURN count(row)
```

<!-- ![Graph Data Model](https://graphacademy.neo4j.com/courses/importing-data/1-preparing/5-data-model/images/movie-data-model.png) -->

There are 2 methods to import CSV data into Neo4j.
- Neo4j Data Importer
- Cypher Query

### Neo4j Data Importer

#### Requirements for using the Data Importer

- You must use CSV files for import.
- CSV files must reside on your local system so you can load them into the graph app.
- CSV data must be clean.
- IDs must be unique for all nodes you will be creating.
- The CSV file must have headers.
- The Neo4j DBMS must be started.

#### Steps for using the Data Importer

1. Place the CSV file(s) in your local system, ensure they have headers and are clean.
2. Open the Neo4j Importer
   The Neo4j importer can be accessed via https://data-importer.graphapp.io/?acceptTerms=true
3. Load the CSV from your local system to the Neo4j Importer
4. Examining the CSV header names used in the CSV files
   You will examine the first rows of each CSV file to determine:
   - Files to be used to create nodes.
   - Files to be used to create relationships.
   - How IDs are used to uniquely identify data.
5. Add node(s)
   1. Add the node in the UI by clicking the Add Node icon.
   2. Specify a label for the node in the Mapping Details pane.
   3. Select the CSV file to use in the Mapping Details pane.
6. Define mapping details for the node
   1. Specify properties for the node (select Add from File where we select all fields).
   2. If you want a property to use a different name or type, edit the property.
   3. Specify the unique ID property for the node.
7. Create relationships
   1. Add the relationship in the UI by dragging the edge of a node to itself or another node.
   2. Specify a type for the relationship in the Mapping Details pane.
   3. Select the CSV file to use in the Mapping Details pane.
8. Define the mapping details for the relationship
   1. In the Mapping Details pane, specify the from and to unique property IDs to use.
   2. If applicable, add properties for the relationship from the file (optional).
   3. Confirm CSV in the left panel is all set for import.
9. Perform the import
   1. Specify connection URL and credentials.
   2. View the import results.
10. Viewing the imported data in Neo4j Browser

##### Advantages & Disadvantages of using Neo4j Importer

Advantages:
- You don't need to write cypher query
- The graph is visually represented by Neo4j Importer (planning can be made during import)

Disadvanteges:
- Data Importer creates all properties as strings, integers, floats, or booleans
- Can't handle date data type
- May need refactoring using cypher
- Good for data less than 1M rows

### Cypher Query

#### Steps for using Cypher Query To Import CSV Data

1. Plan the graph data model and import procedure & transformations
2. Ensure all constraints exists (are already defined) in the DBMS
3. Import the CSV file(s)
4. Sanity check


#### Advantages & Disadvantages of using Cypher Query To Import CSV Data

Advantages:
- Data is loaded in batches using the `:auto USING PERIODIC COMMIT` clause (can handle large datasets with more than 1m rows)
- Type transformations done instantly

Disadvantages:
- Need prior planning

#### Cypher Query Sample

```cypher
:auto USING PERIODIC COMMIT
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
//process only Movie rows
WITH row WHERE row.Entity = "Movie"
MERGE (m:Movie {movieId: row.movieId})
ON CREATE SET
m.tmdbId = row.tmdbId,
m.imdbId = row.imdbId,
m.imdbRating = toFloat(row.imdbRating),
m.released = row.released,
m.title = row.title,
m.year = toInteger(row.year),
m.poster = row.poster,
m.runtime = toInteger(row.runtime),
m.countries = split(coalesce(row.countries,""), "|"),
m.imdbVotes = toInteger(row.imdbVotes),
m.revenue = toInteger(row.revenue),
m.plot = row.plot,
m.url = row.url,
m.budget = toInteger(row.budget),
m.languages = split(coalesce(row.languages,""), "|")
WITH m,split(coalesce(row.genres,""), "|") AS genres
UNWIND genres AS genre
WITH m, genre
MERGE (g:Genre {name:genre})
MERGE (m)-[:IN_GENRE]->(g)
```

```cypher
:auto USING PERIODIC COMMIT
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/importing/2-movieData.csv'
AS row
WITH row WHERE row.Entity = "Person"
MERGE (p:Person {tmdbId: row.tmdbId})
ON CREATE SET
p.imdbId = row.imdbId,
p.bornIn = row.bornIn,
p.name = row.name,
p.bio = row.bio,
p.poster = row.poster,
p.url = row.url,
p.born = CASE row.born WHEN "" THEN null ELSE date(row.born) END,
p.died = CASE row.died WHEN "" THEN null ELSE date(row.died) END
```

# Certification

[Importing CSV Data Into Neo4j](https://graphacademy.neo4j.com/u/0b1bed14-3f76-40ad-9442-046ec8b1274b/importing-data)



















