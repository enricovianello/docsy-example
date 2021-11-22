---
title: "WebDAV User Guide"
linkTitle: "WebDAV"
noComment: true
weight: 2
description: >
---

## The WebDAV protocol

<img src="webdav-logo.jpg" alt="webdav-logo" width="100" style="float: left; margin-right: 30px; margin-left: 0px; margin-top: 10px;"/>


Web Distributed Authoring and Versioning (WebDAV) protocol consists of a set of methods, headers, and content-types ancillary to HTTP/1.1 for the management of resource properties, creation and management of resource collections, URL namespace manipulation, and resource locking. The purpose of this protocol is to present a Web content as a writable medium in addition to be a readable one. [WebDAV on Wikipedia](http://en.wikipedia.org/wiki/WebDAV) and the [WebDAV website](http://www.webdav.org/) provide information on this protocol.

In a few words, the WebDAV protocol mainly abstracts concepts such as resource properties, collections of resources, locks in general, and write locks specifically. These abstractions are manipulated by the WebDAV-specific HTTP methods and the extra HTTP headers used with WebDAV methods. The WebDAV added methods include:

* PROPFIND - used to retrieve properties, stored as XML, from a web resource. It is also overloaded to allow one to retrieve the collection structure (a.k.a. directory hierarchy) of a remote system.
* PROPPATCH - used to change and delete multiple properties on a resource in a single atomic act.
* MKCOL - used to create collections (a.k.a. a directory).
* COPY - used to copy a resource from one URI to another.
* MOVE - used to move a resource from one URI to another.
* LOCK - used to put a lock on a resource. WebDAV supports both shared and exclusive locks.
* UNLOCK - used to remove a lock from a resource.

While the status codes provided by HTTP/1.1 are sufficient to describe most error conditions encountered by WebDAV methods, there are some errors that do not fall neatly into the existing categories, so the WebDAV specification defines some extra status codes. Since some WebDAV methods may operate over many resources, the Multi-Status response has been introduced to return status information for multiple resources.
WebDAV uses XML for property names and some values, and also uses XML to marshal complicated requests and responses.

## StoRM WebDAV

From [StoRM v.1.11.7](http://italiangrid.github.io/storm/release-notes/StoRM-v1.11.7.html) release, the StoRM service that provides valid WebDAV endpoints for each managed storage area is *StoRM WebDAV*.

*StoRM WebDAV* replaces the *StoRM gridhttps service*. All sites installing StoRM and providing HTTP and WebDAV endpoints should upgrade to the StoRM WebDAV service for improved performance and stability of the service as soon as possible.

**Important**: The StoRM WebDAV service is released and supported only on SL/CENTOS 6.

### Installation and configuration

See the [System Administration Guide][webdavconf] to learn how to install and configure the service.

### Endpoints

For each Storage Area, both/either a plain HTTP and/or a HTTP over SSL endpoint can be enabled. The default ports are **8085** (HTTP) and **8443** (HTTPS).
All the following URLs are valid endpoints for a storage area:

    http://example.infn.it:8085/storage_area_accesspoint
    https://example.infn.it:8443/storage_area_accesspoint

To fully support the old *StoRM GridHTTPs* webdav endpoints, used until StoRM v1.11.6, all the URLs with *webdav* context path are accepted by *StoRM WebDAV*:

    http://example.infn.it:8085/webdav/storage_area_accesspoint
    https://example.infn.it:8443/webdav/storage_area_accesspoint

### Authentication and authorization

Users authentication within *StoRM WebDAV* is made through a valid VOMS proxy. All the users that provide a valid x509 VOMS proxy are authorized to access all the content of the storage area in read/write mode.

The most common way to authenticate and be authorized to read/write data into a Storage Area is by providing the right VOMS credentials through a valid VOMS Proxy. Otherwise, through the definition of a VOMS map file, a Storage Area can be configure to accept the list of VO members as obtained by running the `voms-admin list-users` command.
When VOMS mapfiles are enabled, users can authenticate to the StoRM webdav
service using the certificate in their browser and be granted VOMS attributes
if their subject is listed in one of the supported VOMS mapfile. For each supported VO, a file having the same name as the VO is put in the voms-mapfiles directory (`/etc/storm/storm-webdav/vo-mapfiles.d`).

Example: to generate a VOMS mapfile for the cms VO, run the following command

```bash
  voms-admin --host voms.cern.ch --vo cms list-users > cms
```

See more details [here][vomapfiles]. Read permissions of the content of a storage area can also be extendend to anonymous user (it's disabled by default).

### Notes

Both the old ```storm-gridhttps-server``` and the new ```storm-webdav``` components implements WebDAV protocol by using [*Milton*](http://milton.io/) open source java library.

![milton](milton.png)

## Examples

The most common WebDAV clients are:

* browsers
* command-line tools like cURLs, davix and gfal
* a third-party GUI

Currently, users are used to connect to a WebDAV endpoint providing a valid username and password or as anonymous users (if supported). But, as seen in the [Authentication and authorization](#authandauth) paragraph, in our case the most common use case is providing a valid VOMS proxy. The VOMS proxies are supported only by command-line tools. Browsers can be used to navigate into the storage area content in the some cases:

* if VO users access through their x509 certificate is enabled (HTTPS endpoint)
* if anonymous read-only access is enabled (HTTP endpoint)

The use of a third party client (in read only mode) can happen only if anonymous read is enabled.

You can also develop a client on your own, for example by using the <a href="http://jackrabbit.apache.org/">Apache Jackrabbit API</a>.

The following paragraphs will give an example for each WebDAV/HTTP method by using cURLS, DAVIX and gFAL client command line tools. cURL is a command line tool for transferring data with URL syntax (see [cURL website](http://curl.haxx.se/)).

All the requests have been done:

* against our WebDAV test endpoint `https://omii006-vm03.cnaf.infn.it:8443/`
* using `test0.p12` of our `igi-test-ca` credentials:

```bash
$ yum install igi-test-ca
$ cp /usr/share/igi-test-ca/test0.p12 $HOME
$ chmod 600 test0.p12
```

* after the creation of a valid VOMS proxy for `test.vo` VO

```bash
$ cd $HOME
$ voms-proxy-init --voms test.vo --cert test0.p12
```

### Download file

Having the remote file:

**/test.vo/test.txt**

```bash
Hello world
```

use:

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem -o test.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    12  100    12    0     0    101      0 --:--:-- --:--:-- --:--:--   102
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-get -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt test3.txt
Performing Read operation on: https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
[====================================] 100%     12B/12B         0B/s
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-copy davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt test2.txt
Copying davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt   [DONE]  after 0s
{{< /tab >}}
{{< /tabpane >}}

### Download multiple file ranges

Having the remote file:

**/test.vo/test.txt**

```bash
Hello world
```

use:

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem \
-o test.txt -H "Range: 0-1,3-4,8-10"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   295  100   295    0     0   1804      0 --:--:-- --:--:-- --:--:--  1809
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-get -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt test3.txt \
-H "Range: 0-1,3-4,8-10"
Performing Read operation on: https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
Not supported!
{{< /tab >}}
{{< /tabpane >}}

### Read file content

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem
Hello world
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-get -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
Hello world
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-cat davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
Hello world
{{< /tab >}}
{{< /tabpane >}}

### Read multiple file ranges

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem -H "Range: 0-1,3-4,8-10"
--jetty1198079116kuzifrgh
Content-Type: text/plain
Content-Range: bytes 0-1/12

He
--jetty1198079116kuzifrgh
Content-Type: text/plain
Content-Range: bytes 3-4/12

lo
--jetty1198079116kuzifrgh
Content-Type: text/plain
Content-Range: bytes 8-10/12

rld
--jetty1198079116kuzifrgh--
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-get -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
-H "Range: 0-1,3-4,8-10"
--jetty77231386kuzigjn3
Content-Type: text/plain
Content-Range: bytes 0-1/12

He
--jetty77231386kuzigjn3
Content-Type: text/plain
Content-Range: bytes 3-4/12

lo
--jetty77231386kuzigjn3
Content-Type: text/plain
Content-Range: bytes 8-10/12

rld
--jetty77231386kuzigjn3--
{{< /tab >}}

{{< tab header="Gfal" >}}
Not supported!
{{< /tab >}}
{{< /tabpane >}}

### Upload file

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -T test.txt https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-put -P Grid test.txt https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-copy test.txt davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
Copying file:///home/user/test.txt   [DONE]  after 0s
{{< /tab >}}
{{< /tabpane >}}

### Check if resource exists

To check if a resource exists without download any data in case of a file, the HTTP HEAD method is used. HEAD acts like HTTP/1.1, so HEAD is a GET without a response message body.

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X HEAD https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem
curl: (18) transfer closed with 12 bytes remaining to read
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-http -P Grid -X HEAD https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-ls davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}
{{< /tabpane >}}

### Create directory

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X MKCOL https://omii006-vm03.cnaf.infn.it:8443/test.vo/testDir \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-mkdir -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/testDir
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-mkdir davs://omii006-vm03.cnaf.infn.it:8443/test.vo/testDir
{{< /tab >}}
{{< /tabpane >}}

### Delete file

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X DELETE https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-rm -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-rm davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt	DELETED
{{< /tab >}}
{{< /tabpane >}}

### Copy or duplicate file

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X COPY https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem \
-H "Destination: https://omii006-vm03.cnaf.infn.it:8443/test.vo/test3.txt"
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-cp -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
https://omii006-vm03.cnaf.infn.it:8443/test.vo/test2.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-copy davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test-copy.txt
Copying davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt   [DONE]  after 0s
{{< /tab >}}
{{< /tabpane >}}

### Move or rename file

As already seen for COPY method, also for every MOVE the **Destination** header MUST be present.

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X MOVE https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem \
-H "Destination: https://omii006-vm03.cnaf.infn.it:8443/test.vo/test2.txt"
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-mv -P Grid https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
https://omii006-vm03.cnaf.infn.it:8443/test.vo/test2.txt
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-rename davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test-renamed.txt
{{< /tab >}}
{{< /tabpane >}}

### List directory

There are two ways to get the list of the resources into a remote directory:

- the first is through an HTTP GET which returns a human readable version of all the content of the directory;
- the second is through a WebDAV PROPFIND which returns a structured XML body.

The PROPFIND operation retrieves, in XML format, the properties defined on the resource identified by the Request-URI. Clients must submit a `Depth` header with a value of `"0"`, `"1"`, or `"infinity"` (default is `"Depth: infinity"`).

Clients may submit through the body of the request a 'propfind' XML element. It's used to describe what information is being requested:

* a particular property value (by using the 'prop' element)

```xml
<?xml version="1.0" encoding="utf-8" ?>
  <D:propfind xmlns:D="DAV:">
    <D:prop xmlns:R="http://ns.example.com/boxschema/">
      <R:author/>
      <R:title/>
    </D:prop>
  </D:propfind>
```

In this example, the propfind XML element specifies the name of two properties whose values are being requested.

* all property values (by using the 'allprop' element);

```xml
<?xml version="1.0" encoding="utf-8" ?>
  <D:propfind xmlns:D="DAV:">
    <D:allprop/>
  </D:propfind>
```

In this example, the request should return the name and value of all the properties defined by WebDAV specification plus the user defined properties.

* the list of names of all the properties defined on the resource (by using the 'propname' element).

```xml
<?xml version="1.0" encoding="utf-8" ?>
  <propfind xmlns="DAV:">
    <propname/>
  </propfind>
```

To list all the content of a remote directory we can use the 'allprop' XML body, with depth equal to 1:

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X PROPFIND https://omii006-vm03.cnaf.infn.it:8443/test.vo \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem \
--data "<?xml version='1.0' encoding='utf-8' ?><D:propfind xmlns:D='DAV:'><D:allprop/></D:propfind>" \
-H "Depth:1" | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 54572    0 54481  100    91   414k    708 --:--:-- --:--:-- --:--:--  412k
<?xml version="1.0" encoding="utf-8"?>
<d:multistatus xmlns:cal="urn:ietf:params:xml:ns:caldav" xmlns:ns1="http://storm.italiangrid.org/2014/webdav" xmlns:cs="http://calendarserver.org/ns/" xmlns:card="urn:ietf:params:xml:ns:carddav" xmlns:d="DAV:">
  <d:response>
    <d:href>/test.vo/</d:href>
    <d:propstat>
      <d:prop>
        <d:getcreated>2021-10-15T12:10:17Z</d:getcreated>
        <d:creationdate>2021-10-15T12:10:17Z</d:creationdate>
        <d:getcontenttype/>
        <d:getlastmodified>Fri, 15 Oct 2021 12:10:17 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo_-2082722476"</d:getetag>
        <d:iscollection>TRUE</d:iscollection>
        <d:displayname>test.vo</d:displayname>
        <d:isreadonly>FALSE</d:isreadonly>
        <d:name>test.vo</d:name>
        <d:supported-report-set/>
        <d:getcontentlength/>
        <d:resourcetype>
          <d:collection/>
        </d:resourcetype>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
<d:response>
    <d:href>/test.vo/test2.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum/>
        <d:getcreated>2021-10-14T09:08:43Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:43Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:43 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test2.txt_2114950148"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test2.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test2.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
  <d:response>
    <d:href>/test.vo/test-renamed.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:10:18Z</d:getcreated>
        <d:creationdate>2021-10-14T09:10:18Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:10:18 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test-renamed.txt_2115045868"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test-renamed.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test-renamed.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
  <d:response>
    <d:href>/test.vo/testDir/</d:href>
    <d:propstat>
      <d:prop>
        <d:getcreated>2021-10-14T08:57:48Z</d:getcreated>
        <d:creationdate>2021-10-14T08:57:48Z</d:creationdate>
        <d:getcontenttype/>
        <d:getlastmodified>Thu, 14 Oct 2021 08:57:48 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/testDir_2114295708"</d:getetag>
        <d:iscollection>TRUE</d:iscollection>
        <d:displayname>testDir</d:displayname>
        <d:isreadonly>FALSE</d:isreadonly>
        <d:name>testDir</d:name>
        <d:supported-report-set/>
        <d:getcontentlength/>
        <d:resourcetype>
          <d:collection/>
        </d:resourcetype>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
<d:response>
    <d:href>/test.vo/test.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:08:23Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:23Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:23 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test.txt_2114930212"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
</d:multistatus>
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-http -P Grid -X PROPFIND https://omii006-vm03.cnaf.infn.it:8443/test.vo \
--data "<?xml version='1.0' encoding='utf-8' ?><D:propfind xmlns:D='DAV:'><D:allprop/></D:propfind>" \
-H "Depth:1" | xmllint --format -
<?xml version="1.0" encoding="utf-8"?>
<d:multistatus xmlns:cal="urn:ietf:params:xml:ns:caldav" xmlns:ns1="http://storm.italiangrid.org/2014/webdav" xmlns:cs="http://calendarserver.org/ns/" xmlns:card="urn:ietf:params:xml:ns:carddav" xmlns:d="DAV:">
  <d:response>
    <d:href>/test.vo/</d:href>
    <d:propstat>
      <d:prop>
        <d:getcreated>2021-10-15T12:10:17Z</d:getcreated>
        <d:creationdate>2021-10-15T12:10:17Z</d:creationdate>
        <d:getcontenttype/>
        <d:getlastmodified>Fri, 15 Oct 2021 12:10:17 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo_-2082722476"</d:getetag>
        <d:iscollection>TRUE</d:iscollection>
        <d:displayname>test.vo</d:displayname>
        <d:isreadonly>FALSE</d:isreadonly>
        <d:name>test.vo</d:name>
        <d:supported-report-set/>
        <d:getcontentlength/>
        <d:resourcetype>
          <d:collection/>
        </d:resourcetype>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
<d:response>
    <d:href>/test.vo/test2.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum/>
        <d:getcreated>2021-10-14T09:08:43Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:43Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:43 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test2.txt_2114950148"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test2.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test2.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
  <d:response>
    <d:href>/test.vo/test-renamed.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:10:18Z</d:getcreated>
        <d:creationdate>2021-10-14T09:10:18Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:10:18 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test-renamed.txt_2115045868"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test-renamed.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test-renamed.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
  <d:response>
    <d:href>/test.vo/testDir/</d:href>
    <d:propstat>
      <d:prop>
        <d:getcreated>2021-10-14T08:57:48Z</d:getcreated>
        <d:creationdate>2021-10-14T08:57:48Z</d:creationdate>
        <d:getcontenttype/>
        <d:getlastmodified>Thu, 14 Oct 2021 08:57:48 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/testDir_2114295708"</d:getetag>
        <d:iscollection>TRUE</d:iscollection>
        <d:displayname>testDir</d:displayname>
        <d:isreadonly>FALSE</d:isreadonly>
        <d:name>testDir</d:name>
        <d:supported-report-set/>
        <d:getcontentlength/>
        <d:resourcetype>
          <d:collection/>
        </d:resourcetype>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
<d:response>
    <d:href>/test.vo/test.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:08:23Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:23Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:23 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test.txt_2114930212"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
</d:multistatus>
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-ls davs://omii006-vm03.cnaf.infn.it:8443/test.vo
test2.txt
test-renamed.txt
testDir
test.txt
{{< /tab >}}
{{< /tabpane >}}

To list all the properties of a remote file we can use the same 'allprop' XML body:

{{< tabpane lang="bash" >}}
{{< tab header="CURL" >}}
$ curl -X PROPFIND https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--cert $X509_USER_PROXY \
--capath /etc/grid-security/certificates \
--cacert /usr/share/igi-test-ca/test0.cert.pem \
--data "<?xml version='1.0' encoding='utf-8' ?><D:propfind xmlns:D='DAV:'><D:allprop/></D:propfind>" \
-H "Depth:1" | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1053    0   962  100    91   3985    377 --:--:-- --:--:-- --:--:--  3975
<?xml version="1.0" encoding="utf-8"?>
<d:multistatus xmlns:cal="urn:ietf:params:xml:ns:caldav" xmlns:ns1="http://storm.italiangrid.org/2014/webdav" xmlns:cs="http://calendarserver.org/ns/" xmlns:card="urn:ietf:params:xml:ns:carddav" xmlns:d="DAV:">
  <d:response>
    <d:href>/test.vo/test.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:08:23Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:23Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:23 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test.txt_2114930212"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
</d:multistatus>
{{< /tab >}}

{{< tab header="Davix" >}}
$ davix-http -P Grid -X PROPFIND https://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt \
--data "<?xml version='1.0' encoding='utf-8' ?><D:propfind xmlns:D='DAV:'><D:allprop/></D:propfind>" | xmllint --format -
<?xml version="1.0" encoding="utf-8"?>
<d:multistatus xmlns:cal="urn:ietf:params:xml:ns:caldav" xmlns:ns1="http://storm.italiangrid.org/2014/webdav" xmlns:cs="http://calendarserver.org/ns/" xmlns:card="urn:ietf:params:xml:ns:carddav" xmlns:d="DAV:">
  <d:response>
    <d:href>/test.vo/test.txt</d:href>
    <d:propstat>
      <d:prop>
        <ns1:Checksum>1cf20447</ns1:Checksum>
        <d:getcreated>2021-10-14T09:08:23Z</d:getcreated>
        <d:creationdate>2021-10-14T09:08:23Z</d:creationdate>
        <d:getcontenttype>text/plain</d:getcontenttype>
        <d:getlastmodified>Thu, 14 Oct 2021 09:08:23 GMT</d:getlastmodified>
        <d:getetag>"/storage/test.vo/test.txt_2114930212"</d:getetag>
        <d:iscollection>FALSE</d:iscollection>
        <d:displayname>test.txt</d:displayname>
        <d:isreadonly>TRUE</d:isreadonly>
        <d:name>test.txt</d:name>
        <d:supported-report-set/>
        <d:getcontentlength>12</d:getcontentlength>
        <d:resourcetype/>
      </d:prop>
      <d:status>HTTP/1.1 200 OK</d:status>
    </d:propstat>
  </d:response>
</d:multistatus>
{{< /tab >}}

{{< tab header="Gfal" >}}
$ gfal-ls davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
davs://omii006-vm03.cnaf.infn.it:8443/test.vo/test.txt
{{< /tab >}}
{{< /tabpane >}}
