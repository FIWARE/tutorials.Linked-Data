[![FIWARE Banner](https://jason-fox.github.io/tutorials.Linked-Data/img/Fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-ld-red.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/004/01.01.01_60/gs_CIM004v010101p.pdf)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial introduces linked data concepts to the FIWARE Platform. The supermarket chain’s store finder application
is recreated using **NGSI-LD** the differences between the **NSGI v2** and **NGSI-LD** interfaces are highlighted and
discussed. The tutorial is a direct analogue of the original getting started tutorial but uses API calls from the
**NGSI-LD** interface.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://jason-fox.github.io/tutorials.Linked-Data/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/d6671a59a7e892629d2b)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Architecture](#architecture)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Docker Compose (Optional)](#docker-compose-optional)
-   [Starting the containers](#starting-the-containers)
    -   [Option 1) Using Docker commands directly](#option-1-using-docker-commands-directly)
    -   [Option 2) Using Docker Compose](#option-2-using-docker-compose)
-   [Creating your first "Powered by FIWARE" app](#creating-your-first-powered-by-fiware-app)
    -   [Checking the service health](#checking-the-service-health)
    -   [Creating Context Data](#creating-context-data)
        -   [Data Model Guidelines](#data-model-guidelines)
        -   [Attribute Metadata](#attribute-metadata)
    -   [Querying Context Data](#querying-context-data)
        -   [Obtain entity data by ID](#obtain-entity-data-by-id)
        -   [Obtain entity data by type](#obtain-entity-data-by-type)
        -   [Filter context data by comparing the values of an attribute](#filter-context-data-by-comparing-the-values-of-an-attribute)
        -   [Filter context data by comparing the values of a geo:json attribute](#filter-context-data-by-comparing-the-values-of-a-geojson-attribute)
-   [Next Steps](#next-steps)
    -   [Iterative Development](#iterative-development)

</details>

# Adding Linked Data concepts to FIWARE Data Entities.

The introduction to FIWARE [Getting Started tutorial](https://github.com/FIWARE/tutorials.Getting-Started) introduced
the [NSGI v2](https://jason-fox.github.io/specifications/OpenAPI/ngsiv2) interface that is commonly used to create and
manipulate context data entities. An evolution of that interface has created a supplementary specification called
[NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
as a mechanism to enhance context data entities though adding the concept of **linked data**. This tutorial will
introduce the background of the ideas behind the new interface and compare and contrast how to create and manipulate
data entites as linked data.

Addtional tutorials in the series will further discuss data relationships an how to create context data entities using
linked data enabling the full knowledge graph to be traversed.

## What is Linked Data?

All users of the Internet will be familiar with the concept of hypertext links, the way that a link on one web page is
able to guide the browser to loading another page from a known location.

Whilst humans are able to understand relationship discoverability and how links work, computers find this much more
difficult, and require a well-defined protocol to be able to traverse from one data element to another held in a
separate location.

Creating a system of readable links for computers requires the use of a well defined data format
([JSON-LD](http://json-ld.org/)) and assignation of unique IDs (URNs) for both data entities and the relationships
between entities so that semantic meaning can be programmatically retrieved from the data itself.

Properly defined linked data can be used to help answer big data questions, and the data relationships can be traversed
to answer questions like _"Which products are currently avaiable on the shelves of Store X and what prices are they sold
at?"_

### :arrow_forward: Video: What is Linked Data?

[![](http://img.youtube.com/vi/4x_xzT5eF5Q/0.jpg)](https://www.youtube.com/watch?v=4x_xzT5eF5Q "Introduction")

Click on the image above to watch an introductory video on linked data concepts

JSON-LD is an extension of JSON , it is a standard way of avoiding ambiguity when expressing linked data in JSON so that
the data is structured in a format which is parsable by machines. It is a method of ensuring that all data attributes
can be easily compared when coming from a multitude of separate data sources, which could have a different idea as to
what each attribute means. For example, when two data entities have a `name` attribute how can the computer be certain
that is refers to a _"Name of a thing"_ in the same sense (rather than a **Username** or a **Surname** or something).
URLs and datamodels are used to remove ambiguity by allowing attributes to have a both short form (such as `name`) and a
fully specified long form (such `http://schema.org/name`) which means it is easy to discover which attribute have a
common meaning within a data structure.

JSON-LD introduces the concept of the `@context` element which provides additional information allowing the computer to
interpret the rest of the data with more clarity and depth.

Furthermore the JSON-LD specification enables you to define a unique `@type` associating a well-defined
[data model](https://fiware-datamodels.readthedocs.io/en/latest/guidelines/index.html) to the data itself.

## :arrow_forward: Video: What is JSON-LD?

[![](http://img.youtube.com/vi/vioCbTo3C-4/0.jpg)](https://www.youtube.com/watch?v=vioCbTo3C-4 "JSON-LD")

Click on the image above to watch a video describing the basic concepts behind JSON-LD.

## What is NGSI-LD?

**NGSI-LD** is an evolution of the **NGSI v2** information model, which has been modified to improve support for linked
data (entity relationships), property graphs and semantics (exploiting the capabilities offered by JSON-LD). This work
has been conducted under the ETSI ISG CIM initiative and the updated specification hhas been branded as
[NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf). The main constructs
of NGSI-LD are: _Entity_, _Property_ and _Relationship_. NGSI-LD Entities (instances) can be the subject of Properties
or Relationships. In terms of the traditional NGSI v2 data model, Properties can be seen as the combination of an
attribute and its value. Relationships allow to establish associations between instances using linked data.

### NGSI v2 Data Model

As a reminder, the NGSI v2 data model is quite simple. It can be summarized as shown below:

![](https://jason-fox.github.io/tutorials.Linked-Data/img/ngsi-v2.png)

The core element of NGSI v2 is the data _entity_, typically a real object with a changing state (such as a **Store**, a
**Shelf** and so on) Entities have _attributes_ (such as `name` and `location`) and these in turn hold _metadata_ such
as `accuracy` - i.e. the accuracy of a `location` reading.

Every _entity_ must have a `type` which defines the sort of thing the entity describes, but giving an NGSI v2 entity the
`type=Store` is relatively meaningless as no-one is obliged to give shape their own **Store** entities in the same
fashion. Similarly adding a `name` attribute doesn't suddenly make it hold the same data as someone else's `name`
attribute.

Relationships can be defined using NGSI v2, but only so far as giving the attribute a well-defined attribute name (by
convention starting with `ref`, such as `refManagedBy`) and defining the attribute `type=Relationship` which again is
purely a naming convention with no real weight.

### NGSI LD Data Model

The NGSI LD data model is more complex, with more rigid definitions of use which lead to a navigable knowledge graph.

![](https://jason-fox.github.io/tutorials.Linked-Data/img/ngsi-ld.png)

Once again, _entity_ can be considered to be the core element. Every entity must use a unique '`id`
[URN](https://en.wikipedia.org/wiki/Uniform_resource_name), there is also a `type`, to define the structure of the data
held, which is a URN, whcih should correspond to a well-defined data model which can be found on the web. For example
the URN `https://uri.fiware.org/ns/datamodels/Building` is used to define common data model for a
[building](https://fiware-datamodels.readthedocs.io/en/latest/Building/Building/doc/spec/index.html).

_Entities_ can have _properties_ and _relationships_. Ideally the name of each _property_ should be a well defined URN
which corresponds to a common concept found across the web (e.g. `http://schema.org/address` is a common URN for the
physical address of an item). The _property_ will also have a value which will reflect the state of that property.
Finally a property may itself have further properties (a.k.a. _properties-of-properties_) which reflect furthe
information about the property itself. Properties and relationships may in turn have a linked embedded structure (of
_properties-of-properties_ or _properties-of-relationships or relationships-of-properties_ or
_relationships-of-relationships_) which lead to the following:

An NGSI LD Data Entity (e.g. a supermarket):

-   Has an `id`: e.g. `urn:ngsi-ld:Building:store001`, which must be unique
-   Has `type` which should be a fully qualified URN of a well defined data model : e.g.
    `https://uri.fiware.org/ns/datamodels/Building`
-   Has an `address` which is a _property_ of the entity. This can be expanded into `http://schema.org/address`, which
    is a fully qualified name,
-   Has _value_ corresponding to the _property_ `address` (e.g. _Bornholmer Straße 65, 10439 Prenzlauer Berg, Berlin_
-   Has a _property-of-a-property_ (e.g. `verified` field for an `address`)
-   Has a _relationship_ `managedBy`, where the relationship `managedBy` corresponds to another data entity :
    `urn:ngsi-ld:Person:bob-the-manager`
-   The relationship `managedBy`, may itself have a _property-of-a-relationship_ (e.g. `since`), this holds the date Bob
    started working.
-   The relationship `managedBy`, may itself have a _relationship-of-a-relationship_ (e.g. `subordinateTo`), this holds
    the URN of the area manager above Bob in the hierarchy.

As you can see the knowledge graph is well defined and can be expanded indefinitely.

Relationships will be dealt with in more detail in a subsequent tutorial.

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Identity-Management/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

The demo application will send and receive NGSI-LD calls to a compliant context broker. Since both NSGI v2 and NSGI-LD
interfaces are available to an experimental version fo the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), our demo application will only make use of one
FIWARE component.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data it holds. Therefore, the architecture will consist of two elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations

Since all interactions between the two elements are initiated by HTTP requests, the entities can be containerized and
run from exposed ports.

![](https://jason-fox.github.io/tutorials.Linked-Data/img/architecture.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file:

```yaml
orion:
    image: fiware/orion-ld
    hostname: orion
    container_name: fiware-orion
    depends_on:
        - mongo-db
    networks:
        - default
    ports:
        - "1026:1026"
    command: -dbhost mongo-db -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion:1026/version || exit 1
```

```yaml
mongo-db:
    image: mongo:3.6
    hostname: mongo-db
    container_name: db-mongo
    expose:
        - "27017"
    ports:
        - "27017:27017"
    networks:
        - default
    command: --nojournal
```

Both containers are residing on the same network - the Orion Context Broker is listening on Port `1026` and MongoDB is
listening on the default port `27071`. Both containers are also exposing the same ports externally - this is purely for
the tutorial access - so that cUrl or Postman can access them without being part of the same network. The command-line
initialization should be self explanatory.

The only notable difference to the introductory tutorials is that the required image name is currently
`fiware/orion-ld`.

# Start Up

All services can be initialised from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Linked-Data/blob/master/services) Bash script provided within the
repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone git@github.com:FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data

./services start
```

This command will also import seed data from the previous [Store Finder tutorial](getting-started.md) on startup.

> **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---

# Creating a "Powered by FIWARE" app based on Linked Data

This tutorial recreates the same data entities as the intial _"Powered by FIWARE"_ supermarket finder app, but using
NGSI-LD linked data entities rather than NGSI v2.

## Checking the service health

As usual, you can check if the Orion Context Broker is running by making an HTTP request to the exposed port:

#### :one: Request:

```console
curl -X GET \
  'http://localhost:1026/version'
```

#### Response:

The response will look similar to the following:

```json
{
    "orion": {
        "version": "1.15.0-next",
        "uptime": "0 d, 3 h, 1 m, 51 s",
        "git_hash": "af440c6e316075266094c2a5f3f4e4f8e3bb0668",
        "compile_time": "Tue Jul 16 15:46:18 UTC 2019",
        "compiled_by": "root",
        "compiled_in": "51b4d802385a",
        "release_date": "Tue Jul 16 15:46:18 UTC 2019",
        "doc": "https://fiware-orion.readthedocs.org/en/master/"
    }
}
```

The format of the version response has not changed. The `release_date` must be 16th July 2019 or later to be able to
work with the requests defined below.

## Creating Context Data

When creating linked data entities, it is important to use common data models. This will allow us to easily combine data
from multiple sources

```text
TO DO
```

The `type=Store` example used in the getting....

At its heart, FIWARE is a system for managing context information, so lets add some context data into the system by
creating two new entities (stores in **Berlin**). Any entity must have a `id` and `type` attributes, additional
attributes are optional and will depend on the system being described. Each additional attribute should also have a
defined `type` and a `value` attribute.

#### :two: Request:

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "category": {
        "type": "Property",
        "value": ["commercial"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
             "coordinates": [13.3986, 52.5547]
        }
    },
    "name": {
        "type": "Property",
        "value": "Bösebrücke Einkauf"
    },
    "@context": [
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld",
        "https://schema.lab.fiware.org/ld/fiware-datamodels-context.jsonld"
    ]
}'
```

#### :three: Request:

Each subsequent entity must have a unique `id` for the given `type`

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities/ \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store002",
    "type": "Building",
    "category": {
        "type": "Property",
        "value": ["commercial"]
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
     "location": {
        "type": "GeoProperty",
        "value": {
             "type": "Point",
              "coordinates": [13.3903, 52.5075]
        }
    },
    "name": {
        "type": "Property",
        "value": "Checkpoint Markt"
    },
    "@context": [
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld",
        "https://schema.lab.fiware.org/ld/fiware-datamodels-context.jsonld"
    ]
}'
```

### Data Model Guidelines

Although the each data entity within your context will vary according to your use case, the common structure within each
data entity should be standardized order to promote reuse. The full FIWARE data model guidelines can be found
[here](https://fiware-datamodels.readthedocs.io/en/latest/guidelines/index.html). This tutorial demonstrates the usage
of the following recommendations:

#### All terms are defined in American English

Although the `value` fields of the context data may be in any language, all attributes and types are written using the
English language.

#### Entity type names must start with a Capital letter

In this case we only have one entity type - **Store**

#### Entity IDs should be a URN following NGSI-LD guidelines

NGSI-LD has recently been published as a full ETSI
[specification](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf), the proposal is
that each `id` is a URN follows a standard format: `urn:ngsi-ld:<entity-type>:<entity-id>`. This will mean that every
`id` in the system will be unique

#### Data type names should reuse schema.org data types where possible

[Schema.org](http://schema.org/) is an initiative to create common structured data schemas. In order to promote reuse we
have deliberately used the [`Text`](http://schema.org/PostalAddress) and
[`PostalAddress`](http://schema.org/PostalAddress) type names within our **Store** entity. Other existing standards such
as [Open311](http://www.open311.org/) (for civic issue tracking) or [Datex II](http://www.datex2.eu/) (for transport
systems) can also be used, but the point is to check for the existence of the same attribute on existing data models and
reuse it.

#### Use camel case syntax for attribute names

The `streetAddress`, `addressRegion`, `addressLocality` and `postalCode` are all examples of attributes using camel
casing

#### Location information should be defined using `address` and `location` attributes

-   We have used an `address` attribute for civic locations as per [schema.org](http://schema.org/)
-   We have used a `location` attribute for geographical coordinates.

#### Use GeoJSON for codifying geospatial properties

[GeoJSON](http://geojson.org) is an open standard format designed for representing simple geographical features. The
`location` attribute has been encoded as a geoJSON `Point` location.

### Attribute Metadata

Metadata is _"data about data"_, it is additionl data used to describe properties of the attribute value itself like
accuracy, provider, or a timestamp. Several built-in metadata attribute already exist and these names are reserved

-   `dateCreated` (type: DateTime): attribute creation date as an ISO 8601 string.
-   `dateModified` (type: DateTime): attribute modification date as an ISO 8601 string.
-   `previousValue` (type: any): only in notifications. The value of this
-   `actionType` (type: Text): only in notifications.

One element of metadata can be found within the `address` attribute. a `verified` flag indicates whether the address has
been confirmed.

## Querying Context Data

A consuming application can now request context data by making HTTP requests to the Orion Context Broker. The existing
NGSI interface enables us to make complex queries and filter results.

At the moment, for the store finder demo all the context data is being added directly via HTTP requests, however in a
more complex smart solution, the Orion Context Broker will also retrieve context directly from attached sensors
associated to each entity.

Here are a few examples, in each case the `options=keyValues` query parameter has been used shorten the responses by
stripping out the type elements from each attribute

### FNQ Type

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -d 'type=https://uri.fiware.org/ns/datamodels/Building'
```

### Obtain entity data by ID

This example returns the data of `urn:ngsi-ld:Store:001`

#### :four: Request:

```console
curl -G -X GET \
   'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001'
```

#### Response:

The response

```json
{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "https://uri.fiware.org/ns/datamodels/Building",
    "http://schema.org/address": {
        "type": "Property",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "verified": {
            "type": "Property",
            "value": true
        }
    },
    "http://schema.org/name": {
        "type": "Property",
        "value": "Bösebrücke Einkauf"
    },
    "https://uri.fiware.org/ns/datamodels/category": {
        "type": "Property",
        "value": ["commercial"]
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    "@context": "https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/defaultContext/defaultContext.jsonld"
}
```

### Obtain entity data by type

This example returns the data of all `Store` entities within the context data The `type` parameter limits the response
to store entities only.

#### :five: Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://schema.lab.fiware.org/ld/fiware-datamodels-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    'http://localhost:1026/v2/entities' \
    -d 'type=Building' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute `type` and
`metadata` elements.

```json
[
    {
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        },
        "@context": "https://schema.lab.fiware.org/ld/fiware-datamodels-context.jsonld"
    },
    {
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "@context": "https://schema.lab.fiware.org/ld/fiware-datamodels-context.jsonld"
    }
]
```

### Filter context data by comparing the values of an attribute

This example returns all stores with the `name` attribute _Checkpoint Markt_. Filtering can be done using the `q`
parameter - if a string has spaces in it, it can be URL encoded and held within single quote characters `'` = `%27`

#### :six: Request:

```console
curl -G -X GET \
    'http://localhost:1026/v2/entities' \
    -H 'Link: <https://schema.lab.fiware.org/ld/context>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
    -d 'type=Building' \
    -d 'q=name==%27Checkpoint%20Markt%27' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute `type` and
`metadata` elements.

```json
[
    {
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "@context": "https://schema.lab.fiware.org/ld/context"
    }
]
```

### Filter context data by comparing the values of an attribute in an Array

```console
curl -G -X GET \
    'http://localhost:1026/v2/entities' \
    -H 'Link: <https://schema.lab.fiware.org/ld/context>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
    -d 'type=Building' \
    -d 'q=category==%27commercial%27,%27office%27 \
    -d 'options=keyValues'
```

```json
[
    {
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        },
        "@context": "https://schema.lab.fiware.org/ld/context"
    },
    {
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "@context": "https://schema.lab.fiware.org/ld/context"
    }
]
```

### Filter context data by comparing the values of a sub-attribute

This example returns all stores found in the Kreuzberg District.

Filtering can be done using the `q` parameter - sub-attributes are annotated using the bracket syntax e.g.
`q=address[addressLocality]==Kreuzberg`

#### :seven: Request:

```console
curl -G -X GET \
    'http://localhost:1026/v2/entities' \
    -H 'Link: <https://schema.lab.fiware.org/ld/context>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
    -d 'type=Building' \
    -d 'q=q=address[addressLocality]==Kreuzberg' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute `type` and
`metadata` elements.

```json
[
    {
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": ["commercial"],
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "@context": "https://schema.lab.fiware.org/ld/context"
    }
]
```

### Filter context data by querying metadata

This example returns the data of all `Store` entities with a verified address.

Metadata queries (i.e. Properties of Properties) are annotated using the dot syntax e.g. `q=address.verified==true`

#### :eight: Request:

```console
curl -G -X GET \
    'http://localhost:1026/v2/entities' \
    -H 'Link: <https://schema.lab.fiware.org/ld/context>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
    -d 'type=Building' \
    -d 'mq=address.verified==true' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute `type` and
`metadata` elements.

```json
[
    {
        "id": "urn:ngsi-ld:Store:001",
        "type": "Store",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        },
        "name": "Bösebrücke Einkauf"
    },
    {
        "id": "urn:ngsi-ld:Store:002",
        "type": "Store",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "name": "Checkpoint Markt"
    }
]
```

### Filter context data by comparing the values of a geo:json attribute

This example return all Stores within 1.5km the **Brandenburg Gate** in **Berlin** (_52.5162N 13.3777W_)

#### :nine: Request:

```console
curl -G -X GET \
  'http://localhost:1026/v2/entities' \
  -H 'Link: <https://schema.lab.fiware.org/ld/context>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
  -d 'type=Building' \
  -d 'geometry=Point' \
  -d 'coordinates=[52.5162,13.3777]' \
  -d 'georel=within' \
  -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute `type` and
`metadata` elements.

```json
[
    {
        "id": "urn:ngsi-ld:Store:002",
        "type": "Store",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        },
        "name": "Checkpoint Markt"
    }
]
```

---

## License

[MIT](LICENSE) © 2019 FIWARE Foundation e.V.
