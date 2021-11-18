=======================================================
-- Steps of setting up Projects in Neo4J Desktop App --
=======================================================

1. Create New Project
2. Name the Project "Road Trip"
3. Click Add > Local DBMS, Name "Road Trip Graph DBMS", Password "TAFE2021", Start the DBMS
4. On the DBMS(-Active), Click 3 dots icon on the right, click "Open folder" > "Import", this will open the "import" folder. Duplicate the 2 CSVs in this folder. (use the  "restareas.csv" and "postcodes.csv" files in "modified-csv" folder).
5. Click "Open" to launch Neo4j browser


===============================================
-- Dataset Sources & Codes for Neo4j Browser --
===============================================


// 1st set of Data: Postcode/Suburb within NSW

// Get dataset from https://gist.github.com/randomecho/5020859#file-australian-postcodes-sql
// convert .sql to .csv (I used phpMyAdmin)
// Use excel to modifiy csv data (remove rows of states other than NSW)
// Put "Postcode.csv" file into "import" folder

CREATE CONSTRAINT ON (p:Postcode) ASSERT p.PostcodeID IS UNIQUE;

LOAD CSV WITH HEADERS FROM 'file:///postcodes.csv' AS row 
MERGE (p:Postcode {PostcodeID: row.PostcodeID})
ON CREATE SET p.Postcode = row.Postcode,
p.SuburbName = row.SuburbName,
p.Latitude = toFloat(row.Latitude),
p.Longitude = toFloat(row.Longitude);


// Run the Postcode table (bubbles) (>>> In SQL: Select * FROM Postcode)

MATCH (p:Postcode) RETURN (p);



--

// 2nd set of Data: NSW Rest Areas
// Get dataset in .csv from https://opendata.transport.nsw.gov.au/dataset/nsw-rest-areas (login required)
// Use excel to modifiy csv data (remove rows where Rest Area Names are empty, delete columns that not used)
// Put "Restareas.csv" file into "import" folder

CREATE CONSTRAINT ON (r:Restarea) ASSERT r.RestareaID IS UNIQUE;

LOAD CSV WITH HEADERS FROM 'file:///restareas.csv' AS row 
MERGE (r:Restarea {RestareaID: row.cartodb_id}) 
ON CREATE SET r.RestareaNumber = row.rest_area_number, 
r.RestareaName = row.rest_area_name, 
r.SuburbName = row.suburb, 
r.Latitude = toFloat(row.latitude), 
r.Longitude = toFloat(row.longitude),
r.Lookout = row.lookout, 
r.BBQ = row.bbq, 
r.Playground = row.playground, 
r.PicnicTable = row.picnic_table,
r.RoadNumber = row.road_number;

// Run the Restarea table

MATCH (r:Restarea) RETURN (r);

--

// ** In case error, delete the table, and go back to 1st step  (>>> In SQL: DROP DATABASE RestArea) --

MATCH (p) DETACH DELETE p
MATCH (r) DETACH DELETE r

// Display constraints and indexes
:schema

// Remove constraints and indexes

DROP CONSTRAINT constraint_xxxx
DROP INDEX index_xxxx

--

// Merge 2 tables together 

// Make sure APOC Plugin is installed (May need to relaunch Neo4j if error occurs..)

:param relType => ("IS_IN_SUBURB");
:param properties => null;
MATCH (p:Postcode), (r:Restarea)
WHERE (r.SuburbName = p.SuburbName)
CALL apoc.create.relationship (r, $relType, $properties, p)
YIELD rel
RETURN rel;

// Now when you click the bubbles relationship should be built...

--

// Delete Relationship to start over if needed

MATCH p=()-[r:IS_IN_SUBURB]->() DELETE r;

--


// 3rd set of Data: NSW - State Heritage Inventory
// Import dataset from https://www.hms.heritage.nsw.gov.au/ 
// (Refine Search > Listind Details > Heritage Listings > NPW Act - Aboriginal Place)
// (View Results > Table view > download icon)
// Modify data in excel by Breaking full address to columns (Data > Text to Columns)
// Put "NPW_Act_Aboriginal_Place_modified.csv" into "import" folder

CREATE CONSTRAINT ON (a:AboriginalPlace) ASSERT a.AboriginalplaceID IS UNIQUE;

LOAD CSV WITH HEADERS FROM 'file:///NPW_Act_Aboriginal_Place_modified.csv' AS row 
MERGE (a:AboriginalPlace {AboriginalplaceID: row.PlaceID})
ON CREATE SET a.Postcode = row.Postcode,
a.ItemName = row.ItemName,
a.SuburbName = row.Suburb;

// Run the AboriginalPlace table

MATCH (a:AboriginalPlace) RETURN (a);

// Build the relationship 

:param relType => ("HAS_ABORIGINAL_PLACE");
:param properties => null;
MATCH (a:AboriginalPlace),(p:Postcode)
WHERE (p.SuburbName = a.SuburbName)
CALL apoc.create.relationship (p, $relType, $properties, a)
YIELD rel
RETURN rel;



--

// To Do: Not sure how to make them work yet
//// 1. Add more Node properties to TABLE (e.g. r:Restarea )
//// 2. Merge Dupe Suburb? Trying below codes but maybe better way...?

MATCH (a:AboriginalPlace)
WITH a.SuburbName as name, COLLECT(a) AS as
WHERE size(as) > 1
CALL apoc.refactor.mergeNodes(as) YIELD node
RETURN node;

MATCH (p:Postcode)
WITH p.SuburbName as name, COLLECT(p) AS ps
WHERE size(ps) > 1
CALL apoc.refactor.mergeNodes(ps) YIELD node
RETURN node;

--

// Filter -- this can be used in Broom
// Perspective  >> Add Search Phrase

// HasLookout
MATCH (r:Restarea) 
WHERE r.Lookout = 'TRUE'
RETURN r;

// HasBBQ
MATCH (r:Restarea) 
WHERE r.BBQ = 'TRUE'
RETURN r;

// HasPinicTable
MATCH (r:Restarea) 
WHERE r.PinicTable = 'TRUE'
RETURN r;


// HasPlayground
MATCH (r:Restarea) 
WHERE r.Playground = 'TRUE'
RETURN r;

// Suburb Starts with P in Postcode Table
MATCH (p:Postcode)
WHERE p.SuburbName STARTS WITH 'P'
RETURN p

--

// Below is Karen's testing code, trying to play with Lat Long..
// later to study: https://neo4j.com/videos/building-spatial-search-algorithms-for-neo4j/

MATCH (p:Postcode)
WHERE p.SuburbName = "Warialda"
RETURN p

// RestareaName: Tigers Gap westbound  >> Latitude: -29.56968, Longitude: 150.62
// RestareaName: Nancy Coulton Lookout >> Latitude: -29.58259, Longitude: 150.48888

// 1st Attempt - using Lat Long 
WITH
  point({latitude:toFloat('-29.56968'), longitude:toFloat('150.62')}) AS p1,
  point({latitude:toFloat('-29.58259'), longitude:toFloat('150.48888')}) AS p2
RETURN toInteger(distance(p1, p2)/1000) AS km

// 2nd Attempt - assign new values and search by node

MATCH (a:Restarea) WHERE a.RestareaName = 'Tigers Gap westbound'
MATCH (b:Restarea) WHERE b.RestareaName = 'Nancy Coulton Lookout'
WITH
  point({latitude:a.Latitude, longitude:a.Longitude}) AS p1,
  point({latitude:b.Latitude, longitude:b.Longitude}) AS p2
RETURN toInteger(distance(p1, p2)/1000) AS km


// Trying to build relationship not sure if this is right.. need to check the unit

:param relType => ("WITHIN_10KM_RADIUS");
:param properties => null;
MATCH (p:Postcode)
MATCH (r:Restarea)
WHERE distance( point({latitude: r.Latitude, longitude:r.Longitude}), point({latitude: p.Latitude, longitude:p.Longitude})) < 10000. 
CALL apoc.create.relationship (p, $relType, $properties, r)
YIELD rel
RETURN rel;



// Delete Relationship to start over if needed

MATCH p=()-[r:WITHIN_10KM_RADIUS]->() DELETE r;

