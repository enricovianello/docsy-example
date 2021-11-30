---
title: Argus blacklisting
version: 1.11.4
---

### Example of how to avoid a specific user to access a storage-area using Argus authorization system.

_**Components installed**_: <span class="label label-important">StoRM Backend</span> <span class="label label-info">StoRM Frontend</span> <span class="label">StoRM GridFTP</span> <span class="label label-warning">Argus</span>

#### Prerequisites:

You need:

- basic StoRM services installated 
- an Argus authorization system installed (see [Argus twiki][argustwiki])

#### StoRM configuration

In order to create and manage a list of banned users, StoRM can be configured to use ARGUS authorization system.
The Argus authorization service allows administrators to define policies that answer questions like _Can user X perform action Y on resource Z at this time?_ See [Argus twiki][argustwiki] to get more information about this framework.
StoRM doesn't integrate all the services provided by Argus. It allows only to define a list of banned users. 

Supposing you have installed StoRM on a single host, modify your *storm-frontend-server.conf* StoRM Frontend configuration file to integrate ARGUS service. It's necessary to instruct the Frontend to communicate with Argus PEP server.

You need to specify the complete service endpoint of Argus PEP server:
```bash
argus-pepd-endpoint=https://omii005-vm02.cnaf.infn.it:8154/authz
```

Then you have to choose an entity identifier for StoRM service (it's a string that represents the entity-id that you'll use for each StoRM's Argus policy):
```bash
argus.resource-id=storm
```

and enable Frontend's user blacklisting with:
```bash
check.user.blacklisting=true
```
An alternative is to use the StoRM Frontend puppet module. You can modify the *manifest.pp* file by adding the same parameters in the StoRM Frontend class:

```bash
class { 'storm::frontend':
  ...
  check_user_blacklisting => true,
  argus_pepd_endpoint => 'https://omii005-vm02.cnaf.infn.it:8154/authz',
  argus_resource_id => 'storm',
}
```
The StoRM configuration is completed. But before running the configuration it's necessary to define at least a policy for every VO that your StoRM instance supports. To do this, we have to add some valid policies. We can write a file in [Simplified Policy Language][SPLguide] and then import it or we can use the [pap-admin CLI][papadminCLI] directly.
For example, if your StoRM instance supports ```dteam``` VO, you have to write the following policy:

```bash
$ pap-admin add-policy permit --resource "storm" --action ".*" vo="dteam"
$ pap-admin list-policies

default (local):

resource "storm" {

    action ".*" {
        rule permit { vo="dteam" }
    }
}
```

See [Simplified Policy Language guide][SPLguide] to learn how to write valid Argus policies.
If you have a storage area not owned by a VO but readable and writable with a particular x509 certificate (see [this example][X509_SA_conf_example]), you can add an ARGUS policy as follow:

```bash
$ pap-admin add-policy permit --resource "storm" --action ".*" subject-issuer="CN=Test1, O=IGI, C=IT"
$ pap-admin list-policies

default (local):

resource "storm" {

    action ".*" {
        rule permit { subject-issuer="CN=Test1, O=IGI, C=IT" }
    }
}
```

Finally, if you want to ban a particular user you can add a policy as follow:

```bash
$ pap-admin add-policy deny --resource ".*" --action ".*" subject="CN=Test0, O=IGI, C=IT"
$ pap-admin list-policies

default (local):

resource ".*" {

    action ".*" {
        rule deny { subject="CN=Test0, O=IGI, C=IT" }
    }
}
```
or you can simply do:

```bash
$ pap-admin ban subject "CN=Test0, O=IGI, C=IT"
```

Save all these policies on a file and import it via pap-admin command ```add-policies-from-file```:

```bash
$ pap-admin add-policies-from-file <filepath>
```

This command append the contained policies so, if you want to replace the existing policies, you have to do a:

```bash
$ pap-admin rap
```

before, and then import your file.
Clear the cache and reload the policies to finish:

```bash
$ pepdctl clearResponseCache
$ pdpctl reloadPolicy
```

Trying to do a srmLs with the banned certificate we get:

{{< tabpane lang="bash" >}}
{{< tab header="Gfal" >}}
$ gfal-ls srm://omii005-vm03.cnaf.infn.it:8444/test.vo/
gfal-ls error: 13 (Permission denied) - srm-ifce err: Permission denied, err: [SE][Ls][SRM_AUTHORIZATION_FAILURE] httpg://omii005-vm03.cnaf.infn.it:8444/srm/managerv2: Request authorization error: user is blacklisted.
{{< /tab >}}

{{< tab header="ClientSRM" >}}
$ clientSRM ls -e omii005-vm03.cnaf.infn.it:8444 -s srm://omii005-vm03.cnaf.infn.it:8444/test.vo
============================================================
Sending Ls request to: omii005-vm03.cnaf.infn.it:8444
Before execute:
Afer execute:
Request Status Code 3
Poll Flag 0
============================================================
Request status:
  statusCode="SRM_AUTHORIZATION_FAILURE"(3)
  explanation="Request authorization error: user is blacklisted."
============================================================
SRM Response:
============================================================
{{< /tab >}}
{{< /tabpane >}}

with the following logs:

```bash
11/30 14:43:35.720 Thread 7 -  INFO [523dfd3f-aae1-4ec2-bafd-0cdc0e223fc2]: process_request : Connection from 131.154.100.192
11/30 14:43:35.735 Thread 7 -  INFO [523dfd3f-aae1-4ec2-bafd-0cdc0e223fc2]: ns1__srmLs : Request: Ls. IP: 131.154.100.192. Client DN: /C=IT/O=IGI/CN=test0
11/30 14:43:35.813 Thread 7 -  ERROR [523dfd3f-aae1-4ec2-bafd-0cdc0e223fc2]: ns1__srmLs : Request authorization error: user is blacklisted.
11/30 14:43:35.813 Thread 7 -  INFO [523dfd3f-aae1-4ec2-bafd-0cdc0e223fc2]: Result for request 'Ls' is 'SRM_AUTHORIZATION_FAILURE'
```

[X509_SA_conf_example]: http://italiangrid.github.io/storm/documentation/how-to/storage-area-configuration-examples/1.11.3/index.html#sa-anonymous-rw-x509
[SPLguide]: https://twiki.cern.ch/twiki/bin/view/EGEE/SimplifiedPolicyLanguage
[papadminCLI]: https://twiki.cern.ch/twiki/bin/view/EGEE/AuthZPAPCLI
[argustwiki]: https://twiki.cern.ch/twiki/bin/view/EGEE/AuthorizationFramework