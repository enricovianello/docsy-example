---
title: "Repositories"
linkTitle: "Repositories"
date: 2021-09-21
weight: 1
noComment: true
description: >
  Before installing StoRM components, make sure you have the needed **repositories**.
---

StoRM packages can be obtained from both StoRM product team package repository and UMD4 repository. EPEL and EGI Trust Anchors repositories are also required.

### **EPEL**

StoRM requires [EPEL][epel-repo] repositories. Install them on CentOS 7 as follows:

```
yum install epel-release
```

Or:

```
yum localinstall https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-14.noarch.rpm
```

[epel-repo]: https://fedoraproject.org/wiki/EPEL/it


### **EGI Trust Anchors**

Install *EGI Trust Anchors repository* by following [EGI instructions][egi-instructions].

In short:

```
wget http://repository.egi.eu/sw/production/cas/1/current/repo-files/EGI-trustanchors.repo -O /etc/yum.repos.d/EGI-trustanchors.repo
yum install ca-policy-egi-core
```

The *DAG repository* must be disabled. If needed, set to 0 the enabled property in your */etc/yum.repos.d/dag.repo* file.

[egi-instructions]: https://wiki.egi.eu/wiki/EGI_IGTF_Release#Using_YUM_package_management

### **UMD**

StoRM requires UMD repositories.
First of all you should install UMD pgp key:

```
rpm --import http://repository.egi.eu/sw/production/umd/UMD-RPM-PGP-KEY
```

Then, install latest UMD-4 repository for CentOS 7 as follows:

```
yum localinstall http://repository.egi.eu/sw/production/umd/4/centos7/x86_64/updates/umd-release-4.1.3-1.el7.centos.noarch.rpm
```

More information about UMD installation can be found [here][UMD-instructions].

[UMD-instructions]: http://repository.egi.eu/category/umd_releases/distribution/umd-4/

### **StoRM Product Team**

The latest certified StoRM packages can be found in the StoRM production repository:

- [Browse SL7 packages][stable-storm-sl7-repoview]

Note that you should also have UMD repositories installed (as detailed above) for your setup to work as expected.

To install the StoRM production repository files, run the following command (as root):

```
yum-config-manager --add-repo https://repo.cloud.cnaf.infn.it/repository/storm/storm-stable-centos7.repo
```

[stable-storm-sl6-repoview]: https://repo.cloud.cnaf.infn.it/service/rest/repository/browse/storm-rpm-stable/centos6/
[stable-storm-sl7-repoview]: https://repo.cloud.cnaf.infn.it/service/rest/repository/browse/storm-rpm-stable/centos7/


