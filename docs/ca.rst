Managing SSL certificates
=========================

When managing large amounts of iLO interfaces, the constant SSL warnings are a
nuisance, so let's make sure we have proper SSL certificates. This script will
make that easy to do for you by taking care of all the CA work. All you need to
do is add the CA's certificate to your browsers trusted CA list.

First thing to do is configure the CA. Add something like the following to your
:file:`~/.ilo.conf`::

  [ca]
  path = ~/.hpilo_ca
  country = NL
  state = Flevoland
  locality = Lelystad
  organization = Kaarsemaker.net
  organizational_unit = Sysadmin

This path can point to an existing CA, the only requirement is that
:file:`openssl.cnf` for this CA lives inside that directory. The other config
values are used when generating the certificates for the iLOs and are all
optional, they default to what HP puts in there.

If the CA does not yet exist, you can create it as follows::

  $ hpilo_ca init
  Generating RSA private key, 2048 bit long modulus
  .+++
  ..................+++
  e is 65537 (0x10001)
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Country Name (2 letter code) [NL]:
  State or Province Name (full name) [Flevoland]:
  Locality Name (eg, city) [Lelystad]:
  Organization Name (eg, company) [Kaarsemaker.net]:
  Common Name (eg, your name or your servers hostname) [hpilo_ca]:

This generates the needed directories, an openssl config and a self-signed
certificate for your CA.

When your CA is set up, you can start signing certificates. :file:`hpilo_ca`
will check several things:

 * Firmware is upgraded if necessary
 * The hostname is set to the name you use to connect to it, if needed
 * iLO2 is configured to use FQDN's for certificate signing requests

It will then download the certificate signing request, sign it and upload the
signed certificate. Here's an example of it at work::

  $ ./hpilo_ca sign example-server.int.kaarsemaker.net
  (1/5) Checking certificate config of example-server.int.kaarsemaker.net
  (2/5) Retrieving certificate signing request
  (3/5) Signing certificate
  Using configuration from /home/dennis/.hpilo_ca/openssl.cnf
  Check that the request matches the signature
  Signature ok
  Certificate Details:
          Serial Number: 4 (0x4)
          Validity
              Not Before: Oct  5 09:48:26 2015 GMT
              Not After : Oct  3 09:48:26 2020 GMT
          Subject:
              countryName               = NL
              stateOrProvinceName       = Flevoland
              organizationName          = Kaarsemaker.net
              organizationalUnitName    = Sysadmin
              commonName                = example-server.int.kaarsemaker.net
          X509v3 extensions:
              X509v3 Basic Constraints:
                  CA:FALSE
              X509v3 Subject Key Identifier:
                  59:E5:B8:37:C5:30:8D:38:47:29:3E:C1:0E:B3:0A:97:95:48:3E:D1
              X509v3 Authority Key Identifier:
                  keyid:89:17:37:C5:E3:2D:EA:5C:83:0A:52:36:79:B0:EC:B7:A4:D5:D4:EF

              Netscape Comment:
                  Certificate generated by iLO CA
              X509v3 Subject Alternative Name
                  DNS:example-server.int.kaarsemaker.net, DNS:example-server, IP Address:10.4.2.13
  Certificate is to be certified until Oct  3 09:48:26 2020 GMT (1825 days)

  Write out database with 1 new entries
  Data Base Updated
  (4/5) Uploading certificate
  (5/5) Resetting iLO
