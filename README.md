# Neo4j

## Basics
  - Node
    - label (like person)
  
  - Relationship
    - types (like owns)

  - Properties
    - keys and values (like name: "Geroge") 

Properties are attached to Node or Relationship

Neo4j sandbox: https://sandbox.neo4j.com/

### Basic Cypher queries
Query to see schema
```
CALL db.schema()

```

Query to get a node
```
MATCH (m:Movie)  # get movie node, asign it to variable m 
RETURN m
LIMIT 1
```

Reruen just certain property
```
MATCH (m:Movie)
RETURN m.title
LIMIT 1

```

Specify one Node to another Node with specific relationship and property
```
MATCH (m:Movie)-[:IN_GENRE]->(g:Genre)  # movie node to genre with relationship in_genre
WHERE g.name = 'Comedy' 
RETURN m.title
LIMIT 10

```

```
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.title = 'What We Do in the Shadows' 
RETURN a.name

```

This is where things becoming interesting. Get actor and movie and rated (knowing the relationships)
```
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)<-[r:RATED]-(u:User)
WHERE a.name = 'Taika Waititi' 
RETURN a, m, r, u  # return notes Actor and Movie and User with the relationships for rated

```
Perform calculation - get average rating
```
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)<-[r:RATED]-(u:User)
WHERE a.name = 'Taika Waititi' 
RETURN m.title, sum(r.rating)/count(r) as rating

```

## Installation

Install from https://neo4j.com/download-neo4j-now/
You can start your neo4j instance and access it via: localhost:7474
Go to 

You can also use the ne04j browser app.


## Get familir with the neo4j browser
- you can save your favorite query
- you can change the database, "system" will let you do something like:
  - CREATE DATABASE myNewDatabase
  - SHOW ALL DATABASE
  - but you can't run query under system database


## Cypher query 

### Match

```
MATCH (anything)  # without specifying the node, you get back all nodes
RETURN anything LIMIT 5

```
#### Relationships

- "--" represents relationship. However, this will NOT return you the relationship
```
# Limit is based on matching of pattern, not number of nodes
MATCH(node1)--(node2) RETURN node1, node2 LIMIT 5  # without specifying relationships, you get back all relationships
```

- Use "-[rel]-" to return the relationship 
```
MATCH (node1)-[rel]-(node2)
RETURN node1, rel, node2 
LIMIT 1
```
- Use "->" to specify the direction of relationship, and specify the nodes and relationship type. Use "|" for or statement
```
# get person acted in movie pattern
MATCH (p:Person)-[rel:ACTED_IN | DIRECTED]->(m:Movie)
RETURN p, rel, m
LIMIT 1

```
#### Optional Match
Use double "MATCH", the key here is we can use the same variable.

This is saying "match movie nodes, and then match person node that has relationship type as directed to movie, and then match the directors who acted in the movies. So we are narrowing down the results
```
# get person acted in movie pattern
MATCH (m:Movie)
MATCH(d:Person)-[rel:DIRECTED]->(m)
MATCH(d)-[:ACTED_IN]->(m)
RETURN m.title, d.name
```

However, use the "Optional MATCH", you will be getting all movies, if the the director does not act in that movie you get null for the "d.name".
```
MATCH (m:Movie)  // match all the movies
OPTIONAL MATCH (d:Person)-[:DIRECTED]->(m)<-[:ACTED_IN]-(d) //becomes optional 
RETURN m.title, d.name
```

#### Exercise
Get a person that has contant with another person that has contact with another person
```
MATCH (p1:Person)-[:HAS_CONTACT]->(p2)-[:HAS_CONTACT]->(p3)
RETURN p1, p2, p3
LIMIT 1
```

List a contact's name aling with a movie they directed if they directed one
```
MATCH (p:Person)-[:HAS_CONTACT]->(p2)
Optional MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, p2.name, m.title
```

## Filtering, Transforming
Use {} to specify property
```
MATCH (tom:Person{name:'Tom Hanks', born:1956})
RETURN tom
LIMIT 1

```

Use "where" clause to achive the same
```
MATCH (tom:Person)
WHERE tom.name = 'Tom Hanks' AND tom.born = 1956
RETURN tom
LIMIT 1
```

comparison operators:
- >, <, <>, >=, <=, 

boolean operators
- AND, OR, IN, NOT

```
MATCH (p:Person)
WHERE p.born in [1956, 1970]
RETURN tom
LIMIT 1

```

### Path
Let's say we want to get all ppl involved in the movie "unforgiven" but not the person who directed it
```
MATCH (p:Person)-->(m:Movie)
WHERE m.title = 'Unforgiven' AND NOT (person)-[:DIRECTED]->(m)
RETURN p

```

### Regex match
```
MATCH (m:Movie)
WHERE m.title =~ 'The .*'
RETURN m.title

```

To make it case insensitive
```
MATCH (m:Movie)
WHERE m.title =~ '(?i).*The .*'
RETURN m.title

```

### Order by
```
MATCH (a:Person)-[role:ACTED_IN]->(m:Movie)
WHERE m.title = 'Top Gun'
RETURN a.name, role.earnings
ORDER by role.earnings DESC

```

You can also do "skip"
```
MATCH (a:Person)-[role:ACTED_IN]->(m:Movie)
WHERE m.title = 'Top Gun'
RETURN a.name AS actor_name, role.earnings as earned
ORDER by role.earnings DESC
SKIP 1 // skip top 1

```

#### Exercise
Get Tom Hanks's every contact that has earned more than 10000000 from a movie
```
MATCH (p:Person)-[:HAS_CONTACT]->(p2)-[role:ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks' AND p2.born >= 1960 AND role.earnings > 10000000
RETURN p2.name AS ContactName, p2.born AS Born, role.earnings
ORDER BY role.earnings
```


## Distinct, Aggregation
Distinct results
```
MATCH (p:Person)-[role:ACTED_IN]->(:Movie)
WHERE role.earnings > 10000000
RETURN DISTINCT p.name

```

### Aggregatiom functions
Count
```
MATCH(tom:Person{name:'Tom Hanks'})-[role:ACTED_IN]->(m:Movie) 
RETURN COUNT(m) AS MovieCount
```
Sum earnings
```
MATCH(tom:Person{name:'Tom Hanks'})-[role:ACTED_IN]->(m:Movie) 
RETURN SUM(role.earnings) AS TotalMovieEarnings
```

### String functions

toString()
```
RETURN true as Boolean, toString(true) as String
```
Others string functions:
- toUpper()
- toLower()
- trim()  # trim spaces
- replace(orig_str, search_str, replac_with)


## Create Node
Create a node by using "CREATE"
```
CREATE (:Cat)  // create a Cat Node
 
```
Create a Node with label of Cat and Animal
```
CREATE (cat:Cat:Animal)
RETURN cat

```
Create a node with property
```
CREATE (cat:Cat:Animal{sound: 'Meow', eats: 'birds'})
RETURN cat

```

### Create relationship between Nodes
Create relation to the cat Node itself
```
CREATE (cat:Cat{name:'fluffy'})-[:GROOMS]->(cat)

```
Asign properties to the relationship
```
CREATE (cat:Cat{name:'fluffy'})-[:GROOMS{period: 'daily'}]->(cat)
```
### Add relationship to existing Nodes
First create these two nodes
```
CREATE (joe:Bunny:Animal{name:'Joe Bunny'}),
(sarah: Bunny:Animal{name:'Sarah Bunny'})
RETURN joe, sarah
```
Now we want to add relationships, but here intead of using Create we should use "Merge". Merge will create the relationship only once you don't end up creating duplicate relationships
```
MATCH (joe:Bunny{name:'Joe Bunny'}), (sarah: Bunny{name:'Sarah Bunny'})
MERGE (joe)-[:LIKES]->(sarah)
MERGE (sarah)-[:LIKES]->(joe)
RETURN joe, sarah
```

"Merge" can also create Node

```
MATCH (joe:Bunny{name:'Joe Bunny'}), (sarah: Bunny{name:'Sarah Bunny'})
MERGE (joe)-[:LIKES]->(sarah)
MERGE (sarah)-[:LIKES]->(joe)
MERGE (foxy:Bunny{name:'foxy'})-[:LIKES]->(sarah)
RETURN joe, sarah, foxy
```

## Delete
start with match, you have to also delte the relationships. Below will delete every node that has relationships
```
MATCH (node)-[rel]-()
DELETE node, rel

```
However, if a node does not have relationship then you have to delete it separetely
```
MATCH (node)
DELETE node
```

You can use DETACH to achieve this in one shot
```
MATCH (node)
DETACH DELETE node
```

## Update
Update existing nodes using "SET" clause

```


```


