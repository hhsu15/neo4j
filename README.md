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

### Basic queries
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
