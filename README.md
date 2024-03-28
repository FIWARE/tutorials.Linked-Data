# Linked Data[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf)[<img src="https://fiware.github.io/tutorials.Getting-Started/img/Fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/FIWARE/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)


This tutorial introduces the idea of using an existing **NGSI-v2** Context Broker as a Context Source within an **NGSI-LD** data space, and how to combine both JSON-based
and JSON-LD based context data into a unified structure through introducing a simple
**NGSI-LD** façade pattern with a fixed `@context`. The tutorial re-uses the data from
the  original **NGSI-v2** getting started tutorial but uses API calls from the
**NGSI-LD** interface.

**NGSI-LD** interface.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://fiware.github.io/tutorials.Linked-Data/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/125db8d3a1ea3dab8e3f)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-LD)


> [!NOTE]
>  This tutorial is designed for exisiting **NGSI-v2** developers looking to attach existing
>  **NGSI-v2** systems to an **NGSI-LD** federation.
>  if you are building a linked data system from scratch you should start from the beginnning
>   of the [NGSI-LD developers tutorial](https://ngsi-ld-tutorials.readthedocs.io/) documentation.
>
> Similarly, if you an existing **NGSI-v2** developer and you want to understand linked
> data concepts, checkout the equivalent [**NGSI-v2** tutorial](https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-v2) on upgrading **NGSI-v2** data sources to **NGSI-LD**

## Contents



# Adding Linked Data concepts to FIWARE Data Entities.

> “Always be a first rate version of yourself and not a second rate version of someone else.”
>
> ― Judy Garland

The first introduction to FIWARE [Getting Started tutorial](https://github.com/FIWARE/tutorials.Getting-Started) introduced
the [NGSI v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) JSON-based interface that is commonly used to create and
manipulate context data entities. [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json) is an evolution of that interface which enhances context data entities through adding the concept of **linked data**.

There is a need for two separate interfaces since:

-  **NGSI-v2** offers [JSON](https://www.json.org/json-en.html) based interoperability used in individual Smart Systems
-  **NGSI-LD** offers [JSON-LD](https://json-ld.org/) based interoperability used for Federations and Data Spaces

**NGSI-v2** is ideal for creating individual applications offering interoperable interfaces for web services
or IoT devices. It is easier to understand than NGSI-LD and does not require a JSON-LD `@context`, However **NGSI-v2** without linked data falls short when attempting to offer federated data across a
data space. Once multiple apps and organisations are involved each individual data owner is no longer in
control of the structure of the data used internally by the other participants within the data space. This is where
the `@context` of **NGSI-LD** comes in, acting as a mapping mechanism for attributes allowing the each local system to continue to use its own preferred terminology within the data it holds and for federated data from other users within the data space to be renamed using a standard expansion/compaction operation allowing each individual system understand data  holistically from across the whole data space.



## Creating a common data space


For example. Imagine a simple "Buildings" data space consisting of two participants sharing their data together:

-  Details of a series of [supermarkets](https://fiware-tutorials.readthedocs.io/en/latest/)
-  Details of a series of [farms](https://ngsi-ld-tutorials.readthedocs.io/en/latest/)

Although the two participants have agreed to use a common [data model](https://ngsi-ld-tutorials.readthedocs.io/en/latest/datamodels.html#building) between them, internally
they hold their data in a slightly different structure.


#### Farm (**NGSI-LD**)

Within the **NGSI-LD** Smart Farm, all of the Building Entities are marked using `"type":"Building"` as a shortname which can be expanded to a full URI https://uri.fiware.org/ns/dataModels#Building` using JSON-LD expansion rules. All the attributes of each Building entity
(such as `name`, `category` etc
are defined using the [User `@context`](./data-models/ngsi-context.jsonld) and are structured as NGSI-LD attributes.
The standard NGSI-LD Keywords are used to define the nature of each attribute - e.g. `Property`,
`GeoProperty`, `VocabularyProperty`, `Relationship` and each of these terms is also defined within the [core `@context`](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld).



```json
{
    "id": "urn:ngsi-ld:Building:farm001",
    "type": "Building",
    "category": {
        "type": "VocabularyProperty",
        "vocab": "farm"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Großer Stern 1",
            "addressRegion": "Berlin",
            "addressLocality": "Tiergarten",
            "postalCode": "10557"
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3505, 52.5144]
        }
    },
    "name": {
        "type": "Property",
        "value": "Victory Farm"
    },
    "owner": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Person:person001"
    }
}
```

#### Supermarket (**NGSI-v2**)

Within the **NGSI-v2** Smart Supermarket, every entity must have a `id` and `type` - this is part of the data model and a prerequisite for joining the data space.
However due to the backend systems used, the internal short name is `"type":"Store"`. Within an **NGSI-v2** system there is no concept of `@context` - every attribute has a `type` and a `value` and the attribute `type` is usually aligned with the datatype (e.g. `Text`, `PostalAddress`) although since **NGSI-LD** keyword types such as `Relationship` `VocabularyProperty` are also permissible and set by convention.

```json
{
    "id": "urn:ngsi-ld:Store:001",
    "type": "Store",
    "address": {
        "type": "PostalAddress",
        "value": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        }
    },
    "category": {
        "type": "VocabularyProperty",
        "value": "supermarket"
    },
    "location": {
        "type": "geo:json",
        "value": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    "name": {
        "type": "Text",
        "value": "Bösebrücke Einkauf"
    },
    "owner": {
        "type": "Relationship",
        "value": "urn:ngsi-ld:Person:person001"
    }
}
```


### Types of data space

When joining a data space, a participant must abide by the rules that govern that data space. One of the first decisions a common data space must make is to defined the nature of the data space itself. There are three primary approaches to this, which can broadly be defined as follows:

- An **Integrated** data space requires that every participant uses exactly the same payloads and infrastructure - _e.g. "Use Scorpio 4.1.15 only"_ . This could be a requirement for a lottery ticketing system where every terminal sends ticket data in the same format.
- A **unified** data space defines a common data format, but allows for the underlying infrastructure to differ between participants - _e.g. "Use NGSI-LD only for all payloads"_
- A **federated** data space is even looser and defines a common meta structure so each participants has more flexibility regarding it underlying technologies  - _e.g. "All payloads must be translatable as GeoJSON"_ for a combined GIS, NGSI-LD data space.

Using this terminology, in this example we are creating a **unified** data space in this example, since participants are using NGSI-LD for data exchange, but their underlying systems are different.




# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Linked-Data/master/docker-compose/orion-ld.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)


# Architecture

The demo application will send and receive NGSI-LD calls within a data space. This demo application will only make use of three
FIWARE components.

Currently, both Orion and Orion-LD Context Brokers rely on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data they hold. Therefore, the architecture will consist of two elements:

-   The [Smart Farm Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   The [Smart Supermarket Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-v2](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
-   Two is instances of underlying [MongoDB](https://www.mongodb.com/) database
     -   Used by both NGSI-v2 and NGSI-LD Context Brokers to hold context data information such as data entities, subscriptions and registrations
-   The FIWARE [Lepus](https://github.com/jason-fox/lepus) proxy which will translate NGSI-LD to NGSI-v2 and vice-versa
-   An HTTP **Web-Server** which offers static `@context` files defining the context entities within the system.


Since all interactions between the two elements are initiated by HTTP requests, the elements can be containerized and
run from exposed ports.

![](https://fiware.github.io/tutorials.Linked-Data/img/architecture-lepus.png)

The necessary configuration information can be seen in the services section of the associated `common.yml` and `orion-ld.yml` files:

#### Orion-LD (NGSI-LD)

```yaml
orion-ld:
    image: quay.io/fiware/orion-ld
    hostname: orion
    container_name: fiware-orion
    depends_on:
        - mongo-db
    networks:
        - default
    ports:
        - '1026:1026'
    command: -dbhost mongo-db -logLevel DEBUG
    healthcheck:
        test: curl --fail -s http://orion:1026/version || exit 1
```

#### Orion (NGSI-v2)

```yaml
orion:
    image: quay.io/fiware/orion
    hostname: orion2
    container_name: fiware-orion-v2
    depends_on:
      - mongo-for-orion-v2
    networks:
      - default
    ports:
      - "1027:1026" # localhost:1026
    command: -dbhost mongo-db2 -logLevel DEBUG
    healthcheck:
      test: curl --fail -s http://orion2:1026/version || exit 1
```

#### Lepus

```yaml
lepus:
    image: quay.io/fiware/lepus
    hostname: adapter
    container_name: fiware-lepus
    networks:
      - default
    expose:
      - "3005"
    ports:
      - "3005:3000"
    environment:
      - DEBUG=adapter:*
      - NGSI_V2_CONTEXT_BROKER=http://orion2:1026
      - USER_CONTEXT_URL=http://context/fixed-context.jsonld
      - CORE_CONTEXT_URL=https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.8.jsonld
      - NOTIFICATION_RELAY_URL=http://adapter:3000/notify

```


All containers are residing on the same network - the Orion-LD Context Broker is listening on Port `1026` and the first MongoDB is
listening on the default port `27017`. The Orion NGSI-v2 Context Broker is listening on Port `1027` and second MongoDB is
listening on port `27018`. Lepus uses port `3000`, but since that clashes with the the tutorial application, it has been amended externally to `3005`.


# Start Up

All services can be initialised from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Linked-Data/blob/NGSI-v2/services) Bash script provided within the
repository. Please clone the repository and create the necessary images by running the commands as shown:

```bash
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
git checkout NGSI-LD

./services orion|scorpio|stellio
```

> [!NOTE]
> If you want to clean up and start over again you can do so with the following command:
>
> ```
> ./services stop
> ```

---

# Creating a "Powered by FIWARE" app based on Linked Data

This tutorial recreates the same data entities as the initial _"Powered by FIWARE"_ supermarket finder app, but using
NGSI-LD linked data entities rather than NGSI v2.

## Checking the service health

As usual, you can check if the Orion Context Broker is running by making an HTTP request to the exposed port:

#### 1️⃣ Request:

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
from multiple sources and remove ambiguity when comparing data coming from different sources.

Creating linked data using fully qualified names throughout would be painful, as each attribute would need to be a URI,
so JSON-LD introduces the idea of an `@context` attribute which can hold pointers to context definitions. To add a Smart
Data [Building](https://github.com/smart-data-models/dataModel.Building) data entity, the following `@context` would be
required

```json
{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    ...  other data attributes
    "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld"

}
```

### Core Context

[https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld)
refers to the Core `@context` of NGSI-LD, this defines terms such as `id` and `type` which are common to all NGSI
entities, as well as defining terms such as `Property` and `Relationship`. The core context is so fundamental to
NGSI-LD, that it is added by default to any `@context` sent to a request.

### Smart Data Models

[https://smart-data-models.github.io/dataModel.Building/context.jsonld](https://schema.lab.fiware.org/ld/context) refers to a User `@context` - a definition of
a standard data models. Adding this to the `@context` will load a common definition of a **Building**
[data model](https://smartdatamodels.org/) defined by the FIWARE Foundation in collaboration with other organizations
such as GSMA or TM Forum. A summary of the FQNs related to **Building** can be seen below:

```json
{
    "@context": {
        "Building": "https://smartdatamodels.org/dataModel.Building/Building",
        ... etc

        "address": "https://smartdatamodels.org/address",
        "addressCountry": "https://smartdatamodels.org/addressCountry",
        "addressLocality": "https://smartdatamodels.org/addressLocality",
        "addressRegion": "https://smartdatamodels.org/addressRegion",
        "category": "https://smartdatamodels.org/dataModel.Building/category",
        "name": "https://smartdatamodels.org/name",
        ...etc
    }
}
```

If we include this context definition, it means that we will be able to use short names for `Building`, `address`,
`name` for our entities, but computers will also be able to read the FQNs when comparing with other sources.

#### Context terms are IRIs not URLs

It should be noted that According to the [JSON-LD Spec](https://www.w3.org/TR/json-ld/#the-context) : _"a context is
used to map terms to IRIs."_ - An IRI (Internationalized Resource Identifier) is not necessarily a URL - see
[here](https://fusion.cs.uni-jena.de/fusion/blog/2016/11/18/iri-uri-url-urn-and-their-differences/) and therefore it is
not unexpected if elements such as `https://smartdatamodels.org/name` do not actually resolve to a web page. However
many IRIs within JSON-LD `@context` files, such as `http://schema.org/address` do indeed return web pages with more
information about themselves.

If you take the NGSI-LD [Core @context](https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld)

```json
{
  "@context": {

    "ngsi-ld": "https://uri.etsi.org/ngsi-ld/",
    "geojson": "https://purl.org/geojson/vocab#",
    "id": "@id",
    "type": "@type",
...
    "@vocab": "https://uri.etsi.org/ngsi-ld/default-context/"
  }
}
```

You can see that any unresolved short-name for an attribute will be mapped onto the default context i.e.:

-   Unknown attribute `xxx` => `https://uri.etsi.org/ngsi-ld/default-context/xxx`

And unsurprisingly these default-context IRIs don't exist as valid web pages either.

To create a valid **Building** data entity in the context broker, make a POST request to the
`http://localhost:1026/ngsi-ld/v1/entities` endpoint as shown below. It is essential that the appropriate
`Content-Type: application/ld+json` is also used, so that the data entity is recognized as Linked data.

#### 2️⃣ Request:

```console
curl -iX POST \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store001",
    "type": "Building",
    "category": {
        "type": "VocabularyProperty",
        "vocab": "commercial"
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
        "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld"
    ]
}'
```

The first request will take some time, as the context broker must navigate and load all of the files mentioned in the
`@context`.

> [!NOTE]
>  if `https://smart-data-models.github.io/dataModel.Building/context.jsonld` is unavailable for some reason the request will
> fail
>
> For a working production system it is essential that the `@context` files are always available to ensure third parties
> can read the context. High availability infrastructure has not been considered for this tutorial to keep the
> architecture simple.

#### 3️⃣ Request:

Each subsequent entity must have a unique `id` for the given `type`

```console
curl -iX POST \
  http://localhost:1026/ngsi-ld/v1/entities/ \
  -H 'Content-Type: application/ld+json' \
  -d '{
    "id": "urn:ngsi-ld:Building:store002",
    "type": "Building",
    "category": {
        "type": "VocabularyProperty",
        "vocab": "commercial"
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
        "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld"
    ]
}'
```

### Defining Properties within the NGSI-LD entity definition

The attributes `id` and `type` should be familiar to anyone who has used NGSI v2, and these have not changed. As
mentioned above, the type should refer to an included data model, in this case `Building` is being used as a short name
for the included URN `https://uri.fiware.org/ns/dataModels#Building`. Thereafter each _property_ is defined as a JSON
element containing two attributes, a `type` and a `value`.

The `type` of a _property_ attribute must be one of the following:

-   `"GeoProperty"`: `"http://uri.etsi.org/ngsi-ld/GeoProperty"` for locations. Locations should be specified as
    Longitude-Latitude pairs in [GeoJSON format](https://tools.ietf.org/html/rfc7946). The preferred name for the
    primary location attribute is `location`
-   `"VocabularyProperty"` holds enumerated values and is a mapping of a URI to a value within the user'`@context`
-   `"LanguageProperty"` holds a set of internationalized strings.
-   `"Property"`: `"http://uri.etsi.org/ngsi-ld/Property"` - for everything else.
-   For time-based values, `"Property"` shall be used as well, but the property value should be Date, Time or DateTime
    strings encoded in the [ISO 8601 format](https://en.wikipedia.org/wiki/ISO_8601) - e.g. `YYYY-MM-DDThh:mm:ssZ`

> [!NOTE]
> Note that for simplicity, this data entity has no relationships defined. Relationships must be given the
> `type=Relationship` or one of its defined subtypes. Relationships will be discussed in a subsequent tutorial.

### Defining Properties-of-Properties within the NGSI-LD entity definition

_Properties-of-Properties_ is the NGSI-LD equivalent of metadata (i.e. _"data about data"_), it is use to describe
properties of the attribute value itself like accuracy, provider, or the units to be used. Some built-in metadata
attributes already exist and these names are reserved:

-   `createdAt` (type: DateTime): attribute creation date as an ISO 8601 string.
-   `modifiedAt` (type: DateTime): attribute modification date as an ISO 8601 string.

Additionally `observedAt`, `datasetId` and `instanceId` may optionally be added in some cases, and `location`,
`observationSpace` and `operationSpace` have special meaning for Geoproperties.

In the examples given above, one element of metadata (i.e. a _property-of-a-property_) can be found within the `address`
attribute. a `verified` flag indicates whether the address has been confirmed. The commonest _property-of-a-property_ is
`unitCode` which should be used to hold the UN/CEFACT
[Common Codes](http://wiki.goodrelations-vocabulary.org/Documentation/UN/CEFACT_Common_Codes) for Units of Measurement.

> [!NOTE]
> A `valueType` _property-of-a-property_ can be used to describe the **Datatype** of an attribute.
>
> -  For Native JSON Properties, the `valueType` can align with a well-known **Datatype** schema, such as [schema.org](https://schema.org/DataType)  or
>    [XML Schema](https://www.w3.org/TR/2004/REC-xmlschema-2-20041028/) - typically values such as: `Time`, `Boolean`, `DateTime`, `Number`,
>    `Text`, `Date`, `Float`, `Integer` etc.
> -  For other JSON Objects, where possible use a datatype from an existing ontology - for example `PostalAddress` aligns with `https://schema.org/PostalAddress`.
>
> When using `valueType`, the mapping of the given short name value to a full URI should be placed in the User `@context`


## Querying Context Data

A consuming application can now request context data by making NGSI-LD HTTP requests to the Orion Context Broker. The
existing NGSI-LD interface enables us to make complex queries and filter results and retrieve data with FQNs or with
short names.

### Obtain entity data by FQN Type

This example returns the data of all `Building` entities within the context data The `type` parameter is mandatory for
NGSI-LD and is used to filter the response. The Accept HTTP header is needed to retrieve JSON-LD content.

#### 4️⃣ Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -H 'Accept: application/ld+json' \
  -d 'type=https%3A%2F%2Fsmartdatamodels.org%2FdataModel.Building%2FBuilding'
```

#### Response:

The response returns the Core `@context` by default (`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld`) and
all attributes are expanded whenever possible.

-   `id`, `type` and `location` are defined in the core context and are not expanded.
-   `address` has been mapped to `http://smartdatamodels.org/address`
-   `name` has been mapped to `http://smartdatamodels.org/name`
-   `category` has been mapped to `https://smartdatamodels.org/dataModel.Building/category`

Note that if an attribute has not been not associated to an FQN when the entity was created, the short name will
**always** be displayed - `verified` and `commercial` are examples of this since they were missing from the supplied user `@context` when inserting the context data.

```json
[
    {
        "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld",
        "id": "urn:ngsi-ld:Building:store001",
        "type": "https://smartdatamodels.org/dataModel.Building/Building",
        "https://smartdatamodels.org/dataModel.Building/category": {
            "type": "VocabularyProperty",
            "vocab": "commercial"
        },
        "https://smartdatamodels.org/address": {
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
                "coordinates": [
                    13.3986,
                    52.5547
                ]
            }
        },
        "https://smartdatamodels.org/name": {
            "type": "Property",
            "value": "Bösebrücke Einkauf"
        }
    },
    {
        "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "https://smartdatamodels.org/dataModel.Building/Building",
        "https://smartdatamodels.org/dataModel.Building/category": {
            "type": "VocabularyProperty",
            "vocab": "commercial"
        },
        "https://smartdatamodels.org/address": {
            "type": "Property",
            "value": {
                "https://smartdatamodels.org/streetAddress": "Friedrichstraße 44",
                "https://smartdatamodels.org/addressRegion": "Berlin",
                "https://smartdatamodels.org/addressLocality": "Kreuzberg",
                "https://smartdatamodels.org/postalCode": "10969"
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
                "coordinates": [
                    13.3903,
                    52.5075
                ]
            }
        },
        "https://smartdatamodels.org/name": {
            "type": "Property",
            "value": "Checkpoint Markt"
        }
    }
]
```

### Obtain entity data by ID

This example returns the data of `urn:ngsi-ld:Building:store001`

#### 5️⃣ Request:

```console
curl -G -X GET \
   -H 'Accept: application/ld+json' \
   'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001'
```

#### Response:

The response returns the Core `@context` by default (`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld`) and
all attributes are expanded whenever possible.

```json
{
    "@context": "https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context-v1.6.jsonld",
    "id": "urn:ngsi-ld:Building:store001",
    "type": "https://smartdatamodels.org/dataModel.Building/Building",
    "https://smartdatamodels.org/dataModel.Building/category": {
        "vocab": "commercial"
    },
    "https://smartdatamodels.org/address": {
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
    "https://smartdatamodels.org/name": "Bösebrücke Einkauf"
}
```

### Obtain entity data by type

If a reference to the supplied data is supplied, it is possible to return short name data and limit responses to a
specific `type` of data. For example, the request below returns the data of all `Building` entities within the context
data. Use of the `type` parameter limits the response to `Building` entities only, use of the `options=keyValues` query
parameter reduces the response down to standard JSON-LD.

A [`Link` header](https://www.w3.org/wiki/LinkHeader) must be supplied to associate the short form `type="Building"`
with the FQN `https://uri.fiware.org/ns/data-models/Building`. The full link header syntax can be seen below:

```text
Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json
```

The standard HTTP `Link` header allows metadata (in this case the `@context`) to be passed in without actually touching
the resource in question. In the case of NGSI-LD, the metadata is a file in `application/ld+json` format.

#### 6️⃣ Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues`, the response consists of JSON only without the attribute definitions
`type="Property"` or any _properties-of-properties_ elements. You can see that `Link` header from the request has been
used as the `@context` returned in the response.

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

### Filter context data by comparing the values of an attribute

This example returns all `Building` entities with the `name` attribute _Checkpoint Markt_. Filtering can be done using
the `q` parameter - if a string has spaces in it, it can be URL encoded and held within double quote characters `"` =
`%22`.

#### 7️⃣  Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=name==%22Checkpoint%20Markt%22' \
    -d 'options=keyValues'
```

#### Response:

The `Link` header `https://smart-data-models.github.io/dataModel.Building/context.jsonld` includes the FIWARE Building model.

This means that use of the `Link` header and the `options=keyValues` parameter reduces the response to short form
JSON-LD as shown:

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

### Filter context data by comparing the values of an attribute in an Array

Within the standard `Building` model, the `category` attribute refers to an array of enumerated strings. This example returns all
`Building` entities with a `category` attribute which contains either `commercial` or `office` strings. Filtering can be
done using the `q` parameter, comma separating the acceptable values.

> [!NOTE]
>
> `category` has been defined as a **VocabularyProperty**, which would usually mean that the `vocab`
> value should be a URI defined in the `@context`. The `expandValues` hint indicates that URI
> expansion is required for the `category` attribute when querying the context data.

#### 8️⃣  Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=category==%22commercial%22,%22office%22' \
    -d 'options=keyValues' \
    -d 'expandValues=category'
```

#### Response:

The response is returned in JSON-LD format with short form attribute names:

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

### Filter context data by comparing the values of a sub-attribute

This example returns all stores found in the Kreuzberg District.

Filtering can be done using the `q` parameter - sub-attributes are annotated using the bracket syntax e.g.
`q=address[addressLocality]=="Kreuzberg"`. This differs from NGSI v2 where dot syntax was used.

#### 9️⃣ Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/ld+json' \
    -d 'type=Building' \
    -d 'q=address%5BaddressLocality%5D==%22Kreuzberg%22' \
    -d 'options=keyValues'
```

#### Response:

Use of the `Link` header and the `options=keyValues` parameter reduces the response to JSON-LD.

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

### Filter context data by querying metadata

This example returns the data of all `Building` entities with a verified address. The `verified` attribute is an example
of a _Property-of-a-Property_

Metadata queries (i.e. Properties of Properties) are annotated using the dot syntax e.g. `q=address.verified==true`.
This supersedes the `mq` parameter from NGSI v2.

#### 1️⃣0️⃣ Request:

```console
curl -G -X GET \
    'http://localhost:1026/ngsi-ld/v1/entities' \
    -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
    -H 'Accept: application/json' \
    -d 'type=Building' \
    -d 'q=address.verified==true' \
    -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues` together with the Accept HTTP header (`application/json`), the response
consists of JSON only without the attribute `type` and `metadata` elements.

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store001",
        "type": "Building",
        "address": {
            "streetAddress": "Bornholmer Straße 65",
            "addressRegion": "Berlin",
            "addressLocality": "Prenzlauer Berg",
            "postalCode": "10439"
        },
        "name": "Bösebrücke Einkauf",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3986, 52.5547]
        }
    },
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

### Filter context data by comparing the values of a geo:json attribute

This example return all Stores within 2km the **Brandenburg Gate** in **Berlin** (_52.5162N 13.3777W_). To make a
geo-query request, three parameters must be specified, `geometry`, `coordinates` and `georel`.

The syntax for NGSI-LD has been updated, the `coordinates` parameter is now represented in
[geoJSON](https://tools.ietf.org/html/rfc7946) including the square brackets rather than the simple lat-long pairs
required in NGSI v2.

Note that by default the geo-query will be applied to the `location` attribute, as this is default specified in NGSI-LD.
If another attribute is to be used, an additional `geoproperty` parameter is required.

#### 1️⃣1️⃣ Request:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities' \
  -H 'Link: <https://smart-data-models.github.io/dataModel.Building/context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
  -H 'Accept: application/json' \
  -d 'type=Building' \
  -d 'geometry=Point' \
  -d 'coordinates=%5B13.3777,52.5162%5D' \
  -d 'georel=near%3BmaxDistance==2000' \
  -d 'options=keyValues'
```

#### Response:

Because of the use of the `options=keyValues` together with the Accept HTTP header (`application/json`), the response
consists of JSON only without the attribute `type` and `metadata` elements.

```json
[
    {
        "@context": "https://smart-data-models.github.io/dataModel.Building/context.jsonld",
        "id": "urn:ngsi-ld:Building:store002",
        "type": "Building",
        "address": {
            "streetAddress": "Friedrichstraße 44",
            "addressRegion": "Berlin",
            "addressLocality": "Kreuzberg",
            "postalCode": "10969"
        },
        "name": "Checkpoint Markt",
        "category": {
            "vocab" :"commercial"
        },
        "location": {
            "type": "Point",
            "coordinates": [13.3903, 52.5075]
        }
    }
]
```

---

## License

[MIT](LICENSE) © 2019-2024 FIWARE Foundation e.V.
