In this tutorial, we will a create a simple data snippet, which represents a building, in accordance with the OSLO Application Profile (AP) ["Building registry"](https://data.vlaanderen.be/doc/applicatieprofiel/gebouwenregister).
In the next tutorial, We will ingest this snippet in a context broker.

## OSLO Building registry

On [https://data.vlaanderen.be](https://data.vlaanderen.be), all available vocabularies and APs that are governed by the Flemish government are listed:

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/datavlaanderenbe.PNG" width=400>

The Building registry AP (Dutch: Gebouwenregister) is used to describe a building.
In the UML diagram we see that "Building" (Dutch: Gebouw) is the core class and contains a list of attributes, and relations to other classes.
For our data snippet, we will only provide two properties: geometry (Dutch: geometrie) and buildingName (Dutch: gebouwNaam):

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/gebouwuml.PNG" width=400>

Our data snippet will not be compliant with the AP, because we will not provide the mandatory property "status".

## Data snippet

Every AP has a dedicated JSON-LD context, which we can reuse.
You can find the context at the bottom of the AP specification: http://data.vlaanderen.be/context/gebouwenregister.jsonld
This is the starting point of our JSON-LD snippet:

```
{
  "@context": "http://data.vlaanderen.be/context/gebouwenregister.jsonld"
}
```

As example, we will publish data about the public library of Ghent "De Krook".
First, following the principles of Linked Data, we must reuse an existing identifier of this building.
We will use its Wikidata identifier, which can be found by copying the link at "Concept URI":

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/dekrookuri.PNG" width=400>

```
{
  "@context": "http://data.vlaanderen.be/context/gebouwenregister.jsonld",
  "@id": "http://www.wikidata.org/entity/Q28962266"
}
```

To describe that this is a building, we need to look in the OSLO specification for "Building" (Gebouw).
When you hover over it, you can see the URI that is linked with this term:

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/gebouwuri.PNG" width=400>

Make sure to check in the context what the correct term is to use in your snippet:

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/gebouwuricontext.PNG" width=400>

We can now safely add the '@type' property:

```
{
  "@context": "http://data.vlaanderen.be/context/gebouwenregister.jsonld",
  "@id": "http://www.wikidata.org/entity/Q28962266",
  "@type": "Gebouw"
}
```

Of course, we can add a translation in the context if English is preferred:

```
{
  "@context": [ "http://data.vlaanderen.be/context/gebouwenregister.jsonld", {
    "Building": "Gebouw"
  },
  "@id": "http://www.wikidata.org/entity/Q28962266",
  "@type": "Building"
}
```

## Adding properties

For the geometry property, we need to create a 2D-Gebouwgeometrie object.
This an intermediary object and also contains a geometry property that will contain a WKT literal for the exact coordinates.
Again, it is important to hover in the specification for every property what its URI is and then use this to find out the correct term in the JSON-LD context. For "geometrie" of a building, the URI is "https://data.vlaanderen.be/ns/gebouw#Gebouw.geometrie". In the context, we see that we need to use "Gebouw.geometrie" as term in our snippet. Note that not all terms are mapped in the context, such as "2DGebouwgeomtrie". These terms need to be added manually to the context of the snippet.

```
{
  "@context": ["http://data.vlaanderen.be/context/gebouwenregister.jsonld", {
    "2DGebouwgeometrie": "https://data.vlaanderen.be/ns/gebouw#2DGebouwgeometrie",
    "Geometry": "http://www.w3.org/ns/locn#Geometry"
  }],
  "@id": "http://www.wikidata.org/entity/Q28962266",
  "@type": "Gebouw",
  "Gebouw.geometrie": {
    "@type": "2DGebouwgeometrie",
    "geometrie": {
      "@type": "Geometry",
      "wkt": {
        "@value": "POINT(3.7288391590118404, 51.04909701806207)",
        "@type": "http://www.opengis.net/ont/geosparql#wktLiteral"
      }
    }
  },
  "gebouwnaam": {
    "@value": "De Krook",
    "@lang": "nl"
  }
}
```

We used the [WKT playground](https://clydedacruz.github.io/openstreetmap-wkt-playground/) to determine the WKT location of De Krook.
The building name expects a Geografic name (GeografischeNaam). When you hover it, you see that this is a language-tagged string. This is solved in JSON-LD with the `@lang` property. 

Finally, use the [JSON-LD playground](https://json-ld.org/playground/#startTab=tab-table&json-ld=%7B%22%40context%22%3A%5B%22http%3A%2F%2Fdata.vlaanderen.be%2Fcontext%2Fgebouwenregister.jsonld%22%2C%7B%222DGebouwgeometrie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%232DGebouwgeometrie%22%2C%22Geometry%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Geometry%22%7D%5D%2C%22%40id%22%3A%22http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266%22%2C%22%40type%22%3A%22Gebouw%22%2C%22Gebouw.geometrie%22%3A%7B%22%40type%22%3A%222DGebouwgeometrie%22%2C%22geometrie%22%3A%7B%22%40type%22%3A%22Geometry%22%2C%22wkt%22%3A%7B%22%40value%22%3A%22POINT(3.7288391590118404%2C%2051.04909701806207)%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23wktLiteral%22%7D%7D%7D%2C%22gebouwnaam%22%3A%7B%22%40value%22%3A%22De%20Krook%22%2C%22%40lang%22%3A%22nl%22%7D%7D) to check if all triples are correctly returned:

<img src="https://github.com/brechtvdv/tutorials.NGSI-LDF/raw/master/playground-snippet.PNG" width=400>

## NGSI-LDify

Our data snippet is conform with OSLO. However, to ingest this in an NGSI-LD context broker, we need to apply the NGSI-LD metamodel to our data snippet. The snippet will not have the semantic meaning it is intended by OSLO, but we will be able to process it in a context broker and fix this later on with 1) the keyValues option or 2) NGSI-LDF.

NGSI-LDifying means that we will add an intermediary ngsi:Property object when a triple's object is a Literal, and an intermediary ngsi:Relationship object when the triple's object is another resource. We added the NGSI core context for these terms.

The NGSI-LD data snippet looks as follows:

```
{
  "@context": ["http://data.vlaanderen.be/context/gebouwenregister.jsonld", 
    "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld", {
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
      "@value": "De Krook",
      "@lang": "nl"
    }
  }
}
```

If we want our building to be retrievable with geographic queries using the temporal query API, we need to add the `ngsi:location` property to the building. This location requires a GeoJSON-LD value, so we also add the GeoJSON-LD context to add context to "Point" and "coordinates" :

```
{
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
      "@value": "De Krook",
      "@lang": "nl"
    }
  },
  "location": {
      "type": "GeoProperty",
      "value": {
          "type": "Point",
          "coordinates": [3.7288391590118404, 51.04909701806207]
      }
    }
}
```

Use the [JSON-LD playground](https://json-ld.org/playground/#startTab=tab-table&json-ld=%7B%22%40context%22%3A%5B%22http%3A%2F%2Fdata.vlaanderen.be%2Fcontext%2Fgebouwenregister.jsonld%22%2C%22https%3A%2F%2Furi.etsi.org%2Fngsi-ld%2Fv1%2Fngsi-ld-core-context.jsonld%22%2C%22https%3A%2F%2Fgeojson.org%2Fgeojson-ld%2Fgeojson-context.jsonld%22%2C%7B%222DGebouwgeometrie%22%3A%22https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%232DGebouwgeometrie%22%2C%22Geometry%22%3A%22http%3A%2F%2Fwww.w3.org%2Fns%2Flocn%23Geometry%22%7D%5D%2C%22%40id%22%3A%22http%3A%2F%2Fwww.wikidata.org%2Fentity%2FQ28962266%22%2C%22%40type%22%3A%22Gebouw%22%2C%22Gebouw.geometrie%22%3A%7B%22%40type%22%3A%22Relationship%22%2C%22object%22%3A%7B%22%40type%22%3A%222DGebouwgeometrie%22%2C%22geometrie%22%3A%7B%22%40type%22%3A%22Relationship%22%2C%22object%22%3A%7B%22%40type%22%3A%22Geometry%22%2C%22wkt%22%3A%7B%22%40type%22%3A%22Property%22%2C%22value%22%3A%7B%22%40value%22%3A%22POINT(3.7288391590118404%2C%2051.04909701806207)%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.opengis.net%2Font%2Fgeosparql%23wktLiteral%22%7D%7D%7D%7D%7D%7D%2C%22gebouwnaam%22%3A%7B%22%40type%22%3A%22Property%22%2C%22value%22%3A%7B%22%40value%22%3A%22De%20Krook%22%2C%22%40lang%22%3A%22nl%22%7D%7D%2C%22location%22%3A%7B%22type%22%3A%22GeoProperty%22%2C%22value%22%3A%7B%22type%22%3A%22Point%22%2C%22coordinates%22%3A%5B3.7288391590118404%2C51.04909701806207%5D%7D%7D%7D) to check whether everything parses correctly.
