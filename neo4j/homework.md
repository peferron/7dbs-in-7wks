# Neo4j homework

## Day 1

### Find, 1.

[Neo4j Documentation](https://neo4j.com/docs/)

### Find, 2.

[GremlinDocs](http://gremlindocs.spmallette.documentup.com/)

### Find, 3.

?

### Do, 1.

```cypher
MATCH (n) RETURN n
```

### Do, 2.

```cypher
MATCH (n) DETACH DELETE n
```

### Do, 3.

Skipped.

## Day 2

### Find, 1.

[Neo4j REST API Documentation](https://neo4j.com/docs/rest-docs/current/)

### Find, 2.

[Jung API Javadoc](http://jung.sourceforge.net/doc/api/index.html)

### Find, 3.

[Using Neo4j from Python](https://neo4j.com/developer/python/)

### Do, 1.

```groovy
def actor(g, name) {
  g.V.filter{it.name == name}.next()
}

Gremlin.defineStep('costars', [Vertex, Pipe], {
  _().sideEffect{start = it}
     .outE('ACTED_IN')
     .inV
     .inE('ACTED_IN')
     .outV
     .filter{!start.equals(it)}
     .dedup
})

Gremlin.defineStep('path', [Vertex, Pipe], { target ->
  _().costars
     .loop(1){it.loops < 4 & !it.object.equals(target)}
     .filter{it.equals(target)}
     .path
})

def compare_distance(actor1, actor2, target) {
  distance = {a, b -> (a.path(b).next().size() - 3) / 2}
  actor1.name + ': ' + distance(actor1, target) + ', ' + actor2.name + ': ' + distance(actor2, target)
}

g = new Neo4jGraph('/Users/peferron/Downloads/neo4j-1.9.7/data/graph.db')
bacon = actor(g, 'Kevin Bacon')
elvis = actor(g, 'Elvis Presley')
roddy = actor(g, 'Roddy McDowall')
compare_distance(elvis, roddy, bacon)
```

### Do, 2 & 3.

Skipped.
