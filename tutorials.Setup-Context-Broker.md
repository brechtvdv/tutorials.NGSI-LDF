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

Run the command below in your CLI to add a building with URN 'urn:DeKrook' to the NGSI-LD endpoint.
The coordinates match with the location of public library "De Krook" in the city of Ghent.
// TODO: change to URL `http://localhost:9090/ngsi-ld/v1/entities/myHouse`
We must use the NGSI-LD metadata model to make it retrievable with the temporal query API.
To retrieve the subject page of this specific entity, we added a `rdfs:seeAlso` link to the /entities/{id} endpoint.

```
curl --location --request POST 'localhost:9090/ngsi-ld/v1/entities' \
--header 'Content-Type: application/ld+json' \
--data-raw '{
      "@id": "urn:DeKrook",
      "@type": "Building",
      "seeAlso": {
        "@type": "Relationship",
        "object": "base:urn%3AmyHouse"
      },
      "location": {
            "type": "GeoProperty",
            "value": {
                "type": "Point",
                "coordinates": [3.7288391590118404, 51.04909701806207]
            }
      },
      "hasHouseNumber": {
          "@type": "Property",
          "value": "14"
      },
      "hasNeighbouringHouse": {
        "@type": "Relationship",
        "object": "urn:myNeighboursHouse"
      },
      "@context": [{
            "base": "http://localhost:9090/ngsi-ld/v1/entities/",
            "seeAlso": {
                "@id": "http://www.w3.org/2000/01/rdf-schema#seeAlso",
                "@type": "@id"
            },
            "Building": "https://data.vlaanderen.be/ns/gebouw#Gebouw",
            "hasHouseNumber": {
              "@id": "http://example.org/hasHouseNumber",
              "@type": "http://www.w3.org/2001/XMLSchema#string"
            },
            "hasNeighbouringHouse": {
              "@id": "http://example.org/hasNeighbouringHouse",
              "@type": "@id"
            }
    },
    "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld",
    "https://geojson.org/geojson-ld/geojson-context.jsonld"
    ]
}'
```

# Retrieve entity

Our building can be retrieved using following link: `http://localhost:9090/ngsi-ld/v1/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw`
A `type` query parameter is mandatory in NGSI-LD and must be encoded.

This should return following data snippet:
```
{
  "id" : "urn:myHouse",
  "type" : "https://data.vlaanderen.be/ns/gebouw#Gebouw",
  "http://example.org/hasHouseNumber" : {
    "type" : "Property",
    "value" : "3"
  },
  "http://example.org/hasNeighbouringHouse" : {
    "type" : "Relationship",
    "object" : "houses:myNeighboursHouse"
  },
  "http://www.w3.org/2000/01/rdf-schema#seeAlso" : {
    "type" : "Relationship",
    "object" : "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Flocalhost%3A9090%2Fngsi-ld%2Fv1%2Fentities%2FmyHouse&options=keyValues"
  },
  "location" : {
    "type" : "GeoProperty",
    "value" : {
      "type" : "Point",
      "coordinates" : [ 3.7288391590118404, 51.04909701806207 ]
    }
  },
  "@context" : [ "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld" ]
}
```

Use the `&options=keyValues` query parameter to retrieve the simplified, RDFS Class/Property compatible representation:
`http://localhost:9090/ngsi-ld/v1/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw&options=keyValues`

```
{
  "id" : "urn:myHouse",
  "type" : "https://data.vlaanderen.be/ns/gebouw#Gebouw",
  "http://example.org/hasHouseNumber" : "3",
  "http://example.org/hasNeighbouringHouse" : "houses:myNeighboursHouse",
  "http://www.w3.org/2000/01/rdf-schema#seeAlso" : "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Flocalhost%3A9090%2Fngsi-ld%2Fv1%2Fentities%2FmyHouse&options=keyValues",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 3.7288391590118404, 51.04909701806207 ]
  },
  "@context" : [ "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld" ]
}
```
Note that that "Point" and "coordinates" lost its correct mapping to GeoJSON-LD due to a bug.

# GeoTemporal query

# Retrieve OSLO compliant object

