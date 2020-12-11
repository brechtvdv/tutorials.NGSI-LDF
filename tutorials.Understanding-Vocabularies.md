# Understanding vocabularies

In the previous tutorial, we explained how two house instances can be related to eachother using RDF triples. However, the houses are not "classified" as a house. We are able to understand through the URI that `houses:myHouse` probably is a `House`, but for a machine this is only a Web adress. This is important for queries later on, e.g. "give me all the houses". Also, predicates, which are relations (`hasNeighbouringHouse`) or attributes, must be classified so machines and humans understand its meaning and are able to act upon it (such as `reasoning`).

In this tutorial, we will briefly explain the RDF Schema (RDFS) vocabulary and domain-specific vocabularies like Schema.org.

## RDFS

> Prefixes: **rdf** https://www.w3.org/1999/02/22-rdf-syntax-ns# and **rdfs** https://www.w3.org/2000/01/rdf-schema#

A data modeller that wants to create a domain-specific vocabulary needs to specify classes, so we can classify resources (`House`), and properties (`hasHouseNumber`). 
RDF Schema (RDFS) is a vocubary to create vocabularies providing a basic set of classes and properties:

> Source: [Web Fundamentals](https://rubenverborgh.github.io/WebFundamentals/semantic-web/#rdfs-vocabulary) by Ruben Verborgh 

- `rdf:type` is a property stating that a resource is an instance of a class
- `rdfs:Resource` is a class of which everything is an instance
- `rdfs:Class` is a class for resources that conceptually define a set of things
- `rdf:Property` is a class for resources that can be used as triple predicates
- `rdfs:subClassOf` is a property stating all members of a class belong to another
- `rdfs:subPropertyOf` is a property stating a property is more specific than another

We can give more human-readable explanation to classes and properties with these properties:

- `rdfs:label` is a property that gives a human-readable name to a resource
- `rdfs:comment` is a property that clarifies human-readable meaning and usage

Now we can apply this to create a vocabulary for our knowledge of the previous tutorial by creating a class for houses and specifying that our predicates are properties.
For demonstration purposes, we added the class for `Buildings` from which a house is a specific type.
In JSON-LD this looks like this:

```
[{
  "@id": "http://example.org/Building",
  "@type": "rdfs:Class",
  "rdfs:label": "Building",
  "rdfs:comment": "This class allows to describe that a resource is a building."
},
{
  "@id": "http://example.org/House",
  "@type": "rdfs:Class",
  "rdfs:subClassOf": "http://example.org/Building",
  "rdfs:label": "Building",
  "rdfs:comment": "This class allows to describe that a resource is a house."
},
{
  "@id": "http://example.org/hasHouseNumber",
  "@type": "rdf:Property",
  "rdfs:label": "has house number",
  "rdfs:comment": "This property allows to specify a house number of a house."
},
{
  "@id": "http://example.org/hasNeighbouringHouse",
  "@type": "rdf:Property",
  "rdfs:label": "has neighbouring house",
  "rdfs:comment": "This property allows to specify the neighbouring house of a house."
}]
```
Note that `@type` is a JSON-LD keyword for `rdf:type`.

Finally, to specify where a predicate can be used, RDFS uses following properties:

- `rdfs:domain` is a property that states the class of possible subjects of a property
- `rdfs:range` is a property that states the class of possible objects of a property

In our example we can define that `hasNeighbouringHouse` can only be used on houses and points to a house:
```{
  "@id": "http://example.org/hasNeighbouringHouse",
  "@type": "rdf:Property",
  "rdfs:domain": "http://example.org/House",
  "rdfs:range": "http://example.org/House"
}
```
Note 1: it is possible to include a list of domains and ranges.
Note 2: this locks down the usage of this predicate to houses. When you use this predicate, machines will infer that your subject and object are houses.

To soften these formal domains and ranges, the schema.org initiative introduced two similar properties:

- `schema:domainIncludes` is a property that suggests the class of possible subjects of a property
- `schema:rangeIncludes` is a property that suggests the class of possible objects of a property

The goal here is to help users understand where a property could or should be used.

Here is an example how [schema.org](https://schema.org/version/latest/schemaorg-all-https.jsonld) defines a ´Place´ and see how similar this is with our Building description:
> Prefixes: **schema** https://www.schema.org/ and **schema** http://www.schema.org/

```
{
  "@id": "schema:Place",
  "@type": "rdfs:Class",
  "rdfs:comment": "Entities that have a somewhat fixed, physical extension.",
  "rdfs:label": "Place",
  "rdfs:subClassOf": {
    "@id": "schema:Thing"
  },
}
```
Below you can find more similar examples of W3C SSN and OSLO.

# NGSI-LD Metadata model

In the previous section, we saw that RDFS allows to define classes and properties for a certain vocabulary.
However, complex situations... (n-ary relations etc.)
-> RDF*


 ### NGSI-LD
> Prefixes: **ngsi-ld** "https://uri.etsi.org/ngsi-ld/" and **fiware** "https://uri.fiware.org/ns/data-models#"



## Domain vocabularies


### W3C SSN
> Prefixes: **ssn**  and **sosa** 

### OSLO
> Prefixes: **oslo** https://www.data.vlaanderen.be/ns/

### Schema.org




