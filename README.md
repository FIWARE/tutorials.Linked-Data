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
> Similarly, if you an exisitng **NGSI-v2** developer and you want to understand linked
> data concepts, checkout the equivalent [**NGSI-v2** tutorial](https://github.com/FIWARE/tutorials.Linked-Data/tree/NGSI-v2) on upgrading **NGSI-v2** data sources to **NGSI-LD**

---

## License

[MIT](LICENSE) © 2019-2024 FIWARE Foundation e.V.
