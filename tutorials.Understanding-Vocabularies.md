# Understanding vocabularies

In the previous tutorial, we explained how two house instances can be related to eachother using RDF triples. However, the houses are not "classified" as a house. We are able to understand through the URI that `houses:myHouse` probably is a `House`, but for a machine this is only a Web adress. This is important for queries later on, e.g. "give me all the houses". Also, predicates, which are relations (`hasNeighbouringHouse`) or attributes, must be classified so machines and humans understand its meaning and are able to act upon it (such as `reasoning`).

In this tutorial, we will briefly explain the RDF Schema (RDFS) vocabulary and domain-specific vocabularies like Schema.org.

# RDFS

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

But what if I want to add metadata to triples? (like how certain it is or when was this generated)
RDF has multiple ways to solve this (use the PROV-O ontology to add provenance, or N-ary relations, reification, named graphs etc.), but these methods are not intuitive. See [this article](https://enterprise-knowledge.com/rdf-what-is-it-and-why-do-i-need-it/) for an in-depth explanation.

Currently, there are two new approaches that also tackle this:
- **RDF\***: an extension of RDF, which preserves the strength and simplicity of RDF (it's backwards compatible), but it also allows referencing triples in an intuitive way. [Here](https://blog.liu.se/olafhartig/2019/01/10/position-statement-rdf-star-and-sparql-star/) you can find more information about RDF*.
- The **NGSI-LD metadata model**: a metadata model built upon RDF, but taking a different approach from the RDFS Class and Property way of working.

# NGSI-LD Metadata model
> Prefixes: **ngsi-ld** "https://uri.etsi.org/ngsi-ld/" and **fiware** "https://uri.fiware.org/ns/data-models#"

In previous section, we explained that RDFS allows us to define nodes with rdfs:Class and the relationships between nodes with rdfs:Property. With context brokers, we want to add metadata to relationships (e.g. when was this created). However, this is not possible with a rdfs:Property, because instantiation occurs on a rdfs:Class and a rdfs:Property only connects two instances. To solve this, NGSI-LD created its own metadata model based on rdfs:Resource, which encompasses rdfs:Class and rdfs:Property. Concretely, this means that a resourcy of NGSI-LD can be used as both, as rdfs:Property to connect two classes and as rdfs:Class to add metadata.

There are four types of rdfs:Resource created by NGSI-LD:

- `ngsi-ld:Entity` is a class to describe something that is supposed to exist in the real world, physically or
conceptually
- `ngsi-ld:Relationship` is a class to describe a directed link between a subject which is either an NGSI-LD Entity, an
NGSI-LD Property, or another NGSI-LD Relationship on one hand, and an object, which is an NGSI-LD Entity, on the
other hand, and which uses the special `hasObject` property to define its target object
- `ngsi-ld:Property` is a class to associates a main characteristic, i.e. an NGSI-LD Value, to either an
NGSI-LD Entity, an NGSI-LD Relationship or another NGSI-LD Property and which uses the special `hasValue`
property to define its target value
- `ngsi-ld:Value` is a class for resources that have a JSON value (i.e. a string, a number, true or false, an object, an array), or a JSON-LD typed value
(i.e. a string as the lexical form of the value together with a type, defined by an XSD base type or more generally an
IRI), or a JSON-LD structured value (i.e. a set, a list, a language-tagged string)

and two types of rdfs:Property:
- `ngsi-ld:hasObject` is a property that states the ngsi-ld:Entity of a ngsi-ld:Relationship
- `ngsi-ld:hasValue` is a property that states the ngsi-ld:Value of a ngsi-ld:Property

Let's describe our house "vocabulary" in NGSI-LD to make things more clear:
Note that we leave out rdfs:label and rdfs:comment for abbreviation.

```
[{
  "@id": "http://example.org/Building",
  "@type": "ngsi-ld:Entity",
},
{
  "@id": "http://example.org/House",
  "@type": "ngsi-ld:Entity",
  "rdfs:subClassOf": "http://example.org/Building",
},
{
  "@id": "http://example.org/hasHouseNumber",
  "@type": "ngsi-ld:Property",
},
{
  "@id": "http://example.org/hasNeighbouringHouse",
  "@type": "ngsi-ld:Relationship",
}]
```
A `ngsi-ld:Entity` is similar, if not the same, as `rdfs:Class`.
However, the trick comes when applying `ngsi-ld:Property` and `ngsi-ld:Relationship`.
When reading their definition, they have respectively the special `hasValue` and `hasObject`
property, which means they are a `rdfs:Class` with properties.
However, in contrast with the definition but possible as a `rdfs:Resource`, NGSI-LD also uses an `ngsi-ld:Property` and `ngsi-ld:Relationship` as a `rdfs:Property` to link a `ngsi-ld:Entity` with respectively an instance of the `ngsi-ld:Property` and `ngsi-ld:Relationship`.


```
{
  "@context": {
    "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
    "Entity": "ngsi-ld:Entity",
    "Property": "ngsi-ld:Property",
    "hasValue": "ngsi-ld:hasValue",
    "Relationship": "ngsi-ld:Relationship",
    "hasObject": "ngsi-ld:hasObject",
    "houses": "http://example.org/houses/id/",
    "hasHouseNumber": {
      "@id": "http://example.org/hasHouseNumber",
      "@type": "ngsi-ld:Value"
    },
    "hasNeighbouringHouse": {
      "@id": "http://example.org/hasNeighbouringHouse",
      "@type": "@id"
    }
  },
  "@id": "houses:myHouse",
  "@type": "Entity",
  "hasHouseNumber": {
    "@type": "hasHouseNumber", // they only mention ngsi-ld:Property
    "hasValue": "14"
  },
  "hasNeighbouringHouse": {
    "@type": "hasNeighbouringHouse", // they only mention ngsi-ld:Relationship
    "hasObject": "houses:myNeighboursHouse"
  }
}
```

## But what if you want to use existing vocabularies inside this metadata model?

Reusing classes of a vocabulary is not a problem in the NGSI-LD metadata model.
Reusing properties of a vocabulary is however less convenient.
It depends on whether it uses the domainIncludes/rangeIncludes of schema.org or domain/range approach of RDFS.

### Domain/range includes

When the domain and range of a property are defined with the includes-approach of schema.org,
then this is less formal and thus allows the property to point to something else like an instance of ngsi-ld:Relationship.

For example, to reuse the rdfs:Property `https://schema.org/address`, you would create something like this when following the metadata model strictly:

```
"schema:address": {
  "@type": "schema:address", // schema:address is not a rdfs:Class
  "hasObject": {
    "@id": "houses:myAddress",
    "type": "schema:Address"
   }
}
```
However, you cannot make an instance of schema:address as this is not a rdfs:Class.
This can be solved by setting `@type` to `ngsi-ld:Relationship`, which is actually the default strategy used in NGSI-LD examples.
Now "schema:address" does not point to an address directly, but to a ngsi-ld:Relationship instance. This is possible thanks to less formal semantics of the property.

### Domain/range with RDFS


However, you cannot reuse vocabularies that defined properties with `rdfs:Domain` and `rdfs:Range`.
When you use domain/range, then the instance of the object must be of...


## Domain vocabularies


### W3C SSN
> Prefixes: **ssn**  and **sosa** 

### OSLO
> Prefixes: **oslo** https://www.data.vlaanderen.be/ns/

### Schema.org




