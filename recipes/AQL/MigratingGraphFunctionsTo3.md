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
````

NOTE: The `@graphName` is not required anymore, we may have to adjust bindParameters.

The AQL graph features can work with an id directly, no need to call `DOCUMENT` before if we just need this to find a starting point.

#### Example is `null` or the empty object

### GRAPH_EDGES
