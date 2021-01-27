In this tutorial, we will a create a simple data snippet, which represents a building, in accordance with the OSLO Application Profile (AP) ["Building registry"](https://data.vlaanderen.be/doc/applicatieprofiel/gebouwenregister).
In the next tutorial, We will ingest this snippet in a context broker.

## OSLO Building registry

On [https://data.vlaanderen.be](https://data.vlaanderen.be), all available vocabularies and APs that are governed by the Flemish government is listed:

TODO: insert figure

The Building registry AP (Dutch: Gebouwenregister) is used to describe a building.
In the UML diagram we see that "Building" (Dutch: Gebouw) is the core class and contains a list of attributes, and relations to other classes.
For our data snippet, we will only provide two properties: geometry (Dutch: geometrie) and buildingName (Dutch: gebouwNaam):

TODO: figure with properties indicated

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

TODO figure wikidata

```
{
  "@context": "http://data.vlaanderen.be/context/gebouwenregister.jsonld",
  "@id": "http://www.wikidata.org/entity/Q28962266"
}
```

To describe that this is a building, we need to look in the OSLO specification for "Building" (Gebouw).
When you hover over it, you can see the URI that is linked with this term:

TODO add figure gebouwuri

Make sure to check in the context what the correct term is to use in your snippet:

TODO add gebouwuricontext

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


