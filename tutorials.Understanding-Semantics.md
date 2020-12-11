# Understanding Semantics

This tutorial introduces basics of vocabularies to describe things in a certain domain. 
These vocabularies ensure that applications speak the same language (are semantically interoperable) and are one of the prerequisites to publish Linked Data.
For NGSI-LDF, Linked Data aligned with vocabularies enable interoperability with other Digital Twin components. 

We will introduce the RDF data model and investigate how these are applied in the *Schema.org*, *NGSI-LD*, *OSLO* and *W3C* ecosystems. 
Then, we will explain what Application Profiles are and how they can help developers achieving data compliance.

In the next tutorial, we will create a OSLO-compliant data snippet of a Building which we will use further in the tutorials.

## RDF

RDF or the Resource Description Framework is a standardized model to describe knowledge.
Knowledge is represented as a set of facts, which are described with triple-statements.

For example, if I want to express that the housenumber of my house is 14,
than you describe this with an RDF triple:
```
<myHouse> <hasHouseNumber> "14" .
```

In a RDF triple, you have three parts:
- the subject (\<myHouse\>)
- the predicate (\<hasHouseNumber\>)
- the object ("14")
  
In this example, the subject and predicate are `nodes` while the object is a `literal`.
It's also possible to have a node in the object, for example:
```
<myHouse> <hasNeighbouringHouse> <myNeighoursHouse> .
```

Nodes are the glue for Linked Data: other datasets can also reference \<myHouse\> when they want to state facts about my house (e.g. how good is the air quality on a working day). Also, it's important that datasets use the same reference for predicates, like \<hasHouseNumber\>, to allow automatic integration of different datasets.

There are two types of nodes: named nodes and blank nodes. 
`Named nodes` are resources that are referenced by an URI. This is an unique, global identifier on the Web and as best-practice it should be dereferenceable allowing others to look it up.
`Blank nodes` are unnamed resources and are locally identified within the datadump or Web document that created this node.

There are multiple syntaxes available to describe RDF (JSON-LD, Turtle, N-Quads...). In these tutorials, we will focus on the usage of JSON-LD.
JSON-LD uses the JSON syntax, but maps JSON terms to URIs through a (JSON-LD) context.
`@id` is a keyword to reference a resource and thus creates a named node for the subject of triple(s).
Note 1: we added "http://example.org/houses/id/" as base URI for houses and "http://example.org/" for the predicate. 
This should be replaced with a real location on the Web that can return more information about those things (cfr. dereferencing).
Note 2: myNeighboursHouse also has a `@id`. This way, a JSON-LD parser will know that myNeighboursHouse is a resource and not a literal.
```
{
  "@id": "http://example.org/houses/id/myHouse",
  "http://example.org/hasHouseNumber": "14",
  "http://example.org/hasNeighbouringHouse": {
    "@id": "http://example.org/houses/id/myNeighboursHouse"
  }
}
```
You can try the above example in the [JSON-LD playground](https://json-ld.org/playground/#startTab=tab-table&json-ld=%7B%22%40id%22%3A%22http%3A%2F%2Fexample.org%2Fhouses%2Fid%2FmyHouse%22%2C%22http%3A%2F%2Fexample.org%2FhasHouseNumber%22%3A%2214%22%2C%22http%3A%2F%2Fexample.org%2FhasNeighbouringHouse%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fexample.org%2Fhouses%2Fid%2FmyNeighboursHouse%22%7D%7D). Click on "Table" to retrieve an overview of parsed triples.

To ease the readability of this example, we can add a JSON-LD context (`@context`) that allows defining mappings for JSON terms.
We created a mapping `houses` so we can compact the URIs of myHouse and myNeighboursHouse with the prefix `houses`.
We also created a mapping for the predicates "hasHouseNumber" and "hasNeighbouringHouse", which don't need to be used as a prefix.
We used an expanded mapping to not only map to a URI with `@id`, but also indicate what datatype it is using [XSD datatypes](https://www.w3.org/TR/xmlschema11-2/#built-in-primitive-datatypes) or if its a resource "@type": "@id".

```
{
  "@context": {
    "houses": "http://example.org/houses/id/",
    "hasHouseNumber": {
      "@id": "http://example.org/hasHouseNumber",
      "@type": "http://www.w3.org/2001/XMLSchema#string"
    },
    "hasNeighbouringHouse": {
      "@id": "http://example.org/hasNeighbouringHouse",
      "@type": "@id"
    }
  },
  "@id": "houses:myHouse",
  "hasHouseNumber": "14",
  "hasNeighbouringHouse": "houses:myNeighboursHouse"
}
```

You can try the above example in the [JSON-LD playground](https://json-ld.org/playground/#startTab=tab-table&json-ld=%7B%22%40context%22%3A%7B%22houses%22%3A%22http%3A%2F%2Fexample.org%2Fhouses%2Fid%2F%22%2C%22hasHouseNumber%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fexample.org%2FhasHouseNumber%22%2C%22%40type%22%3A%22http%3A%2F%2Fwww.w3.org%2F2001%2FXMLSchema%23string%22%7D%2C%22hasNeighbouringHouse%22%3A%7B%22%40id%22%3A%22http%3A%2F%2Fexample.org%2FhasNeighbouringHouse%22%2C%22%40type%22%3A%22%40id%22%7D%7D%2C%22%40id%22%3A%22houses%3AmyHouse%22%2C%22hasHouseNumber%22%3A%2214%22%2C%22hasNeighbouringHouse%22%3A%22houses%3AmyNeighboursHouse%22%7D). Click on "Table" to retrieve an overview of parsed triples.


