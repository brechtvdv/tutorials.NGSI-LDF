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

Our building can be retrieved using following link: `http://localhost:9090/ngsi-ld/v1/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw`
A `type` query parameter is mandatory in NGSI-LD and must be encoded.

You can also add a context link through the HTTP Link-header to make it more readable:
```
curl --location --request GET 'localhost:9090/ngsi-ld/v1/entities?type=Gebouw' \
--header 'Accept: application/ld+json' \
--header 'Link: <http://data.vlaanderen.be/context/gebouwenregister.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

The latter should return following data snippet:
```
[
    {
        "id": "http://www.wikidata.org/entity/Q28962266",
        "type": "Gebouw",
        "Gebouw:.geometrie": {
            "type": "Relationship",
            "object": {
                "type": "https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie",
                "https://data.vlaanderen.be/ns/gebouw#geometrie": {
                    "type": "Relationship",
                    "object": {
                        "type": "http://www.w3.org/ns/locn#Geometry",
                        "http://www.opengis.net/ont/geosparql#asWKT": {
                            "type": "Property",
                            "value": {
                                "type": "http://www.opengis.net/ont/geosparql#wktLiteral",
                                "@value": "POINT(3.7288391590118404, 51.04909701806207)"
                            }
                        }
                    }
                }
            }
        },
        "https://data.vlaanderen.be/ns/gebouw#gebouwnaam": {
            "type": "Property",
            "value": "De Krook"
        },
        "location": {
            "type": "GeoProperty",
            "value": {
                "type": "Point",
                "coordinates": [
                    3.7288391590118404,
                    51.04909701806207
                ]
            }
        },
        "@context": [
            "http://data.vlaanderen.be/context/gebouwenregister.jsonld",
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"
        ]
    }
]
```
Note that that "Point" and "coordinates" lost its correct mapping to GeoJSON-LD due to a bug.

# GeoTemporal query

# Retrieve OSLO compliant object

