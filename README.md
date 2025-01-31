## CFSSL TRUST

This is the trust stores Cloudflare uses for
[CFSSL](https://github.com/cloudflare/cfssl). It also includes the
sources of the trust chain that can be built using the `mkbundle`
utility from CFSSL.

Files:

```
.
├── ca-bundle.crt
├── ca-bundle.crt.metadata
├── certdata
│   └── trusted_roots
│       ├── froyo.pem
│       ├── gingerbread.pem
│       ├── honeycomb.pem
│       ├── ics.pem
│       ├── ios.pem
│       ├── kitkat.pem
│       ├── nss.pem
│       ├── osx.pem
│       ├── ubuntu.pem
│       └── windows.pem
├── int-bundle.crt
├── README.md
```

The `ca-bundle.crt` file contains the trusted roots. CFSSL uses the
`ca-bundle.crt.metadata` when building bundles to assist in building
bundles that need to verified in the maximum number of trust stores
on different systems. The `int-bundle.crt` file contains a number of
known intermediates; these are preloaded for performance reasons and
occasionally updated as CFSSL finds more intermediates. If an intermediate
isn't in this bundle, but can be found through following the AIA `CA
Issuers` fields, it will be downloaded and eventually merged into here.

The `trusted_roots` directory contains the root stores from a number of
systems. Currently, we have trust stores from

* NSS (Firefox, Chrome)
* OS X
* Windows
* Android 2.2 (Frozen Yogurt)
* Android 2.3 (Gingerbread)
* Android 3.x (Honeycomb)
* Android 4.0 (Ice Cream Sandwich)
* Android 4.4 (KitKat)

### Release

#### Prerequisites

```
$ go get -u github.com/kisom/goutils/cmd/certdump
$ go get -u github.com/cloudflare/cfssl/cmd/...
$ go get -u github.com/cloudflare/cfssl_trust/...
```

#### Build

The final bundles (i.e. `ca-bundle.crt` and `int-bundle.crt`) may be
built as follows:

```
$ ./release.sh
```

This command automatically removes expiring certificates, and pushes the
changes to a new release branch.

The content of 'ca-bundle.crt.metadata' is crucial to building
ubiquitous bundle. Feel free to tune its content. Make sure the paths to
individual trust root stores are correctly specified.

#### Adding new roots or intermediates

New roots and intermediates can be added using the same command, just by
providing values for the `NEW_ROOTS` and `NEW_INTERMEDIATES` variables:

```
$ NEW_ROOTS="/path/to/root1 /path/to/root2" NEW_INTERMEDIATES="/path/to/int1 /path/to/int22" ./release.sh
```

#### Check for expiring roots or intermediates

To verify that an intermediate or root certificate is expiring or revoked without creating a release, the `expiring` command can be used from the project root directory.

To check for expiring or revoked intermediate certificates in the database provided in this repo:
```
$ cfssl-trust -d ./cert.db -b int expiring
```
To check for expiring or revoked root certificates:
```
$ cfssl-trust -d ./cert.db -b ca expiring
```

`./cert.db` which is specified as the database using the `-d` flag, contains both intermediate and root certificates.
Any certificate database can be used here in place of `./cert.db`

These calls to the `expiring` command will provide an output showing if there are any expiring or revoked certificates.
```
...
1 certificates expiring.
0 certificates revoked.
```
