# How to add XML data to ArangoDB?

## Problem
You want to store XML data files into a database to have the ability to make queries onto them.

**Note**: ArangoDB > 2.6 and the javaDriver => 2.7.2 is needed.

## Solution
Since version 2.7.2 the aragodb-java-driver supports writing (createDocumentRaw(...)), reading (getDocumentRaw(...)) and querying (executeAqlQueryRaw(...)) of raw JSON strings.

With [JsonML](http://www.jsonml.org/) you can convert a XML string into a JSON string and back to XML again.

Convert XML into JSON with JsonML example:
``` java
String string = "<recipe name=\"bread\" prep_time=\"5 mins\" cook_time=\"3 hours\"> "
		+ "<title>Basic bread</title> "
		+ "<ingredient amount=\"8\" unit=\"dL\">Flour</ingredient> "
		+ "<ingredient amount=\"10\" unit=\"grams\">Yeast</ingredient> "
		+ "<ingredient amount=\"4\" unit=\"dL\" state=\"warm\">Water</ingredient> "
		+ "<ingredient amount=\"1\" unit=\"teaspoon\">Salt</ingredient> "
		+ "<instructions> "
		+ "<step>Mix all ingredients together.</step> "
		+ "<step>Knead thoroughly.</step> "
		+ "<step>Cover with a cloth, and leave for one hour in warm room.</step> "
		+ "<step>Knead again.</step> "
		+ "<step>Place in a bread baking tin.</step> "
		+ "<step>Cover with a cloth, and leave for one hour in warm room.</step> "
		+ "<step>Bake in the oven at 180(degrees)C for 30 minutes.</step> "
		+ "</instructions> "
		+ "</recipe> ";

JSONObject jsonObject = JSONML.toJSONObject(string);
```

Saving the converted JSON to ArangoDB example:
``` java
DocumentEntity<String> entity = arangoDriver.createDocumentRaw("testCollection", jsonObject.toString(), true,false);
String documentHandle = entity.getDocumentHandle();
```
Reading the stored JSON as a string and convert it back to XML example :
``` java
String rawJsonString = arangoDriver.getDocumentRaw(documentHandle, null, null);
String xml = JSONML.toString(rawJsonString);
System.out.println("XML value: " + xml);
```

Query raw data example:
``` java
String queryString = "FOR t IN " + COLLECTION_NAME + " FILTER t.cook_time == \"3 hours\" RETURN t";
CursorRawResult cursor = arangoDriver.executeAqlQueryRaw(queryString, null, null);
Iterator<String> iter = cursor.iterator();
while (iter.hasNext()) {
	JSONObject jsonObject = new JSONObject(iter.next());
	String xml = JSONML.toString(jsonObject);
	System.out.println("XML value: " + xml);
}
```

## Other resources
More documentation about the ArangoDB java driver is available:
 - [Arango DB Java in ten minutes](https://www.arangodb.com/tutorial-java/)
 - [java driver at Github](https://github.com/arangodb/arangodb-java-driver)
 - [Raw JSON string example](https://github.com/arangodb/arangodb-java-driver/blob/master/src/test/java/com/arangodb/example/document/RawDocumentExample.java)

**Author**: [a-brandt](https://github.com/a-brandt)

**Tags**: #java #driver