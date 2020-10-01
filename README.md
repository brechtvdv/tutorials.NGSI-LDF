# tutorials.NGSI-LDF
This is work in progress
Tutorials on publishing Linked Data Fragments on top of NGSI-LD

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A YAML file is used to
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

Open data reusers want to build their own APIs (prefix TREE, OGC APIs...), store the data in their own databases for faster lookups etc.

To harvest from a NGSI-LD API, either all entities must be queried or the temporal query endpoint can be used for filtering.
This requires server-side processing for every HTTP request that is send. Therefore, HTTP caching can not be applied and the NGSI-LD API is less suited for open data publishing.

With Linked Data Fragments, we investigate how we can create cacheable documents on top of a NGSI-LD endpoint and in such way that open data reusers can easily discover and harvest these documents. This is done by introducing the concept of **mutable** and **immutable** objects and using the event streams specification.

# Start Up

All services can be initialised from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Linked-Data/blob/NGSI-v2/services) Bash script provided within the
repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone https://github.com/brechtvdv/tutorials.NGSI-LDF.git
cd tutorials.NGSI-LDF

./services scorpio
```
TODO: add ngsi-ldf on top of scorpio

> **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---
# Setup a NGSI-LD Context Broker based on Linked Data

For this tutorial, we work with the Scorpio context broker, because this one supports temporal queries.
First, we will ingest some entities. Afterwards, we inspect these entities through the NGSI-LDF interface that runs on top of the context broker.

## Context

### NGSI-LD core context
[https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld)
refers to the Core `@context` of NGSI-LD, this defines terms such as `id` and `type` which are common to all NGSI
entities, as well as defining terms such as `Property` and `Relationship`. The core context is so fundamental to
NGSI-LD, that it is added by default to any `@context` sent to a request.

### Data standards

Next to the NGSI-LD core context, add or describe in the `@context` the JSON-LD context or URI mappings of other standards, such as W3C SOSA/SSN, FIWARE [data models](https://fiware-datamodels.readthedocs.io) or OSLO Application Profiles.

```jsonld
{
    "id": "http://example.org/observations/1",
    "type": "Observation",
    "createdAt": {
      "type": "Property",
      "value": "2020-02-14T12:00:00Z"
    },
    "modifiedAt": {
      "type": "Property",
      "value": "2020-02-15T12:00:00Z"
    },
    ...  other data attributes
    "@context": [{
        "Observation": "http://www.w3.org/ns/sosa/Observation"
      },
      "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
     ]
}
```

The context broker allows every entity, also this one `urn:ngsi-ld:Observation:1`, to be updated at anytime. In order that we can create cacheable fragments for open data reuse, we need objects to become immutable. To do this, we use the event streams specification (work in progress) that describes a stream of versioned objects. The latter is an object that contains the `dcterms:isVersionOf` and `prov:generatedAtTime` properties and will be published by the NGSI-LDF interface.

```jsonld
{
    "id": "http://example.org/observations/1?modifiedAt=2020-02-15T12:00:00Z",
    "type": "Observation",
    "createdAt": {
      "type": "Property",
      "value": "2020-02-15T12:00:00Z"
    },
    "modifiedAt": {
      "type": "Property",
      "value": "2020-02-15T12:00:00Z"
    },
    "dcterms:isVersionOf": {
      "type": "Relationship",
      "value": "http://example.org/observations/1"
    },
    "prov:generatedAtTime": {
      "type": "Property",
      "value": "2020-02-15T12:00:00Z"
    },
    ...  other data attributes
    "@context": [{
        "Observation": "http://www.w3.org/ns/sosa/Observation",
        "dcterms": "http://purl.org/dc/terms/",
        "prov": "http://www.w3.org/ns/prov#"
      },
      "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.3.jsonld"
     ]
}
```


#### :one: Request:

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities \
  -H 'Content-Type: application/ld+json' \
  -d '{
        "id": "",
        "type": "http://www.w3.org/ns/sosa/Observation",
        "http://www.w3.org/2002/07/owl#sameAs": {
            "type": "Relationship",
            "object": "http://localhost:9090/ngsi-ld/v1/entities/urn:ngsi-ld:observations:21"
        },
        "http://www.w3.org/ns/sosa/observedProperty": {
            "type": "Property",
            "value": "http://example.org#CO2"
        },
        "http://www.w3.org/ns/sosa/hasSimpleResult": {
            "type": "Property",
            "value": 22
        },
        "location": {
                "type": "GeoProperty",
                "value": {
                        "type": "Point",
                        "coordinates": [3.70944, 51.01328]
                }
        },
        "@context": [
            "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld"
        ]
}'
```
Scorpio has an issue with entity id's that are URLs. This is why we created a URN for the entity ID and owl:sameAs relation to its dereferenceable URI.

