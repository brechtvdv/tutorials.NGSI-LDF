We expect for this tutorial that you have a running context broker with `De krook` as example data. If this is not the case, see previous tutorial: https://github.com/brechtvdv/tutorials.NGSI-LDF/blob/master/tutorials.Setup-Context-Broker.md
NGSI-LDF is a Web API on top of a NGSI-LD context broker that republishes entities inside cacheable Linked Data Fragments and repackages entities' temporal evolution following the [Event Stream specification](https://treecg.github.io/SEMIC-Linked-Data-Event-Streams/eventstreams.html).

# Install NGSI-LDF

```
git clone git@github.com:brechtvdv/ngsi-ldf.git
cd ngsi-ldf
npm install
npm start
```

Test if you receive a response with following URL: http://localhost:3001/13/4180/2740/latest?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw

# Fragmentations

NGSI-LDF fragments the NGSI-LD into geotemporal Linked Data Fragments and exposes its entities as an event stream.

There are three geospatial fragmentation methods implemented: slippy tiles, H3 and Geohash. 
In this tutorial, we will focus on slippy tiles, because this is similar to map interfaces (Open Street Map, Google Maps).
Fragments that use this fragmentation adhere to the following path template: `/{z}/{x}/{y}` where:

* `z` is the zoom level. Zoom level 0 covers the entire world, zoom level 1 contains 4 tiles, ... We support values that lie in the interval [13, 14].
* `x` is the tile longitude. Possible values lie in the interval [0, 2^n], where 0 corresponds to the westernmost tile and 2^n corresponds to the easternmost tile.
* `y`is the tile longitude. Possible values lie in the interval [0, 2^n], where 0 corresponds to the *northernmost* (**not** the southernmost) tile and 2^n corresponds to the southernmost tile. 

Example functions to translate a WGS84 coordinate to a tile coordinate as well as functions to translate tiles coordinates to their *top left* corners can be found on the [OpenStreetMap wiki](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames).

In the previous tutorial, we added `De Krook` as example entity. Using the formula from the OpenStreetMap wiki, we found that following path can be used for the tile that contains De Krook: `/13/4180/2740`.

The temporal fragmentation is easier, because it uses only one variable: the `page` query parameter.
Temporal fragmentation is currently implemented with fixed buckets of one hour: between 1am and 2am, between 2am and 3am etc.

Lastly, it is necessary to provide the `type` query parameter as this will be passed to the underlying NGSI-LD endpoint.

Every fragment is a tree:node and part of the collection:
```
@id: "http://localhost:3001/13/4180/2740?page=2021-02-15T13:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
@type: "tree:Node"
```

## Hypermedia controls

Every fragment contains hypermedia controls, such as:

**Search form**
```
{
@id: "http://localhost:3001/13/4180/2740?page=2021-02-15T13:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
"dcterms:isPartOf": {
   @id: "http://localhost:9001/ngsi-ld/v1/entities?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
  "@type": "tree:Collection", 
  "hydra:search": {
    "@type": "hydra:IriTemplate",
    "hydra:template": "http://localhost:3001/{z}/{x}/{y}{?type,page}",
    "hydra:variableRepresentation": "hydra:BasicRepresentation",
    "hydra:mapping": [{
      "@type": "hydra:IriTemplateMapping",
      "hydra:variable": "z",
      "hydra:property": "tiles:zoom",
      "hydra:required": true
    }, {
      "@type": "hydra:IriTemplateMapping",
      "hydra:variable": "x",
      "hydra:property": "tiles:longitudeTile",
      "hydra:required": true
    }, {
      "@type": "hydra:IriTemplateMapping",
      "hydra:variable": "y",
      "hydra:property": "tiles:latitudeTile",
      "hydra:required": true
    }, {
      "@type": "hydra:IriTemplateMapping",
      "hydra:variable": "type",
      "hydra:property": "rdf:type",
      "hydra:required": true
    }, {
      "@type": "hydra:IriTemplateMapping",
      "hydra:variable": "page",
      "hydra:property": "schema:startDate",
      "hydra:required": false
    }]
   }
  }
}
```


**TREE relations**
```
@id: "http://localhost:3001/13/4180/2740?page=2021-02-15T13:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
"tree:relation": [{
	"@type": "tree:LessThanRelation",
	"tree:node": "http://localhost:3001/13/4180/2740?page=2021-02-15T11:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
	"tree:path": "ngsi-ld:modifiedAt",
	"tree:value": {
		"schema:startDate": "2021-02-15T11:00:00.000Z",
		"schema:endDate": "2021-02-15T12:00:00.000Z"
	}
}, {
	"@type": "tree:GreaterThanRelation",
	"tree:node": "http://localhost:3001/13/4180/2740?page=2021-02-15T13:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
	"tree:path": "ngsi-ld:modifiedAt",
	"tree:value": {
		"schema:startDate": "2021-02-15T13:00:00.000Z",
		"schema:endDate": "2021-02-15T14:00:00.000Z"
	}
}]
```

**Event Stream**:

To receive our building, we can use following URL: http://localhost:3001/13/4180/2740?page=2021-02-15T12:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw
Change the page query parameter to the time you created De Krook.

This should return following response:
```
{
	"@context": [{
			"xsd": "http://www.w3.org/2001/XMLSchema#",
			"schema": "http://schema.org/",
			"schema:endDate": {
				"@type": "xsd:dateTime"
			},
			"dcterms": "http://purl.org/dc/terms/",
			"tiles": "https://w3id.org/tree/terms#",
			"hydra": "http://www.w3.org/ns/hydra/core#",
			"hydra:variableRepresentation": {
				"@type": "@id"
			},
			"hydra:property": {
				"@type": "@id"
			},
			"tree": "https://w3id.org/tree/terms#",
			"tree:node": {
				"@type": "@id"
			},
			"sh": "https://www.w3.org/ns/shacl#",
			"tree:path": {
				"@type": "@id"
			},
			"ngsi-ld": "https://uri.etsi.org/ngsi-ld/"
		},
		["https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"]
	],
	"@id": "http://localhost:3001/13/4180/2740?page=2021-02-15T12:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
	"@type": "tree:Node",
	"tiles:zoom": 13,
	"tiles:longitudeTile": 4180,
	"tiles:latitudeTile": 2740,
	"tree:relation": [{
		"@type": "tree:LessThanRelation",
		"tree:node": "http://localhost:3001/13/4180/2740?page=2021-02-15T11:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
		"tree:path": "ngsi-ld:modifiedAt",
		"tree:value": {
			"schema:startDate": "2021-02-15T11:00:00.000Z",
			"schema:endDate": "2021-02-15T12:00:00.000Z"
		}
	}, {
		"@type": "tree:GreaterThanRelation",
		"tree:node": "http://localhost:3001/13/4180/2740?page=2021-02-15T13:00:00.000Z&type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
		"tree:path": "ngsi-ld:modifiedAt",
		"tree:value": {
			"schema:startDate": "2021-02-15T13:00:00.000Z",
			"schema:endDate": "2021-02-15T14:00:00.000Z"
		}
	}],
	"tree:path": "ngsi-ld:modifiedAt",
	"tree:value": {
		"schema:startDate": "2021-02-15T12:00:00.000Z",
		"schema:endDate": "2021-02-15T13:00:00.000Z"
	},
	"dcterms:isPartOf": {
		"@id": "http://localhost:3001/https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw",
		"@type": "tree:Collection",
		"hydra:search": {
			"@type": "hydra:IriTemplate",
			"hydra:template": "http://localhost:3001/{z}/{x}/{y}{?type,page}",
			"hydra:variableRepresentation": "hydra:BasicRepresentation",
			"hydra:mapping": [{
				"@type": "hydra:IriTemplateMapping",
				"hydra:variable": "z",
				"hydra:property": "tiles:zoom",
				"hydra:required": true
			}, {
				"@type": "hydra:IriTemplateMapping",
				"hydra:variable": "x",
				"hydra:property": "tiles:longitudeTile",
				"hydra:required": true
			}, {
				"@type": "hydra:IriTemplateMapping",
				"hydra:variable": "y",
				"hydra:property": "tiles:latitudeTile",
				"hydra:required": true
			}, {
				"@type": "hydra:IriTemplateMapping",
				"hydra:variable": "type",
				"hydra:property": "rdf:type",
				"hydra:required": true
			}, {
				"@type": "hydra:IriTemplateMapping",
				"hydra:variable": "page",
				"hydra:property": "schema:startDate",
				"hydra:required": false
			}]
		}
	},
	"@included": [{
		"id": "http://www.wikidata.org/entity/Q28962266/2021-02-15T12:08:09.436Z",
		"type": "https://data.vlaanderen.be/ns/gebouw#Gebouw",
		"https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie": {
			"type": "Relationship",
			"createdAt": "2021-02-15T12:08:09.930757Z",
			"object": {
				"id": "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#2DGebouwgeometrie",
				"type": "https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie",
				"https://data.vlaanderen.be/ns/gebouw#geometrie": {
					"type": "Relationship",
					"object": {
						"id": "http://localhost:9090/ngsi-ld/v1/entities/http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266#geometrieClass",
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
			},
			"instanceId": "urn:ngsi-ld:b911425e-93fd-4b5d-8109-2d9610a20421",
			"modifiedAt": "2021-02-15T12:08:09.930757Z"
		},
		"https://data.vlaanderen.be/ns/gebouw#gebouwnaam": {
			"type": "Property",
			"createdAt": "2021-02-15T12:08:09.930757Z",
			"value": {
				"@language": "nl",
				"@value": "De Krook"
			},
			"instanceId": "urn:ngsi-ld:92b309a3-573d-4261-afd9-23700881f31c",
			"modifiedAt": "2021-02-15T12:08:09.930757Z"
		},
		"createdAt": "2021-02-15T12:08:09.4368909Z",
		"location": {
			"type": "GeoProperty",
			"createdAt": "2021-02-15T12:08:09.930757Z",
			"value": {
				"type": "Point",
				"coordinates": [3.7288391590118404, 51.04909701806207]
			},
			"instanceId": "urn:ngsi-ld:3c477c41-c3a4-4524-94dc-8ee977f3d5d9",
			"modifiedAt": "2021-02-15T12:08:09.930757Z"
		},
		"modifiedAt": "2021-02-15T12:08:09.4368909Z",
		"@context": ["https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"],
		"dcterms:isVersionOf": "http://www.wikidata.org/entity/Q28962266",
		"prov:generatedAtTime": "2021-02-15T12:08:09.436Z",
		"memberOf": "http://localhost:3001/https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw"
	}]
}
```
