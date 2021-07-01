---
title: "StoRM Frontend"
linkTitle: "Frontend"
noComment: true
description: >
  The Frontend component exposes the SRM web service interface, manages user authentication and stores the data of the SRM requests into the database. It relies on the GSOAP framework to expose the SRM interface and it uses the CGSI-GSOAP plugin to manage secure connection with clients.
---

### Pool of worker threads

The FE uses a pool of worker threads to manage SRM requests. Once a request has been authorized, the FE assigns it as a new task for a worker thread. In case there are no free threads in the system, the request is maintained in an internal queue. The size of the pool and the size of the queue are important parameters, their value have to be defined depending on hardware resources and performance required. Depending on the type of SRM request, each thread should have two main task to do, as explained in the next paragraph.

### XML-RPC communication

Synchronous SRM requests are a category of SRM calls that return the control to the client only when the request has been executed by the system. Most of the SRM call belongs to this category:

* Namespace operations (srmLs, srmMkdir, etc.)
* Discovery operation (srmPing)
* Space operations (srmReserveSpace, srmGetSpaceMetadata, etc.)

For this type of request, the FE perform a direct communication to the Backend using a RPC approach, based on the XML-RPC protocol. XML-RPC is a simple protocol to exchange XML structured data over HTTP. The Backend provides an XML-RPC server and the FE(s) acts as client. A worker threads in case of synchronous requests performs this steps:

* structure the SRM data in XML
* send a request to the BE XML-RPC server
* wait until the execution
* get result from XML and unmarshall it in SOAP
* return the control to the client

<img src="/images/fe-architecture.jpg" width="400" style="margin-top: 30px; margin-bottom: 30px;"/>
