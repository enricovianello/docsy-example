---
title: "StoRM Architecture Overview"
linkTitle: "Overview"
weight: 1
noComment: true
description: >
  StoRM is a storage resource manager for disk based storage systems
---

StoRM is a lightweight storage resource manager (SRM) solution developed at [INFN](https://home.infn.it/it/), which powers the Italian Tier-1 data center at [INFN-CNAF](https://www.cnaf.infn.it/), as well as more than 30 other sites.

The SRM ("Storage Resource Manager") is a protocol for Storage Resource Management. The SRM protocol does not do any data transfer. The protocol is used to ask a Mass Storage System (MSS) to make a file ready for transfer, or to create space in a disk cache to which a file can be uploaded. The file is then transferred to or from a Transfer URL or TURL.

*Read more:*
- [The Storage Resource Manager Interface Specification Version 2.2](https://sdm.lbl.gov/srm-wg/doc/SRM.v2.2.html)

## StoRM typical deployment

StoRM implements the [SRM version 2.2](https://sdm.lbl.gov/srm-wg/doc/SRM.v2.2.html) data management specification and is typically deployed on top of a cluster file system like [IBM GPFS](https://www.ibm.com/docs/en/gpfs). StoRM has a layered architecture (Figure 1), split between two main components: the StoRM frontend and backend services. The StoRM frontend service implements the SRM interface exposed to client applications and frameworks. The StoRM backend service implements the actual storage management logic by interacting directly with the underlying file system. Data transfer is provided by GridFTP, HTTP/WebDAV and XRootD services accessing directly the file system underlying the StoRM deployment. StoRM WebDAV, besides HTTP data transfer functionality, also provides a WebDAV-based data management interface.

StoRM supports tape through integration with GEMSS, a tape library manager component also developed at INFN-CNAF.

![StoRM typical deployment architecture](/storm-docs/images/storm-architecture.png "Figure 1. StoRM typical deployment architecture")

StoRM provides flexible AuthN/Z support:
* VOMS & OAuth tokens (WebDAV & CDMI)
* File access control is enforced via POSIX ACLs

