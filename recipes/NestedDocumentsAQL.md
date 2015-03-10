# AQL update syntax for nested documents

##Problem

You use ArangoDB with [our PHP driver]() and want to use an AQL update syntax for nested documents.

##Solution

First of all you need to register an AQL user function named `my::addToSet` via the [ArangoShell](https://docs.arangodb.com/Arangosh/README.html):

```
var aqlfunctions = require("org/arangodb/aql/functions");
var addToSet = function (previous, toAdd) {
  if (previous === undefined ||
      previous === null ||
      ! Array.isArray(previous)) {
    /* initial value for the set */
    return [ toAdd ];
  }

  /* compare each existing element against toAdd */
  var compare = JSON.stringify(toAdd);

  for (var i = 0; i < previous.length; ++i) {
    if (JSON.stringify(previous[i]) === compare) {
      /* toAdd is already in the set */
      return previous;
    }
  }

  /* toAdd is not yet in the set. now add it */
  previous.push(toAdd);
  return previous;
});

aqlfunctions.register("my::addToSet", addToSet);
```

After that the function is callable from AQL via `my::addToSet()`. For this you can for example use the following query:

```
FOR p IN profiles
  FILTER p.doc == @doc && p._key == @key

  LET contacts = my::addToSet(p.detail.cnx.e, @data)
  LET detail = { cnx: { e: contacts } }

  UPDATE p WITH {detail: detail} IN profiles
```

**Author:** [Jan Steemann](https://github.com/jsteemann)

**Tags:** #php #aql
