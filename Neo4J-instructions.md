
-- Neo4J Desktop App --
1. Create New Project
2. Name the Project "Road Trip"
3. Click Add > Local DBMS, Name "Road Trip Graph DBMS", Password "TAFE2021"
4. Click Add > File, Select file "nsw_rest_areas.csv" (Download from github or from NSW Open Dataset)
5. Run Database


-- Test Data and Display as Table Example --

LOAD CSV WITH HEADERS FROM 'file:///nsw_rest_areas.csv' AS row
WITH row WHERE row.bbq = 'true'
WITH row.rest_area_number AS RestAreaNumber, row.rest_area_name AS RestAreaName, row.litter_bins AS LitterBins, row.toilets AS NoOfToilets
RETURN RestAreaNumber, RestAreaName, LitterBins, NoOfToilets
LIMIT 3;


-- Code --

CREATE CONSTRAINT ON (r:RestArea) ASSERT r.CartoDBID IS UNIQUE;

LOAD CSV WITH HEADERS FROM 'file:///nsw_rest_areas.csv' AS row 
MERGE (r:RestArea {CartoDBID: row.cartodb_id})
ON CREATE SET 
    r.RestAreaNumber = row.rest_area_number,
	r.RestAreaName = row.rest_area_name,
    r.Suburb = row.suburb,
	r.Lat = row.latitude,
    r.Long = row.longitude,
    r.VehicleType = row.vehicle_type,
    r.NoOfToilets = row.toilets,
    r.HasShelter = row.shelter,
    r.HasPicnicTable = row.picnic_table,
    r.HasPlayground = row.playground,
    r.HasBBQ = row.bbq,
    r.HasLookout = row.lookout;  



-- Show (SELECT * FROM (TABLE) RestArea )--

MATCH (r:RestArea)
RETURN (r);



-- Delete (DROP DATABASE RestArea) --

MATCH (r)
DETACH DELETE r
