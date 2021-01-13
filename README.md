# rootcerts

Package rootcerts provides an embedded copy of the [Mozilla Included CA Certificate List],
more specifically the [PEM of Root Certificates in Mozilla's Root Store with the Websites (TLS/SSL) Trust Bit Enabled].
If this package is imported anywhere in the program, then if the [`crypto/x509`]
package cannot find the system certificate pool, it will use this embedded information.

Additionally, the usage of this embedded information can be forced by setting the the environment
variable `GO_ROOTCERTS_ENABLE=1` while running a program, which includes this package.

Importing this package will increase the size of a program by about 250 KB.

This package should normally be imported by a program's main package, not by a library. Libraries
normally shouldn't decide whether to include the "Mozilla Included CA Certificate List" in a program.

```Go
import (
    _ "github.com/breml/rootcerts"
)
```

## Trustworthiness of the Mozilla Included CA Certificate List

Most operation systems as well as web browsers do include a list of certificate authorities and the respective
root certificates that are trusted by default. Some major software vendors operate their own [root programs] and
so does the Mozilla Foundation for their well known products like the [Firefox] web browser or [Thunderbird] email
client.

In contrast to most of the other software vendors, Mozilla does maintain its Included CA Certificate List publicly and
distribute it under an open source license. This is also the reason, why most of the Linux distributions as well as
other free unix derivates use this list of CA Certificates as part of their distribution, here some examples:

* Debian (and its derivates): [ca-certificates](https://packages.debian.org/en/sid/ca-certificates)
* Red Hat / Fedora / CentOS: [ca-certificates](https://src.fedoraproject.org/rpms/ca-certificates) / [ca-certificates](https://centos.pkgs.org/7/centos-x86_64/ca-certificates-2020.2.41-70.0.el7_8.noarch.rpm.html)
* Alpine Linux: [ca-certificates](https://pkgs.alpinelinux.org/package/v3.12/main/x86/ca-certificates)
* FreeBSD: [ca_root_nss](https://www.freshports.org/security/ca_root_nss/)
* NetBSD: [ca-certificates](https://pkgsrc.se/security/ca-certificates)

Additionally Mozilla is operating the [Common CA Database] (used/supported by other major software vendors). The Common
CA Database describes it self as:

> The Common CA Database (CCADB) is a repository of information about externally operated Certificate Authorities (CAs)
whose root and intermediate certificates are included within the products and services of CCADB root store members.

To summarize: it is safe to say, that the Mozilla Included CA Certificate List is well established and widely used.
In fact, if your Go program is run on Linux or an other free Unix derivate, chances are high, that the root
certificates used by your program is the Mozilla Included CA Certificate List already today.

## Motivation

For me, there are mainly two use cases for this package:

1. Docker containers from scratch
2. Poorly maintained operation systems, e.g. in hardware appliances

### Docker Containers from Scratch

If one is building a Docker container from scratch, containing a Go program, there are usually two issues:

1. Timezone data is missing
2. CA certificates are missing

The first issue can be addressed with the [`time/tzdata`] package, introduced into the Go standard library
with version 1.15.
The second case can now be mitigated by this package.

### Poorly maintained appliances

By the term appliance I mainly have hardware appliances like small NAS (network attached storage) systems from
vendors like QNAP or Synology in mind. These systems are in most cases based on Linux and do offer SSH access.
This allows the user to run custom tools on these systems. Unfortunately, whenever the vendor of these systems decide
to stop shipping firmware updates, also the system certificates are no longer updated and it often is difficult or even
impossible to update the system certificates manually.

Therefore it is a great advantage, if a program like a tool built with Go does embed the root certificates.

The following two properties of Go do make it a really good candidate for building programs for hardware appliances:

1. Go programs are statically linked and can be distributed by simply copying the executable.
2. Go does provide greate support for cross compiling for multiple CPU architectures.

## Words of Caution ‒ or why you should not use this package

The root certificates are the top-most certificates in the trust chain and used to ensure the trustworthiness of the
certificates signed by them either directly (intermediate certificates) or indirectly (through intermediate
certificates). As a user of this package, you have the obligation to double check the source as well as the integrity
of the root certificates provided in this package. This is absolutely crucial and should not be taken lightly. All
certificates, that are validated by programs that built upon this package, e.g. by using TLS for communication, rely
on the trustworthiness of these root certificates.

Beside of the trust, put into the certificates included in this package, there is an other topic to keep in mind and
this is how the certificates get updated.

In the "normal" case, where a Go program is run on a recent operating system, the certificates get updated whenever
the operating system is updated (and a new version of the CA certificates is available).\
With the use of this package, this stays true if both of the following conditions are met:

* the [`crypto/x509`] package is able to find the CA certificates on the system.
* the environment variable `GO_ROOTCERTS_ENABLE=1` is not set.

Additionally, it is worth mentioning, that the [`crypto/x509`] package by default does not provide the necessary
mechanics to detect and reload the CA certificates, if they change. By default, a restart of the Go program is
necessary to leverage the updated certificates.

If the above conditions are not met, the CA certificates from this package are used. These certificates are only
updated if all of the following conditions are met:

* An updated list of certificates is available from Mozilla.
* An updated version of this package, containing the updated certificates, is available.
* The dependencies of the Go program are updated (`go get -u github.com/breml/rootcerts`).
* A rebuilt version of the Go program is used

To link back to the motivation for this package again, the main use case for this package should be cases, where:

1. the Go program is frequently updated (automated via CI) and distributed in a minimalistic form like a Docker
container from scratch
2. the Go program is run in a out of date environment like a poorly maintained or no longer updateable system

In all other cases, it is recommended to stick to the CA certificates maintained with the operating system.

## Inspiration

This package is heavily inspired by the [`time/tzdata`] package from the Go standard library.

## License

Software: [BSD 2-Clause “Simplified” License](LICENSE)\
Embedded certificates: [MPL-2.0](LICENSE.certificates)

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"\
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE\
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE\
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE\
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL\
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR\
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER\
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,\
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE\
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[`crypto/x509`]: https://golang.org/pkg/crypto/x509/
[Mozilla Included CA Certificate List]: https://wiki.mozilla.org/CA/Included_Certificates
[PEM of Root Certificates in Mozilla's Root Store with the Websites (TLS/SSL) Trust Bit Enabled]: https://ccadb-public.secure.force.com/mozilla/IncludedRootsPEMTxt?TrustBitsInclude=Websites
[root programs]: https://en.wikipedia.org/wiki/Public_key_certificate#Root_programs
[Firefox]: https://www.mozilla.org/en-US/firefox/
[Thunderbird]: https://www.thunderbird.net/en-US/
[Common CA Database]: https://www.ccadb.org/
[`time/tzdata`]: https://golang.org/pkg/time/tzdata/
