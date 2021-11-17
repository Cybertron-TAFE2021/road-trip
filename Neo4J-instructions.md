
-- Neo4J Desktop App --
1. Create New Project
2. Name the Project "Road Trip"
3. Click Add > Local DBMS, Name "Road Trip Graph DBMS", Password "TAFE2021", Start the DBMS
4. On the DBMS(-Active), Click 3 dots icon on the right, click "Open folder" > "Import", this will open the "import" folder. Duplicate the 2 CSVs in this folder. (use the  "restareas.csv" and "postcodes.csv" files in "modified-csv" folder).
5. Click "Open" to launch Neo4j browser

-- Code --

// 1st code: Import Postcode.csv

CREATE CONSTRAINT ON (p:Postcode) ASSERT p.PostcodeID IS UNIQUE;

LOAD CSV WITH HEADERS FROM 'file:///postcodes.csv' AS row 
MERGE (p:Postcode {PostcodeID: row.PostcodeID})
ON CREATE SET p.Postcode = row.Postcode,
p.SuburbName = row.SuburbName,
p.Latitude = toFloat(row.Latitude),
p.Longitude = toFloat(row.Longitude);

--

// 2nd code: Run the Postcode table (bubbles) (>>> In SQL: Select * FROM Postcode)

MATCH (p:Postcode) RETURN (p);

--

// 3rd code: Import Restareas.csv
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

--

// 4th code: Run the Restarea table

MATCH (r:Restarea) RETURN (r);

--

// ** In case error, delete the table, and go back to 1st step  (>>> In SQL: DROP DATABASE RestArea) --

MATCH (p) DETACH DELETE p
MATCH (r) DETACH DELETE r

--

// 5th code: Merge 2 tables together 
// 1. Make sure APOC Plugin is installed. 
// 2. May need to relaunch Neo4j if error occurs..

:param relType => ("IS_IN_SUBURB");
:param properties => null;
MATCH (p:Postcode), (r:Restarea)
WHERE (r.SuburbName = p.SuburbName)
CALL apoc.create.relationship (r, $relType, $properties, p)
YIELD rel
RETURN rel;

--

// Now when you click the bubbles relationship should be built...

--

// Delete Relationship (DON'T DO)

MATCH p=()-[r:IS_IN_SUBURB]->() DELETE r;

--

// 2nd Attempt to Build another relationship 

:param relType => ("WITHIN_10KM_HAS");
:param properties => null;
MATCH (r:Restarea),(p:Postcode)
WHERE distance(point({longitude: r.Longitude, latitude: r.Latitude}),point({longitude: p.Longitude, latitude: p.Latitude})) < 10 * 1000

CALL apoc.create.relationship (r, $relType, $properties, p)
YIELD rel
RETURN rel;


// Add more Node properties to r:Restarea 

// Not sure how to make this work yet

--


// Filter -- this can be used in Broom
// Perspective  >> Add Search Phrase

MATCH (r:Restarea) 
WHERE r.Lookout = 'TRUE'
RETURN r;

