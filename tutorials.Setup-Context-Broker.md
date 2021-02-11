In this tutorial, we will a setup a NGSI-LD context broker.
The NGSI-LDified OSLO data snippet of the previous [tutorial](https://github.com/brechtvdv/tutorials.NGSI-LDF/blob/master/tutorials.Data-Snippet.md#ngsi-ldify) will be ingested to showcase some of the API calls that are possible with NGSI-LD.
Also, we will demonstrate how the query parameter `options=keyValues` allows to output OSLO-compliant data.

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
  "@type": "Gebouw",
  "Gebouw.geometrie": {
    "@type": "Relationship",
    "object": {
      "@type": "2DGebouwgeometrie",
      "geometrie": {
        "@type": "Relationship",
        "object": {
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
      "@value": "De Krook"
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

A second method is to use the `GET /entities/:id` endpoint with following link: `http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266`

# GeoTemporal query



# Retrieve OSLO compliant object

