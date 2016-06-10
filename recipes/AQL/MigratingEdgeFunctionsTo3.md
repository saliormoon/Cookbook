# Migrating anonymous graph Functions from 2.8 or earlier to 3.0

## Problem

With the release of 3.0 all GRAPH functions have been dropped from AQL in favor of a more
native integration of graph features into the query language. I have used the old graph
functions and want to upgrade to 3.0.

Graph functions covered in this recipe:

* EDGES
* NEIGHBORS
* PATHS
* TRAVERSAL
* TRAVERSAL_TREE

## Solution

### EDGES

The EDGES can be simply replaced by a call to the AQL traversal.

#### No options

The syntax is slightly different but mapping should be simple:

```
// OLD
[..] FOR e IN EDGES(@@edgeCollection, @startId, 'outbound') RETURN e

// NEW
[..] FOR v, e IN OUTBOUND @startId @@edgeCollection RETURN e
```

#### Using EdgeExamples

Examples have to be transformed into AQL filter statements.
How to do this please read the GRAPH_VERTICES section
in [Migrating GRAPH_* Functions from 2.8 or earlier to 3.0](MigratingGraphFunctionsTo3.html).
Apply these filters on the edge variable `e`.

#### Option incluceVertices

In order to include the vertices you just use the vertex variable v as well:

```
// OLD
[..] FOR e IN EDGES(@@edgeCollection, @startId, 'outbound', null, {includeVertices: true}) RETURN e

// NEW
[..] FOR v, e IN OUTBOUND @startId @@edgeCollection RETURN {edge: e, vertex: v}
```

NOTE: The direction cannot be given as a bindParameter any more it has to be hard-coded in the query.

### NEIGHBORS

The NEIGHBORS is a breadth-first-search on the graph with a global unique check for vertices. So we can replace it by a an AQL traversal with these options.
Due to syntax changes the vertex collection of the start vertex is no longer mandatory to be given.
You may have to adjust bindParameteres for this query.

#### No options

The default options did just return the neighbors `_id` value.

```
// OLD
[..] FOR n IN NEIGHBORS(@@vertexCollection, @@edgeCollection, @startId, 'outbound') RETURN n

// NEW
[..] FOR n IN OUTBOUND @startId @@edgeCollection OPTIONS {bfs: true, uniqueVertices: 'global'} RETURN n._id
```

NOTE: The direction cannot be given as a bindParameter any more it has to be hard-coded in the query.

#### Using edgeExamples

Examples have to be transformed into AQL filter statements.
How to do this please read the GRAPH_VERTICES section
in [Migrating GRAPH_* Functions from 2.8 or earlier to 3.0](MigratingGraphFunctionsTo3.html).
Apply these filters on the edge variable `e` which is the second return variable of the traversal statement.

However this is a bit more complicated as it interferes with the global uniqueness check.
For edgeExamples it is sufficent when any edge pointing to the neighbor matches the filter. Using `{uniqueVertices: 'global'}` first picks any edge randomly. Than it checks against this edge only.
If we know there are no vertex pairs with multiple edges between them we can use the simple variant which is save:

```
// OLD
[..] FOR n IN NEIGHBORS(@@vertexCollection, @@edgeCollection, @startId, 'outbound', {label: 'friend'}) RETURN n

// NEW
[..] FOR n, e IN OUTBOUND @startId @@edgeCollection OPTIONS {bfs: true, uniqueVertices: 'global'}
FILTER e.label == 'friend'
RETURN n._id
```

If there may be multiple edges between the same pair of vertices we have to make the distinct check ourselfes and cannot rely on the traverser doing it correctly for us:

```
// OLD
[..] FOR n IN NEIGHBORS(@@vertexCollection, @@edgeCollection, @startId, 'outbound', {label: 'friend'}) RETURN n

// NEW
[..] FOR n, e IN OUTBOUND @startId @@edgeCollection OPTIONS {bfs: true}
FILTER e.label == 'friend'
RETURN DISTINCT n._id
```

#### Option includeData

If you want to include the data simply return the complete document instead of only the `_id`value.

```
// OLD
[..] FOR n IN NEIGHBORS(@@vertexCollection, @@edgeCollection, @startId, 'outbound', null, {includeData: true}) RETURN n

// NEW
[..] FOR n, e IN OUTBOUND @startId @@edgeCollection OPTIONS {bfs: true, uniqueVertices: 'global'} RETURN n
```

### PATHS

This function computes all paths of the entire edge collection (with a given minDepth and maxDepth) as you can imagine this feature is extremely expensive and should never be used.
However paths can again be replaced by AQL traversal.

#### No options
By default paths of length 0 to 10 are returned. And circles are not followed.

```
// OLD
RETURN PATHS(@@vertexCollection, @@edgeCollection, "outbound")

// NEW
FOR start IN @@vertexCollection
FOR v, e, p IN 0..10 OUTBOUND start @@edgeCollection RETURN {source: start, destination: v, edges: p.edges, vertices: p.vertices}
```

#### followCycles

If this option is set we have to modify the options of the traversal by modifying the `uniqueEdges` property:

```
// OLD
RETURN PATHS(@@vertexCollection, @@edgeCollection, "outbound", {followCycles: true})

// NEW
FOR start IN @@vertexCollection
FOR v, e, p IN 0..10 OUTBOUND start @@edgeCollection OPTIONS {uniqueEdges: 'none'} RETURN {source: start, destination: v, edges: p.edges, vertices: p.vertices}
```

#### minDepth and maxDepth

If this option is set we have to give these parameters directly before the direction.

```
// OLD
RETURN PATHS(@@vertexCollection, @@edgeCollection, "outbound", {minDepth: 2, maxDepth: 5})

// NEW
FOR start IN @@vertexCollection
FOR v, e, p IN 2..5 OUTBOUND start @@edgeCollection
RETURN {source: start, destination: v, edges: p.edges, vertices: p.vertices}
```

### TRAVERSAL and TRAVERSAL_TREE

These have been removed and should be replaced by the
[native AQL traversal](https://docs.arangodb.com/3/Manual/Graphs/Traversals/index.html).
As they are way to complex for a cookbook please read the native traversal documentation.
If you need further help with this please contact us via our social channels.

**Author:** [Michael Hackstein](https://github.com/mchacki)

**Tags**: #howto #aql #migration
