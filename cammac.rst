**CAMMAC (Container Authenticated by Multiple MACs)**

When a Kerberos client requests a service ticket from a KDC, it can submit
authorization data for the service that implies a privilege restriction (ie. a
subset of a user's normal file permissions) to the request. The KDC can copy
this authorization data into the issued ticket, without verifying or
inspecting its contents.

::

    tgs-req
        pvno: 5
        msg-type: krb-tgs-req (12)
        padata: 2 items
            ...
        req-body
            ...
            realm: KRBTEST.COM
            sname
                name-type: kRB5-NT-PRINCIPAL (1)
                name-string: 2 items
                    KerberosString: host
                    KerberosString: localhost
            till: 2016-09-08 17:19:09 (UTC)
            nonce: 1473268749
            etype: 6 items
                ENCTYPE: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
                ...
    --->    enc-authorization-data
    |           etype: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
    |           cipher: d29b82454191c7a4c06fdd8e38039ec3df7762d59ea232ca...
    |
    --- encrypted in the TGT-obtained session key (or sub-session key),
        by the client

Sometimes the intent of the authorization data is a privilege elevation (ie.
principal user is in the admin group) in which the KDC cannot just copy it to
the ticket, since the client can modify it at-will when requesting the ticket
and therefore elevate their own privileges.

There is the KDC-ISSUED container, by which the KDC inspects the client added
authorization data, runs policy checks, and then adds new authorization data
to the ticket that is integrity protected by a checksum generated with the
server's key. The server can then verify that the authorization data comes
from the KDC.

However, the S4U2Proxy scenario [MS-SFU_] requires that servers present
previously-issued service tickets to a KDC in order to impersonate a user with
a derived ticket.  Since the server has the server key, the KDC can't be sure
that the server did not tamper with the authorization data from the evidence
ticket.

The CAMMAC_ type container takes the place of KDC-ISSUED, as it provides the
same service checksum, but also adds a second checksum for the KDC's own use.
The KDC checksum is generated with its own TGS key to be able to verify that
it previously issued the authorization data. The PAC structure in an Active
Directory ticket has a similar ``PAC_SIGNATURE_DATA`` container that provides
server and KDC signatures.

The addition of CAMMAC in MIT krb5 was a prerequisite for `Authentication
Indicators`_, which is CAMMAC-carried authorization data that can inform a
service of the strength of pre-authentication performed by the client.

.. _MS-SFU: https://msdn.microsoft.com/en-us/library/cc246071.aspx
.. _CAMMAC: https://tools.ietf.org/html/rfc7751
.. _`Authentication Indicators`: https://tools.ietf.org/html/draft-ietf-kitten-krb-auth-indicator-02

