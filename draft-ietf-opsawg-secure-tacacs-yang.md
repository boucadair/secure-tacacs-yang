---
title: "A YANG Data Model for Terminal Access Controller Access-Control System Plus (TACACS+)"
abbrev: "YANG for TACACS+ over TLS"
category: std
obsoletes: 9105

docname: draft-ietf-opsawg-secure-tacacs-yang-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Operations and Management Area Working Group"
keyword:
 - TLS
 - device management
 - network operator
 - provider network
 - AAA
 - authentication
 - authorization
 - accounting

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    role: editor
    email: mohamed.boucadair@orange.com
 -
    fullname: Bo Wu
    organization: Huawei Technologies
    email: mlana.wubo@huawei.com
normative:

informative:


--- abstract

   This document defines a Terminal Access Controller Access-Control
   System Plus (TACACS+) client YANG module that augments the System
   Management data model, defined in RFC 7317, to allow devices to make
   use of TACACS+ servers for centralized Authentication, Authorization,
   and Accounting (AAA). Specifically, this document defines a YANG module for TACACS+ over TLS 1.3.

   This document obsoletes RFC 9105.

--- middle

#  Introduction

The System Management data model {{!RFC7317}} defines separate functionality to support local and Remote Authentication Dial In User Service (RADIUS) authentication:

   User Authentication Model:
   :  Defines a list of user names with
      associated passwords and a configuration leaf to decide the order
      in which local or RADIUS authentication is used.

   RADIUS Client Model:
   : Defines a list of RADIUS servers used by a
      device for centralized user authentication.

{{?RFC9105}} defines a YANG module ("ietf-system-tacacs-plus") that augments the System Management data model {{!RFC7317}} for the management of Terminal Access Controller Access-Control System Plus (TACACS+) clients as an alternative to RADIUS servers {{?RFC2865}}. Typically, the "ietf-system-tacacs-plus" module is used to configure a TACACS+ client on a device to support deployment scenarios with centralized authentication, authorization, and accounting servers.

This document defines a YANG module for managing TACACS+ clients ({{sec-module}}), including TACACS+ over TLS 1.3 clients {{!I-D.ietf-opsawg-tacacs-tls13}}. This document obsoletes {{?RFC9105}}.

The YANG module in this document conforms to the Network Management
   Datastore Architecture (NMDA) defined in {{!RFC8342}}.

## Changes Since RFC 9105

   The following changes have been made to {{?RFC9105}}:

   * Add support for TLS {{!I-D.ietf-opsawg-tacacs-tls13}}
   * Add a constraint to ensure that the list of servers is unique per address/port number
   * Update the description of 'address' to be consistent with the type
   * Fix a must statement under 'tacacs-plus'
   * Fix errors in the example provided in Appendix A of {{?RFC9105}}
   * Add an example to illustrate the use of VRF
   * Add new examples to illustrate the use of TACACS+TLS data nodes

Detailed YANG changes are listed in {{sec-module}}.

## Editorial Note (To be removed by RFC Editor)

> Note to the RFC Editor: This section is to be removed prior to publication.

This document contains placeholder values that need to be replaced with finalized values at the time of publication. This note summarizes all of the substitutions that are needed.

Please apply the following replacements:

* XXXX --> the assigned RFC number for this I-D
* SSSS --> the assigned RFC number for {{!I-D.ietf-opsawg-tacacs-tls13}}
* TBD  --> the assigned port number in {{Section 7 of !I-D.ietf-opsawg-tacacs-tls13}}
* 2024-12-11 --> the actual date of the publication of this document

# Conventions and Definitions

{::boilerplate bcp14-tagged}

   The terminology for describing YANG data models is defined in
   {{!RFC7950}}.

The document uses the terms defined in {{Section 2 of !I-D.ietf-opsawg-tacacs-tls13}} and {{Section 3 of ?RFC8907}}.

'client' refers to TACACS+ client, while 'server' refers to TACACS+ server.

## Tree Diagrams

   The tree diagram used in this document follows the notation defined
   in {{?RFC8340}}.

# Design of the TACACS+ Data Model

   This module is used to configure a TACACS+ client on a device to
   support deployment scenarios with centralized authentication,
   authorization, and accounting servers.  Authentication is used to
   validate a user's username and password, authorization allows the
   user to access and execute commands at various privilege levels
   assigned to the user, and accounting keeps track of the activity of a
   user who has accessed the device.

   The "ietf-system-tacacs-plus" module augments the '/sys:system' path
   defined in the "ietf-system" module with the contents of the 'tacacs-plus'
   grouping.  Therefore, a device can use local, RADIUS, or
   TACACS+ authentication to validate users who attempt to access the
   device by several mechanisms, e.g., a command line interface or a
   web-based user interface.

   The 'server' list, which is directly under the 'tacacs-plus'
   container, holds a list of TACACS+ servers and uses 'server-type' to
   distinguish between Authentication, Authorization, and Accounting
   (AAA) services. The list of servers is for redundancy.

   When there are multiple interfaces connected to a TACACS+ client or
   server, the source address of outgoing TACACS+ packets could be
   specified, or the source address could be specified through the
   interface IP address setting or derived from the outbound interface
   from the local Forwarding Information Base (FIB). For a TACACS+
   server located in a Virtual Private Network (VPN), a VPN Routing and
   Forwarding (VRF) instance needs to be specified.

   The 'statistics' container under the 'server' list is a collection of
   read-only counters for sent and received messages from a configured
   server.

   The YANG module for TACACS+ client has the structure shown in {{tree-overview}}.

~~~
{::include-fold ./trees/tree-overview.txt}
~~~
{: #tree-overview title="Tree Structure Overview"}

Specifically, the module is designed to cover the following key requirements specified in {{!I-D.ietf-opsawg-tacacs-tls13}}:

* TLS 1.3 {{!RFC8446}} MUST be used for transport.
* Earlier TLS versions MUST NOT be used.
* The cipher suites offered or accepted SHOULD be configurable.
* Implementations MAY support Raw Public Keys (RPKs) and Pre-Shared Keys (PSKs).
* Implementations MUST support the ability to configure the server's domain name, so that it may be included in the TLS Server Name Indication (SNI) extension.

The following new data nodes are supported compared to {{?RFC9105}}:

'client-credentials' and 'server-credentials':
: Defines a set credentials that can be globally provisioned and then referenced under specific servers.

'domain-name':
: Provides a domain name of the server per {{Section 3.3 of !I-D.ietf-opsawg-tacacs-tls13}}. This is the TLS TACACS+ server's domain name that is  included in the SNI extension. This domain name is distinct from the IP address/hostname used for the underlying transport connection.

'sni-enabled':
: Controls activation of Server Name Indication (SNI) ({{Section 3 of !RFC6066}}). This parameter can be used only if a domain name is provided.

'client-identity':
: Specifies the identity credentials that the client may present when
  establishing a connection to a server. Client identities can be configured at the top level and then referenced for specific server instances. Alternatively, client identities can be configured explicitly under each server instance.

'server-authentication':
: Specifies how a client authenticates servers. Server credentials can be configured at the top level and then referenced for specific server instances. Alternatively, client identities can be configured explicitly under each server instance.

'hello-params':
: Controls TLS versions and cipher suites to be used when establishing TLS sessions.

'discontinuity-time':
: The time of the most recent occasion at which the client suffered a discontinuity
(a configuration action to reset all counters, re-initialization, etc.).

'cert-errors':
: Number of connection failures due to certificate issues.

'rpk-errors':
: Number of raw public key related connection failures.

# TACACS+ Client Module {#sec-module}

This YANG module uses types and groupings defined in {{!RFC6991}}, {{!RFC8341}}, {{!RFC8343}}, {{!RFC8529}}, {{!RFC9640}}, {{!RFC9641}},
{{!RFC9642}}, and {{!RFC9645}}.

The module augments {{!RFC7317}}.

The module also cites {{!RFC6520}}, {{!RFC9257}}, and {{!RFC9258}}.

~~~ yang
<CODE BEGINS> file "ietf-system-tacacs-plus@2025-01-23.yang"
{::include-fold ./yang/ietf-system-tacacs-plus.yang}
<CODE ENDS>
~~~

# Operational Considerations

The same operational considerations discussed in {{Section 6 of !I-D.ietf-opsawg-tacacs-tls13}} apply for this document.

# Security Considerations

This section is modeled after the template described in {{Section 3.7 of ?I-D.ietf-netmod-rfc8407bis}}.

The "ietf-ac-common" YANG module defines a data model that is
designed to be accessed via YANG-based management protocols, such as
NETCONF {{?RFC6241}} and RESTCONF {{?RFC8040}}. These YANG-based management protocols (1) have to
use a secure transport layer (e.g., SSH {{?RFC4252}}, TLS {{?RFC8446}}, and
QUIC {{?RFC9000}}) and (2) have to use mutual authentication.

The Network Configuration Access Control Model (NACM) {{!RFC8341}}
   provides the means to restrict access for particular NETCONF or
   RESTCONF users to a preconfigured subset of all available NETCONF or
   RESTCONF protocol operations and content.

There are a number of data nodes defined in this YANG module that are
writable/creatable/deletable (i.e., "config true", which is the
default).  All writable data nodes are likely to be
sensitive or vulnerable in some network environments.  Write
operations (e.g., edit-config) and delete operations to these data
nodes without proper protection or authentication can have a negative
effect on network operations.  The following subtrees and data nodes
have particular sensitivities/vulnerabilities:

   'server':
   :  This list contains the data nodes used
      to control the TACACS+ servers used by the device. Unauthorized
      access to this list could enable an attacker to assume complete
      control over the device by pointing to a compromised TACACS+
      server, or to modify the counters to hide attacks against the
      device.

   'shared-secret':
   :  This leaf controls the key
      known to both the TACACS+ client and server. Unauthorized access
      to this leaf could make the device vulnerable to attacks;
      therefore, it has been restricted using the "default-deny-all"
      access control defined in {{!RFC8341}}.  When setting, it is highly
      recommended that the leaf is at least 32 characters long and
      sufficiently complex with a mix of different character types,
      i.e., upper case, lower case, numeric, and punctuation.

   'client-identity' and 'server-authentication':
   : Any modification to a key or reference to a key may dramatically alter the implemented security policy. For this reason, the NACM extension "default-deny-write" has been set.

This YANG module uses groupings from other YANG modules that define nodes that may be considered sensitive or vulnerable in network environments. Refer to {{Section 5.3 of !RFC9642}} and {{Section 5.3 of !RFC9645}} for information as to which nodes may be considered sensitive or vulnerable in network environments.

# IANA Considerations

   IANA is requested to update the following URI in the "ns" subregistry within
   the "IETF XML Registry" {{!RFC3688}}:

~~~~
   URI:  urn:ietf:params:xml:ns:yang:ietf-system-tacacs-plus
   Registrant Contact:  The IESG.
   XML:  N/A; the requested URI is an XML namespace.
~~~~

   IANA is requested to register the following YANG module in the "YANG Module
   Names" registry {{!RFC6020}} within the "YANG Parameters" registry group:

~~~~
   Name:  ietf-system-tacacs-plus
   Namespace:  urn:ietf:params:xml:ns:yang:ietf-system-tacacs-plus
   Prefix:  sys-tcs-plus
   Maintained by IANA?  N
   Reference:  RFC XXXX
~~~~


--- back

# Example TACACS+ Authentication Configuration with Shared Secret

{{ex9105}} shows an example where a TACACS+ authentication server instance is configured using shared secret for authentication. This mode is not recommended per {{!I-D.ietf-opsawg-tacacs-tls13}}.

~~~ json
{::include-fold ./json-example/9105-ex.json}
~~~
{: #ex9105 title="Example with Shared Secret"}

{{ex9105-vrf}} provides an example to associate a TACACS+ server with a VRF.

~~~ json
{::include-fold ./json-example/9105-with-vrf.json}
~~~
{: #ex9105-vrf title="Example with VRF"}

# TACACS+TLS Examples

This section provides examples to illustrate the configuration of TACACS+TLS clients.

These examples follow the convention used in {{Section 1.5 of !RFC9645}} for binary data that has been base64 encoded.

## Example TACACS+ Authentication Configuration with Explicit Certificate Definitions

{{exin}} shows a configuration example with 'inline-definition' for the client identity and server authentication.

~~~ json
{::include-fold ./json-example/tls-ex-inline.json}
~~~
{: #exin title="Example with TACACS+TLS with Inline Certificate Definitions"}

## Example TACACS+ Authentication Configuration with Certificate References

{{ex-ref}} shows a configuration example with credential references for multiple service instances: four server instances are configured with all using the same credentials. These instances form a redundancy group for both IPv4 and IPv6.

~~~ json
{::include-fold ./json-example/tls-ex-with-ref.json}
~~~
{: #ex-ref title="Example with TACACS+TLS with References"}

# Full Tree {#sec-full}

The full tree structure is shown below:

~~~~~~~~~~
{::include-fold ./trees/full-tree.txt}
~~~~~~~~~~

# Acknowledgments
{:numbered="false"}

The document leverages data structures defined in {{!RFC9645}}.

Thanks to Joe Clarke and Tom Petch for the review and comments.

Thanks to Reshad Rahman for the yangdoctors review, Tina Tsou for
the opsdir review, and Ines Robles for the genart review.

Thanks Mahesh Jethanandani for the AD review.

Thanks Erik Kline for the IESG review.

Authors of RFC 9105:
: Bo Wu
: Guangying Zheng
: Michael Wang

Acknowledgments from RFC 9105:
: The authors wish to thank Alex Campbell, John Heasley, Ebben Aries,
   Alan DeKok, Joe Clarke, Tom Petch, Robert Wilton, and many others for
   their helpful comments and suggestions.
