# Migrating GRAPH_* Functions from 2.8 or earlier to 3.0

## Problem

With the release of 3.0 all GRAPH functions have been dropped from AQL in favor of a more
native integration of graph features into the query language. I have used the old graph
functions and want to upgrade to 3.0.


## Solution 1 Quick and Dirty (not recommended)

### When to use this solution

I am not willing to invest a lot if time into the upgrade process and i am
willing to surrender some performance in favor of less effort.
Some constellations may not work with this solution due to the nature of
user-defined functions.
Especially check for AQL queries that do both modifications
and `GRAPH_*` functions.

### Registering user-defined functions

This step has to be executed once on ArangoDB for every database we are using.

We connect to `arangodb` with `arangosh` to issue the following commands two:

```js
var graphs = require("@arangodb/general-graph");
graphs._registerCompatibilityFunctions();
```

### Modify the application code

These have registered all old `GRAPH_*` functions as user-defined functions again, with the prefix `arangodb::`.

Next we have to go through our application code and replace all calls to `GRAPH_*` by `arangodb::GRAPH_*`.
Now run a testrun of our application and check if it worked.
If it worked we are ready to go.

### Important Information

The user defined functions will call translated subqueries (as described in Solution 2).
The optimizer does not know anything about these subqueries beforehand and cannot optimize the whole plan.
Also there might be read/write constellations that are forbidden in user-defined functions, therefore
a "really" translated query may work while the user-defined function work around may be rejected.

## Solution 2 Translating the queries (recommended)

### When to use this solution

I am willing to invest some time on my queries in order to get
maximum performance, full query optimization and a better
control of my queries. No forcing into the old layout
any more.

### Before you start

If you are using `vertexExamples` which are not only `_id` strings do not skip
the GRAPH_VERTICES section, because it will describe how to translate them to
AQL. All graph functions using a vertexExample are identical to executing a
GRAPH_VERTICES before and using it's result as start point.
Example with NEIGHBORS:

```
FOR res IN GRAPH_NEIGHBORS(@graph, @myExample) RETURN res
```

Is identical to:

```
FOR start GRAPH_VERTICES(@graph, @myExample)
  FOR res IN GRAPH_NEIGHBORS(@graph, start) RETURN res
```

All non GRAPH_VERTICES functions will only explain the transformation for a single input document's `_id`.

### GRAPH_VERTICES

First we have to branch on the example.
There we have three possibilities:

1. The example is an `_id` string.
2. The example is `null` or `{}`.
3. The example is a non empty object or an array.

#### Example is '_id' string

This is the easiest replacement. In this case we simply replace the function with a call to `DOCUMENT`:

```
// OLD
[..] GRAPH_VERTICES(@graphName, @idString) [..] 

// NEW
[..] DOCUMENT(@idString) [..] 
```

NOTE: The `@graphName` is not required anymore, we may have to adjust bindParameters.

The AQL graph features can work with an id directly, no need to call `DOCUMENT` before if we just need this to find a starting point.

#### Example is `null` or the empty object

This case means we use all documents from the graph.
Here we first have to now the vertex collections of the graph.

1. If we only have one collection (say `vertices`) we can replace it with a simple iteration over this collection:

```
// OLD
[..] FOR v IN GRAPH_VERTICES(@graphName, {}) [..] 

// NEW
[..] FOR v IN vertices [..] 
````

NOTE: The `@graphName` is not required anymore, we may have to adjust bindParameters.


2. We have more than one collection. This is the unfortunate case for a general replacement.
So in the general replacement we assume we do not want to exclude any of the collections in
the graph. Than we unfortunately have to form a `UNION`over all these collections.
Say our graph has the vertex collections `vertices1`, `vertices2`, `vertices3` we create a sub-query for
a single collection for each of them and wrap them in a call to `UNION`.

```
// OLD
[..] FOR v IN GRAPH_VERTICES(@graphName, {}) [..] 

// NEW
[..]
FOR v IN UNION( // We start a UNION
  (FOR v IN vertices1 RETURN v), // For each vertex collection
  (FOR v IN vertices2 RETURN v), // we create the same subquery
  (FOR v IN vertices3 RETURN v)
) // Finish with the UNION
[..] 
````

NOTE: If you have any more domain knowledge of your graph apply it at this point to identify which
collections are actually relevant as this `UNION` is a rather expensive operation.

If we use the option `vertexCollectionRestriction` in the original query. The `UNION` has to be formed
by the collections in this restriction instead of ALL collections.

#### Example is a non-empty object

First we follow the instructions for the empty object above.
In this section we will just focus on a single collection `vertices`, the UNION for multiple collections
is again wrapped around a subquery for each of these collections built in the following way.

Now we have to transform the example into an AQL `FILTER` statement.
Therefore we take all top-level attributes of the example and do an equal comparison with their values.
All of these comparisons are joined with an `AND` because the all have to be fulfilled.

Example:

```
// OLD
[..] FOR v IN GRAPH_VERTICES(@graphName, {foo: 'bar', the: {answer: 42}}}) [..] 

// NEW
[..] FOR v IN vertices
  FILTER v.foo == 'bar' // foo: bar
  AND    v.the == {answer: 42} //the: {answer: 42}
[..] 
```

#### Example is an array

The idea transformation is almost identical to a single non-empty object.
For each element in the array we create the filter conditions and than we
`OR`-combine them (mind the brackets):

```
// OLD
[..] FOR v IN GRAPH_VERTICES(@graphName, [{foo: 'bar', the: {answer: 42}}, {foo: 'baz'}])) [..] 

// NEW
[..] FOR v IN vertices
  FILTER (v.foo == 'bar' // foo: bar
    AND   v.the == {answer: 42}) //the: {answer: 42}
  OR (v.foo == 'baz')
[..] 
```

### GRAPH_EDGES

The GRAPH_EDGES can be simply replaced by a call to the AQL traversal.
The option `maxIterations` is removed without replacement.

#### No options

The default options did use a direction `ANY` and returned a distinct result of the edges.
Also it did just return the edges `_id` value.

```
// OLD
[..] FOR e IN GRAPH_EDGES(@graphName, @startId) RETURN e

// NEW
[..] FOR v, e IN ANY @startId GRAPH @graphName RETURN DISTINCT e._id
```

#### Option direction

The direction has to be placed before the start id.
Note here: The direction has to be placed as Word it cannot be handed in via a bindParameter
anymore:

```
// OLD
[..] FOR e IN GRAPH_EDGES(@graphName, @startId, {direction: 'inbound'}) RETURN e

// NEW
[..] FOR v, e IN INBOUND @startId GRAPH @graphName RETURN DISTINCT e._id
```

#### Options minDepth, maxDepth

If we use the options minDepth and maxDepth (both default 1 if not set) we can simply
put them in front of the direction part in the Traversal statement:

```
// OLD
[..] FOR e IN GRAPH_EDGES(@graphName, @startId, {minDepth: 2, maxDepth: 4}) RETURN e

// NEW
[..] FOR v, e IN 2..4 ANY @startId GRAPH @graphName RETURN DISTINCT e._id
```

#### Option includeData

If we use the option includeData we simply return the edge directly instead of only the _id:

```
// OLD
[..] FOR e IN GRAPH_EDGES(@graphName, @startId, {includeData: true}) RETURN e

// NEW
[..] FOR v, e IN ANY @startId GRAPH @graphName RETURN DISTINCT e
```

#### Option edgeCollectionRestriction

In order to use edge Collection restriction we just use the feature that the traverser
can walk over a list of edge collections directly. So the edgeCollectionRestrictions
just form this list:

```
// OLD
[..] FOR e IN GRAPH_EDGES(@graphName, @startId, {edgeCollectionRestriction: [edges1, edges2]}) RETURN e

// NEW
[..] FOR v, e IN ANY @startId edges1, edges2 RETURN DISTINCT e._id
```

Note: The `@graphName` bindParameter is not used anymore and probably has to be removed from the query.

#### Option edgeExamples.

See `GRAPH_VERTICES` on how to transform examples to AQL FILTER. Apply the filter on the edge variable `e`.

### GRAPH_NEIGHBORS
