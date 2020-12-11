# Understanding vocabularies

In the previous tutorial, we explained how two house instances can be related to eachother using RDF triples. However, the houses are not "classified" as a house. We are able to understand through the URI that `houses:myHouse` probably is a `House`, but for a machine this is only a Web adress. This is important for queries later on, e.g. "give me all the houses". Also, predicates, which are relations (`hasNeighbouringHouse`) or attributes, must be classified so machines and humans understand its meaning and are able to act upon it (such as `reasoning`).

In this tutorial, we will briefly explain the RDF Schema (RDFS) vocabulary and domain-specific vocabularies like Schema.org.

## RDFS

> Prefixes: **rdf** https://www.w3.org/1999/02/22-rdf-syntax-ns# and **rdfs** https://www.w3.org/2000/01/rdf-schema#

## Domain vocabularies


### W3C SSN
> Prefixes: **ssn**  and **sosa** 

### OSLO
> Prefixes: **oslo** https://www.data.vlaanderen.be/ns/

### Schema.org
> Prefixes: **schema** https://www.schema.org/ and **schema** http://www.schema.org/

### NGSI-LD
> Prefixes: **ngsi-ld**  and **fiware** 




