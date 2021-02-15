In this tutorial, we will a setup a NGSI-LD context broker.
The NGSI-LDified OSLO data snippet of the previous [tutorial](https://github.com/brechtvdv/tutorials.NGSI-LDF/blob/master/tutorials.Data-Snippet.md#ngsi-ldify) will be ingested to showcase some of the API calls that are possible with NGSI-LD.
Also, we will demonstrate how the query parameter `options=keyValues` is not sufficient to output OSLO-compliant data.

# Install NGSI-LD

Make sure you have Docker installed, otherwise follow these instructions: https://github.com/FIWARE/tutorials.Linked-Data#docker

There are two ways to install NGSI-LD context broker:
## 1) From Fiware tutorials

```
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
./services scorpio
```

If you want to clean up and start again, run following command:
```
./services stop
```
## 2) From Scorpio

Or run following commands:

```
curl https://raw.githubusercontent.com/ScorpioBroker/ScorpioBroker/development/docker-compose-aaio.yml > docker-compose-aaio.yml
docker-compose -f docker-compose-aaio.yml up -d
```

By default the broker runs on port 9090.
Visit the info URL of the broker (`http://localhost:9090/scorpio/v1/info`) to see if you receive the error page.

# Create entity

We described "De Krook", the public library of the city of Ghent, in our previous [tutorial](https://github.com/brechtvdv/tutorials.NGSI-LDF/blob/master/tutorials.Data-Snippet.md#ngsi-ldify). Note that NGSI-LDified this to make it retrievable through the temporal query API of NGSI-LD.
We will add this building (or entity) to the NGSI-LD endpoint using the cURL command below.
Run this in your command line interface: 

```
curl --location --request POST 'localhost:9090/ngsi-ld/v1/entities' \
--header 'Content-Type: application/ld+json' \
--data-raw '{
  "@context": ["http://data.vlaanderen.be/context/gebouwenregister.jsonld", 
    "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld",
    "https://geojson.org/geojson-ld/geojson-context.jsonld", {
    "2DGebouwgeometrie": "https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie",
    "Geometry": "http://www.w3.org/ns/locn#Geometry"
  }],
  "@id": "http://www.wikidata.org/entity/Q28962266",
  "@type": "https://data.vlaanderen.be/ns/gebouw#Gebouw",
  "Gebouw.geometrie": {
    "@type": "Relationship",
    "object": {
      "@id": "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#2DGebouwgeometrie",
      "@type": "2DGebouwgeometrie",
      "geometrie": {
        "@type": "Relationship",
        "object": {
          "@id": "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#geometrieClass",
          "@type": "Geometry",
          "wkt": {
            "@type": "Property",
            "value": {
              "@value": "POINT(3.7288391590118404, 51.04909701806207)",
              "@type": "http://www.opengis.net/ont/geosparql#wktLiteral"
            }
          }
        }
      }
    }
  },
  "gebouwnaam": {
    "@type": "Property",
    "value": {
      "@value": "De Krook",
      "@language": "nl"
    }
  },
  "location": {
      "type": "GeoProperty",
      "value": {
          "type": "Point",
          "coordinates": [3.7288391590118404, 51.04909701806207]
      }
    }
}'
```

# Retrieve entity

A first method to retrieve our building is to use the `GET /entities` endpoint with following link: `http://localhost:9090/ngsi-ld/v1/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw`
A `type` query parameter is mandatory for this endpoint and must be encoded.

This will return an array of buildings with length 1:
```
[ {
  "id" : "http://www.wikidata.org/entity/Q28962266",
  "type" : "https://data.vlaanderen.be/ns/gebouw#Gebouw",
  "https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie" : {
    "type" : "Relationship",
    "object" : {
      "id" : "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#2DGebouwgeometrie",
      "type" : "https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie",
      "https://data.vlaanderen.be/ns/gebouw#geometrie" : {
        "type" : "Relationship",
        "object" : {
          "id" : "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#geometrieClass",
          "type" : "http://www.w3.org/ns/locn#Geometry",
          "http://www.opengis.net/ont/geosparql#asWKT" : {
            "type" : "Property",
            "value" : {
              "type" : "http://www.opengis.net/ont/geosparql#wktLiteral",
              "@value" : "POINT(3.7288391590118404, 51.04909701806207)"
            }
          }
        }
      }
    }
  },
  "https://data.vlaanderen.be/ns/gebouw#gebouwnaam" : {
    "type" : "Property",
    "value" : {
      "@language" : "nl",
      "@value" : "De Krook"
    }
  },
  "location" : {
    "type" : "GeoProperty",
    "value" : {
      "type" : "Point",
      "coordinates" : [ 3.7288391590118404, 51.04909701806207 ]
    }
  },
  "@context" : [ "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld" ]
}]
```
A second method is to use the `GET /entities/:id` endpoint with following link: `http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266`

# Retrieve OSLO compliant entity

We received our entity in the NGSI-LD metadata model, which allows adding metadata to properties and relations (cfr. property graph model).
In the NGSI-LD specification, it is mentioned that you use the options query parameter set to `keyValues` to retrieve a simplified representation of the response (`?options=keyValues`).
This way, we can retrieve data conform OSLO, however, the simplified representation only focuses at the first level of the graph.
To demonstrate this, run following request: `http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266?options=keyValues`

This returns our building as follows:

```
{
  "id" : "http://www.wikidata.org/entity/Q28962266",
  "type" : "https://data.vlaanderen.be/ns/gebouw#Gebouw",
  "https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie" : "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#2DGebouwgeometrie",
  "https://data.vlaanderen.be/ns/gebouw#gebouwnaam" : {
    "@language" : "nl",
    "@value" : "De Krook"
  },
  "location" : {
    "type" : "Point",
    "coordinates" : [ 3.7288391590118404, 51.04909701806207 ]
  },
  "@context" : [ "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld" ]
}
```

The value of `https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie` is only referring to the URI of the 2DGebouwgeometrie. A client is able to dereference `http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#2DGebouwgeometrie` as this uses the same entity page of our building.

When validating this snippet with the SHACL shapes of `OSLO Gebouwenregister`, we receive following validation errors:

* Value does not have class <https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie>
* Less than 1 values (<https://data.vlaanderen.be/ns/gebouw#Gebouw.status>). This is expected as this mandatory property was not provided in our data snippet.

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/validatie-gebouw.PNG" width=400>

The value of `https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie` (2DGebouwgeometrie) links to another object (datatype in this case). OSLO expects that at least the type of a datatype or class is mentioned with `@type`. Retrieving OSLO compliant relationships (datatypes or classes) is thus not possible in the simplified representation.
Although, there is no validation error for the `https://data.vlaanderen.be/ns/gebouw#gebouwnaam` property. This means that properties can be retrieved in an OSLO compliant way.

# Geotemporal query

Now that we hadded an entity to the NGSI-LD context broker, we can use the temporal query API to retrieve our building according to a certain time window and location.
This will be important in the NGSI-LDF tutorial where Linked Data Fragments are generated through temporal queries.

Following query parameters can be used to retrieve the building:

| Query parameter  | Value |
| ------------- | ------------- |
| type  | https://data.vlaanderen.be/ns/gebouw#Gebouw |
| georel  | within  |
| geometry  | Polygon  |
| coordinates  | [[[3.69140625,51.06901665960391],[3.69140625,51.04139389812637],[3.7353515625,51.04139389812637],[3.7353515625,51.06901665960391]]]  |
| timerel  | between  |
| time  | 2021-02-15T12:00:00.000Z  |
| endTime  | 2021-02-15T13:00:00.000Z  |
| timeproperty  | modifiedAt  |


Change the time interval with `time` and `endTime` to your current time.
The coordinates reflect a polygon around the center of the city of Ghent:

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/tile-ghent.PNG" width=400>

The, use following URL (adapt the time interval): http://localhost:9090/ngsi-ld/v1/temporal/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw&georel=within&geometry=Polygon&coordinates=[[[3.69140625,51.06901665960391],[3.69140625,51.04139389812637],[3.7353515625,51.04139389812637],[3.7353515625,51.06901665960391]]]&timerel=between&time=2021-02-15T12:00:00.000Z&endTime=2021-02-15T13:00:00.000Z&timeproperty=modifiedAt&options=sysAttrs

Note that we also added `options=sysAttrs` to retrieve extra system metadata, such as `createdAt` and `modifiedAt`.
