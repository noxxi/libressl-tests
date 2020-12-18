# Problems

- New validator cannot deal with wildcards in label, i.e. www*.example.com
  If such wildcards are detected in SAN the certificate is considered invalid.
  This means the verification fails, no matter if the hostname in question would
  even need this specific SAN entry.
- s_client in LibreSSL 3.2.2 does not seem to validate certificates at all

# How to check

## Test#1 - unusual wildcard cert, no CA given to client

- start server
  $ openssl s_server -cert server-unusual-wildcard.pem -key server-unusual-wildcard.pem -www
- start client w/o explicit certificate
  $ openssl s_client -connect 127.0.0.1:4433
- Results
    - with OpenSSL 1.1.1: Verify return code: 21 (unable to verify the first certificate)
    - with LibreSSL 3.2.2: Verify return code: 0 (ok) \
      The result and checks with strace/ktrace show that no chain validation was done
    - With LibreSSL 3.3.0: Verify return code: 53 (unsupported or invalid name syntax)

## Test#2 - unusual wildcard cert, CA given to client

- start server
  $ openssl s_server -cert server-unusual-wildcard.pem -key server-unusual-wildcard.pem -www
- start client w/o explicit certificate
  $ openssl s_client -connect 127.0.0.1:4433 -CAfile caR.pem
- Results
    - with OpenSSL 1.1.1: Verify return code: 0 (ok)
    - with LibreSSL 3.2.2: Verify return code: 0 (ok) \
      The result and checks with strace/ktrace show that no chain validation was done
    - With LibreSSL 3.3.0: Verify return code: 53 (unsupported or invalid name syntax)

## Test#3 - common wildcard cert, no CA given to client

- start server
  $ openssl s_server -cert server-common-wildcard.pem -key server-common-wildcard.pem -www
- start client w/o explicit certificate
  $ openssl s_client -connect 127.0.0.1:4433
- Results
    - with OpenSSL 1.1.1: Verify return code: 21 (unable to verify the first certificate)
    - with LibreSSL 3.2.2: Verify return code: 0 (ok) \
      The result and checks with strace/ktrace show that no chain validation was done
    - With LibreSSL 3.3.0: Verify return code: 20 (unable to get local issuer certificate)

## Test#4 - common wildcard cert, CA given to client

- start server
  $ openssl s_server -cert server-common-wildcard.pem -key server-common-wildcard.pem -www
- start client w/o explicit certificate
  $ openssl s_client -connect 127.0.0.1:4433 -CAfile caR.pem
- Results
    - with OpenSSL 1.1.1: Verify return code: 0 (ok)
    - with LibreSSL 3.2.2: Verify return code: 0 (ok) \
      The result and checks with strace/ktrace show that no chain validation was done
    - With LibreSSL 3.3.0: Verify return code: 0 (ok)

## Test#5 - openssl verify, unusual wildcard cert

- Command: openssl verify -CAfile caR.pem server-unusual-wildcard.pem
- Results
    - with OpenSSL 1.1.1: "server-unusual-wildcard.pem: OK"
    - with LibreSSL 3.2.2: "server-unusual-wildcard.pem:" (no actual result given)
    - With LibreSSL 3.3.0: "server-unusual-wildcard.pem: server-unusual-wildcard.pem: verification failed: 53 (unsupported or invalid name syntax)" \
      (Yes, in the error case the certificate name is output twice)

## Test#5 - openssl verify, common wildcard cert

- Command: openssl verify -CAfile caR.pem server-common-wildcard.pem
- Results
    - with OpenSSL 1.1.1: "server-common-wildcard.pem: OK"
    - with LibreSSL 3.2.2: "server-common-wildcard.pem: OK"
    - with LibreSSL 3.3.0: "server-common-wildcard.pem: OK"
      (Yes, in the success case the certificate name is output only one)
