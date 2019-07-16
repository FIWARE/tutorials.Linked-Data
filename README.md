[![FIWARE Banner](https://fiware.github.io/tutorials.Linked-Data/img/Fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-ld-red.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/004/01.01.01_60/gs_CIM004v010101p.pdf)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial introduces linked data concepts to the FIWARE Platform. The supermarket chain’s store
finder application is recreated using **NGSI-LD** the differences between the **NSGI v2** and **NGSI-LD** interfaces are highlighted and discussed.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.Linked-Data/)

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


# Linked Data

##  :arrow_forward: Video: What is Linked Data?

[![](http://img.youtube.com/vi/4x_xzT5eF5Q/0.jpg)](https://www.youtube.com/watch?v=4x_xzT5eF5Q "Introduction")

##  :arrow_forward: Video: What is JSON-LD?

[![](http://img.youtube.com/vi/vioCbTo3C-4/0.jpg)](https://www.youtube.com/watch?v=vioCbTo3C-4 "JSON-LD")

##  :arrow_forward: Video: JSON-LD: Compaction and Expansion

[![](http://img.youtube.com/vi/Tm3fD89dqRE/0.jpg)](https://www.youtube.com/watch?v=Tm3fD89dqRE "JSON-LD: Compaction and Expansion")

##  :arrow_forward: Video: JSON-LD: Core Markup

[![](http://img.youtube.com/vi/UmvWk_TQ30A/0.jpg)](https://www.youtube.com/watch?v=UmvWk_TQ30A "JSON-LD: Core Markup")





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

Our demo application will only make use of one FIWARE component - the
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/). Usage of the Orion Context Broker is sufficient
for an application to qualify as _“Powered by FIWARE”_.

Currently, the Orion Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data it holds. Therefore, the architecture will consist of two elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
-   The underlying [MongoDB](https://www.mongodb.com/) database :
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations

Since all interactions between the two elements are initiated by HTTP requests, the entities can be containerized and
run from exposed ports.

![](https://fiware.github.io/tutorials.Linked-Data/img/architecture.png)

# Creating a "Powered by FIWARE" app based on Linked Data

## Checking the service health

You can check if the Orion Context Broker is running by making an HTTP request to the exposed port:

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
        "version": "1.12.0-next",
        "uptime": "0 d, 0 h, 3 m, 21 s",
        "git_hash": "e2ff1a8d9515ade24cf8d4b90d27af7a616c7725",
        "compile_time": "Wed Apr 4 19:08:02 UTC 2018",
        "compiled_by": "root",
        "compiled_in": "2f4a69bdc191",
        "release_date": "Wed Apr 4 19:08:02 UTC 2018",
        "doc": "https://fiware-orion.readthedocs.org/en/master/"
    }
}
```

## Creating Context Data

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
        "value": [
            "commercial"
        ]
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3986,
                52.5547
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
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
        "category": [
            "commercial"
        ],
        "location": {
            "type": "Point",
            "coordinates": [
                13.3903,
                52.5075
            ]
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
            "coordinates": [
                13.3986,
                52.5547
            ]
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
            "coordinates": [
                13.3903,
                52.5075
            ]
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
            "coordinates": [
                13.3903,
                52.5075
            ]
        },
        "name": "Checkpoint Markt"
    }
]
```

---

## License

[MIT](LICENSE) © 2019 FIWARE Foundation e.V.
