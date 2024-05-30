[![FIWARE Banner](https://fiware.github.io/tutorials.Linked-Data/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)

The **NGSI-v2** tutorial introduces linked data concepts to the FIWARE Platform. For **NGSI-v2** developers, the supermarket chain’s store finder application
is recreated using **NGSI-LD** and the differences between the **NGSI-v2** and **NGSI-LD** interfaces are highlighted
and discussed. The tutorial is a direct analogue of the original getting started tutorial but uses API calls from the
**NGSI-LD** interface.

The **NGSI-LD** tutorial shows how to encapsulate an existing **NGSI-v2** data source (such as the Smart Supermarket buildings) and use registrations 
to allow an **NGSI-v2** source to federate within an **NGSI-LD** system using a fixed `@context`.

Both tutorials are available as [Postman documentation](https://www.postman.com/downloads/).

# Start-Up

## NGSI-LD Smart Supermarket

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this **NGSI-LD** tutorial for **NGSI-v2** developers, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-v2) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Linked-Data/) |
| --- | --- | --- |

## NGSI-LD Smart Farm

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this tutorial with **NGSI-LD**, use the `NGSI-LD` branch.

```console
git clone https://github.com/FIWARE/tutorials.Linked-Data.git
cd tutorials.Linked-Data
git checkout NGSI-LD

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-LD) | <img  src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Linked-Data/ngsi-ld.html) |
| --- | --- | --- |


---

## License

[MIT](LICENSE) © 2019-2024 FIWARE Foundation e.V.
