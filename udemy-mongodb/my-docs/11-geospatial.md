# 11:  Geospatial Data in MongoDB
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11851158#overview
- mongodb-slides.pdf: pg 92-95 

**Documentation:**
- Geospatial Indexes: https://www.mongodb.com/docs/manual/core/indexes/index-types/index-geospatial/
  - 2dsphere: https://www.mongodb.com/docs/manual/core/indexes/index-types/geospatial/2dsphere/create/
- Geospatial Queries: https://www.mongodb.com/docs/manual/geospatial-queries/
- GeoJSON Objects: https://www.mongodb.com/docs/manual/reference/geojson/

# Key concepts
## GeoJSON Objects
- To calculate geometry over an Earth-like sphere, store your location data as GeoJSON objects.
- To specify GeoJSON data, use an embedded document with:
  - a field named type that specifies the GeoJSON object type, and
  - a field named coordinates that specifies the object's coordinates.
```js
<field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

**Important**:
- If specifying latitude and longitude coordinates, list the longitude first, and then latitude.
  - Valid longitude values are between -180 and 180, both inclusive.
  - Valid latitude values are between -90 and 90, both inclusive.

For example, to specify a GeoJSON Point:
```js
location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
}
```

---

# Restaurants Turorial Example
- SEE: https://www.mongodb.com/docs/manual/tutorial/geospatial-tutorial/

## Download Datasets
```js
//After downloading the datasets, import them into the database
//NOTE: the format ISN'T jsonArray (no square brackets around the entire file)
mongoimport --db places --collection restaurants --file downloads/geospatial/restaurants.json 
//Results:  25359 documents

mongoimport --db places --collection neighborhoods --file downloads/geospatial/neighborhoods.json
// Results: 195 documents
```

---

## Create Geospatial Indexes
- A geospatial index, and almost always improves performance of `$geoWithin` and `$geoIntersects` queries.
- Because this data is geographical, create a 2dsphere index on each collection using mongosh:

```js
//Connect to the DB
mongosh --port 27017
show dbs
// Switch to places DB
use places

// Create 2dsphere indexes
db.restaurants.createIndex({ location: "2dsphere" })
db.neighborhoods.createIndex({ geometry: "2dsphere" })
```

---

## Exploring the Data

```js
//Inspect an entry in the newly-created restaurants collection in mongosh:
db.restaurants.findOne()
//Result:
{
   location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
   },
   name: "Morris Park Bake Shop"
}


//Now inspect an entry in the neighborhoods collection:
db.neighborhoods.findOne()

//Results: This query will return a document like the following:
{
   geometry: {
      type: "Polygon",
      coordinates: [[
         [ -73.99, 40.75 ],
         ...
         [ -73.98, 40.76 ],
         [ -73.99, 40.75 ]
      ]]
    },
    name: "Hell's Kitchen"
}
```

## Find the Current Neighborhood
- Assuming the user's mobile device can give a reasonably accurate location for the user, it is simple to find the user's current neighborhood with $geoIntersects.

```js
//Suppose the user is located at -73.93414657 longitude and 40.82302903 latitude. 
// To find the current neighborhood, you will specify a point using the special $geometry field in GeoJSON format:
db.neighborhoods.findOne({ geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } })

//Results: This query will return the following result:
{
    "_id" : ObjectId("55cb9c666c522cafdb053a68"),
    "geometry" : {
        "type" : "Polygon",
        "coordinates" : [
            [
                [
                    -73.93383000695911,
                    40.81949109558767
                ],
                ...
            ]
        ]
    },
    "name" : "Central Harlem North-Polo Grounds"
}

// Find all Restaurants in the Neighborhood
// You can also query to find all restaurants contained in a given neighborhood.
//Run the following in mongosh to find the neighborhood containing the user, and then count the restaurants within that neighborhood:

var neighborhood = db.neighborhoods.findOne( { geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } } )
db.restaurants.find( { location: { $geoWithin: { $geometry: neighborhood.geometry } } } ).count()
//Results: This query will tell you that there are 127 restaurants in the requested neighborhood,
```

## Find Restaurants within a Distance
- To find restaurants within a specified distance of a point, you can use either `$geoWithin` with `$centerSphere` to return results in unsorted order, or `$nearSphere` with `$maxDistance` if you need results sorted by distance.

### Unsorted with $geoWithin
- To find restaurants within a circular region, use $geoWithin with `$centerSphere`. `$centerSphere` is a MongoDB-specific syntax to denote a circular region by specifying the center and the radius in radians.
- `$geoWithin` does not return the documents in any specific order, so it may show the user the furthest documents first.
- `$centerSphere`'s second argument accepts the radius in radians, so you must divide it by the radius of the earth in miles. See Convert Distance to Radians for Spherical Operators for more information on converting between distance units: https://www.mongodb.com/docs/manual/core/indexes/index-types/geospatial/2d/calculate-distances/#std-label-calculate-distance-spherical-geometry

```js
//The following will find all restaurants within five miles of the user:
db.restaurants.find({ location:
   { $geoWithin:
      { $centerSphere: [ [ -73.93414657, 40.82302903 ], 5 / 3963.2 ] } } })
```

###  Sorted with $nearSphere
- You may also use $nearSphere and specify a `$maxDistance` term in meters. This will return all restaurants within five miles of the user in sorted order from nearest to farthest:

```js
//Define the number of meters in a mile
var METERS_PER_MILE = 1609.34

//Find all restaurants within five miles of the user, sorted by distance
db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] }, $maxDistance: 5 * METERS_PER_MILE } } })

```



