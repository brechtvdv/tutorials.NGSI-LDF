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
\<myHouse\> \<hasHouseNumber\> "14" .

In a RDF triple, you have three parts:
- the subject (\<myHouse\>)
- the predicate (\<hasHouseNumber\>)
- the object ("14")
  
In this example, the subject and predicate are `nodes` while the object is a `literal`.
It's also possible to have a node in the object, for example:
\<myHouse\> \<hasNeighbouringHouse\> \<myNeighoursHouse\> .

Nodes are the glue for Linked Data: other datasets can also reference \<myHouse\> when they want to state facts about my house (e.g. how good is the air quality on a working day). Also, it's important that datasets use the same reference for predicates, like \<hasHouseNumber\>, to allow automatic integration of different datasets.

There are two types of nodes: named nodes and blank nodes. 
`Named nodes` are resources that are referenced by an URI. This is an unique, global identifier on the Web and as best-practice it should be dereferenceable allowing others to look it up.
`Blank nodes` are unnamed resources and are locally identified within its datadump or Web document.

There are multiple syntaxes available to describe RDF (JSON-LD, Turtle, N-Quads...). In these tutorials, we will focus on the usage of JSON-LD.
JSON-LD uses the JSON syntax, but maps JSON terms to URIs through a (JSON-LD) context.
`@id` is a keyword to reference a resource and thus creates a named node for the subject of triple(s).

```
{
  "@id": "http://example.org/houses/id/1",
  "http://example.org/hasHouseNumber": "14",
  "http://example.org/hasNeighbouringHouse": {
    "@id": "http://example.org/houses/id/2"
  }
}
```

To ease the readability of this example, we can add a JSON-LD context that allows mappings between JSON terms and URIs, but also prefixes can be added.



### Schema.org

### NGSI-LD

### OSLO

### W3C

## Application Profiles

