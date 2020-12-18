# Problem

Not all chain certificates are sent in s_server (fixed in LibreSSL 3.3.0)

# Setup of certificates

    "server.subca.local"  - server-subca.pem - Leaf, signed by Intermediate
    [S] "SubCA.of.ROOT"   - subcaR.pem - Intermediate, signed by ROOT
    [R] "ROOT" - caR.pem  - root certificate, same subject and key as X

# Expectation

If s_server is called with server-subca-chainS.pem it should provide the
full chain to s_client, i.e. leaf certificate "server.subca.local" and
intermediate "SubCA.of.ROOT". s_client can then validate this against "ROOT".

# How to check

    $ openssl s_server -cert server-subca.pem -CAfile subcaR.pem -www

# Results

- Server with OpenSSL 1.1.1
    - provides full chain as can be seen from output of s_client, no matter if
      client is using TLS 1.3 or TLS 1.2
- Server with LibreSSL 3.2.2
    - client TLS 1.3: only leaf certificate is provided (validation thus fails)
    - client TLS 1.2: leaf and chain is provided
- Server with LibreSSL 3.3.0
    - issue seems to be fixed, i.e. leaf + chain with both TLS 1.3 and TLS 1.2



