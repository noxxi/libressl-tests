# Problem

Validation fails in when rebasing a CA or similar. The new validation probably
sees an old chain as untrusted but does not find the new valid chain.

# Setup of test certificates

    "server.subca.local"  - server-subca.pem - Leaf, signed by Intermediate
    [S] "SubCA.of.ROOT"   - subcaR.pem - Intermediate, signed by ROOT
    [X] "ROOT" - caRO.pem - Intermediate, signed by OLD_ROOT
    [R] "ROOT" - caR.pem  - root certificate, same subject and key as X
    "OLD_ROOT" - caO.pem  - old root certificate used to sign X

There are multiple trust chains with this:

- with [R] "ROOT" being in the trust store\
  "server.subca.local" - [S] "SubCA.of.ROOT" - [R] "ROOT"
- with "OLD_ROOT" being in the trust store\
  "server.subca.local" - [S] "SubCA.of.ROOT" - [X] "ROOT" - "OLD_ROOT"

# Situation where such setup happens

This situations happens if ROOT was once a subCA of OLD_ROOT but later moved to
being its own CA. Or in case of cross-signing. This is common when "rebasing" a
CA like LetsEncrypt from being indirectly trusted through a previously trusted
party to being trusted itself.

# Expectation

In these situations it is common that servers still have the old setup of using
[S] and [X] in the chain, even though some clients already have [R] as trusted
root certificate and some even no longer have OLD_ROOT as trused root.

The expection is that this setup with [S] and [X] as chain should work no matter
if the client has [R] or OLD_ROOT in the trust store, one should be sufficient.
In case of only [R] in the trust store the certificate [X] should be ignored.

OpenSSL had in the past problems with this kind of setup which cause problems in
real-life. A fix with introduced with X509_V_FLAG_TRUSTED_FIRST in OpenSSL
1.0.2. Note that this flag no longer works when the new validation inside
LibreSSL is used. See also https://stackoverflow.com/questions/27804710/

# How to check

   $ openssl verify -verbose -trusted caR.pem -untrusted chainSX.pem server-subca.pem

# Results

- OpenSSL 1.1.1: server-subca.pem: Ok
- LibreSSL 3.2.2, LibreSSL 3.3.0

   server-subca.pem: CN = ROOT
   error 20 at 2 depth lookup:unable to get local issuer certificate
   OK
